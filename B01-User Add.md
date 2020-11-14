# B01-User Add
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
//新增
//------------------------------------------
var add = async function(newData){
    var result;
    await sql('INSERT INTO "user" ("userno", "username", "schemail", "isEmployee", "restrict") VALUES ($1, $2, $3, $4, $5)', [newData.userno, newData.username, newData.schemail, newData.isEmployee, newData.restrict])
        .then((data) => {
            result = 0;  
        }, (error) => {
            result = -1;
        });
		
    return result;
}
//匯出
module.exports = {add};
```
## user_add_form.js
```
var express = require('express');
var router = express.Router();

//接收GET請求
router.get('/', function(req, res, next) {
    res.render('user_add_form'); 
});

module.exports = router; 
```
## user_add_form.ejs
``` 
<h2>人員新增</h2>
<form action = "/user/add" method = "post">
    <div class="form">
        <span class="name">編號: </span>
        <span class="value"><input type="text" name="userno"></span>
        <br/>

        <span class="name">姓名: </span>
        <span class="value"><input type="text" name="username"></span>
        <br/>

        <span class="name">信箱: </span>
        <span class="value"><input type="text" name="schemail" ></span>
        <br/>

        <span class="name">是否為員工: </span>
        <span class="value">
            <input type="radio" name="isEmployee" value="t">是
            <input type="radio" name="isEmployee" value="f">否
        </span>
        <br/>

        <span class="name">限制: </span>
        <span class="value">
            <input type="radio" name="restrict" value="0">正常使用
            <input type="radio" name="restrict" value="1">暫時停權
            <input type="radio" name="restrict" value="2">永久停權
        </span>
        <br/>
                                            
        <span class="name"></span>
        <span class="value"><input type="submit" value="增加" ></span>    
    </div>    
</form>  
```
## user_add.js
```
var express = require('express');
var router = express.Router();

//增加引用函式
const user = require('./utility/user');

//接收POST請求
router.post('/', function(req, res, next) {
    var userno = req.body.userno;                 //學號
    var username = req.body.username;             //姓名
    var schemail = req.body.schemail;           //信箱
    var isEmployee = req.body.isEmployee;       //是否為員工
    var restrict = req.body.restrict;           //限制

    // 建立一個新資料物件
    var newData={
        userno:userno,
        username:username,
        schemail:schemail,
        isEmployee:isEmployee,
        restrict:restrict
    } 
    
    user.add(newData).then(d => {
        if (d==0){
            res.render('addSuccess');
        }else{
            res.render('addFail');
        }  
    })
});

module.exports = router;
```
## addSuccess.ejs
```
<script>
    alert("新增成功");
    location.href = "/";
</script>   
```
## addFail.ejs
```
<script>
    alert("新增失敗");
    history.go(-1);
</script>   
```
## app.js
```
var user_add_form = require('./routes/user_add_form'); //新增
var user_add = require('./routes/user_add');

app.use('/user/add/form', checkAuth_E, user_add_form); //新增
app.use('/user/add', checkAuth_E, user_add);
```