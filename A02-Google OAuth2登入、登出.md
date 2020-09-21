# A02-Google OAuth2登入、登出
## node掛件
```
npm install passport --save
npm install passport-google-oauth20 --save
npm install express-session --save
```

## app.js
```
var express = require('express');
var path = require('path');
var favicon = require('serve-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');

var index = require('./routes/index');
var users = require('./routes/users');

var app = express();
//---------------------------------------------
// 使用passport-google-oauth2套件進行認證
//---------------------------------------------
var passport = require('passport');

app.use(require('express-session')({
    secret: 'keyboard cat',
    resave: true,
    saveUninitialized: true
}));

app.use(passport.initialize());
app.use(passport.session());

passport.serializeUser(function(user, done) {
    done(null, user);
});

passport.deserializeUser(function(user, done) {
    done(null, user);
});

//載入google oauth2
var GoogleStrategy = require('passport-google-oauth20').Strategy;

//填入自己在google cloud platform建立的憑證
passport.use(
    new GoogleStrategy({
        clientID: '自己的用戶端ID', 
        clientSecret: '自己的用戶端密碼',
        callbackURL: "http://localhost/auth/google/callback"  
    },
    function(accessToken, refreshToken, profile, done) {
      console.log(profile);
        if (profile) {
            return done(null, profile);
        }else {
            return done(null, false);
        }
        
    }
));

//---------------------------------------------

// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'ejs');

// uncomment after placing your favicon in /public
//app.use(favicon(path.join(__dirname, 'public', 'favicon.ico')));
app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));

app.use('/', index);
app.use('/users', users);

//---------------------------------------------
// Get profile from google
//---------------------------------------------
app.get('/auth/google',
  passport.authenticate('google', { scope: ['profile','email'] }));

app.get('/auth/google/callback', 
  passport.authenticate('google', { failureRedirect: '/login' }),
  function(req, res) {
    // Successful authentication, redirect home.
    console.log(req.session.passport.user.id);                //ID
    console.log(req.session.passport.user.displayName);       //姓名
    console.log(req.session.passport.user.emails[0].value);	  //帳號
    req.session.email = req.session.passport.user.emails[0].value;  //把帳號放到session裡，之後都從session抓就號
    res.redirect('/user_type');
  });

app.get('/user/logout', function(req, res){    
    req.logout();        //將使用者資料從session移除
    res.redirect('/logout');   //導向登出頁面
  });    
```

## logout.js
```
var express = require('express');
var router = express.Router();

//接收POST請求
router.get('/', function(req, res, next) {       
    res.render('logout');  //傳至登出    
});

module.exports = router;
```

## logout.ejs
```
<script>
    alert("登出成功");
    location.href = "/";
</script>  
```