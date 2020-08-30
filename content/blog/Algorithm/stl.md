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



---

## Vector

### 벡터 초기화, 원소삭제

벡터를 초기화한다. 특정 인덱스를 삭제하고 뒤의 인덱스를 앞으로 당길 수 있다.

```c
vector<int> v{ 1, 2, 3, 4, 5}; // 벡터 초기화
v.erase(v.begin() + 3);  // v{1, 2, 3, 5}
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





---

## Queue

인덱스로 접근이 불가능함

iterator도 못씀