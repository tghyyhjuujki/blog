---
title: Docker) 볼륨마운트/DockerFile
date: 2020-07-21
category: Docker
draft: false
---

---

### 볼륨마운트 테스트

볼륨마운트는 호스트(node1)의 디렉토리와 컨테이너(nginx)의 디렉토리를 공유하는 기법으로, 컨테이너가 날아간다해도 node1의 데이터는 무사하기 떄문에 다시 컨테이너만 구축하면 기존과 같이 사용할 수 있게 된다

```shell
# nginx에서 디렉토리 공유할거임
$ docker run -d -p 80:80 -v /root/html:/usr/share/nginx/html nginx
# -v는 볼륨마운트, : 앞은 node1에서 공유할(마운트할) 디렉토리, : 뒤는 docker container인 nginx안의 디렉토리 
```



### 컨테이너 가동확인

```shell
$ docker stats [컨테이너ID]
```



### 이미지 생성 - build

### FROM 명령어

```shell
$ vi Dockerfile # 도커파일 만들고 FROM ubuntu:16.04 쓰고 저장

$ docker build -t fromtest:1.0 ./Dockerfile # 또는
$ docker build -t fromtest:1.0 . # 로 빌드(이미지 생성)

$ docker images # 우분투와 내가 만든 fromtest의 이미지 ID가 같은 것을 확인할 수 있다. 베이스 이미지가 같기 때문이다
```



### RUN  명령어

```shell
$ vi Dockerfile # 이번에는 아래 내용 추가

FROM ubuntu:16.04
RUN mkdir /mydata
RUN echo "Hi, there!"

$ docker build -t fromtest:1.0 . # 이미지 생성
$ docker run -it fromtest:1.0 # 콘솔 들어감
$ root@f4fa9e3d22a4:/$ dir # mydata라는 디렉토리 생성된거 확인 가능
```



### ADD 명령어

문제 : cestos 이미지로 새로운 도커 이미지 생성하고(ID달라야함) mydata라는 디렉토리를 dcoker container에 생성. mydata라는 디렉토리에 test.txt라는 파일을 복사

```shell
$ vi Dockerfile3 # 하고 아래 내용 추가

FROM centos
RUN mkdir /mydata
COPY test.txt /mydata # ADD 사용해도 되지만 가끔 버그가 발생하므로 복사할 때는 COPY를 사용해준다

$ docker build -t addtest:1.0 -f Dockerfile2 . # 이거나 아래명령어 사용
$ docker build -t addtest:1.0 -f ./Dockerfile2 . # -t는 태그, -f는 파일 현재경로에 dockerfile2를 사용하겠다
$ docker run -it addtest:1.0 /bin/bash
$ root@a2e7ab3da2c6:/$ cat /mydata/test.txt # mydata 안에 생성된거 확인 가능

# 번외
$ docker start -i [컨테이너ID] # 정지되어있는 컨테이너 콘솔에 접속
```



문제 : nginx 이미지를 base로 하는 mywebserver 이미지 생성, index.html 파일을 생성한 다음, nginx의 기본 index.html 파일을 변경

```shell
$ vi Dockerfile4 # 하고 아래 내용 입력

FROM nginx:latest

$ docker build -t webservser:1.0 -f ./Dockerfile4 .

# /root/work 디렉토리 안에 index.html작성하고 Run
$ docker run -it -d -p 80:80 -v /root/work:/usr/share/nginx/html webservser:1.0

------------------------------------------------

# 만약 dockerfile 안에 마운트 내용 추가한다면,
$ vi Dockerfile4 # 하고 아래 내용 입력

FROM nginx:latest
COPY ./index.html /usr/share/nginx/html/index.html

$ docker build -t webservser:1.0 -f ./Dockerfile4 .

# /root/work 디렉토리 안에 index.html작성하고 Run
$ docker run -it -d -p 80:80 webservser:1.0

$ curl -X GET http://localhost # 정상적으로 내용 출력되는지 확인
```



### ENTRYPOINT : 파일이 만들어지고 처음 실행되는 구문

```shell
$ vi run.sh # 파일생성 및  echo 'hi' 라는 내용추가
$ ./run.sh # 실행권한 없어서 에러 발생 -rw-r--r--

$ chmod 744 run.sh # 숫자로 권한변경
$ chmod +x run.sh # +로 권한 변, 전부 다 X 권한 들어감

$ vi Dockerfile5 # 아래 내용 추가

FROM centos
COPY ./run.sh .
RUN chmod +x /run.sh
ENTRYPOINT /run.sh

$ docker build -t test5:1.0 -f Dockerfile5 . # 빌드
$ docker run test5:1.0 # 실행
```



### CMD

