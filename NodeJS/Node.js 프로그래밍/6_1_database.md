## 데이터베이스 사용하기

노드에서 데이터 저장이 필요할 때 몽고디비를 사용하는 경우가 많습니다. 몽고디비는 기존에 자주 사용하던 관계형 데이터베이스(Relational Database)와 달리 SQL을 사용하지 않습니다.
또 자바스크립트 객체를 그대로 저장할 수 있어서 데이터 조회 방식도 SQL과 다릅니다. 하지만 데이터를 저장하거나 조회하는 방법을 따로 제공하기 때문에 몇 가지 사용법만 알아 두면
쉽게 사용할 수 있다.

**몽고디비 시작**

```

mongod --dbpath /Users/user/database/local

```

**몽고디비 데이터 추가 및 조회**

```

mongo

use local

```

**데이터 삽입**

```

db.users.insert({name:'소녀시대',age:20})

```

**조회**

```

db.users.find().pretty()

```

pretty() 메소드는 결과가 출력될 때 예쁘게 보이도록 조회된 결과에 띄어쓰기나 줄 바꿈을 자동으로 넣어 준다.

**정규표현식 이용한 데이터 삭제**

```

db.users.remove('{name':/소녀/})

```

**Content Type**

HTTP 메시지(요청과 응답 모두)에 담겨 보내는 데이터의 형식을 알려주는 헤더

> application/x-www-form-urlencoded

&으로 분리되고, "=" 기호로 값과 키를 연결하는 key-value tuple로 인코딩되는 값이다. 

영어 알파벳이 아닌 문자들은 percent encoded을 인코딩된다.

