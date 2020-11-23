---
title: Spring Boot에서 REST API 사용 환경 만들기
date: 2020-11-04
category: Spring
draft: false
---

지난 글 [Spring Boot란?(feat.IntelliJ IDAE) + 프로젝트 생성](https://jeonghoon.netlify.app/Spring/SpringBoot2/)에 이어서 진행한다. 

<br/>

### Annotation?

일단 Annotation에 대해 설명하자면, 자바 소스코드에 추가하여 사용할 수 있는 메타데이터(자신의 정보를 담고 있는데이터)의 일종이다. 자바에서 오버라이딩 할 때 @Overriding 으로 사용한 기억이 있을 것이다. 얘 역시 Annotation.

<br/>

### RestController Annotation

기존 `@Controller`은 MVC의 View 기술을 사용했지만,  `@RestController`는 객체를 반활할 때, JSON/SML 타입의 HTTP 응답을 직접 리턴하게 된다.

이제 테스트를 해보자.

먼저 HelloWorldcontroller 라는 클래스를 하나 만들어준다.

```java
package com.example.restfulwebservice;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloWorldController {
    // Get
    // hello-world (endpoint)
    // @RequestMapping(method=requestMethod.GET, path="/hello-world")
    @GetMapping(path = "/hello-world")
    public String helloWorld(){
        return "Hello World";
    }
}
```

@GetMapping은 