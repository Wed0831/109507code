# F03-BookingDevice Return
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
// 歸還
//------------------------------------------
var query_return = async function(userno){
    var result={};
    
    await sql('SELECT * FROM bookingdevice AS a JOIN bookingdevice_detail AS b on a.bookingdeviceno = b.bookingdeviceno WHERE a.userno = $1 and return is false', [userno])
        .then((data) => {
            result = data.rows;
        }, (error) => {
            result = null;
        });   

    return result;  
}

var update_return = async function(newData){
    var results = 0;

    for(var i=0; i<newData.bookingdeviceno.length;i++){
        await sql('UPDATE bookingdevice SET return=$1 WHERE bookingdeviceno = $2', [newData.return[i], newData.bookingdeviceno[i]])
            .then((data) => {
                results = data.rowCount;  
            }, (error) => {
                results = -1;
            });
    }

    return results;
}
//匯出
module.exports = {query_return, update_return};
```
## devicereturn_update_no.js
```
var express = require('express');
var router = express.Router();

/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('devicereturn_update_no');
});

//匯出
module.exports = router;
```
## devicereturn_update_no.ejs
```
<h2>設備審核</h2>
<form action = "/devicereturn/update/form" method = "get">
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

    deviceverify.query_return(no).then(d => {
        if(d==null){
            res.render('notFound');  //導向錯誤頁面
        }else if(d.length > 0){
            res.render('devicereturn_update_form', {items:d, moment:moment});  //將資料傳給更新頁面
        }else{
            res.render('notFound');  //導向找不到頁面
        }  
    })
});

//匯出
module.exports = router;
```
## devicereturn_update_form.ejs
```
<h2>產品更新 </h2>
<form action = "/devicereturn/update" method = "post">
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
                    <td>是否歸還</td>
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
                        <td><%= items[i].deviceno %></td>
                        <td>
                            <% if(items[i].return == 't'){ %>
                                <% name = 'return' + i %>
                                <input type="radio" name="<%= name %>" value="t" checked>是
                                <input type="radio" name="<%= name %>" value="f">否
                            <% } else{ %>
                                <% name = 'return' + i %>
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
## devicereturn_update.js
```
var express = require('express');
var router = express.Router();

//增加引用函式
const deviceverify = require('./utility/deviceverify');

//接收POST請求
router.post('/', function(req, res, next) {
    var bookingdeviceno = req.body.bookingdeviceno;   //取得產品編號
    if(typeof(bookingdeviceno) === 'string'){
        bookingdeviceno = [bookingdeviceno];
    }
    var t = [req.body.return0, req.body.return1, req.body.return2, req.body.return3, req.body.return4, req.body.return5, req.body.return6, req.body.return7, req.body.return8, req.body.return9
        ,req.body.return10, req.body.return11, req.body.return12, req.body.return13, req.body.return14, req.body.return15, req.body.return16, req.body.return17, req.body.return18, req.body.return19
        ,req.body.return20, req.body.return21, req.body.return22, req.body.return23, req.body.return24, req.body.return25, req.body.return26, req.body.return27, req.body.return28, req.body.return29
        ,req.body.return30, req.body.return31, req.body.return32, req.body.return33, req.body.return34, req.body.return35, req.body.return36, req.body.return37, req.body.return38, req.body.return39
        ,req.body.return40, req.body.return41, req.body.return42, req.body.return43, req.body.return44, req.body.return45, req.body.return46, req.body.return47, req.body.return48, req.body.return49];
    var rt = [];

    for(var i=0;i<bookingdeviceno.length;i++){
        rt.push(t[i]);
    }

    var newData={
        bookingdeviceno:bookingdeviceno,                   
        return:rt
    } 

    deviceverify.update_return(newData).then(d => {
        if (d>=0){
            res.render('returnSuccess');
        }else{
            res.render('returnFail');
        }  
    })
});

//匯出
module.exports = router;
```
## notFound.ejs
```
<script>
    alert("找不到資料");
    location.href = "/";
</script>   
```
## returnSuccess.ejs
```
<script>
    alert("歸還成功");
    history.go(-1);
</script>  
```
## returnFail.ejs
```
<script>
    alert("歸還失敗");
    history.go(-1);
</script>   
```
## app.js
```
var devicereturn_update_no = require('./routes/devicereturn_update_no');  //歸還
var devicereturn_update_form = require('./routes/devicereturn_update_form');
var devicereturn_update = require('./routes/devicereturn_update');

app.use('/devicereturn/update/no', checkAuth_E, devicereturn_update_no); //歸還
app.use('/devicereturn/update/form', checkAuth_E, devicereturn_update_form);
app.use('/devicereturn/update', checkAuth_E, devicereturn_update);
```