---
title: Windows에서 nvm으로 node 버전 변경, 관리(feat. Netlify)
date: 2020-08-30 19:08:85
category: Node
draft: false
---





Netlify 배포하는 과정에서 실패를 했다

<br/>

사건의 전말은 이렇다.

1. Netlify에서 어제까지만 해도 잘 되던 배포가 fail이 떳다.
2. Version '12.18.0' not found. Failed to install node version. 라는 내용의 에러가 떳다.
3. 내 node 버전이 12.16.0.이라 떠서 버전을 변경해줘야하는줄 알았다.
4. 버전을 변경해줘도 fail이 떴고, Netlify 커뮤니티를 가보니 나와 같은 증상을 호소하는 환자들이 많았다.
5. 네트워크 제공업체의 문제라는 공지가 올라왔다. 아마 CloudFlare 문제인듯싶다. https://news.ycombinator.com/item?id=24322513 여기서 실시간으로 고통 받는 전세계 사람들을 확인할 수 있었다..ㅋㅋㅋ
6. 아무튼 포스트는 만들었으니 nvm 버전관리로 올린다.

![image-20200830194923572](nvm.assets/image-20200830194923572.png)

<br/>

에러 내용은 다음과 같았다.  Version '12.18.0' not found. Failed to install node version.

![image-20200830195057605](nvm.assets/image-20200830195057605.png)

<br/>

왜 찾을수 없지? 내 node 버전을 확인해봤다.

![image-20200830195133567](nvm.assets/image-20200830195133567.png)

<br/>

위에 뜬 에러 내용대로 nvm ls-remote를 실행했지만 안 먹혔다. nvm을 설치해줘야 할 것 같다.

여기까지 난 내 node 버전을 변경해야하는줄 알았다. 이게 원인은 아니지만, 이왕 작성한 거 nvm설치 및 관리 포스팅을 해본다.

---

## NVM 설치 및 실행

nvm은 node version manager로, 말 그대로 버전관리자이다. 

Windows 환경일 경우 [여기](https://github.com/coreybutler/nvm-windows/releases)서 다운로드 가능하다. zip 파일 안에 .exe만 실행하면 따로 환경설정을 해줘야하는 것 같다. setup까지 같이 받아 실행하면 자동으로 설정된다.

![image-20200830201347734](nvm.assets/image-20200830201347734.png)

<br/>

다시 확인해보면 설치된 것을 확인할 수 있다.

```sh
$ nvm ls-remote
```



![image-20200830201454227](nvm.assets/image-20200830201454227.png)

<br/>

다음과 같이 사용하는 것 같다.

![image-20200830201654032](nvm.assets/image-20200830201654032.png)

<br/>

굿, 설치가 잘 되었다.

```sh
$ nvm install [버전]
```



![image-20200830201855439](nvm.assets/image-20200830201855439.png)

<br/>

이제 명령어 하나로 12.16.0 버전을 12.18.0버전으로 변경할 수 있다(반대도 가능).

```sh
$ nvm use [버전]
```

![image-20200830211233778](nvm.assets/image-20200830211233778.png)

당연한 얘기지만 내 문제가 아니었기에 배포는 fail이 떴다. 아마 시간이 조금 지나면 개발팀이 빠르게 복구하겠지. 겸사겸사 `nvm` 설치와 실행에 대해 알아봤다.

<br/>

PS. 얼마 안지나서 Netlify는 복구되었다.