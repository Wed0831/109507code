# B03-User Update
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

//----------------------------------
// 更新商品
//----------------------------------
var update = async function(newData){
    var results;

    await sql('UPDATE "user" SET "username"=$1, "isEmployee"=$2, "restrict"=$3 WHERE "userno" = $4', [newData.username, newData.isEmployee, newData.restrict, newData.userno])
        .then((data) => {
            results = data.rowCount;  
        }, (error) => {
            results = -1;
        });
		
    return results;
}
//匯出
module.exports = {query, update};
```
## user_update_no.js
```
var express = require('express');
var router = express.Router();

/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('user_update_no');
});

//匯出
module.exports = router;
```
## user_update_no.ejs
```
<h2>人員更新</h2>
<form action = "/user/update/form" method = "get">
    <div class="form">
        <span class="name">編號: </span>
        <span class="value"><input type="text" name="userno"></span>
        <br/>    

        <span class="name"></span>
        <span class="value"><input type="submit" value="查詢" ></span>    
    </div>    
</form> 
```
## user_add_form.js
```
var express = require('express');
var router = express.Router();

//增加引用函式
var moment = require('moment');
const user = require('./utility/user');

//接收GET請求
router.get('/', function(req, res, next) {
    var no = req.query.userno;

    user.query(no).then(d => {
        if (d!=null && d!=-1){
            var data = {
                userno: d.userno,
                usrname: d.usrname,
                isEmployee: d.isEmployee,
                restrict: d.restrict
            }
            res.render('user_update_form', {item:data});  //將資料傳給更新頁面
        }else{
            res.render('notFound')  //導向找不到頁面
        }  
    })
});

//匯出
module.exports = router;
```
## user_add_form.ejs
```
<h2>人員更新 </h2>
<form action = "/user/update" method = "post">
    <div class="form">

        <span class="name">編號</span>
        <span class="value"><input type="text" name="userno" value="<%= item.userno %>"></span>
        <br/>

        <span class="name">姓名</span>
        <span class="value"><input type="text" name="username" value="<%= item.username %>" ></span>
        <br/>

        <span class="name">是否為員工</span>
        <span class="value"><input type="text" name="isEmployee" value="<%= item.isEmployee %>"></span>
        <br/>

        <span class="name">備註</span>
        <span class="value"><input type="text" name="restrict" value="<%= item.restrict %>"></span>
        <br/>

        <span class="value"><input type="submit" value="更新" ></span>    
    </div>
</form>
```
## user_update.js
```
var express = require('express');
var router = express.Router();

//增加引用函式
const user = require('./utility/user');

//接收POST請求
router.post('/', function(req, res, next) {
    var userno = req.body.userno;   //取得編號

    var newData={
        userno:userno,                   
        username: req.body.username,   
        isEmployee: req.body.isEmployee,
        restrict: req.body.restrict  
    } 
    
    user.update(newData).then(d => {
        if (d>=0){
            res.render('updateSuccess');
        }else{
            res.render('updateFail');
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
var user_update_no = require('./routes/user_update_no');  //更新
var user_update_form = require('./routes/user_update_form');
var user_update = require('./routes/user_update');

app.use('/user/update/no', checkAuth_E, user_update_no); //更新
app.use('/user/update/form', checkAuth_E, user_update_form);
app.use('/user/update', checkAuth_E, user_update);
```