# F01-BookingDevice
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
## bookingdevice.js(Utility)
```
'use strict';

//引用操作資料庫的物件
const sql = require('./asyncDB');

//------------------------------------------
// 新增商品
//------------------------------------------
var add = async function(newData){
    var result={};

    await sql('INSERT INTO bookingdevice (userno, category, bookingdate, borrowdate, returndate, teleno) VALUES ($1, $2, $3, $4, $5, $6)', [newData.userno, newData.category, newData.bookingdate, newData.borrowdate, newData.returndate, newData.teleno])
        .then((data) => {
            result = 0;  
        }, (error) => {
            result = -1;
        });
        
    return result;
}

var query = async function(bookingdate){
    var result={};
    
    await sql('SELECT * FROM "bookingdevice" WHERE "bookingdate" = $1', [bookingdate])
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
    var result={};

    await sql('INSERT INTO bookingdevice_detail (bookingdeviceno) VALUES ($1)', [newData.bookingdeviceno])
        .then((data) => {
            result = 0;  
        }, (error) => {
            result = -1;
        });
        
    return result;
}

//------------------------------------------
// 查詢是否有違規
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

//匯出
module.exports = {add, query, add_detail, query_verify};
```
## bookingdevice_form.js
```
var express = require('express');
var router = express.Router();

//接收GET請求
router.get('/', function(req, res, next) {
    res.render('bookingdevice_form'); 
});

module.exports = router; 
```
## bookingdevice_form.ejs
```
<h2>設備申請</h2>
<form action = "/bookingdevice" method = "post">
    <div class="form">
        <span class="name">學號: </span>
        <span class="value"><input type="text" name="userno" required></span>
        <br/>

        <span class="name">連絡電話: </span>
        <span class="value"><input type="text" name="teleno" required></span>
        <br/>  

        <span class="name">設備: </span>
        <span class="value">
            <select name="category" required>
                <option value="電腦">電腦</option>
                <option value="滑鼠">滑鼠</option>
                <option value="Zenbo">Zenbo</option>
                <option value="音源線">音源線</option>
                <option value="鍵盤">鍵盤</option>
                <option value="計算機">計算機</option>
            </select>
        </span>    
        <br/>
                                                        
        <span class="name">借用日: </span>
        <span class="value"><input type="date" name="borrowdate" required></span>
        <br/>
                                            
        <span class="name">歸還日: </span>
        <span class="value"><input type="date" name="returndate" required></span>
        <br/>	    
                                    
        <span class="name"></span>
        <span class="value"><input type="submit" value="新增" ></span>    
    </div>    
</form>      
```
## bookingdevice.js
```
var express = require('express');
var router = express.Router();

//增加引用函式
var moment = require('moment');
var today = require('silly-datetime');
const bookingdevice = require('./utility/bookingdevice');

//接收POST請求
router.post('/', function(req, res, next) {
    var userno = req.body.userno;               //學號
    var category = req.body.category;           //設備
    var bookingdate = today.format("YYYY-MM-DD HH:mm:ss"); //申請日期 
    var borrowdate = moment(req.body.borrowdate).format("YYYY-MM-DD HH:mm:ss");       //借用日
    var returndate = moment(req.body.returndate).format("YYYY-MM-DD HH:mm:ss");       //歸還日
    var teleno = req.body.teleno;               //連絡電話
    var borrowweek =  moment(req.body.borrowdate).format("E");  //借用日的星期
    var returnwweek =  moment(req.body.returndate).format("E");  //歸還日的星期

    req.session.bookingdate = bookingdate;

    // 建立一個新資料物件
    var newData={
        userno:userno,
        category:category,
        bookingdate:bookingdate,
        borrowdate:borrowdate,
        returndate:returndate,
        teleno:teleno
    } 
    console.log(newData);

    bookingdevice.query_verify(userno).then(data => {
        if (data==null){
           console.log('error');  //導向錯誤頁面
        }else if(data==-1){

            if (borrowdate >= returndate){
                res.render('bt_less_et');
            }else if(borrowweek == '6' || borrowweek == '7'){
                res.render('borrowweek');
            }else if(returnwweek == '6' || returnwweek == '7'){
                res.render('returnwweek');
            }else{
                bookingdevice.add(newData).then(d => {
                    if (d==0){
                        console.log("addSuccess bookingdevice");

                        bookingdevice.query(bookingdate).then(data => {
                            if (data==null){
                               console.log('error');  //導向錯誤頁面
                               res.render('error');
                            }else if(data==-1){
                                console.log('notFound');     
                                res.render('notFound');     
                            }else{
                                var bookingdeviceno = data.bookingdeviceno;
                                // 建立一個新資料物件到bookingdevice_detail
                                var newData_detail={bookingdeviceno:bookingdeviceno}  

                                bookingdevice.add_detail(newData_detail).then(d => {
                                    if (d==0){
                                        res.render('audit');    //已送交審核
                                    }else{
                                        res.render('auditFail');
                                    }  
                                })
                            }  
                        })
                    }else{
                        res.render('addFail');     //導向錯誤頁面
                    }  
                })
            }
            
        }else{
            var data = {
                userno: userno,
                startdate:  moment(data.startdate).format("YYYY-MM-DD"),   //懲罰開始日
                finishdate: moment(data.finishdate).format("YYYY-MM-DD")    //懲罰結束日
            }
            console.log(data);
            if(bookingdate > data.startdate & bookingdate < data.finishdate){ 
                res.render('inPunish');
            }else{
                if (borrowdate >= returndate){
                    res.render('bt_less_et');
                }else if(borrowweek == '6' || borrowweek == '7'){
                    res.render('borrowweek');
                }else if(returnwweek == '6' || returnwweek == '7'){
                    res.render('returnwweek');
                }else{
                    bookingdevice.add(newData).then(d => {
                        if (d==0){
                            console.log("addSuccess bookingdevice");
    
                            bookingdevice.query(bookingdate).then(data => {
                                if (data==null){
                                   res.render('error');
                                }else if(data==-1){
                                    res.render('notFound');     
                                }else{
                                    var bookingdeviceno = data.bookingdeviceno;
                                    // 建立一個新資料物件到bookingdevice_detail
                                    var newData_detail={bookingdeviceno:bookingdeviceno}  
    
                                    bookingdevice.add_detail(newData_detail).then(d => {
                                        if (d==0){
                                            res.render('audit');    //已送交審核
                                        }else{
                                            res.render('auditFail');
                                        }  
                                    })
                                }  
                            })
                        }else{
                            res.render('addFail');     //導向錯誤頁面
                        }  
                    })
                }
            }
        }  
    })
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
## returnweek.ejs
```
<script>
    alert("假日不可以歸還");
    history.go(-1);
</script> 
```
## inPunish.ejs
```
<script>
    alert("在懲罰期間不能借用");
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
    alert("送交審核失敗");
    location.href = "/";
</script> 
```
## app.js
```
var bookingdevice_form = require('./routes/bookingdevice_form');  //設備借用
var bookingdevice = require('./routes/bookingdevice');//新增 

app.use('/bookingdevice/form', checkAuth_ST, bookingdevice_form); //設備借用
app.use('/bookingdevice', checkAuth_ST, bookingdevice); //新增
```