```shell
# 위 Dockerfile5에 아래 내용 추가
CMD ["ping", "127.0.0.1"]

# 다시 빌드하고 run하면 핑이 계속 나오는 것을 확인할 수 있다(실은 안나옴 문제가 뭔지 찾아보자)
```



### 이제 Doker가 여러 어플리케이션도 돌릴수 있음을 보자 >> NodeJS

```shell
$ yum install epel-release # 레파지토리 등록
$ yum insatll nodejs # node, npm 설치
$ npm init
$ vi pakage.json

# 다음과 같이 일부분만 내용 변경
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node index.js"
  },
  "dependencies": {
    "express": "*"
  },
  
$ vi index.js

# 아래와 같이 입력  
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('how are you?');
});

app.listen(8080, () => {
  console.log('listening on port 8080');
});

$ npm install # pakage.json 내용 기반으로 설치
$ npm start # 192.168.56.11:8080로 접속 가능
```



### 위 작업을 Dokcerfile로 작성해보자

dockerfil 작성

```dockerfile
FROM node:alpine

WORKDIR /home/node # 작업디렉토리 먼저 생성

COPY ./package.json ./package.json # 앞의 pakage는 리눅스의 내 파일, 뒤는 복사될 파일의 경로와 이름
COPY ./index.js ./index.js
RUN npm install

CMD ["npm", "start"]
```

이미지 빌드

```shell
$ docker build -t myweb:1.0 -f . # dockerfile 경로에서 이미지 빌드
$ docker run -p 8080:8080 myweb:1.0 # 실행
```

돌아가는 것은 확인했다. 이제 이미지 이름을 허브에 올릴 수 있게 태그 새로 만들어야한다

```shell
$ docker tag myweb:1.0 tghyyhju/myweb:1.0 # 이미지, 태그 변경
$ docker push tghyyhju/myweb:1.0 # 허브에 push
```

node1에서 push한 내용을 node3에서 pull

```shell
$ docker pull tghyyhju/myweb:1.0
$ docker run -d -p 8080:8080 tghyyhju/myweb:1.0 # 정상적으로 실행된다.
# 이렇게 배포가 끝났다 이걸 travis를 사용해 자동으로 배포하면 된다
```

### 하지만 배포할 때마다 dockerfile 실행하고 복사하고 푸시하는 것은 비효율적

때문에 가상의 공유폴더에 index.js를 만들어 놓을 것이다. 개발용 리눅스와 배포된 곳 각각에서 볼륨마운트로 만들면 된다.

또 다른 방법은 데이터를 볼륨마운트 된 같은곳에 저장해놓는것이다

설명이 좀 부족한데 다음 실습을 따라해보자

### MySQL로 볼륨마운트 실습

```shell
$ docker run -v /root/work/datadir:/var/lib/mysql -d -p 3306:3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=true --name mysql mysql:5.7 # mysql을 볼륨마운트를 걸면서 생성. 이제 mysql이 삭제되어도 데이터가 안날아감

$ docker exec -it mysql mysql # 실행

mysql> create database mydb;
mysql> use mydb;
mysql> create table member(id int, name varchar(50));
mysql> insert into member(id, name) values(1, 'admin');

root@5a1973652c9c:/$ cd var/lib/mysql # 여기에 mydb있는데,
$ ls -l # 여기에도 있다

$ docker stop 5a && docker rm 5a # 컨테이너 다시 삭제하고
$ docker run -v /root/work/datadir:/var/lib/mysql -d -p 3306:3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=true --name mysql mysql:5.7 # 올린다

$ docker exec -it mysql mysql # 실행하고
mysql> show databases; # 확인하면 mydb가 살아있는 것을 확인할 수 있다
```



### 볼륨마운트 예제

```shell
$ vi Dockerfile4 # 하고 아래 내용 입력

FROM nginx:latest
COPY ./index.html /usr/share/nginx/html/index.html

$ docker build -t webservser:1.0 -f ./Dockerfile4 .

# /root/work 디렉토리 안에 index.html작성하고 Run
$ docker run -it -d -p 80:80 -v /root/work:/usr/share/nginx/html webtest:1.0

# 이제 리눅스의 index.html 수정하면 웹 화면도 수정된다!
$ curl -X GET http://localhost # 정상적으로 내용 출력되는지 확인
```

볼륨마운트가 -v 뒤에 디렉토리가 하나만 올 수도 있다. 가상의 디렉토리를 공유하는 것이다.

---



### 포트 에러시

만약 포트충돌 일어나면 다음과 같이 확인해준다

```shell
$ yum install net-tools -y # netstat을 사용하기 위해 설치
$ netstat -ntlp # 사용중인 포트 확인
$ kill -9 [PID] # 포트 강제종료
```
