---
title: 2018 kakao blind [1차]) 뉴스 클러스터링
date: 2020-08-23 18:08:57
category: Algorithm
draft: false
---

설명은 나중에,

문제는 [뉴스 클러스터링](https://programmers.co.kr/learn/courses/30/lessons/17677)에서 확인가능하다.

```c
#include <string>
#include <vector>
#include <algorithm>
#include <map>
using namespace std;

int solution(string str1, string str2) {
	int answer = 65536;
	int size1 = str1.size() - 1, size2 = str2.size() - 1, mom = 0, son = 0;
	transform(str1.begin(), str1.end(), str1.begin(), ::tolower);
	transform(str2.begin(), str2.end(), str2.begin(), ::tolower);
	map<string, int> m;
	for (int i = 0; i < size1; ++i) {
		string temp = str1.substr(i, 2);
		if (temp[0] < 97 || temp[0]>122 || temp[1] < 97 || temp[1]>122)continue;
		m[temp]++;
		mom++;
	}


	for (int i = 0; i < size2; ++i) {
		string temp = str2.substr(i, 2);
		if (temp[0] < 97 || temp[0]>122 || temp[1] < 97 || temp[1]>122)continue;
		if (m.count(temp) > 0) {
			if (m[temp] == 0) {
				mom++;
			}
			else {
				son++;
				m[temp]--;
			}
		}
		else mom++;
	}
	if (mom != 0)
		answer = ((double)son / (double)mom)*answer;
	return answer;
}

```

