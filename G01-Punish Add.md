# G01-Punish Add
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
//執行資料庫動作的函式-新增產品資料
//------------------------------------------
var add = async function(newData){
    var result;

    await sql('INSERT INTO bookingpunish (userno, startdate, finishdate, punishdetail) VALUES ($1, $2, $3, $4)', [newData.userno, newData.startdate, newData.finishdate, newData.punishdetail])
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
## punish_add_form.js
```
var express = require('express');
var router = express.Router();

//接收GET請求
router.get('/', function(req, res, next) {
    res.render('punish_add_form'); 
});

module.exports = router; 
```
## punish_add_form.ejs
```
<h2>新增懲罰 </h2>
<form action = "/punish/add" method = "post">
    <div class="form">                                    
        <span class="name">學號: </span>
        <span class="value"><input type="text" name="userno" ></span>
        <br/>
                                                        
        <span class="name">起始日: </span>
        <span class="value"><input type="date" name="startdate" ></span>
        <br/>
                                            
        <span class="name">結束日: </span>
        <span class="value"><input type="date" name="finishdate" ></span>
        <br/>
                    
        <span class="name">逞罰原因: </span>
        <span class="value"><input type="text" name="punishdetail" ></span>
        <br/>
                                    
        <span class="name"></span>
        <span class="value"><input type="submit" value="新增" ></span>    
    </div>    
</form>   
```
## punish_add.js
```
var express = require('express');
var router = express.Router();

//增加引用函式
const punish = require('./utility/punish');

//接收POST請求
router.post('/', function(req, res, next) {                 
    var userno = req.body.userno;                 
    var startdate = req.body.startdate;  
    var finishdate = req.body.finishdate;                  
    var punishdetail = req.body.punishdetail;  

    // 建立一個新資料物件
    var newData={
        userno:userno,
        startdate:startdate,
        finishdate:finishdate,
        punishdetail:punishdetail
    } 
    
    punish.add(newData).then(d => {
        if (d==0){
            res.render('addSuccess');  //傳至成功頁面
        }else{
            res.render('addFail');     //導向錯誤頁面
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
var punish_add_form = require('./routes/punish_add_form'); //新增
var punish_add = require('./routes/punish_add');

app.use('/punish/add/form', checkAuth_E, punish_add_form); //新增
app.use('/punish/add', checkAuth_E, punish_add);
```