> application/json

 json 형식의 데이터
 
 ## mongoDB 
 
 **app.js**
 
 ```
 
// Express 모듈 불러오기
let express = require('express');
let http = require('http');
let path = require('path');

// Express 미들웨어 불러오기
let bodyParser = require('body-parser');
let cookieParser = require('cookie-parser');
let static = require('serve-static');

// 오류 핸들러 사용
let expressErrorHandler = require('express-error-handler');

// Session 미들웨어 불러오기
let expressSession = require('express-session');

// 몽고디비 모듈 사용
let MongoClient = require('mongodb').MongoClient;

let app = express();

app.set('port', process.env.PORT || 3000);

// body-parser를 사용해 application/x-www-form-urlenceded 파싱
app.use(bodyParser.urlencoded({ extended: false }));

// body-parsr를 사용해 application/json 파싱
app.use(bodyParser.json());

// public 폴더를 static으로 오픈
app.use('/public', static(path.join(__dirname, 'public')));

// cookie-parser 설정
app.use(cookieParser());

// 세션 설정
app.use(
  expressSession({
    secret: 'my key',
    resave: true,
    saveUninitialized: true,
  })
);

// 데이터베이스 객체를 위한 변수 선언
let database;

// 데이터베이스 연결
function connectDB() {
  // 데이터베이스 연결 정보
  let databaseUrl = 'mongodb://localhost:27017/local';

  // 데이터베이스 연결
  MongoClient.connect(databaseUrl, function (err, db) {
    if (err) throw err;

    console.log('데이터베이스에 연결되었습니다.' + databaseUrl);

    // database 변수에 할당
    database = db.db('local');
  });
}

// 사용자를 인증하는 함수
let authUser = function (database, id, password, callback) {
  console.log('authUser 호출됨');

  // users 컬렉션 참조
  let users = database.collection('users');

  // 아이디와 비밀번호를 사용해 검색
  users.find({ id: id, password: password }).toArray(function (err, docs) {
    if (err) {
      callback(err, null);
      return;
    }

    if (docs.length > 0) {
      console.log(
        '아이디 [%s], 비밀번호 [%s]가 일치하는 사용자 찾음.',
        id,
        password
      );
      callback(null, docs);
    } else {
      console.log('일치하는 사용자를 찾지 못함.');
      callback(null, null);
    }
  });
};

// 사용자를 추가하는 함수
let addUser = function (database, id, password, name, callback) {
  console.log('addUser 호출됨 : ' + id + ', ' + password + ', ' + name);

  // users 컬렉션 참조
  let users = database.collection('users');

  // id, password, username을 사용해 사용자 추가
  users.insertMany(
    [{ id: id, password: password, name: name }],
    function (err, result) {
      if (err) {
        // 오류가 발생했을 때 콜백 함수를 호출하면서 오류 객체 전달
        callback(err, null);
        return;
      }

      // 오류가 아닌 경우, 콜백 함수를 호출하면서 결과 객체 전달
      if (result.insertedCount > 0) {
        console.log('사용자 레코드 추가됨 : ' + result.insertedCount);
      } else {
        console.log('추가된 레코드가 없음');
      }

      callback(null, result);
    }
  );
};

// 라우팅 객체 참조
let router = express.Router();

// 로그인 라우팅 함수 - 데이터베이스의 정보와 비교
router.route('/process/login').post(function (req, res) {
  console.log('/process/login 호출됨.');
});

app.post('/process/login', function (req, res) {
  console.log('/process/login 호출됨.');

  let paramId = req.body.id || req.query.id;
  let paramPassword = req.body.password || req.query.password;

  if (database) {
    authUser(database, paramId, paramPassword, function (err, docs) {
      if (err) {
        throw err;
      }

      if (docs) {
        console.dir(docs);
        let username = docs[0].name;
        res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
        res.write('<h1>로그인 성공</h1>');
        res.write('<div><p>사용자 아이디 : ' + paramId + '</p></div>');
        res.write('<div><p>사용자 이름 : ' + username + '</p></div>');
        res.write("<br><br><a href='/public/login.html'>다시 로그인하기</a>");
        res.end();
      } else {
        res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
        res.write('<hi>로그인 실패</h1>');
        res.write('<div><p>아이디와 비밀번호를 다시 확인하십시오.</p></div>');
        res.write("<br><br><a href='/public/login.html'>다시 로그인하기</a>");
        res.end();
      }
    });
  } else {
    res.wrtieHead('200', { 'Content-Type': 'text/html;charset=utf8' });
    res.write('<h2>데이터베이스 연결 실패</h2>');
    res.write('<div><p>데이터베이스에 연결하지 못했습니다.</p></div>');
    res.end();
  }
});

// 사용자 추가 라우팅 함수 - 클라이언트에서 보내온 데이터를 이용해 데이터베이스에 추가
router.route('/process/adduser').post(function (req, res) {
  console.log('/process/adduser 호출됨.');

  let paramId = req.body.id || req.query.id;
  let paramPassword = req.body.password || req.query.password;
  let paramName = req.body.name || req.query.name;

  console.log(
    '요청 파라미터 : ',
    paramId + ', ' + paramPassword + ' ' + paramName
  );

  // 데이터베이스 객체가 초기화된 경우, addUser 함수 호출하여 사용자 추가
  if (database) {
    addUser(
      database,
      paramId,
      paramPassword,
      paramName,
      function (err, result) {
        if (err) {
          throw err;
        }

        // 결과 객체 확인하여 추가된 데이터 있으면 성공 응답 전송
        if (result && result.insertedCount > 0) {
          console.dir(result);

          res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
          res.write('<h2>사용자 추가 성공</h2>');
          res.end();
        } else {
          // 결과 객체가 없으면 실패 응답 전송
          res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
          res.write('<h2>데이터베이스 연결 실패</h2>');
          res.end();
        }
      }
    );
  } else {
    // 데이터베이스 객체가 초기화되지 않은 경우 실패 응답 전송
    res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
    res.write('<h2>데이터베이스 연결 실패</h2>');
    res.end();
  }
});

// 라우터 객체 등록
app.use('/', router);

//===== 404 오류 페이지 처리 =====//
let errorHandler = expressErrorHandler({
  static: {
    404: './public/404.html',
  },
});

app.use(expressErrorHandler.httpError(404));
app.use(errorHandler);

//===== 서버 시작 =====//
http.createServer(app).listen(app.get('port'), function () {
  console.log('서버가 시작되었습니다. 포트: ' + app.get('port'));

  connectDB();
});
 
```

**login.html**

```

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <form method="post" action="/process/login">
        <table>
            <tr>
                <td><label>아이디</label></td>
                <td><input type="text" name="id"></td>
            </tr>
            <tr>
                <td><label>비밀번호</label></td>
                <td><input type="password" name="password"></td>
            </tr>
        </table>
        <input type="submit" value="전송" name="">
    </form>
</body>
</html>

```

**adduser.html**

