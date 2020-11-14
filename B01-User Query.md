# B01-User Query
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
## User.js(Utility)
```
//------------------------------------------
//查詢
//------------------------------------------
var query = async function(userno){
    var result={};
    
    await sql('SELECT * FROM "user" WHERE "userno" = $1', [userno])
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
## user_query_form.js
```
var express = require('express');
var router = express.Router();

//接收GET請求
router.get('/', function(req, res, next) {
    res.render('user_query_form');
});

module.exports = router;
```
## user_query_form.ejs
```
<h2>人員查詢</h2>
<form action = "/user/query" method = "get">
    <div class="form">
        <span class="name">編號: </span>
        <span class="value"><input type="text" name="userno"></span>
        <br/>    
                                    
        <span class="name"></span>
        <span class="value"><input type="submit" value="查詢" ></span>    
    </div>    
</form>    
```
## user_query.js
```
var express = require('express');
var router = express.Router();

//增加引用函式
const user = require('./utility/user');

//接收GET請求
router.get('/', function(req, res, next) {
    var userno = req.query.userno;   //取出參數

    user.query(userno).then(data => {
        if (data==null || data==-1){
            res.render('notFound');          
        }else{
            res.render('user_query', {item:data});  //將資料傳給顯示頁面
        }  
    })
});

module.exports = router;
```
## user_query.ejs
```
<h2>人員查詢結果</h2>
    <table>
        <tr>
            <td width="25%">編號</td>
            <td><%= item.userno %></td>
        </tr>
        <tr>
            <td>姓名</td>
            <td><%= item.username %></td>
        </tr>
        <tr>
            <td>信箱</td>
            <td><%= item.schemail %></td>
        </tr>
        <tr>
            <td>是否為員工</td>
            <td><%= item.isEmployee %></td>
        </tr>
        <tr>
            <td>限制</td>
            <td><%= item.restrict %></td>
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
var user_query_form = require('./routes/user_query_form');  //查詢
var user_query = require('./routes/user_query');


app.use('/user/query/form', checkAuth_E, user_query_form); //查詢
app.use('/user/query', checkAuth_E, user_query);
```