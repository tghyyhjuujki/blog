---
title: 프로그래머스) 길찾기 게임(슬라이딩 윈도, 이분탐색)
date: 2021-09-01 18:08:57
category: Algorithm
draft: false
---

문제는 [길찾기게임](https://programmers.co.kr/learn/courses/30/lessons/42892)에서 확인가능하다.

간단해 보이지만, 간단한 방식으로 풀면 당연하게도 시간초과가 발생한다.

슬라이딩 윈도, 이분탐색 두 가지 방법으로 풀어봤다.

또한 테스트케이스 3번, 효율성 13번에서 계속 틀렸고, 아래와 같은 테스트케이스를 찾았다.

<br/>

슬라이딩 윈도

```c
#include <string>
#include <vector>
#include <map>

using namespace std;

int solution(vector<int> stones, int k) {
	int answer = 200000001;
	multimap<int, int, greater<int> > m;
	multimap<int, int>::iterator iter;
	int size = stones.size();

	// 길이가 k인 윈도 생성
	for (int i = 0; i < k; ++i)
		m.insert({ stones[i], i});

	//초항 처리
	iter = m.begin();
	if (iter->first < answer)
		answer = iter->first;

	//최대값 추출
	for (int i = 0; i < size - k; ++i) {
		iter = m.begin();
		if (iter->first < answer)
			answer = iter->first;
		
		//1씩 이동
		iter = m.find(stones[i]);
		m.erase(iter);
		m.insert({ stones[i + k], i + k});
	}
	return answer;
}
```