```

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <h1>사용자 추가</h1>
    <form method="post" action="/process/adduser">
        <table>
            <tr>
                <td><label>아이디</label></td>
                <td><input type="text" name="id"></td>
            </tr>
            <tr>
                <td><label>비밀번호</label></td>
                <td><input type="password" name="password"></td>
            </tr>
            <tr>
                <td><label>사용자명</label></td>
                <td><input type="text" name="name"></td>
            </tr>
        </table>
        <input type="submit" value="전송" name="">
    </form>
</body>
</html>

```

## mongoose 사용하기

**app.js**

```

// Express 모듈 불러오기
let express = require('express');
let http = require('http');
let path = require('path');

// Express 미들웨어 불러오기
let bodyParser = require('body-parser');
let cookieParser = require('cookie-parser');
let static = require('serve-static');

// 오류 핸들러 사용
let expressErrorHandler = require('express-error-handler');

// Session 미들웨어 불러오기
let expressSession = require('express-session');

// 몽고디비 모듈 사용
let MongoClient = require('mongodb').MongoClient;

// mongoose 모듈 불러오기
let mongoose = require('mongoose');

let app = express();

app.set('port', process.env.PORT || 3000);

// body-parser를 사용해 application/x-www-form-urlenceded 파싱
app.use(bodyParser.urlencoded({ extended: false }));

// body-parsr를 사용해 application/json 파싱
app.use(bodyParser.json());

// public 폴더를 static으로 오픈
app.use('/public', static(path.join(__dirname, 'public')));

// cookie-parser 설정
app.use(cookieParser());

// 세션 설정
app.use(
  expressSession({
    secret: 'my key',
    resave: true,
    saveUninitialized: true,
  })
);

// 데이터베이스 객체를 위한 변수 선언
let database;

// 데이터베이스 스키마 객체를 위한 변수 선언
let UserSchema;

// 데이터베이스 모델 객체를 위한 변수 선언
let UserModel;

// 데이터베이스 연결
function connectDB() {
  //데이터베이스 연결 정보
  let databaseUrl = 'mongodb://localhost:27017/local';

  // 데이터베이스 연결
  console.log('데이터베이스 연결을 시도합니다.');
  mongoose.Promise = global.Promise;
  mongoose.connect(databaseUrl);
  database = mongoose.connection;

  database.on(
    'error',
    console.error.bind(console, 'mongoose connection error.')
  );
  database.on('open', function () {
    console.log('데이터베이스에 연결되었습니다 : ' + databaseUrl);

    // 스키마 정의
    UserSchema = mongoose.Schema({
      id: { type: String, required: true, unique: true },
      password: { type: String, required: true },
      name: { type: String, index: 'hashed' },
      age: { type: Number, default: -1 },
      created_at: { type: Date, index: { unique: false }, default: Date.now },
      updated_at: { type: Date, index: { unique: false }, default: Date.now },
    });
    console.log('UserSchema 정의함.');

    // 스키마에 static 메소드 추가
    UserSchema.static('findById', function (callback) {
      return this.find({ id: id }, callback);
    });

    UserSchema.static('findAll', function (callback) {
      return this.find({}, callback);
    });

    console.log('UserSchema 정의함.');

    // UserModel 모델 정의
    UserModel = mongoose.model('users2', UserSchema);
    console.log('UserModel 정의함.');
  });

  // 연결 끊어졌을 때 5초 후 재연결
  database.on('disconnected', function () {
    console.log('연결이 끊어졌습니다. 5초 후 다시 연결합니다.');
    setInterval(connectDB, 5000);
  });
}

// 사용자를 인증하는 함수 : 아이디로 먼저 찾고 비밀번호를 그다음에 비교
let authUser = function (database, id, password, callback) {
  console.log('authUser 호출됨');

  // 1. 아이디를 사용해 검색
  UserModel.findById(id, function (err, results) {
    if (err) {
      callback(err, null);
      return;
    }

    console.log('아이디 [%s]로 사용자 검색 결괴', id);
    console.dir(results);

    if (results.length > 0) {
      console.log('아이디와 일치하는 사용자 찾음.');

      // 2. 비밀번호 확인
      if (results[0]._doc.password == password) {
        console.log('비밀번호 일치함');
        callback(null, results);
      } else {
        console.log('비밀번호 일치하지 않음');
        callback(null, null);
      }
    } else {
      console.log('아이디와 일치하는 사용자를 찾지 못함.');
      callback(null, null);
    }
  });
};

// 사용자를 추가하는 함수
let addUser = function (database, id, password, name, callback) {
  console.log('addUser 호출됨 : ' + id + ', ' + password + ', ' + name);

  // UserModel의 인스턴스 생성
  let user = new UserModel({ id: id, password: password, name: name });

  // save()로 저장
  user.save(function (err) {
    if (err) {
      callback(err, null);
      return;
    }
    console.log('사용자 데이터 추가함.');
    callback(null, user);
  });
};

// 라우팅 객체 참조
let router = express.Router();

// 로그인 라우팅 함수 - 데이터베이스의 정보와 비교
router.route('/process/login').post(function (req, res) {
  console.log('/process/login 호출됨.');
});

app.post('/process/login', function (req, res) {
  console.log('/process/login 호출됨.');

  let paramId = req.body.id || req.query.id;
  let paramPassword = req.body.password || req.query.password;

  if (database) {
    authUser(database, paramId, paramPassword, function (err, docs) {
      if (err) {
        throw err;
      }

      if (docs) {
        console.dir(docs);
        let username = docs[0].name;
        res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
        res.write('<h1>로그인 성공</h1>');
        res.write('<div><p>사용자 아이디 : ' + paramId + '</p></div>');
        res.write('<div><p>사용자 이름 : ' + username + '</p></div>');
        res.write("<br><br><a href='/public/login.html'>다시 로그인하기</a>");
        res.end();
      } else {
        res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
        res.write('<hi>로그인 실패</h1>');
        res.write('<div><p>아이디와 비밀번호를 다시 확인하십시오.</p></div>');
        res.write("<br><br><a href='/public/login.html'>다시 로그인하기</a>");
        res.end();
      }
    });
  } else {
    res.wrtieHead('200', { 'Content-Type': 'text/html;charset=utf8' });
    res.write('<h2>데이터베이스 연결 실패</h2>');
    res.write('<div><p>데이터베이스에 연결하지 못했습니다.</p></div>');
    res.end();
  }
});

// 사용자 추가 라우팅 함수 - 클라이언트에서 보내온 데이터를 이용해 데이터베이스에 추가
router.route('/process/adduser').post(function (req, res) {
  console.log('/process/adduser 호출됨.');

  let paramId = req.body.id || req.query.id;
  let paramPassword = req.body.password || req.query.password;
  let paramName = req.body.name || req.query.name;

  console.log(
    '요청 파라미터 : ',
    paramId + ', ' + paramPassword + ' ' + paramName
  );

  // 데이터베이스 객체가 초기화된 경우, addUser 함수 호출하여 사용자 추가
  if (database) {
    addUser(
      database,
      paramId,
      paramPassword,
      paramName,
      function (err, result) {
        if (err) {
          throw err;
        }

        // 결과 객체 확인하여 추가된 데이터 있으면 성공 응답 전송
        if (result) {
          console.dir(result);

          res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
          res.write('<h2>사용자 추가 성공</h2>');
          res.end();
        } else {
          // 결과 객체가 없으면 실패 응답 전송
          res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
          res.write('<h2>데이터베이스 연결 실패</h2>');
          res.end();
        }
      }
    );
  } else {
    // 데이터베이스 객체가 초기화되지 않은 경우 실패 응답 전송
    res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
    res.write('<h2>데이터베이스 연결 실패</h2>');
    res.end();
  }
});

// 사용자 리스트 함수
router.route('/process/listuser').post(function (req, res) {
  console.log('/process/listuser 호출됨.');

  // 데이터베이스 객체가 초기화된 경우, 모델 객체의 findAll 메소드 호출
  if (database) {
    // 1. 모든 사용자 검색
    UserModel.findAll(function (err, results) {
      // 오류가 발생했을 때 클라이언트로 오류 전송
      if (err) {
        console.error('사용자 리스트 조회 중 오류 발생 : ' + err.stack);

        res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
        res.write('<h2>사용자 리스트 조회 중 오류 발생</h2>');
        res.write('<p>' + err.stack + '</p>');
        res.end();

        return;
      }

      if (results) {
        // 결과 객체 있으면 리스트 전송
        console.dir(results);

        res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
        res.write('<h2>사용자 리스트</h2>');
        res.write('<div><ul>');

        for (let i = 0; i < results.length; i++) {
          let curId = results[i]._doc.id;
          let curName = results[i]._doc.name;
          res.write('   <li>#' + i + ' : ' + curId + ', ' + curName + '</li>');
        }

        res.write('</ul></div>');
        res.end();
      } else {
        // 결과 객체가 없으면 실패 응답 전송
        res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
        res.write('<h2>사용자 리스트 조회 실패</h2>');
        res.end();
      }
    });
  } else {
    // 데이터베이스 객체가 초기화되지 않았을 때 실패 응답 전송
    res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
    res.write('<h2>데이터베이스 연결 실패</h2>');
    res.end();
  }
});

// 라우터 객체 등록
app.use('/', router);

//===== 404 오류 페이지 처리 =====//
let errorHandler = expressErrorHandler({
  static: {
    404: './public/404.html',
  },
});

app.use(expressErrorHandler.httpError(404));
app.use(errorHandler);

//===== 서버 시작 =====//
http.createServer(app).listen(app.get('port'), function () {
  console.log('서버가 시작되었습니다. 포트: ' + app.get('port'));

  connectDB();
});


```

