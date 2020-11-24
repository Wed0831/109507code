# E02-BookingRoom Verify
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
## roomverify.js(Utility)
```
//------------------------------------------
//審核
//------------------------------------------
var query = async function(yesorno){
    var result={};

    await sql('SELECT * FROM bookingroom as a JOIN bookingroom_detail as b on a.bookingroomno = b.bookingroomno WHERE yesorno is null or a.yesorno = $1 ORDER BY a.bookingroomno, a."role"', [yesorno])
        .then((data) => {
            result = data.rows;
        }, (error) => {
            result = null;
        });       
    
    return result;
}

var update = async function(newData){
    var results=0;

    for(var i=0; i<newData.bookingroomno.length;i++){
        await sql('UPDATE bookingroom SET yesorno=$1 WHERE bookingroomno=$2', [newData.yesorno[i], newData.bookingroomno[i]])
        .then((data) => {
            results = results + data.rowCount;  
        }, (error) => {
            results = -1;
        });
    }

    return results;
}
//匯出
module.exports = {query, update};
```
## roomverify_update_form.js
```
var express = require('express');
var router = express.Router();

//增加引用函式
var moment = require('moment');
const roomverify = require('./utility/roomverify');

//接收GET請求
router.get('/', function(req, res, next) {
    var no = null;

    roomverify.query(no).then(d => {
        if (d!=null || d!=-1){
            if(d == null){
                res.render('no_verify')  //沒有審核資歷
            }else if(d.length >= 0){
                res.render('roomverify_update_form', {items:d, moment:moment});  //將資料傳給更新頁面
            }
        }else{
            res.render('notFound')  //導向找不到頁面
        }  
    })
});

//匯出
module.exports = router;
```
## roomverify_update_form.ejs
```
<h2>教室審核</h2>
<form action = "/roomverify/update" method = "post">
    <div class="form">
        <table>
            <thead>
                <tr>
                    <td>借用編號</td>
                    <td>學號</td>
                    <td>申請日</td>
                    <td>教室</td>
                    <td>借用原因</td>
                    <td>借用日</td>
                    <td>借用時間</td>
                    <td>歸還時間</td>
                    <td>身分</td>
                    <td>上傳圖片</td>
                    <td>是否同意</td>
                </tr>
            </thead> 
                    
            <tbody>
                <% for(var i=0; i<items.length; i++) {%>
                    <tr>
                        <td><%= items[i].bookingroomno %><input type="text" name="bookingroomno" value="<%= items[i].bookingroomno %>" hidden></td>
                        <td><%= items[i].userno %><input type="text" name="userno" value="<%= items[i].userno %>" hidden></td>
                        <td><%= moment(items[i].bookingdate).format('YYYY-MM-DD') %></td>
                        <td><%= items[i].roomno %></td>
                        <td><%= items[i].reason %></td>
                        <td><%= moment(items[i].borrowdate).format(' YYYY-MM-DD') %></td>
                        <td><%= items[i].borrowtime %></td>
                        <td><%= items[i].endtime %></td>
                        <td><%= items[i].role %></td>
                        <td><img src ='<%= items[i].evidence %>'></td>
                        <td>
                            <% if(items[i].yesorno == 't'){ %>
                                <% name = 'yesorno' + i %>
                                <input type="radio" name="<%= name %>" value="t" checked>是
                                <input type="radio" name="<%= name %>" value="f">否
                            <% } else{ %>
                                <% name = 'yesorno' + i %>
                                <input type="radio" name="<%= name %>" value="t" checked>是
                                <input type="radio" name="<%= name %>" value="f">否
                            <% } %>
                        </td><!---->
                    </tr>
                <% } %> 
            </tbody>    
        </table>
                    
                    
        <span class="value"><input type="submit" value="更新" ></span>    
    </div>
</form>
```
## roomverify_update.js
```
var express = require('express');
var router = express.Router();
var nodemailer = require('nodemailer');

//增加引用函式
const roomverify = require('./utility/roomverify');

//接收POST請求
router.post('/', function(req, res, next) {
    var userno = req.body.userno;
    if(typeof(userno) === 'string'){
        userno = [userno];
    }
    var email = [];

    var bookingroomno = req.body.bookingroomno;   //取得借用編號
    if(typeof(bookingroomno) === 'string'){
        bookingroomno = [bookingroomno];
    }
    var yn = [req.body.yesorno0, req.body.yesorno1, req.body.yesorno2, req.body.yesorno3, req.body.yesorno4, req.body.yesorno5, req.body.yesorno6, req.body.yesorno7, req.body.yesorno8, req.body.yesorno9
    ,req.body.yesorno10, req.body.yesorno11, req.body.yesorno12, req.body.yesorno13, req.body.yesorno14, req.body.yesorno15, req.body.yesorno16, req.body.yesorno17, req.body.yesorno18, req.body.yesorno19
    ,req.body.yesorno20, req.body.yesorno21, req.body.yesorno22, req.body.yesorno23, req.body.yesorno24, req.body.yesorno25, req.body.yesorno26, req.body.yesorno27, req.body.yesorno28, req.body.yesorno29
    ,req.body.yesorno30, req.body.yesorno31, req.body.yesorno32, req.body.yesorno33, req.body.yesorno34, req.body.yesorno35, req.body.yesorno36, req.body.yesorno37, req.body.yesorno38, req.body.yesorno39
    ,req.body.yesorno40, req.body.yesorno41, req.body.yesorno42, req.body.yesorno43, req.body.yesorno44, req.body.yesorno45, req.body.yesorno46, req.body.yesorno47, req.body.yesorno48, req.body.yesorno49];
    var yesorno = [];

    for(var i=0;i<bookingroomno.length;i++){
        yesorno.push(yn[i]);
    }

    var newData={
        bookingroomno:bookingroomno,
        yesorno:yesorno
    };


    roomverify.update(newData).then(d => {
        if (d>=0){
            console.log("updateSuccess");
            for(var i=0;i<bookingroomno.length;i++){
                    email.push(userno[i] + "@ntub.edu.tw");

                //宣告發信物件
                var transporter = nodemailer.createTransport({
                    service: 'Gmail',
                    auth: {
                        user: 'simplesemaster@gmail.com',
                        pass: 'ntub109507'
                    }
                });

                var options = {
                    //寄件者
                    from: 'simplesemaster@gmail.com',
                    //收件者
                    to: userno[i] + "@ntub.edu.tw", 
                    //主旨
                    subject: '審核結果', // Subject line
                    //純文字
                    text: '可以到借用紀錄查看審核結果了, https://simple-semaster.herokuapp.com/' // plaintext body
                };

                transporter.sendMail(options, function(error, info){
                    if(error){
                        console.log(error);
                    }else{
                        res.render('updateSuccess');
                    }
                });
            }
        }else{
            res.render('updateFail');
        }  
    })
});

//匯出
module.exports = router;
```
## no_verify.ejs
```
<script>
    alert("沒有審核資料");
    location.href = "/";
</script>  
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
var roomverify_update_form = require('./routes/roomverify_update_form');//審核
var roomverify_update = require('./routes/roomverify_update');

app.use('/roomverify/update/form', checkAuth_ST, roomverify_update_form);//審核
app.use('/roomverify/update', checkAuth_ST, roomverify_update);
```