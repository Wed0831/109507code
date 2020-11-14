# C03-Room Update
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
//------------------------------------------
//執行資料庫動作的函式-取出單一商品
//------------------------------------------
var query = async function(roomno){
    var result={};
    
    await sql('SELECT * FROM room WHERE roomno = $1', [roomno])
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

    await sql('UPDATE room SET roomname=$1, roomdetail=$2 WHERE roomno = $3', [newData.roomname, newData.roomdetail, newData.roomno])
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
## room_update_no.js
```
var express = require('express');
var router = express.Router();

/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('room_update_no');
});

//匯出
module.exports = router;
```
## room_update_no.ejs
```
<h2>教室更新</h2>
<form action = "/room/update/form" method = "get">
    <div class="form">
        <span class="name">編號: </span>
        <span class="value"><input type="text" name="roomno"></span>
        <br/>    

        <span class="name"></span>
        <span class="value"><input type="submit" value="查詢" ></span>    
    </div>    
</form> 
```
## room_update_form.js
```
var express = require('express');
var router = express.Router();

//增加引用函式
const room = require('./utility/room');

//接收GET請求
router.get('/', function(req, res, next) {
    var no = req.query.roomno;

    room.query(no).then(d => {
        if (d!=null && d!=-1){
            var data = {
                roomno: d.roomno,
                roomname: d.roomname,
                roomdetail: d.roomdetail
            }

            res.render('room_update_form', {item:data});  //將資料傳給更新頁面
        }else{
            res.render("notFound")  //導向找不到頁面
        }  
    })
});

//匯出
module.exports = router;
```
## room_update_form.ejs
```
<h2>教室更新</h2>
<form action = "/room/update" method = "post">
    <div class="form">

        <span class="name">教室編號</span>
        <span class="value"><input type="text" name="roomno" value="<%= item.roomno %>" readonly></span>
        <br/>

        <span class="name">名稱</span>
        <span class="value"><input type="text" name="roomname" value="<%= item.roomname %>" ></span>
        <br/>

        <span class="name">細項</span>
        <span class="value"><input type="text" name="roomdetail" value="<%= item.roomdetail %>"></span>
        <br/>

        <span class="value"><input type="submit" value="更新" ></span>    
        </div>
</form>
```
## room_update.js
```
var express = require('express');
var router = express.Router();

//增加引用函式
const room = require('./utility/room');

//接收POST請求
router.post('/', function(req, res, next) {
    var roomno = req.body.roomno;   //取得產品編號

    var newData={
        roomno:roomno,                   
        roomname: req.body.roomname,   
        roomdetail: req.body.roomdetail 
    } 
    
    room.update(newData).then(d => {
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
var room_update_no = require('./routes/room_update_no');  //更新
var room_update_form = require('./routes/room_update_form');
var room_update = require('./routes/room_update');


app.use('/room/update/no', checkAuth_E, room_update_no); //更新
app.use('/room/update/form', checkAuth_E, room_update_form);
app.use('/room/update', checkAuth_E, room_update);
```