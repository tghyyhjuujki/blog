---
title: 2018 kakao blind [1차]) 캐시
date: 2020-08-30 18:08:57
category: Algorithm
draft: false
---



문제는 [2018 kakao blind [1차]) 캐시](https://programmers.co.kr/learn/courses/30/lessons/17680)에서 확인할 수 있다.

처음에는 65/100 점을 받았다. LRU(Least Recently Used)를 제대로 이해하지 못한 내 잘못이었다. 

LRU는 페이지 교체 알고리즘 중 하나로, 최근에 사용한 동일한 데이터가 있으면 캐시에서 꺼내와 쓰는 방식이다. 

여기서 주의할점은, 캐시에 동일한 데이터가 있으면, 최신의 인덱스로 교체해줘야 하는 것이다.  예를 들어 아래와 같은 데이터가 있고 캐시의 크기가 3이라고 가정해보자.

```c
"Jeju"
"Pangyo"
"seoul"
"Jeju"
"LA"
```

seoul 까지 읽었을 때, 캐시 안에는 `Jeju, Pangyo, seoul` 순으로 저장되어 있을 것이다. 이제 4번째 Jeju 를 읽을 때 캐시 안에 Jeju가 있으므로 캐시 안의 Jeju 를 삭제하고 최신 인덱스에 Jeju 를 넣어줘야한다.

`Pangyo, seoul, Jeju`

만약 본인이 65점을 받았다면 이 점을 다시 한 번 고민해보자.

<br/>

아래에 소스코드와 주석을 달아놨다.

```c
#include<string>
#include<vector>
#include<algorithm>

using namespace std;

int solution(int cacheSize, vector<string> cities) {
	int answer = 0;
	int city_size = cities.size();
	vector<string> cache;
	vector<string>::iterator iter;
	// 일단 전부 소문자로 만들어서 비교하기 쉽게 만든다.
	for (int i = 0; i < city_size; ++i)
		transform(cities[i].begin(), cities[i].end(), cities[i].begin(), ::tolower);

	for (int i = 0; i < city_size; ++i) {
		//cache 안에 문자열 있나 탐색
		iter = find(cache.begin(), cache.end(), cities[i]);
		// 일치하는 문자열이 없다면 +5
		if (iter == cache.end()) answer += 5;
        // 있으면 +1, 그리고 그 캐시를 삭제해서 무조건 최신상태로 갱신가능하게 만든다.
		else {
			cache.erase(cache.begin() + (iter - cache.begin()));
			answer++;
		}
		if (cacheSize != 0) {
			if (cache.size() >= cacheSize)
				cache.erase(cache.begin());
			cache.push_back(cities[i]);
		}
	}
	return answer;
}
```

