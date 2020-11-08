---
title: DB) SQL 연습(2)
date: 2020-08-22
category: Database
draft: false
---



고객별 평균 주문금액을 백원단위로 반올림한 값 구하기

```sql
select custid '고객번호', round(sum(saleprice)/count(*), -2) '평균금액'
from Orders
group by custid;
```

고객 성으로 구분하기

```sql
-- 성과 이름 분리
select substr(name, 1, 1) '성', substr(name, 2, 2) '이름' from Customer group by name;

-- 성끼리 그룹
select substr(name, 1, 1) '성', count(*) '인원' from Customer 
group by substr(name, 1, 1); -- 1번째 자리에서 1개 가져옴
```

​	날짜함수 adddate

```sql
select orderid, orderdate, adddate(orderdate, interval 10 day) from Orders; -- 10일 후
select orderid, orderdate, adddate(orderdate, interval -1 month) from Orders; -- 1달 전
```

date_format, str_to_date

```sql
select orderid, str_to_date(orderdate, '%y %M %d') from Orders -- 문자열 to 날자형
where orderdate=date_format('20140707', '%Y%m%d'); -- 날자형 to 문자열
```

---

## NULL

IFNULL

```sql
select name ,ifnull(phone, '연락처ㄴㄴ') from Customer; -- null이면 연락처ㄴㄴ로 변경
```

---

### 서브쿼리

스칼라 서브쿼리 : select문에 서브쿼리 작성

```sql
-- join으로 구현
select Orders.custid, name, sum(saleprice)
from Orders, Customer
where Orders.custid=Customer.custid
group by Orders.custid;

-- 서브쿼리로 구현
select custid, 
	(select name from Customer c  where c.custid=o.custid),  -- 서브쿼리는 반드시 하나의 속성만 반환해야한다 안그러면 오류 뜸
	sum(saleprice)
from Orders o
group by  custid;
```

인라인 뷰 : from 안에 서브쿼리 작성

인라인 뷰를 사용하여 고객번호가 2 이하인 고객의 판매액 구하기

```sql
select name, sum(saleprice) from 
	(select custid, name from Customer where custid <=2) c, Orders o
where c.custid=o.custid
group by name;
```

---

## 뷰

하나 이상의 테이블을 합하여 만든 가상의 테이블

Book 테이블에서 ‘축구’라는 문구가 포함된 자료만 보여주는 뷰 생성하기

```sql
create view vBook
as
select * from Book where bookname like '축구%';
```

![image-20200724113540315](db_practice2.assets/image-20200724113540315.png)

```sql
select * from vBook;
```

![image-20200724113624229](db_practice2.assets/image-20200724113624229.png)

<br/>

Orders 테이블에 고객이름과 도서이름을 바로 확인할 수 있는 뷰를 생성한 후, ‘김 연아’ 고객이 구입한 도서의 주문번호, 도서이름, 주문액을 보이시오.

```sql
create view vw_Orders(orderid, custid, name, bookid, bookname, saleprice, orderdate)
as
select o.orderid, o.custid, c.name, o.bookid, b.bookname, o.saleprice, o.orderdate
from Orders o, Customer c, Book b
where o.custid=c.custid and o.bookid=b.bookid;
```

![image-20200724114219220](db_practice2.assets/image-20200724114219220.png)

<br/>

### 뷰의 수정

대한민국 국적만 나오는 테이블에서 영국 국적 가진 사람만 나오게 수정

```sql
create or replace view vCustomer(custid, name, address)
as
select custid, name, address
from Customer
where address like '영국%';
```

고객별 총 구매 금액 리스트 뷰 만들기

```sql
create view vOrderTotal(name, totalprice)
as
select c.name, sum(saleprice)
from Customer c, Orders o
where c.custid=o.custid
group by c.custid;
```

<br/>

### 뷰 삭제

```sql
drop view [뷰 이름]; -- 뷰 삭제, 원본 테이블은 삭제 안 된다
```

---

### [도서 데이터베이스]

```sql
-- 1. 판매가격이 20,000원인 도서의 도서번호, 도서이름, 고객이름, 출판사, 판매가격을 보여주 는 highorders 뷰를 생성하시오.
create view highorders
as
select * from Book where price>=20000;

-- 2. highorders 뷰를 변경하고자 한다. 판매가격 속성을 삭제하는 명령을 수행하시오. 삭제 후 (2)번 SQL 문을 다시 수행하시오.
create or replace view highorders(bookid, bookname, publisher)
as
select bookid, bookname, publisher from Book
where price >=20000;


```



### [사원 데이터베이스]

```sql
-- 3. 팀장(MGR)이 없는 직원의 이름을 보이시오.
select concat(first_name, ' ', last_name) 
from employees where manager_id is null;

-- 4. 사원의 이름과 부서의 이름을 보이시오(조인/스칼라 부속질의 사용).
- 조인
- 스칼라 부속질의
select concat(first_name, ' ', last_name), d.department_name 
from employees e, departments d
where e.department_id=d.department_id;

-- 5. ‘Seattle’에 근무하는 사원의 이름을 보이시오
- 조인, 인라인 뷰, 중첩질의, EXISTS
select first_name from employees e, departments d
where e.department_id=d.department_id
and location_id=(
	select location_id
    from locations
    where city like 'Seattle');

-- 6. 평균보다 급여가 많은 직원의 이름을 보이시오.
select concat(first_name, ' ', last_name) from employees 
where salary > (select avg(salary) from employees);

-- 7. 자기 부서의 평균보다 급여가 많은 직원의 이름을 보이시오(상관 부속질의 사용). 
select concat(first_name, ' ', last_name) from employees e, departments d
where e.department_id=d.department_id 
and salary >(
	select avg(salary) 
	from employees 
	where department_id=e.department_id
);
```



---

# Database Programming

비식별관계 : 부모의 키가 일반 속성으로 포함되는 관계

식별관계 :  부모의 키가 주 식별자로 포함되는 관계