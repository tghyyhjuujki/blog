---
title: Docker 연습(2)-사용환경설정(Vagrant, Linux)
date: 2020-07-14
category: Docker
draft: false
---


### 루트 디렉토리 밑의 디렉토리

dev(device): 장치 파일이 담긴 디렉토리

home : 사용자 홈 디렉터리가 생성되는 디렉터리

opt : 추가 패키지가 설치되는 곳

root : root 계정의 홈 디렉터리 루트(/)디렉터리와 다른 것이므로 혼동하지 말자

usr : 기본 실행 파일과 라이브러리 파일, 헤더 파일 등 많은 파일이 있다. 참고로 usr은 universal system resources의 약자이다. 윈도로 치면 programfile같은 것

etc : 리눅스 설정에 필요한 각종 파일을 갖고 있다

---

### ETC

pwd : present working directory의 약자, 현재 디렉토리를 확인함

~ : cd 앞에 붙이면 홈 디렉토리로 이동



vagrantfile

```python
Vagrant.configure("2") do |config|
  # Jenkins (node1)
  config.vm.define:"node1" do |cfg|
    cfg.vm.box = "centos/7"
    cfg.vm.provider:virtualbox do |vb|
        vb.name="node1"
        vb.customize ["modifyvm", :id, "--cpus", 2]
        vb.customize ["modifyvm", :id, "--memory", 2048]
    end
    cfg.vm.host_name="node1"
    cfg.vm.synced_folder ".", "/vagrant", type: "nfs"
    cfg.vm.network "private_network", ip: "192.168.56.11", bridge: "en0: Wi-Fi (AirPort)"
    cfg.vm.network "forwarded_port", guest: 22, host: 19211, auto_correct: false, id: "ssh"
    cfg.vm.network "forwarded_port", guest: 8080, host: 18080
    cfg.vm.network "forwarded_port", guest: 80, host: 18000 # guest는 리눅스, host는 윈도를 뜻함
    cfg.vm.network "forwarded_port", guest: 3306, host: 13306 # db를 위한 포트도 개방
    cfg.vm.provision "shell", path: "bash_ssh_conf_4_CentOS.sh"
  end

  # Tomcat (node2)
  config.vm.define:"node2" do |cfg|
    cfg.vm.box = "centos/7"
    cfg.vm.provider:virtualbox do |vb|
        vb.name="node2"
        vb.customize ["modifyvm", :id, "--cpus", 2]
        vb.customize ["modifyvm", :id, "--memory", 2048]
    end
    cfg.vm.host_name="node2"
    cfg.vm.synced_folder ".", "/vagrant", type: "nfs"
    cfg.vm.network "private_network", ip: "192.168.56.12", bridge: "en0: Wi-Fi (AirPort)"
    cfg.vm.network "forwarded_port", guest: 22, host: 19212, auto_correct: false, id: "ssh"
    cfg.vm.network "forwarded_port", guest: 8080, host: 28080
    cfg.vm.network "forwarded_port", guest: 80, host: 28000
    cfg.vm.network "forwarded_port", guest: 3306, host: 23306
    cfg.vm.provision "shell", path: "bash_ssh_conf_4_CentOS.sh"
  end

  # Docker (node3)
  config.vm.define:"node3" do |cfg|
    cfg.vm.box = "centos/7"
    cfg.vm.provider:virtualbox do |vb|
        vb.name="node3"
        vb.customize ["modifyvm", :id, "--cpus", 2]
        vb.customize ["modifyvm", :id, "--memory", 2048]
    end
    cfg.vm.host_name="node3"
    cfg.vm.synced_folder ".", "/vagrant", type: "nfs"
    cfg.vm.network "private_network", ip: "192.168.56.13", bridge: "en0: Wi-Fi (AirPort)"
    cfg.vm.network "forwarded_port", guest: 22, host: 19213, auto_correct: false, id: "ssh"
    cfg.vm.network "forwarded_port", guest: 8080, host: 38080
    cfg.vm.network "forwarded_port", guest: 80, host: 38000
    cfg.vm.network "forwarded_port", guest: 3306, host: 33306
    cfg.vm.provision "shell", path: "bash_ssh_conf_4_CentOS.sh"
  end
end
```



