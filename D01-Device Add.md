# D01-Device Add
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
// 新增商品
//------------------------------------------
var add = async function(newData){
    var result;

    await sql('INSERT INTO device (deviceno, devicename, devicedetail) VALUES ($1, $2, $3)', [newData.deviceno, newData.devicename, newData.devicedetail])
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
## device_add_form.js
```
var express = require('express');
var router = express.Router();

//接收GET請求
router.get('/', function(req, res, next) {
    res.render('device_add_form'); 
});

module.exports = router; 
```
## device_add_form.ejs
```
<h2>設備新增</h2>
<form action = "/device/add" method = "post">
        <div class="form">
            <span class="name">編號: </span>
            <span class="value"><input type="text" name="deviceno"></span>
            <br/>

            <span class="name">名稱: </span>
            <span class="value"><input type="text" name="devicename"></span>
            <br/>

            <span class="name">細項: </span>
            <span class="value"><input type="text" name="devicedetail" ></span>
            <br/>
                                            
            <span class="name"></span>
            <span class="value"><input type="submit" value="增加" ></span>    
        </div>    
</form>    
```
## device_add.js
```
var express = require('express');
var router = express.Router();

//增加引用函式
const device = require('./utility/device');

//接收POST請求
router.post('/', function(req, res, next) {
    var deviceno = req.body.deviceno;               //編號
    var devicename = req.body.devicename;           //名稱
    var devicedetail = req.body.devicedetail;       //細項

    // 建立一個新資料物件
    var newData={
        deviceno:deviceno,
        devicename:devicename,
        devicedetail:devicedetail,
    } 
    
    device.add(newData).then(d => {
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
var device_add_form = require('./routes/device_add_form'); //新增
var device_add = require('./routes/device_add');


app.use('/device/remove/form', checkAuth_E, device_remove_form); //刪除
app.use('/device/remove', checkAuth_E, device_remove);
```