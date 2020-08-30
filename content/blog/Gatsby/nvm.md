---
title: node 버전 변경, 관리) Failed to install node version
date: 2020-08-30 19:08:85
category: Gatsby
draft: false
---





Netlify 배포하는 과정에서 실패를 하였다







Netlify에서 어제까지만 해도 잘 되던 배포가 fail이 떳다.

![image-20200830194923572](nvm.assets/image-20200830194923572.png)

<br/>

에러 내용은 다음과 같았다.  Version '12.18.0' not found. Failed to install node version.

![image-20200830195057605](nvm.assets/image-20200830195057605.png)

<br/>

왜 찾을수 없지? 내 node 버전을 확인해봤다.

![image-20200830195133567](nvm.assets/image-20200830195133567.png)

<br/>

알 수 없는 이유로 변경되어있었다...(뭐지...). 일단 버전을 업그레이드 해주자.

위에 뜬 에러 내용대로 nvm ls-remote를 실행했지만 안 먹혔다. nvm을 설치해줘야 할 것 같다.