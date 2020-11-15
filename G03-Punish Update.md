# G03-Punish Update
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
## punish.js(Utility)
```
//----------------------------------
// 更新商品
//----------------------------------
var query_update = async function(userno){
    var result={};
    
    await sql('SELECT * FROM bookingpunish WHERE userno = $1', [userno])
        .then((data) => {
            result = data.rows;       
        }, (error) => {
            result = null;
        });
		
    return result;
}

var update = async function(newData){
    var results=0;

    for(var i=0; i<newData.punishno.length;i++){
        await sql('UPDATE bookingpunish SET userno = $1, startdate=$2, finishdate=$3, punishdetail=$4 WHERE punishno=$5', [newData.userno[i], newData.startdate[i], newData.finishdate[i], newData.punishdetail[i], newData.punishno[i]])
            .then((data) => {
                results = results + data.rowCount;  
            }, (error) => {
                results = -1;
            });
    }
    return results;
}
//匯出
module.exports = {query_update, update};
```
## punish_update_no.js
```
var express = require('express');
var router = express.Router();

/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('punish_update_no');
});

//匯出
module.exports = router;
```
## punish_update_no.ejs
```
<h2>懲罰更新</h2>
<form action = "/punish/update/form" method = "get">
    <div class="form">
        <span class="name">編號: </span>
        <span class="value"><input type="text" name="userno"></span>
        <br/>    

        <span class="name"></span>
        <span class="value"><input type="submit" value="查詢" ></span>    
        </div>    
</form> 
```
## punish_update_form.js
```
var express = require('express');
var router = express.Router();

//增加引用函式
var moment = require('moment');
const punish = require('./utility/punish');

//接收GET請求
router.get('/', function(req, res, next) {
    var userno = req.query.userno;

    punish.query_update(userno).then(d => {
        if (d!=null || d!=-1){
            if(d == null){
                res.render('notFound')  //沒有審核資歷
            }else if(d.length >= 0){
                res.render('punish_update_form', {items:d, moment:moment});  //將資料傳給更新頁面
            }
        }else{
            res.render('notFound')  //導向找不到頁面
        }  
    })
});

//匯出
module.exports = router;
```
## punish_update_form.ejs
```
<h2>懲罰更新</h2>
<form action = "/punish/update" method = "post">
    <div class="form">
        <table>
            <thead>
                <tr>
                    <td>懲罰編號</td>
                    <td>學號</td>
                    <td>起始日</td>
                    <td>結束日</td>
                    <td>原因</td>
                </tr>
            </thead> 
                    
            <tbody>
                <% for(var i=0; i<items.length; i++) {%>
                    <tr>
                        <td><%= items[i].punishno %><input type="text" name="punishno" value="<%= items[i].punishno %>" hidden></td>
                        <td><%= items[i].userno %><input type="text" name="userno" value="<%= items[i].userno %>" hidden></td>
                        <td><input type="text" name="startdate" value="<%=  moment(items[i].startdate).format('YYYY-MM-DD') %>"></td>
                        <td><input type="text" name="finishdate" value="<%= moment(items[i].finishdate).format('YYYY-MM-DD') %>"></td>
                        <td><input type="text" name="punishdetail" value="<%= items[i].punishdetail %>"></td>
                    </tr>
                <% } %> 
            </tbody>    
        </table>
                    
                    
        <span class="value"><input type="submit" value="更新" ></span>    
    </div>
</form>
```
## punish_update.js
```
var express = require('express');
var router = express.Router();

//增加引用函式
var moment = require('moment');
const punish = require('./utility/punish');

//接收POST請求
router.post('/', function(req, res, next) {
    var punishno = req.body.punishno;   //取得產品編號
    var userno = req.body.userno;
    if(typeof(punishno) === 'string'){
        punishno = [punishno];
    }
    if(typeof(userno) === 'string'){
        userno = [userno];
    }

    var newData={
        punishno: punishno,
        userno: userno,                     
        startdate: req.body.startdate, 
        finishdate: req.body.finishdate,  
        punishdetail: req.body.punishdetail  
    } 
    
    punish.update(newData).then(d => {
        if (d>=0){
            res.render('updateSuccess');  //傳至成功頁面
        }else{
            res.render('updateFail');     //導向錯誤頁面
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
## updateSuccess.ejs
```
<script>
    alert("已更新");
    location.href = "/";
</script>  
```
## updateFail.ejs
```
<script>
    alert("更新失敗");
    history.go(-1);
</script>     
```
## app.js
```
var punish_update_no = require('./routes/punish_update_no');  //更新
var punish_update_form = require('./routes/punish_update_form');
var punish_update = require('./routes/punish_update');


app.use('/punish/update/no' , punish_update_no); //更新
app.use('/punish/update/form', punish_update_form);
app.use('/punish/update', punish_update);
```