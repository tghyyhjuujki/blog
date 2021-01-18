---
title: Java 알고리즘 풀이를 위한 잡기술 정리(계속 업데이트)
date: 2020-08-30 18:08:84
category: Algorithm
draft: false
---



갑자기 자바로 코테를 준비하는 이유는, 백 엔드 부분에서 C++을 안 보는 기업이 있어서다.

준비 기간이 짧은 만큼, 속성으로 준비했다.

![image-20210117163541512](java.assets/image-20210117163541512.png)

<br/>

## 해시









## 정렬, 배열 할당

프로그래머스 K번째 수

```java
import java.util.*;

class Solution {
    public int[] solution(int[] array, int[][] commands) {
        int[] answer = new int[commands.length];
        
        for(int i=0;i<commands.length;++i){
            
            int[] temp = new int[commands[i][1] - commands[i][0] + 1];
            for(int j=0; j<temp.length; ++j){
                temp[j] = array[j+commands[i][0] - 1];
            }
            Arrays.sort(temp);
            answer[i] = temp[commands[i][2] - 1];
        }
        return answer;
    }
}
```

