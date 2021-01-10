---
title: 프로그래머스/C++) 징검다리 건너기(슬라이딩 윈도, 이분탐색)
date: 2021-01-09 21:37:57
category: Algorithm
draft: false
---

문제는 [징검다리 건너기](https://programmers.co.kr/learn/courses/30/lessons/64062)에서 확인가능하다.

간단해 보이지만, 간단한 방식으로 풀면 당연하게도 시간초과가 발생한다.

슬라이딩 윈도, 이분탐색 두 가지 방법으로 풀어봤다.

또한 테스트케이스 3번, 효율성 13번에서 계속 틀렸다. 반례는 아래와 같은 테스트케이스일 경우이며, 맨 뒤의 2, 1, 1을 탐색하지 않아 생긴 오류였다.

```
k = 3
2, 4, 5, 3, 2, 1, 4, 2, 1, 1
answer = 2
```

<br/>

### 슬라이딩 윈도

문제를 읽어보면, 0이 되는 돌이 연속으로 k개 나오면 건너지 못하는 것을 알 수 있다.

따라서 k 길이의 윈도를 만들고 윈도 안에서 최대값을 뽑아낸다. 왜냐하면 최대값이 0이 될때 나머지 값들도 0이 되기 때문에 건널 수 없다. 따라서 해당 범위에서 뽑아낸 최대값의 수만큼 건널 수 있는 것이다.

이 작업을 윈도의 좌,우를 1씩 늘리며 반복해주면 된다.

시간복잡도는 각 영역에서 최대값을 구하는 작업이 logN 시간 소요되고,

이러한 작업을 stones의 사이즈인 N만큼 반복해야 하므로 

총 소요시간은 N*logN이다.

```c
#include <string>
#include <vector>
#include <map>

using namespace std;

int solution(vector<int> stones, int k) {
	multimap<int, int, greater<int> > m;
	multimap<int, int>::iterator iter;
	int size = stones.size();

	// 길이가 k인 윈도 생성
	for (int i = 0; i < k; ++i)
		m.insert({ stones[i], i});

	//초항 처리
	iter = m.begin();
	int answer = iter->first;

	// 윈도 안에서 최대값 추출
	for (int i = 0; i < size - k; ++i) {

		//1씩 이동
		iter = m.find(stones[i]);
		m.erase(iter);
		m.insert({ stones[i + k], i + k});

		iter = m.begin();
		if (iter->first < answer)
			answer = iter->first;
	}
	return answer;
}
```

<br/>

### 이분탐색

먼저 stones의 최대값을 구한 후, 중간씩 끊어서 최대 몇명이 건널 수 있는지 구해본다.

시간복잡도는, 건널 수 있는 사람의 수를 1/2로 나누는 작업을 반복하므로 logN,

그 사람들을 stones의 사이즈 N만큼 반복해야 하므로  

총 소요시간은 N*logN이다.

```c
#include <string>
#include <vector>
#include <algorithm>

using namespace std;

bool check(int& mid, int& k, vector<int>& stones) {
	int cnt = 0;
	int size = stones.size();
	for (int i = 0; i < size; ++i) {
	
		// 연속가능한 돌이면 +1
		if (stones[i] <= mid)
			cnt++;
		// 연속 불가능하면 초기화
		else 
			cnt = 0;
		// 연속 가능한 돌이 k 이상이면 true
		if (cnt >= k)return true;
	}
	return false;
}

int solution(vector<int> stones, int k) {
	//start, mid, end 포인트 설정
	int s = 1, mid, e = *max_element(stones.begin(), stones.end());

	//이분탐색
	while (s <= e) {
		mid = (s + e) / 2;
		// 중간으로 자른 범위에서 연속하는 구간이 나오면
		if (check(mid, k, stones))
			e = mid - 1;
        // 나오지 않으면
		else
			s = mid + 1;
	}
	return s;
}
```

