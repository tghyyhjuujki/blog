---
title: RDS와 외부 클라이언트 연결하는 node 서버 만들기
date: 2020-09-01 23:09:32
category: Node
draft: false
---



미니프로젝트

1. 다이어그램 작성
2.  파이썬 파일 작성
3. RDS

이번 포스트는 미니 프로젝트에서 EC2 안에서 RDS와 외부 클라이언트를 연결하는 node 서버를 만드는 것이다. 만들고 보니 간단한 프로젝트를 하기에는 Spring boot보다 node 서버가 구축하기 훨씬 편했다.

아키텍처를 보자면, 다음과 같다.

[그림1]

DB를 보면 Master와 Slave로 나뉘어있는데, select는 slave에서, update, insert, delete는 master에서 처리하도록 구성한다. 클라이언트로부터의 요청을 분산시켜 DB의 부하를 줄이기 위해서이다. node file 작성할 때 이 부분을 고려하자.

Python에서는 requests 모듈을 사용하여, node 쪽으로 Json 형식의 데이터를 보내게 만들어놨다. [미니프로젝트를 위한 Python]()에서 확인할 수 있다.

<br/>

먼저 node 모듈과 환경설정을 보자면, http method 사용을 위해 거의 node 서버의 표준이 된 express를 임포트하였다. 또한 RDS의 mysql에 연결하기 위해 node의 mysql모듈도 임포트 하였다. 자세한건 주석으로.

```js
const express    = require('express'); //express import
const mysql      = require('mysql'); // mysql import
const dbconfig   = require('./config/database.js'); //master 연결 정보
const dbconfig_slave = require('./config/database-slave.js'); // slave 연결 정보
const connection = mysql.createConnection(dbconfig); // master 연결
const connection_slave = mysql.createConnection(dbconfig_slave); // slave 연결
const bodyParser = require('body-parser') // json 형식으로 받기 위한 body-parser 모듈
const app = express();

app.set('port', process.env.PORT || 8000); // app을 8000포트로
app.use(bodyParser.urlencoded({ extended: false }));

var res_state='complete'; // 요청이 성공적으로 실행되면 클라이언트에 응답할 문자열
```

<br/>

그리고 slave는 master RDS에서 읽기전용으로 만든, 복사된 DB의 정보를 넣어준다.

```js
master
slave
```

그리고 Python 파일의 정보에 맞게

<br/>

### Http Methods

select는 node에서 get 방식,

insert, update는 node에서 post방식

delete는 node에서 delete방식으로 요청과 응답을 하도록 구현하였다.

클라이언트(Python)에서 get방식으로 모든 DML을 날려도 되지만, 그렇게 구현하면 url에 클라이언트가 요청한 모든 정보가 노출되는 엄청난 보안사고가 발생한다(...). 따라서 각 DML에 맞는 Http Method를 사용하는 것이 바람직하다.

다음과 같이 get, post, delete 메소드를 작성해줬다. req는 request로, 클라이언트에서 node쪽으로 보내는 데이터이다. res는 response로, 클라이언트의 요청에 대한 응답이다. 필요에 따라 적절히 사용하면 된다. 참고로 .body로 데이터를 받기 위해서는 `body-parser`를 설치해야한다. 이걸 설치하지 않아 조금 헤맸었다. 아래는 http 메서드의 코드와 주석으로 DML 하나씩만 넣어놨다.

```js
// 전체 고객 데이터를 불러오는 쿼리
app.get('/Customer/select', (req, res) => {
    connection_slave.query('SELECT * from Customer', (error, rows) => {
    // error 처리. 밑의 쿼리에서는 생략
    if (error) throw error;
    console.log('User info is: ', rows);
    res.send(rows);
    });
});

// 고객의 이름과 전화번호, 생년월일을 RDS로 insert하는 쿼리
app.post('/Customer/insert', (req, res) => {
    // 확인해 보면 클라이언트에 날린 데이터가 json 형식으로 출력된다.
    console.log(req.body); 
    // 각 변수 초기화.
    var cust_name = req.body.cust_name;
    var cust_phone = req.body.cust_phone;
    var cust_birth = req.body.cust_birth;

    // sql에 쿼리를 작성하고 변수 할당할 부분은 ?로 놓는다.
    var sql = 'insert into Customer(cust_name,cust_phone,cust_birth) values(?,?,?)';
    connection.query(sql, [cust_name,cust_phone,cust_birth]);
	res.send(res_state);
});

// 고객의 정보 수정
app.post('/Customer/update', (req, res) => {
    console.log(req.body);
    var cust_name = req.body.cust_name;
    var cust_phone = req.body.cust_phone;
    var cust_birth = req.body.cust_birth;
    var cust_id = req.body.cust_id;

    var sql = 'update Customer set cust_name=?, cust_phone=?, cust_birth=? where cust_id=?';
    connection.query(sql, [cust_name, cust_phone, cust_birth,cust_id]);
    res.send(res_state);
});

// 고객의 정보를 삭제하는 delete 쿼리
app.delete('/Customer/delete', (req, res) => {
    console.log(req.body);
    var cust_id = req.body.cust_id;

    var sql = 'delete from Customer where cust_id=?';
    connection.query(sql, [cust_id]);
    res.send(res_state);
});
```

<br/>

### Test

외부에서 Python 파일을 실행하고 테스트해볼 차례. 간단하게 고객 데이터만 갖고 놀아보겠다.

