---
title: 2018 kakao blind [1차]) 프렌즈4블록
date: 2020-08-30 18:08:57
category: Algorithm
draft: false
---





문제는 [프렌즈4블록](https://programmers.co.kr/learn/courses/30/lessons/17679)에서 확인 가능하다.

문제가 그렇게 어렵진 않았지만 구현할 게 많았다.

<br/>

아래는 소스코드와 주석.

```c
#include <vector>
#include <string>
#include <queue>
using namespace std;

queue<pair<int,int> > q;
vector<string> v;

// 블록 내려가게 중력 적용, 아래부터 체크
void gravity(int m, int n) {
	for (int i = m - 2; i >= 0; --i) {
		for (int j = 0; j < n; ++j) {
			if (v[i][j] != '.' && v[i + 1][j] == '.') {
				int r = i + 1, c = j;
				while (1) {
					if (v[r][c] != '.') {
						v[r - 1][c] = v[i][j];
						v[i][j] = '.';
						break;
					}
					else if (r == m - 1) {
						v[r][c] = v[i][j];
						v[i][j] = '.';
						break;
					}
					else if (v[r][c] == '.')r++;
				 }
			}
		}
	}
}

// 제거할 블록이 있는지 체크
bool check(int m, int n) {
	bool temp = false;
	for (int i = 0; i < m - 1; ++i) {
		for (int j = 0; j < n - 1; ++j) {
			if (v[i][j] != '.' && v[i][j] == v[i][j + 1] && v[i][j] == v[i + 1][j] && v[i][j] == v[i + 1][j + 1]) {
				q.push({ i,j });
				temp = true;
			}
		}
	}
	while (!q.empty()) {
		int i = q.front().first;
		int j = q.front().second;
		q.pop();
		v[i][j] = '.';
		v[i][j + 1] = '.';
		v[i + 1][j] = '.';
		v[i + 1][j + 1] = '.';
	}
	return temp;
}

int solution(int m, int n, vector<string> board) {
	int answer = 0;
	v = board;
	while (1) {
        // 제거할 블록이 있으면 제거하고 중력 적용
		if (check(m, n) == true) gravity(m, n);
		else break;
	}
	for (int i = 0; i < m; ++i)
		for (int j = 0; j < n; ++j)
			if (v[i][j] == '.') answer++;
	cout << answer;
	return answer;
}
```