위 파일 작성하고 각 노드 설치 및 실행

```shell
$ vagrant up
$ vagrant ssh node1 # node1 접속
$ ip addr # node2,3으로 가기 위해 내 ip 확인

# 여기서 eth1에 해당하는192.168.56.11~13번으로 접속 가능
$ ping 192.168.56.11~13 # 각 핑 확인 가능
```



node2에서

```shell
$ ping node1 # ping: node1: Name or service not known 라는 에러 출력
```

에러가 뜨므로 도메인을 등록해줘야한다(마치 네이버에 naver.com로 접근 가능하듯)

```shell
$ vi /etc/hosts # 편집 들어가서 아래 내용 추가하려면 권한 없다고 뜸
$ su - # sudo로 실행해도 되지만 root사용자로 변경해준다
Password: vagrant # 비밀번호 입력
[root@node1 ~]	# 정상적으로 바뀜

$ $ vi /etc/hosts # 이제 가능

# 아래 내용 추가
192.168.56.11   node1
192.168.56.12   node2
192.168.56.13   node3

$ su vagrant # su 다음에 로그인 아이디 입력하면 계정변경
$ exit # 로그아웃

$ ping node2 # 이제 정상적으로 핑 출력된다(도메인 등록된 것)
```



이제 몇가지 작업 해보자

```shell
$ mkdir profile.txt # 만들고 아무거나 입력
$ cat profile.txt # 내용 보기
$ touch template_profile.txt # 크기가 0짜리인 놈 생성

$ rm profile.txt # 파일 삭제
$ rm -rf test # 디렉토리 삭제
$ mkdir one two three # 한번에 여러개 디렉토리 생성가능
$ mkdir -p one/tmp/test # one 아래에 아무것도 없을 때 tmp와 test를 생성해줌

# 파일 복사
$ cp [원본파일] [복사할파일]
$ cp /etc/hosts ./ # hosts 파일을 현재 경로로 복사
$ cp /etc/hosts new_hosts # hosts 파일을 현재경로의 new_hosts라는 이름으로 복사
$ rm -rf * # 확장자 없어서 별표만 입력, f는 force이므로 조심해서 사용, r은 재귀
$ cp /etc/hosts /etc/hostname ./test # hosts와 hostname을 test 디렉토리로 복사

# 디렉토리 복사
$ cp -r ./test ./new_test # r은 recursive로 재귀를 뜻함. 안에 여러가지 파일이 있을 수 있으므로 반복해서 카피하는 -r옵션을 줘야한다.

# history
$ ![히스토리 넘버] # 해당번호 명령어 실행

# 파일 이동, 이름 변경
$ mv hostname ./test # cp와 달리 파일이 test로 이동하고 현재 디렉토리에서는 사라짐
$ mv hostname new_hostname # 만약 hostname 있는 디렉토리에서 새 이름 쓰면 이름변경
```



정리

```shell
# 파일생성
vi, touch, cp
# 파일 복사
cp
# 파일 이동, 파일명 변경
cp, mv
# 파일, 디렉토리 삭제
rm -rf
#디렉토리
mkdir, cd, rmdir
# 현재디렉토리 확인
pwd
#사용자 전환
su -
# 파일목록 상세확인
ls -l
```

---

### 파일 링크

하드링크 : 기존 파일에 새로운 파일명을 추가로 생성

```shell
$ ln hosts hosts.ln # hosts파일 존재하는 디렉토리에서 hosts.ln이라는 링크파일 생성
$ ls #하면 두 파일의 크기가 같다(복사된 것과 같다), 어떤 파일이 원본인지 알 수 없다
```



심볼릭 링크 : 원본파일을 가리키는 새로운 파일 생성(바로가기 파일)

```sh
$ ln -s hosts s_hosts.ln # 심볼릭 링크
$ ls # 하면 심볼릭 링크 파일의 크기가 매우 작은 것을 확인 할 수 있다(바로가기라서) 링크파일을 가리켜서 어떤파일이 원본인지 알 수 있다
```

