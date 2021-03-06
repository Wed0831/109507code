# F04-BookingDevice Query
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
// 查詢
//------------------------------------------
var query_no = async function(userno){
    var result={};
    
    await sql('SELECT * FROM bookingdevice AS a JOIN bookingdevice_detail AS b on a.bookingdeviceno = b.bookingdeviceno WHERE a.userno = $1', [userno])
        .then((data) => {
            result = data.rows;   
        }, (error) => {
            result = null;
        });

    return result;  
}

//匯出
module.exports = {query_no};
```
## devicecerify_query.js
```
var express = require('express');
var router = express.Router();

//增加引用函式
var moment = require('moment');
const deviceverify = require('./utility/deviceverify');

//接收GET請求
router.get('/', function(req, res, next) {
    var email = req.session.email
    var no = email.indexOf("@",1);
    var userno = email.substr(0,no);

    deviceverify.query_no(userno).then(data => {
        if(data==null){
            res.render('notFound');  //導向錯誤頁面
        }else if(data.length > 0){
            res.render('deviceverify_query', {items:data, moment:moment});  //將資料傳給顯示頁面
        }else{
            res.render('notFound');  //導向找不到頁面
        }  
    })
});

module.exports = router;
```
## deviceverify_query.ejs
```
<h2>借用紀錄查詢</h2>
    <table>
        <thead>
            <tr>
                <td>借用編號</td>
                <td>學號</td>
                <td>設備</td>
                <td>申請日</td>
                <td>借用日</td>
                <td>歸還日</td>
                <td>設備編號</td>
                <td>是否歸還</td>
            </tr>
        </thead>
                        
        <tbody>
            <% for(var i=0; i<items.length; i++) {%>
                <tr>
                    <td><%= items[i].bookingdeviceno %></td>
                    <td><%= items[i].userno %></td>
                    <td><%= items[i].category %></td>
                    <td><%= moment(items[i].bookingdate).format(' YYYY-MM-DD  HH:mm:ss') %></td>
                    <td><%= moment(items[i].borrowdate).format('YYYY-MM-DD') %></td>
                    <td><%= moment(items[i].returndate).format('YYYY-MM-DD') %></td>
                    <td><%= items[i].deviceno %></td>
                    <td><%= items[i].return %></td><!---->
                </tr>
            <% } %> 
        </tbody>
    </table>
```
## notBorrow.ejs
```
<script>
    alert("無人借用");
    history.go(-1);
</script> 
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
var deviceverify_query = require('./routes/deviceverify_query');//查詢

app.use('/deviceverify/query', checkAuth_ST, deviceverify_query);//查詢
```