**listuser.html**

```

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <form method="post" action="/process/listuser">
        <table>
            <tr>
                <td><label>아래 [전송] 버튼을 누르세요</label></td>
            </tr>
        </table>
        <input type="submit" value="전송" name="">
    </form>
</body>
</html>

```
## 비밀번호 암호화하여 저장하기

app.js

```

// Express 모듈 불러오기
let express = require('express');
let http = require('http');
let path = require('path');

// Express 미들웨어 불러오기
let bodyParser = require('body-parser');
let cookieParser = require('cookie-parser');
let static = require('serve-static');

// 오류 핸들러 사용
let expressErrorHandler = require('express-error-handler');

// Session 미들웨어 불러오기
let expressSession = require('express-session');

// crypto 모듈 불러들이기
let crypto = require('crypto');

// 몽고디비 모듈 사용
let MongoClient = require('mongodb').MongoClient;

// mongoose 모듈 불러오기
let mongoose = require('mongoose');

let app = express();

app.set('port', process.env.PORT || 3000);

// body-parser를 사용해 application/x-www-form-urlenceded 파싱
app.use(bodyParser.urlencoded({ extended: false }));

// body-parsr를 사용해 application/json 파싱
app.use(bodyParser.json());

// public 폴더를 static으로 오픈
app.use('/public', static(path.join(__dirname, 'public')));

// cookie-parser 설정
app.use(cookieParser());

// 세션 설정
app.use(
  expressSession({
    secret: 'my key',
    resave: true,
    saveUninitialized: true,
  })
);

// 데이터베이스 객체를 위한 변수 선언
let database;

// 데이터베이스 스키마 객체를 위한 변수 선언
let UserSchema;

// 데이터베이스 모델 객체를 위한 변수 선언
let UserModel;

// 데이터베이스 연결
function connectDB() {
  //데이터베이스 연결 정보
  let databaseUrl = 'mongodb://localhost:27017/local';

  // 데이터베이스 연결
  console.log('데이터베이스 연결을 시도합니다.');
  mongoose.Promise = global.Promise;
  mongoose.connect(databaseUrl);
  database = mongoose.connection;

  database.on(
    'error',
    console.error.bind(console, 'mongoose connection error.')
  );
  database.on('open', function () {
    console.log('데이터베이스에 연결되었습니다 : ' + databaseUrl);

    // user 스키마 및 모델 객체 생성
    createUserSchema();
  });

  // 연결 끊어졌을 때 5초 후 재연결
  database.on('disconnected', function () {
    console.log('연결이 끊어졌습니다. 5초 후 다시 연결합니다.');
    setInterval(connectDB, 5000);
  });
}

// user 스키마 및 모델 객체 생성
function createUserSchema() {
  // 스키마 정의
  // password를 hashed_password로 변경, default 속성 모두 추가, salt 속성 추가
  // 스키마 정의
  UserSchema = mongoose.Schema({
    id: { type: String, required: true, unique: true, default: ' ' },
    hashed_password: { type: String, required: true, default: ' ' },
    salt: { type: String, required: true },
    name: { type: String, index: 'hashed', default: ' ' },
    age: { type: Number, default: -1 },
    created_at: { type: Date, index: { unique: false }, default: Date.now },
    updated_at: { type: Date, index: { unique: false }, default: Date.now },
  });
  console.log('UserSchema 정의함.');

  // password를 virtual 메소드로 정의  : MongoDB에 저장되지 않는 편리한 속성임. 특정 속성을 지정하고 set, get 메소드를 정의함
  UserSchema.virtual('password')
    .set(function (password) {
      this._password = password;
      this.salt = this.makeSalt();
      this.hashed_password = this.encryptPassword(password);
      console.log('virtual password 호출됨 : ' + this.hashed_password);
    })
    .get(function () {
      return this._password;
    });

  // 스키마에 모델 인스턴스에서 사용할 수 있는 메소드 추가
  // 비밀번호 암호화 메소드
  UserSchema.method('encryptPassword', function (plainText, inSalt) {
    if (inSalt) {
      return crypto.createHmac('sha1', inSalt).update(plainText).digest('hex');
    } else {
      return crypto
        .createHmac('sha1', this.salt)
        .update(plainText)
        .digest('hex');
    }
  });

  // salt 값 만들기 메소드
  UserSchema.method('makeSalt', function () {
    return Math.round(new Date().valueOf() * Math.random()) + '';
  });

  // 인증 메소드 - 입력된 비밀번호와 비교 (true/false 리턴)
  UserSchema.method(
    'authenticate',
    function (plainText, inSalt, hashed_password) {
      if (inSalt) {
        console.log(
          'authenticate 호출됨 : %s -> %s : %s',
          plainText,
          this.encryptPassword(plainText, inSalt),
          hashed_password
        );
        return this.encryptPassword(plainText, inSalt) === hashed_password;
      } else {
        console.log(
          'authenticate 호출됨 : %s -> %s : %s',
          plainText,
          this.encryptPassword(plainText),
          this.hashed_password
        );
        return this.encryptPassword(plainText) === this.hashed_password;
      }
    }
  );

  // 스키마에 static 메소드 추가
  UserSchema.static('findById', function (id, callback) {
    return this.find({ id: id }, callback);
  });

  UserSchema.static('findAll', function (callback) {
    return this.find({}, callback);
  });

  UserSchema.path('id').validate(function (id) {
    return id.length;
  }, 'id 칼럼의 값이 없습니다.');

  UserSchema.path('name').validate(function (name) {
    return name.length;
  }, 'name 칼럼의 값이 없습니다.');

  // UserModel 모델 정의
  UserModel = mongoose.model('users3', UserSchema);
  console.log('UserModel 정의함.');
}

// 사용자를 인증하는 함수 : 아이디로 먼저 찾고 비밀번호를 그다음에 비교
let authUser = function (database, id, password, callback) {
  console.log('authUser 호출됨');

  // 1. 아이디를 사용해 검색
  UserModel.findById(id, function (err, results) {
    if (err) {
      callback(err, null);
      return;
    }

    console.log('아이디 [%s]로 사용자 검색 결괴', id);
    console.dir(results);

    if (results.length > 0) {
      console.log('아이디와 일치하는 사용자 찾음.');

      // 2. 비밀번호 확인 : 모델 인스턴스를 객체를 만들고 authenticate() 메소드 호출
      let user = new UserModel({ id: id });
      let authenticated = user.authenticate(
        password,
        results[0]._doc.salt,
        results[0]._doc.hashed_password
      );

      if (authenticated) {
        console.log('비밀번호 일치함');
        callback(null, results);
      } else {
        console.log('비밀번호 일치하지 않음');
        callback(null, null);
      }
    } else {
      console.log('아이디와 일치하는 사용자를 찾지 못함.');
      callback(null, null);
    }
  });
};

// 사용자를 추가하는 함수
let addUser = function (database, id, password, name, callback) {
  console.log('addUser 호출됨.');

  // UserModel 인스턴스 생성
  let user = new UserModel({ id: id, password: password, name: name });

  // save()로 저장
  user.save(function (err) {
    if (err) {
      callback(err, null);
      return;
    }

    console.log('사용자 데이터 추가함.');
    callback(null, user);
  });
};

// 라우팅 객체 참조
let router = express.Router();

// 로그인 라우팅 함수 - 데이터베이스의 정보와 비교
router.route('/process/login').post(function (req, res) {
  console.log('/process/login 호출됨.');
});

app.post('/process/login', function (req, res) {
  console.log('/process/login 호출됨.');

  let paramId = req.body.id || req.query.id;
  let paramPassword = req.body.password || req.query.password;

  if (database) {
    authUser(database, paramId, paramPassword, function (err, docs) {
      if (err) {
        throw err;
      }

      if (docs) {
        console.dir(docs);
        let username = docs[0].name;
        res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
        res.write('<h1>로그인 성공</h1>');
        res.write('<div><p>사용자 아이디 : ' + paramId + '</p></div>');
        res.write('<div><p>사용자 이름 : ' + username + '</p></div>');
        res.write("<br><br><a href='/public/login.html'>다시 로그인하기</a>");
        res.end();
      } else {
        res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
        res.write('<hi>로그인 실패</h1>');
        res.write('<div><p>아이디와 비밀번호를 다시 확인하십시오.</p></div>');
        res.write("<br><br><a href='/public/login.html'>다시 로그인하기</a>");
        res.end();
      }
    });
  } else {
    res.wrtieHead('200', { 'Content-Type': 'text/html;charset=utf8' });
    res.write('<h2>데이터베이스 연결 실패</h2>');
    res.write('<div><p>데이터베이스에 연결하지 못했습니다.</p></div>');
    res.end();
  }
});

// 사용자 추가 라우팅 함수 - 클라이언트에서 보내온 데이터를 이용해 데이터베이스에 추가
router.route('/process/adduser').post(function (req, res) {
  console.log('/process/adduser 호출됨.');

  let paramId = req.body.id || req.query.id;
  let paramPassword = req.body.password || req.query.password;
  let paramName = req.body.name || req.query.name;

  console.log(
    '요청 파라미터 : ',
    paramId + ', ' + paramPassword + ' ' + paramName
  );

  // 데이터베이스 객체가 초기화된 경우, addUser 함수 호출하여 사용자 추가
  if (database) {
    addUser(
      database,
      paramId,
      paramPassword,
      paramName,
      function (err, result) {
        if (err) {
          throw err;
        }

        // 결과 객체 확인하여 추가된 데이터 있으면 성공 응답 전송
        if (result) {
          console.dir(result);

          res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
          res.write('<h2>사용자 추가 성공</h2>');
          res.end();
        } else {
          // 결과 객체가 없으면 실패 응답 전송
          res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
          res.write('<h2>데이터베이스 연결 실패</h2>');
          res.end();
        }
      }
    );
  } else {
    // 데이터베이스 객체가 초기화되지 않은 경우 실패 응답 전송
    res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
    res.write('<h2>데이터베이스 연결 실패</h2>');
    res.end();
  }
});

// 사용자 리스트 함수
router.route('/process/listuser').post(function (req, res) {
  console.log('/process/listuser 호출됨.');

  // 데이터베이스 객체가 초기화된 경우, 모델 객체의 findAll 메소드 호출
  if (database) {
    // 1. 모든 사용자 검색
    UserModel.findAll(function (err, results) {
      // 오류가 발생했을 때 클라이언트로 오류 전송
      if (err) {
        console.error('사용자 리스트 조회 중 오류 발생 : ' + err.stack);

        res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
        res.write('<h2>사용자 리스트 조회 중 오류 발생</h2>');
        res.write('<p>' + err.stack + '</p>');
        res.end();

        return;
      }

      if (results) {
        // 결과 객체 있으면 리스트 전송
        console.dir(results);

        res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
        res.write('<h2>사용자 리스트</h2>');
        res.write('<div><ul>');

        for (let i = 0; i < results.length; i++) {
          let curId = results[i]._doc.id;
          let curName = results[i]._doc.name;
          res.write('   <li>#' + i + ' : ' + curId + ', ' + curName + '</li>');
        }

        res.write('</ul></div>');
        res.end();
      } else {
        // 결과 객체가 없으면 실패 응답 전송
        res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
        res.write('<h2>사용자 리스트 조회 실패</h2>');
        res.end();
      }
    });
  } else {
    // 데이터베이스 객체가 초기화되지 않았을 때 실패 응답 전송
    res.writeHead('200', { 'Content-Type': 'text/html;charset=utf8' });
    res.write('<h2>데이터베이스 연결 실패</h2>');
    res.end();
  }
});

// 라우터 객체 등록
app.use('/', router);

//===== 404 오류 페이지 처리 =====//
let errorHandler = expressErrorHandler({
  static: {
    404: './public/404.html',
  },
});

app.use(expressErrorHandler.httpError(404));
app.use(errorHandler);

//===== 서버 시작 =====//
http.createServer(app).listen(app.get('port'), function () {
  console.log('서버가 시작되었습니다. 포트: ' + app.get('port'));

  connectDB();
});

```
































