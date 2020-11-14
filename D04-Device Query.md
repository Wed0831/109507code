# D04-Device Query
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
## device.js(Utility)
```
//------------------------------------------
//查詢商品
//------------------------------------------
var query = async function(deviceno){
    var result={};
    
    await sql('SELECT * FROM device WHERE deviceno = $1', [deviceno])
        .then((data) => {
            if(data.rows.length > 0){
                result = data.rows[0];   
            }else{
                result = -1;
            }    
        }, (error) => {
            result = null;
        });
		
    return result;
}
//匯出
module.exports = {query};
```
## device_query_form.js
```
var express = require('express');
var router = express.Router();

//接收GET請求
router.get('/', function(req, res, next) {
    res.render('device_query_form');
});

module.exports = router;
```
## device_query_form.ejs
```
<h2>設備查詢</h2>
<form action = "/device/query" method = "get">
    <div class="form">
        <span class="name">編號: </span>
        <span class="value"><input type="text" name="deviceno"></span>
        <br/>    
                                    
        <span class="name"></span>
        <span class="value"><input type="submit" value="查詢" ></span>    
    </div>    
</form>   
```
## device_query.js
```
var express = require('express');
var router = express.Router();

//增加引用函式
const device = require('./utility/device');

//接收GET請求
router.get('/', function(req, res, next) {
    var deviceno = req.query.deviceno;   //取出參數

    device.query(deviceno).then(data => {
        if (data==null || data==-1){
            res.render('notFound');          
        }else{
            res.render('device_query', {item:data});  //將資料傳給顯示頁面
        }  
    })
});

module.exports = router;
```
## device_query.ejs
```
<h2>設備查詢結果</h2>
    <table>
        <tr>
            <td width="25%">編號</td>
            <td><%= item.deviceno %></td>
        </tr>
        <tr>
            <td>名稱</td>
            <td><%= item.devicename %></td>
        </tr>
        <tr>
            <td>細項</td>
            <td><%= item.devicedetail %></td>
        </tr>                                         
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
var device_query_form = require('./routes/device_query_form');  //查詢
var device_query = require('./routes/device_query');

app.use('/device/query/form', checkAuth_E, device_query_form); //查詢
app.use('/device/query', checkAuth_E, device_query);
```
