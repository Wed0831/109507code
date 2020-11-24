# E01-BookingRoom
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
## bookingroom.js(Utility)
```
'use strict';

//引用操作資料庫的物件
const sql = require('./asyncDB');

//------------------------------------------
// 新增商品
//------------------------------------------
var add = async function(newData){
    var result={};
    console.log(newData);
    await sql('INSERT INTO bookingroom (userno, reason, bookingdate, borrowdate, endtime, evidence, role, borrowtime) VALUES ($1, $2, $3, $4, $5, $6, $7, $8)', [newData.userno, newData.reason, newData.bookingdate, newData.borrowdate, newData.endtime, newData.evidence, newData.role, newData.borrowtime])
        .then((data) => {
            result = 0;  
        }, (error) => {
            result = -1;
        });
    console.log(result);
    return result;
}

var query = async function(bookingdate){
    var result={};
    
    await sql('SELECT * FROM "bookingroom" WHERE "bookingdate" = $1', [bookingdate])
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

var add_detail = async function(newData){
    var result;

    await sql('INSERT INTO bookingroom_detail (bookingroomno, roomno) VALUES ($1, $2)', [newData.bookingroomno, newData.roomno])
        .then((data) => {
            result = 0;  
        }, (error) => {
            result = -1;
        });
		
    return result;
}

//------------------------------------------
// 查詢是否有違規以及是否已經借用
//------------------------------------------
var query_verify = async function(userno){
    var result={};
    
    await sql('SELECT * FROM "bookingpunish" WHERE "userno" = $1', [userno])
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

var query_borrowdate = async function(borrowdate, borrowtime, endtime, roomno, yesorno){
    var result={};
    
    await sql('SELECT * FROM bookingroom as a JOIN bookingroom_detail as b on a.bookingroomno = b.bookingroomno WHERE a.borrowdate = $1 and a.borrowtime = $2 and a.endtime = $3 and b.roomno = $4 and a.yesorno = $5'
    , [borrowdate, borrowtime, endtime, roomno, yesorno])
        .then((data) => {
            result = data.rows; 
        }, (error) => {
            result = null;
        });
    
	console.log(result);	
    return result;
}

//匯出
module.exports = {add , query, add_detail, query_verify, query_borrowdate};
```
## bookingroom_form.js
```
var express = require('express');
var router = express.Router();

//接收GET請求
router.get('/', function(req, res, next) {
    res.render('bookingroom_form'); 
});

module.exports = router; 
```
## bookingroom_form.ejs
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
                        </td>
                    </tr>
                <% } %> 
            </tbody>    
        </table>
        <span class="value"><input type="submit" value="審核" ></span>    
    </div>
</form>
```
## bookingroom.js
```
var express = require('express');
var router = express.Router();

//增加引用函式
var moment = require('moment');
var today = require('silly-datetime');
const bookingroom = require('./utility/bookingroom');

//---------------------------
// 引用multer外掛
//---------------------------
const multer  = require('multer');

// 宣告上傳存放空間及檔名更改
var storage = multer.diskStorage({
    destination: function (req, file, cb) {
        cb(null, 'public/picture');
    },

    filename: function (req, file, cb) {
        cb(null, Date.now()+"--"+file.originalname);    
    }   
})

// 產生multer的上傳物件
var maxSize=1024*1024;  //設定最大可接受圖片大小(1M)

var upload = multer({
    storage:storage
})

