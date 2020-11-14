# B02-Room Remove
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
## room.js(Utility)
```
//----------------------------------
// 刪除商品
//----------------------------------
var remove = async function(roomno){
    var result;

    await sql('DELETE FROM room WHERE roomno = $1', [roomno])
        .then((data) => {
            result = data.rowCount;  
        }, (error) => {
            result = -1;
        });
		
    return result;
}

//匯出
module.exports = {remove};
```
## room_remove_form.js
```
var express = require('express');
var router = express.Router();

//接收GET請求
router.get('/', function(req, res, next) {
    res.render('room_remove_form'); 
});

module.exports = router;
```
## room_remove_form.ejs
```
<h2>教室刪除</h2> 
<form action = "/room/remove" method = "post">
    <div class="form">
        <span class="name">編號: </span>
        <span class="value"><input type="text" name="roomno"></span>
        <br/>
                                                    
        <span class="name"></span>
        <span class="value"><input type="submit" value="刪除" ></span>    
    </div>    
</form>     
```
## room_remove.js
```
var express = require('express');
var router = express.Router();

//增加引用函式
const room = require('./utility/room');

//接收POST請求
router.post('/', function(req, res, next) {
    var roomrno = req.body.roomrno;   //取得編號
   
    room.remove(roomrno).then(d => {
        if(d>0){  
            res.render('removeSuccess');
        }else{
            res.render('removeFail');
        }
    })    
});

module.exports = router;
```
## removeSuccess.ejs
```
<script>
    alert("已刪除資料");
    location.href = "/";
</script> 
```
## removeFail.ejs
```
<script>
    alert("刪除失敗");
    history.go(-1);;
</script>  
```
## app.js
```
var room_remove_form = require('./routes/room_remove_form');  //刪除
var room_remove = require('./routes/room_remove');

app.use('/room/remove/form', checkAuth_E, room_remove_form); //刪除
app.use('/room/remove', checkAuth_E, room_remove);
```