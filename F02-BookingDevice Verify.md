# F02-BookingDevice Verify
## asyncDB.js
```
'use strict';

//-----------------------
// 引用資料庫模組
//-----------------------
const {Client} = require('pg');

//-----------------------
// 自己的資料庫連結位址
//-----------------------
var pgConn = 'postgres://tcmlcffsmhdxec:1d3f66e3407911142089ee9d9b20b1f84cc62108d5e2aa8c21c1109d88fddb5b@ec2-52-87-135-240.compute-1.amazonaws.com:5432/d87tq6p6hnqbt2';


//postgres://ejagiexdrhgktj:213165a694c98480c2167e67872070fd7d1a4755069c1a292eef8fce99a192aa@ec2-18-210-51-239.compute-1.amazonaws.com:5432/dcmd4haf47dj0c

//產生可同步執行sql的函式
function query(sql, value=null) {
    return new Promise((resolve, reject) => {
        //產生資料庫連線物件
        var client = new Client({
            connectionString: pgConn,
            ssl: true
        })     

        //連結資料庫
        client.connect();

        //執行並回覆結果  
        client.query(sql, value, (err, results) => {                   
            if (err){
                reject(err);
            }else{
                resolve(results);
            }

            //關閉連線
            client.end();
        });
    });
}

//匯出
module.exports = query;
```
## deviceverify.js(Utility)
```
//------------------------------------------
// 審核
//------------------------------------------
var query_verify = async function(userno){
    var result={};
    
    await sql('SELECT * FROM bookingdevice AS a JOIN bookingdevice_detail AS b on a.bookingdeviceno = b.bookingdeviceno WHERE a.userno = $1 and yesorno is NULL', [userno])
        .then((data) => {
            result = data.rows;
        }, (error) => {
            result = null;
        });     
        
    console.log(result);
    return result;
}

var update_verify_detail = async function(newData){
    var results=0;

    for(var i=0; i<newData.bookingdeviceno.length;i++){       
        await sql('UPDATE bookingdevice_detail SET deviceno=$1 WHERE bookingdeviceno = $2', [newData.deviceno[i], newData.bookingdeviceno[i]])
            .then((data) => {
                results = results + data.rowCount;  
            }, (error) => {
                results = -1;
            });
    }

    return results;
}

var update_verify = async function(newData){
    var results=0;

    for(var i=0; i<newData.bookingdeviceno.length;i++){
        await sql('UPDATE bookingdevice SET reviewdate=$1, yesorno=$2, return = $3 WHERE bookingdeviceno = $4', [newData.reviewdate[i], newData.yesorno[i], newData.return[i], newData.bookingdeviceno[i]])
            .then((data) => {
                results = results + data.rowCount;  
            }, (error) => {
                results = -1;
            });
    }

    return results;
}

//匯出
module.exports = {query_verify, update_verify, update_verify_detail};
```
## deviceverify_update_no.js
```
var express = require('express');
var router = express.Router();

/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('deviceverify_update_no');
});

//匯出
module.exports = router;
```
## deviceverify_update_no.ejs
```
<h2>設備審核</h2>
<form action = "/deviceverify/update/form" method = "get">
    <div class="form">
        <span class="name">學號: </span>
        <span class="value"><input type="text" name="userno"></span>
        <br/>    

        <span class="name"></span>
        <span class="value"><input type="submit" value="查詢" ></span>    
    </div>    
</form> 
```
## deviceverify_update_form.js
```
var express = require('express');
var router = express.Router();

//增加引用函式
var moment = require('moment');
const deviceverify = require('./utility/deviceverify');

//接收GET請求
router.get('/', function(req, res, next) {
    var no = req.query.userno;

    deviceverify.query_verify(no).then(d => {
        if(d==null){
            res.render('notFound');  //導向錯誤頁面
        }else if(d.length > 0){
            res.render('deviceverify_update_form', {items:d, moment:moment});  //將資料傳給更新頁面
        }else{
            res.render('notFound');  //導向找不到頁面
        }  
    })
});

//匯出
module.exports = router;
```
## deviceverify_update_form.ejs
```
<h2>產品更新 </h2>
<form action = "/deviceverify/update" method = "post">
    <div class="form">
        <table>
            <thead>
                <tr>
                    <td>學號</td>
                    <td>聯絡電話</td>
                    <td>借用編號</td>
                    <td>類別</td>
                    <td>申請日</td>
                    <td>起始日</td>
                    <td>結束日</td>
                    <td>設備編號</td>
                    <td>是否同意</td>
                </tr>
            </thead> 

            <tbody>
                <% for(var i=0; i<items.length; i++) {%>
                    <tr>
                        <td><%= items[i].userno %></td>
                        <td><%= items[i].teleno %></td>
                        <td><%= items[i].bookingdeviceno %><input type="text" name="bookingdeviceno" value="<%= items[i].bookingdeviceno %>" hidden></td>
                        <td><%= items[i].category %></td>
                        <td><%= moment(items[i].inventorydate).format("YYYY-MM-DD HH:MM:SS") %></td>
                        <td><%= moment(items[i].borrowdate).format("YYYY-MM-DD") %></td>
                        <td><%= moment(items[i].returndate).format("YYYY-MM-DD") %></td>
                        <td><input type="text" name="deviceno" value="<%= items[i].deviceno %>"></td>
                        <td>
                            <% if(items[i].yesorno == 't'){ %>
                                <% name = 'yesorno' + i %>
                                <input type="radio" name="<%= name %>" value="t" checked>是
                                <input type="radio" name="<%= name %>" value="f">否
                                        <% } else{ %>
                                <% name = 'yesorno' + i %>
                                <input type="radio" name="<%= name %>" value="t" checked>是
                                <input type="radio" name="<%= name %>" value="f">否
                            <% } %>
                        </td>
                    </tr>
                <% } %> 
            </tbody>    
        <table>

        <span class="name"></span>
        <span class="value"><input type="submit" value="更新" ></span>    
    </div>
</form>
```
## deviceverify_update.js
```
var express = require('express');
var router = express.Router();

//增加引用函式
const deviceverify = require('./utility/deviceverify');
var today = require('silly-datetime');

//接收POST請求
router.post('/', function(req, res, next) {
    var bookingdeviceno = req.body.bookingdeviceno;   //取得產品編號
    if(typeof(bookingdeviceno) === 'string'){
        bookingdeviceno = [bookingdeviceno];
    }
    var yn = [req.body.yesorno0, req.body.yesorno1, req.body.yesorno2, req.body.yesorno3, req.body.yesorno4, req.body.yesorno5, req.body.yesorno6, req.body.yesorno7, req.body.yesorno8, req.body.yesorno9
        ,req.body.yesorno10, req.body.yesorno11, req.body.yesorno12, req.body.yesorno13, req.body.yesorno14, req.body.yesorno15, req.body.yesorno16, req.body.yesorno17, req.body.yesorno18, req.body.yesorno19
        ,req.body.yesorno20, req.body.yesorno21, req.body.yesorno22, req.body.yesorno23, req.body.yesorno24, req.body.yesorno25, req.body.yesorno26, req.body.yesorno27, req.body.yesorno28, req.body.yesorno29
        ,req.body.yesorno30, req.body.yesorno31, req.body.yesorno32, req.body.yesorno33, req.body.yesorno34, req.body.yesorno35, req.body.yesorno36, req.body.yesorno37, req.body.yesorno38, req.body.yesorno39
        ,req.body.yesorno40, req.body.yesorno41, req.body.yesorno42, req.body.yesorno43, req.body.yesorno44, req.body.yesorno45, req.body.yesorno46, req.body.yesorno47, req.body.yesorno48, req.body.yesorno49];
    var yesorno = [];
    var reviewdate = []
    var rt = [];

    for(var i=0;i<bookingdeviceno.length;i++){
        yesorno.push(yn[i]);
    }

    for(var i=0; i<bookingdeviceno.length; i++){
        reviewdate.push(today.format("YYYY-MM-DD HH:MM:SS"));
        rt.push('f');
    }

    var newData={
        bookingdeviceno:bookingdeviceno,                 
        deviceno: req.body.deviceno,     
        yesorno: yesorno,
        reviewdate:reviewdate,
        return:rt
    } 

    deviceverify.update_verify_detail(newData).then(d => {
        if (d>=0){
            deviceverify.update_verify(newData).then(d => {
                if (d>=0){
                    res.render('updateSuccess');  //傳至成功頁面
                }else{
                    res.render('updateFail');     //導向錯誤頁面
                }  
            })
        }else{
            res.render('updateFail');     //導向錯誤頁面
        }  
    })
});

//匯出
module.exports = router;
```
## no_verify.ejs
```
<script>
    alert("沒有審核資料");
    location.href = "/";
</script>  
```
## notFound.ejs
```
<script>
    alert("找不到資料");
    location.href = "/";
</script>   
```
## borrowSuccess.ejs
```
<script>
    alert("借用成功");
    location.href = "/";
</script>  
```
## borrowFail.ejs
```
<script>
    alert("借用失敗");
    history.go(-1);
</script>     
```
## app.js
```
var deviceverify_update_no = require('./routes/deviceverify_update_no');//審核
var deviceverify_update_form = require('./routes/deviceverify_update_form');
var deviceverify_update = require('./routes/deviceverify_update');

app.use('/deviceverify/update/no', deviceverify_update_no);//審核
app.use('/deviceverify/update/form', deviceverify_update_form);
app.use('/deviceverify/update', deviceverify_update);
```