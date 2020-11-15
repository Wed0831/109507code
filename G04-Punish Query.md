# G04-Punish Query
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
//------------------------------------------
//查詢
//------------------------------------------
var query = async function(userno){
    var result={};
    
    await sql('SELECT * FROM bookingpunish WHERE userno = $1', [userno])
        .then((data) => {
            result = data.rows;       
        }, (error) => {
            result = null;
        });

    return result;
}
//匯出
module.exports = {query};
```
## punish_query.js
```
var express = require('express');
var router = express.Router();

//增加引用函式
var moment = require('moment');
const punish = require('./utility/punish');

//接收GET請求
router.get('/', function(req, res, next) {
    var email = req.session.email
    var no = email.indexOf("@",1);
    var userno = email.substr(0,no);

    punish.query(userno).then(data => {
        if (data==null){
            res.render('notFound');            
        }else if(data.length > 0){
            //console.log(data);
            res.render('punish_query', {items:data, moment:moment});  //將資料傳給顯示頁面
        }else{
            res.render('notFound');  //導向找不到頁面     
        }  
    })
});

module.exports = router;
```
## punish_query.ejs
```
<h2>懲罰查詢</h2>
    <table>
        <thead>
            <tr>
                <td>逞罰編號</td>
                <td>學號</td>
                <td>起始日</td>
                <td>結束日</td>
                <td>逞罰原因</td>
            </tr>
        </thead>

        <tbody>
            <% for(var i=0; i<items.length; i++) {%>
                <tr>
                    <td><%= items[i].punishno %></td>
                    <td><%= items[i].userno %></td>
                    <td><%= moment(items[i].startdate).format(' YYYY-MM-DD') %></td>
                    <td><%= moment(items[i].finishdate).format(' YYYY-MM-DD') %></td>   
                    <td><%= items[i].punishdetail %></td>                     
                </tr> 
            <% } %> 
        </tbody>
    </table>
```
## notFound.ejs
```
<script>
    alert("找不到資料");
    location.href = "/";
</script>   
```
## app.js
```
var punish_query = require('./routes/punish_query');//查詢

app.use('/punish/query', checkAuth_ST, punish_query);//查詢
```