//接收POST請求
router.post('/', upload.single('evidence'), function(req, res, next) {
    // 如果有選擇圖片
    if (typeof req.file != 'undefined'){
        // 傳入檔案不可超過maxSize
        if(req.file.size > maxSize){
            res.render('fileSizeError');  //圖片過大
            return;
        }                      
    }  

    var userno = req.body.userno;      //使用者編號                 
    var reason = req.body.reason;      //原因
    var roomno = req.body.roomno;      //教室 
    var bookingdate = today.format("YYYY-MM-DD HH:mm:ss"); //申請日期
    var borrowdate = moment(req.body.borrowdate).format("YYYY-MM-DD");  //借用日  
    var endtime = req.body.endtime;   //歸還時間
    var evidence;     //圖片歸還時間
    var role = req.body.role;           //身分
    var borrowtime = req.body.borrowtime;   //借用時間

    var borrowweek =  moment(req.body.borrowdate).format("E");  //借用日的星期

    req.session.bookingdate = bookingdate;  //借用日  
    req.session.roomno = roomno;    //教室編號
    req.session.userno = userno;    //使用者編號
    req.session.borrowdate = borrowdate;    //借用日
    req.session.borrowtime = borrowtime;    //借用時間
    req.session.endtime = endtime;    //歸還時間

    // 如果有選擇圖片
    if (typeof(req.file) != 'undefined'){
        evidence=req.file.filename;   //取得上傳照片名稱
    }


    // 建立一個新資料物件
    var newData={
        userno:userno,
        reason:reason,
        bookingdate:bookingdate,
        borrowdate:borrowdate,
        endtime:endtime,
        evidence:evidence,
        role:role,
        borrowtime:borrowtime
    } 

    //計算時間差----------------------------------------------------
    var btime = new Date(borrowdate + " " + borrowtime);
    var etime = new Date(borrowdate + " " + endtime);
    var dif = etime.getTime() - btime.getTime();

    //計算出小時數
    var leave1=dif%(24*3600*1000);    //計算天數後剩餘的毫秒數
    var hours=Math.floor(leave1/(3600*1000));

    //計算相差分鐘數
    var leave2=leave1%(3600*1000);        //計算小時數後剩餘的毫秒數
    var minutes=Math.floor(leave2/(60*1000));

    var dt = hours + ":" + minutes;
    console.log("dt：" + dt);
    //----------------------------------------------------------------
  
    //取得今天後 7 日的時間---------------------------------------------
    var bdate = new Date(bookingdate);
    var mo = bdate.getMonth() + 1;
    console.log("mo：" + mo);       //月

    var bd = bdate.getDate() + 7;
    console.log('bd：' + bd);       //日

    if(mo == '1' || mo == '3' || mo == '5' || mo == '7' || mo == '8' || mo == '10' || mo == '12'){
        if(bd/31 > 0){
            var nbd = bd%31;
            var nmo = mo+1;
        }else{
            var nbd = bd%31;
            var nmo = bdate.getMonth();
        }
    }else{
        if(bd/30 > 0){
            var nbd = bd%30;
            var nmo = mo+1;
        }else{
            var nbd = bd%30;
            var nmo = bdate.getMonth();
        }
    }

    console.log("nmo：" + nmo);       //月
    console.log('nbd：' + nbd);       //日

    var new_bd = nmo + "/" + nbd;
    console.log("new_bd：" + new_bd);
    
    //----------------------------------------------------------------
    bookingroom.query_verify(userno).then(data => {
        if (data==null){
           console.log('error');  //導向錯誤頁面
           res.render('error');
        }else if(data==-1){
            console.log("未有違規事項"); 

            if (borrowtime >= endtime){
                console.log('歸還時間不能小於借用時間');
                res.render('bt_less_et');
            }else if(borrowweek == '6' || borrowweek == '7'){
                console.log('假日無法借用');
                res.render('borrowweek');
            }else if(borrowdate < bookingdate || borrowdate < new_bd){
                console.log('只能借兩週內的教室(包含當週)');
                res.render('bd_more_newbd');
            }else if(borrowtime < "08:00" || endtime > "21:50"){
                console.log("早上8點前以及晚上9：50之後不借用");
                res.render('bt_between_et');
            }else if(role == "4" && dt > "2:00"){
                console.log("借用時間不可以大於2小時");
                res.render('bt_less_two');
            }else if(role == "2" && evidence == ""){
                console.log("TA借用教室必須有證明文件");
                res.render('evidence_null');
            }else{
                bookingroom.add(newData).then(d => {
                    if (d==0){
                        console.log('add：addSuccess');

                        //-----------------------------------------------------
                        var yesorno = 't';
    
                        bookingroom.query_borrowdate(borrowdate, borrowtime, endtime, roomno, yesorno).then(data => {
                            
                            if (data==null){                               
                                bookingroom.query(bookingdate).then(data => {
                                    if (data==null){
                                       console.log('error');  //導向錯誤頁面
                                       res.render('error');
                                    }else if(data==-1){
                                        console.log('notFound');     
                                        res.render('notFound');     
                                    }else{
                                        console.log('data：',data); 
                                        req.session.data = data;

                                        //-----------------------------------------------------
                                        var bookingroomno = data.bookingroomno;

                                        // 建立一個新資料物件
                                        var newData_detail={
                                            bookingroomno:bookingroomno,
                                            roomno:roomno
                                        }   

                                        bookingroom.add_detail(newData_detail).then(d => {
                                            if (d==0){
                                                res.render('audit');    //已送交審核
                                            }else{
                                                res.render('addFail');
                                            }  
                                        })
                                    }  
                                })
                            }else if(data==-1){  
                                res.render('notFound');     
                            }else if(data.length > 0){
                                console.log(data);
                                res.render('haveuse');
                            }else{
                                bookingroom.query(bookingdate).then(data => {
                                    if (data==null){
                                       console.log('error');  //導向錯誤頁面
                                       res.render('error');
                                    }else if(data==-1){
                                        console.log('notFound');     
                                        res.render('notFound');     
                                    }else{
                                        console.log('data：',data); 
                                        req.session.data = data;

                                        //-----------------------------------------------------
                                        var bookingroomno = data.bookingroomno;

                                        // 建立一個新資料物件
                                        var newData_detail={
                                            bookingroomno:bookingroomno,
                                            roomno:roomno
                                        }   

                                        bookingroom.add_detail(newData_detail).then(d => {
                                            if (d==0){
                                                res.render('audit');    //已送交審核
                                            }else{
                                                res.render('addFail');
                                            }  
                                        })
                                    }  
                                })
                            }
                        })
                    }else{
                        res.render('addFail');
                    }
                });
            }
        }else{
            var data = {
                userno: userno,
                startdate:  moment(data.startdate).format("YYYY-MM-DD"),   //懲罰開始日
                finishdate: moment(data.finishdate).format("YYYY-MM-DD")    //懲罰結束日
            }
            console.log(data);
            console.log(bookingdate);
            if(bookingdate > data.startdate & bookingdate < data.finishdate){ 
                res.render('inPunish');
            }else{
                if (borrowtime >= endtime){
                    res.render('bt_less_et');
                }else if(borrowweek == '6' || borrowweek == '7'){
                    res.render('borrowweek');
                }else if(borrowdate < bookingdate || borrowdate < new_bd){
                    res.render('bd_more_newbd');
                }else if(borrowtime < "08:00" || endtime > "21:50"){
                    res.render('bt_between_et');
                }else if(role == "4專題" && dt > "2:00"){
                    res.render('bt_less_two');
                }else if(role == "2TA" && evidence == ""){
                    res.render('evidence_null');
                }else{
                    bookingroom.add(newData).then(d => {
                        if (d==0){
                            console.log('add：addSuccess');

                            //-----------------------------------------------------
                            var yesorno = 't';
        
                            bookingroom.query_borrowdate(borrowdate, borrowtime, endtime, roomno, yesorno).then(data => {
                                
                                if (data==null){
                                    bookingroom.query(bookingdate).then(data => {
                                        if (data==null){
                                           res.render('error');
                                        }else if(data==-1){   
                                            res.render('notFound');     
                                        }else{
                                            console.log('data：',data); 
                                            req.session.data = data;
    
                                            //-----------------------------------------------------
                                            var bookingroomno = data.bookingroomno;
    
                                            // 建立一個新資料物件
                                            var newData_detail={
                                                bookingroomno:bookingroomno,
                                                roomno:roomno
                                            }   
    
                                            bookingroom.add_detail(newData_detail).then(d => {
                                                if (d==0){
                                                    res.render('audit');    //已送交審核
                                                }else{
                                                    res.render('addFail');
                                                }  
                                            })
                                        }  
                                    })
                                }else if(data==-1){
                                    res.render('notFound');     
                                }else if(data.length > 0){
                                    res.render('haveuse');
                                }else{
                                    bookingroom.query(bookingdate).then(data => {
                                        if (data==null){
                                        res.render('error');
                                        }else if(data==-1){
                                            res.render('notFound');     
                                        }else{
                                            console.log('data：',data); 
                                            req.session.data = data;

                                            //-----------------------------------------------------
                                            var bookingroomno = data.bookingroomno;

                                            // 建立一個新資料物件
                                            var newData_detail={
                                                bookingroomno:bookingroomno,
                                                roomno:roomno
                                            }   

                                            bookingroom.add_detail(newData_detail).then(d => {
                                                if (d==0){
                                                    res.render('audit');    //已送交審核
                                                }else{
                                                    res.render('auditFail');
                                                }  
                                            })
                                        }  
                                    })
                                }
                            })
                        }else{
                            res.render('auditFail');
                        }
                    });
                }
            }
        }
    });

});

