---

title: Spring boot) Response 데이터 필터링하기
date: 2020-12-27 16:21:21
category: Spring
draft: false
---


## Response 데이터 필터링하기

클라이언트에게 중요한 데이터가 보여지면 보안상 위험하기 때문에 이번 포스트에서는 중요한 데이터를 필터링하는 방법에 대해 알아보자. 먼저 기존 user 클래스에 다음 필드를 추가한다.

```java
@Data
@AllArgsConstructor
public class User {
    private Integer id;

    @Size(min=4, message = "Name 4글자 이상 입력 해야함")
    private String name;
    @Past
    private Date joinData;
    
    //추가한 내용
    private String password;
    private String ssn;
}
```

<br/>

이에 맞게 데이터도 수정해주고 

```java
@Service
public class UserDaoService {
	static {
        users.add(new User(1, "Jeonghoon1", new Date(), "pass1", "111111"));
        users.add(new User(2, "Jeonghoon2", new Date(), "pass2", "222222"));
        users.add(new User(3, "Jeonghoon3", new Date(), "pass3", "333333"));
    }
}
```

<br/>

조회해보면 다음과 같이 출력된다. 패스워드등의 중요한 정보가 그대로 노출된다.

![image-20201227192617928](SpringBoot11-data_filtering.assets/image-20201227192617928.png)

<br/>

이제 클라이언트가 보지 못하게 필터링하는 방법을 알아보자.

<br/>

### 방법 1. @JsonIgnore, @JsonIgnoreProperties

user 클래스의 필드에 @JsonIgnore를 추가해준다.

```java
@Data
@AllArgsConstructor
public class User {
    private Integer id;

    @Size(min=4, message = "Name 4글자 이상 입력 해야함")
    private String name;
    @Past
    private Date joinData;
    
    //추가한 부분
    @JsonIgnore
    private String password;
    @JsonIgnore
    private String ssn;
}
```

<br/>

조회해보면 나오지 않는다.

![image-20201227193134009](SpringBoot11-data_filtering.assets/image-20201227193134009.png)

<br/>

마찬가지로 @JsonIgnoreProperties 사용하여 나오지 않게 할 수 있다.

```java
@Data
@AllArgsConstructor
//추가한 부분
@JsonIgnoreProperties(value={"password", "ssn"})
public class User {
    private Integer id;

    @Size(min=4, message = "Name 4글자 이상 입력 해야함")
    private String name;
    @Past
    private Date joinData;
    
    private String password;
    private String ssn;
}
```
![image-20201227193505323](SpringBoot11-data_filtering.assets/image-20201227193505323.png)

<br/>

### 방법2. @JsonFilter