---

### 파일 검색, 내용검색

```sh
$ cd /usr/bin
$ ls -l yum* # 현재 디렉토리에서 yum 들어간 파일 검색
$ ls -l | grep yum # |는 and의 의미, grep는 잡는다는 의미. yum들어간 파일 검색
$ cat yum | grep import # yum 파일 안에 import라는 키워드 검색
```

 

### 파일 찾기

```shell
$ sudo find /usr/ -name ls # usr 아래에 어딘가의 ls 라는 파일의 위치 찾을 수 있음
$ ls -l /usr/bin/ls # 위에서 나온 결과를 검색
$ find ./ -name hosts | xargs grep "localhost" # 현재 경로에서 hosts라는 파일검색, 그리고 그 파일 안에서 localhost라는 내용 검색
```



### 명령의 위치 찾기

```shell
$ whereis mv # mv들어가는 명령어 위치 뿐만 아니라 소스, 매뉴얼의 위치도 알 수 있다
$ which mv # mv들어가는 정확한 명령어 찾음
```



---

## 파일 속성

```shell
-rw-r--r--. 1 root root 442 Jul 14 04:58 /etc/hosts

rw- : 2^2 + 2^1 + 0 = 6
r-- : 2^2 + 0 + 0 = 4
r-- : 2^2 + 0 + 0 = 4

즉 -644

처음은 파일 종류 (ex -, d, l)
그 뒤 3개씩 끊어서 소유자, 그룹, 기타사용자의 read, write, excute권한.
1 : 하드링크의 개수
root : 파일소유자의 로그인 ID
root : 파일이 속한 그룹
442: 파일 크기(바이트)
jul 14 04:58 : 마지막 수정된 날짜, 한국시간과 다르므로 +9시간 해주면 된다
/etc/hosts : 경로

# file 명령어
$ file /usr/bin/python # 파일정보를 출력
```



### group

```shell
$ groups # 현재 그룹 보여줌
$ groups root # root계정의 그룹 보여줌
```



### 접근권한 변경

u : 파일소유자 g : 그룹 o : 기타사용자 +: 권한 부여 -: 권한삭제

뭐 이런 명령어가 있지만, 숫자로 권한부여하는 것이 제일 쉽다



```shell
# chmod = change mode
$ chmod 755 hosts # 권한 변경, 만약 하드링크 걸려있는 원본 파일이면 링크된 파일의 권한도 모두 변경된다.
```

---



## 셸의 기능과 종류



### 리다이렉트( < , > , >> 등)

```shell
$ ls -l > my_list # 파일목록을 my_list에 저장
$ $ ls -l /usr/bin/ > bin_list.txt # bin 아래에 있는 파일을 txt에 저장
$ cat bin_list.txt | grep yum > yum_list.txt # bin_list에서 yum만 추출해서 yum_list에 저장

> : 기존 파일의 내용을 삭제하고 새로 저장
>> : 기존 파일 뒤에 내용 추가
```



### 셸 변수와 환경변수 설정

```sh
$ SOME=test # 변수 선언및 저장, 공백 없어야함
$ echo $ SOME # 변수 내용 출력
$ set | grep SOME # SOME 자세히 출력

# export, 다른곳에서 환경변수 사용가능하게 내보내는 역할.
$ export SOME=test # 모든곳에서 사용가능
```



### 에일리어스 (alias), 긴 문장을 짧게 만듦

```sh
$ ls -l --color=auto # 다음 명령어를
$ alias li='ls -l --color=auto' # li에 저장하고(띄어쓰기 하면 안 됨)
$ li # 이것만 입력해도 똑같이 작동한다
```



### 사용자 환경설정파일

```sh
~/.bash_profile
~/.bashrc : 이 밑에 alias명령어 추가가능
~/.bash_logout

$ vi .bashrc # 파일에 들어가서

# 맨 밑에 에일리어스 추가, 로그아웃 후 사용가능
alias rm='rm -i'
alias ls='ls -F'

$ vi .bash_profile # 파일에 들어가서

# 맨 밑에 export 추가, 로그아웃 후 사용가능
export SOME=test
```