module.exports = router;


```
## bt_less_et.ejs
```
<script>
    alert("歸還時間不能小於借用時間");
    history.go(-1);
</script>  
```
## borrowweek.ejs
```
<script>
    alert("假日無法借用");
    history.go(-1);
</script>  
```
## bd_more_newbd.ejs
```
<script>
    alert("只能兩週前借教室(包含當週)");
    history.go(-1);
</script> 
```
## bt_between_et.ejs
```
<script>
    alert("早上8點前以及晚上9：50之後不借用");
    history.go(-1);
</script>  
```
## bt_less_two.ejs
```
<script>
    alert("借用時間不可以大於2小時");
    history.go(-1);
</script>   
```
## evidence_null.ejs
```
<script>
    alert("TA借用教室必須有證明文件");
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
## haveuse.ejs
```
<script>
    alert("此時段有人使用");
    history.go(-1);
</script>   
```
## fileSizeError.ejs
```
<script>
    alert("圖檔過大, 上傳失敗!");
    location.href = "/";
</script> 
```
## audit.ejs
```
 <script>
    alert("已送交審核");
    location.href = "/";
</script>   
```
## auditFail.ejs
```
<script>
    alert("送交申請失敗");
    location.href = "/";
</script> 
```
## app.js
```
var bookingroom_form = require('./routes/bookingroom_form');  //教室借用
var bookingroom = require('./routes/bookingroom');//新增

app.use('/bookingroom/form', checkAuth_ST, bookingroom_form); //教室借用
app.use('/bookingroom', checkAuth_ST, bookingroom); //新增
```