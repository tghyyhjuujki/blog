---
title: C/C++ 알고리즘 풀이를 위한 잡기술 정리(계속 업데이트)
date: 2020-08-30 18:08:84
category: Algorithm
draft: false
---

## String, Char

C++은 Python보다 문자 또는 문자열 처리하기가 상당히 까다로워서 정리해보려한다. 계속 추가할 예정이다.

<br/>

### Char 대문자 <-> 소문자

다음과 같이 사용한다. 반대의 경우 `toupper`로 바꿔준다.

```c
string temp = "ASDF", temp2;
for (int i = 0; i < temp.size(); ++i)
    temp2.push_back(tolower(temp[i]));
cout << temp2;

//asdf 출력.
```



### String 대문자 <->  소문자

다음과 같이 사용하면 된다. 반대의 경우 `::toupper`로 바꿔준다.

```c
#include <algorithm>

string temp = "ASDF";
transform(temp.begin(), temp.end(), temp.begin(), ::tolower);
cout << temp;

//asdf 출력
```



### String 문자열 자르기

```c
#include <string>

string str1 = "ASDF";
for (int i = 0; i < str1.size() - 1; ++i) {
    string temp = str1.substr(i, 2);
    transform(temp.begin(), temp.end(), temp.begin(), ::tolower);
    v1.push_back(temp);
}

// as, sd, df 들어있음
```



### String to int

```c
#include <string>

string temp="";
for (int i = 0; i < 5; ++i)
	temp += to_string(i);    // 방법 1, 가끔 사용 불가능할 때 있음
	temp += i+'0';			// 방법 2, 편법으로 매우 유용하게 사용 가능
```



### int to String

```c
int temp = 0;
char a = 'a';
temp += a -'0'; // 편법으로 유용하게 사용 가능
```



---

## Vector

### 벡터 초기화, 원소삭제

벡터를 초기화한다. 특정 인덱스를 삭제하고 뒤의 인덱스를 앞으로 당길 수 있다.

```c
vector<int> v{ 1, 2, 3, 4, 5}; // 벡터 초기화
v.erase(v.begin() + 3);  // v{1, 2, 3, 5}
```



### 사이즈 동적할당

```c
// 1차원
v.assign(size, 0); // size만큼 0을 할당함, 초기화 뒤에도 사용 가능

// 2차원
int row = 15;
int col = 10;
v.assign(row, vector<int>(col, 1));
```



### find()

String 의 find와는 조금 다르다. iterator를 사용하면 포인터처럼 간접적으로 값을 찾아낸다. 또한 해당 값이 어느 인덱스에 있는지 찾을 수도 있다. 해당 값을 찾지 못하면 end를 가르킨다.

```c
vector<string> v;
vector<string>::iterator iter;
v.push_back("asdf");
v.push_back("asdf");
v.push_back("Jeju");

iter = find(v.begin(), v.end(), "Jeju");
cout << *iter;			// Jeju, 인자가 없으면 NULL
cout << iter - v.begin();	 // 2

iter = find(v.begin(), v.end(), "Jeju2");
if(iter == v.end())
    cout << "없음";		//없음 출력
else cout << "있음";
```

문자열 안에서 문자열 찾기

```c
#include <iostream>

using namespace std;

int main() {
    string a="hi hello";

    if(a.find("hi")>=0){
        cout<<a.find("hi")<<endl;
    }
    if(a.find("hello")>=0){
        cout<<a.find("hello")<<endl;
    }
    return 0;
}
//각각 시작 인덱스인 0, 3이 출력됨
```





---

## Queue

인덱스로 접근이 불가능함

iterator도 못씀





---

## Map

키가 중복 불가능한 해시테이블

```c
#include <map>

map<char, int> m;
string s = "aba";
for(int i=0;i<s.size();++i)
    m[s[i]]++;

//a에 2 b에 1 저장되어있음

// 이 키가 있는지 없는지 확인
if(m.count("a")){
    //키에 value 가 뭔지 확인
    cout<< m["a"];
}
```

### 순회

```c
#include <map>
map<string, int> m; // m에 값이 있다는 가정하에

map<string, int> ::iterator iter;
for (iter = m.begin(); iter != m.end(); ++iter) {
    if (iter->second > 0)
        answer = iter->first;
}
```



---

## Set

중복을 처리해줘야할 때 사용 가능

ex) dfs에서 중복을 처리할  때 사용 가능

[불량사용자](https://programmers.co.kr/learn/courses/30/lessons/64064)가 좋은 예시

```c
#include <set>
string temp="";
for (int i = 0; i < check.size(); ++i) {
        temp += i+'0';
}
s.insert(temp);
```





---

## Tree

### 이진트리

다음 문제가 이진트리 공부하기에 아주 좋다.

[프로그래머스 길찾기게임](https://jeonghoon.netlify.app//Algorithm/Programmers/2019kakao_find_load/)

---

## 정렬

기본

```c
#include <algorithm>

//기본적인 정렬, 오름차순이 디폴트
sort(nodeinfo.begin(), nodeinfo.end());
```

커스텀

```c
bool cmp(TREE a, TREE b) {
	if (a.y == b.y)return a.x < b.x;
	return a.y > b.y;
}
```



우선순위 큐를 사용한 정렬, 구조체 사용

```c
//백준 16236 아기상어
struct SHARK{
	int r, c;
}shark;

priority_queue<SHARK> pq;

bool operator<(SHARK a, SHARK b) {
	if (a.r == b.r) return a.c > b.c;
	return a.r > b.r;
}

//push 하면 오퍼레이터 적용됨
 pq.push({ nr,nc })
```

