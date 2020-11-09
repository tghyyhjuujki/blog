---
title: Docker 연습(3)-기본 명령어
date: 2020-07-20
category: Docker
draft: false
---

# Docker

docker : 컨테이너 기반의 가상화 플랫폼. 운영체제를 가상화시키지 않고, 미들웨어만 가상화시킴



```shell
$ docker pull ubuntu :latest # 우분투 최신버전 이미지 가져옴
$ docker images # docker 이미지 보여줌
$ docker container ls # 컨테이너 보여줌
$ docker run ubuntu:latest # 컨테이너 만듦
$ docker ps -a # 죽거나 살아있는 프로세스 모두 보여줌
$ docker rm [컨테이너id] # 컨테이너 삭제, 이미지는 남아있음
$ docker rmi [이미지id] # 이미지 삭제

$ docker run -it ubuntu:latest /bin/bash # 만약 이미지가 없으면 다운받음 -it는 인터랙티브한 터미널을 열 수 있는 환경에 /bin/bash를 사용하겠다는 의미. 이제 리눅스 사용 가능해진다
root@74e6c018ea95:/$ # 컨테이너 id가 hostname이 된다
root@74e6c018ea95:/$ hostname -i # 172.17.0.2가 나오는데, 도커가 설정한 가상ip
```



### mysql 사용하기

```shell
$ docker pull mysql:5.7
$ docker run -p 3306:3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=true --name mysql mysql:5.7# -p는 포트포워딩,3306:3306은 3306으로 윈도우에서 호출하면 도커에서 3306으로 응답하겠다,-e는 환경설정 MYSQL_ALLOW_EMPTY_PASSWORD=true로 비밀번호 없게 하고 --name을 mysql로 입력하여 컨테이너 이름 지정, 마지막은 버전정보 입력한다

$ docker exec -it mysql bash # exec는 실행하겠다, -it는 터미널 들어가겠다, mysql이름을 bash셀을 이용해서

root@4f82e9206780:/$ mysql -uroot -p # mysql 접속

mysql> show databases;	# db목록 출력
mysql> use mysql;		# 사용
mysql> show tables; 	# 테이블 보여줘
```



### tensorflow 사용하기

```shell
$ docker run -d --rm -p 8888:8888 teamlab/pydata-tensorflow:0.1 # -d는 디태치 모드 즉 백그라운드에서 실행되게함(stop했다가 다시 킬때 바로 켜짐) 즉, powershell 꺼도 실행된다.--rm은 stop됐을때 컨테이너를 아예 삭제시키는 명령어
```

localhost:8888로 웹에서 접속하면 주피터가 정상적으로 실행되는 것을 볼 수 있다.

---

## node1,2,3번에 각각 도커를 설치하고  여러방법으로 접속

node1,2

```shell
$ docker run -d -p 3306:3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=true --name mysql mysql:5.7 # mysql 설치, node2에는 mysql2로 태깅하고 설치했다
$ docker exec -it mysql /bin/bash # 실행
root@5fc4147664c9:/$ mysql # mysql접속
```

2번 docker에서 1번 mysql 접속 

```shell
root@19c3573e2814:/$ mysql -h192.168.56.11 -uroot -p # 1번 mysql에 접속
```



node3번 centos에서 1번 mysql 접속

```shell
$ yum install -y mysql # mysql 직접 설치하고
$ mysql -h192.168.56.11 -uroot -p # node1 mysql에 접속
```

---



### 기타

```shell
$ docker system info # 도커 정보 보여줌
$ df -h # 현재 디스크의 정보를 나타냄
$ free -h # 사용가능한 메모리 보여줌

$ docker system df # 도커의 디스크정보를 보여줌
볼륨마운트 : 외부 스토리지와 연결해놓은것
```



node1번에서, page86 nginx 설치 및 이미지 배포

```shell
$ docker pull nginx #이미지 다운
$ docker run -d -p 80:80 --name webserver nginx # nginx 실행
# 192.168.56.11:80 로 접속해야한다

# 이제 본인만의 이미지 만들자
$ docker login # 본인 도커허브 아이디로 로그인
$ docker tag nginx:latest tghyyhju/nginx:v1.0 # v1.0이라는 새로운 태그의 이미지 생성 및 레파지토리(tghyyhju/nginx)에 파싱
$ docker push tghyyhju/nginx:v1.0 # 본인 레파지토리에 업로드
```



이제 2번에서

```shell
$ docker pull tghyyhju/nginx:v1.0 # 내가 1번에서 올려놓은 이미지 가져올 수 있다
$ docker run --name webserver2 --rm -d -p 80:80 tghyyhju/nginx:v1.0 # 192.168.56.12:80으로 접속가능
```



참고로 run명령어는 create + start 가 합쳐진 명령어

docker logs [컨테이너 ID] : 로그목록확인, 에러 발견할 수 있다

docker system prune : 모든것 삭제