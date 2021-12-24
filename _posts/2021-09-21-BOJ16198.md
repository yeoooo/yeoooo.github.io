---
title: "백준[ 16198 \\| Python ] 에너지모으기 파이썬"
excerpt: "백트래킹"

categories:
    Algorithm
tags:
   백트래킹
   브루트포스
toc: true
comments: true
---
## 문제
<https://www.acmicpc.net/problem/16198>
<p align = "center"><img alt = "boj16198" src = "../../assets/images/boj/gathering_e.png"></p>

## 시도
첫 시도는 전형적인 백트래킹으로 선택한 에너지구슬 i의 모든 경우의 수를 뽑고  
탈출 구문에서 문제에 나온대로 i-1과 i+1을 곱하고 그 수를 value로 갱신한 뒤 리스트의 내용을 변경하며 해결하려 했다.  
하지만 재귀로 인해 리스트 갱신이 원하는 대로 되지 않았고 백트래킹에 관한 문제를 그렇게 적지 않게 풀었다고 생각했는데  
아직 이해가 부족하다는 생각이 들었다.    
조금 아쉬웠지만 다른 블로그를 참고했고 그럴만한 가치가 있었다.  

## 참고
코드는 아래 블로그를 참고했다.  
<https://jokerkwu.tistory.com/160>  
백트래킹과 브루트포스가 사용되었다.

```python
import sys

n = int(sys.stdin.readline().split()[-1])
marbles = list(map(int, sys.stdin.readline().split()))
gathered = 0

def backtrack(val):
    global marbles
    if len(marbles) == 2:
        global gathered
        if gathered < val:
            gathered = val
            return

    for i in range(1, len(marbles)-1):
        val += marbles[i-1]*marbles[i+1]
        chosen = marbles[i]
        marbles = marbles[:i]+marbles[i+1:]# == del marbles[i]
        backtrack(val)
        marbles.insert(i,chosen)
        val -= marbles[i-1]*marbles[i+1]

backtrack(0)
print(gathered)
```
## 생각 & 정리
재귀에 값을 전달해 가면서 stack에 append, pop을 시키는 과정과 비슷하게 문제에 제시 된대로 값을 하나씩 삭제하고, 다시 추가하면서 해결한 풀이였다.  
"안그래도 val은 원상태로 만들어줘야 할텐데 .." 라는 고민을 했었는데, 그 고민도 말끔하게 해결된 풀이였다.  
항상 알고리즘 기법을 새로 익히게 되면 편협하게 푸는 버릇이 생기는데 전체를 이해했다면  
그런 실수를 하지 않았을 것 같다.  
아직도 많이 부족하다는 생각이 들었다.
