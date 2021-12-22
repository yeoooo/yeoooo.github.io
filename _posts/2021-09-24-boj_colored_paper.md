---
title: "백준[ 2563 \\| Python ] 색종이"
excerpt: "구현"
categories:
    Algorithm
tags:
   구현
toc: true
comments: true
---
## 문제  
<https://www.acmicpc.net/problem/2563>
<p align = "center"><img alt = "boj2563" src = "../../assets/images/boj/colored_p.png"></p>

## 시도
평소 문제를 풀 때 시간복잡도로 고생을 많이 했던 터라
도형을 맞이하고는 바로 들었던 생각이 '수'적으로 해결해야 겠다는 생각이었다.  

색종이 끼리 겹치는지 판단한 후 겹치는 부분의 넓이를 계산하고 (겹치는 종이의 수 * 100) - (겹치는 부분의 넓이)로 계산하려 했으나 몇 개의 종이가 얼마나 겹치는지 생각하기에 너무 복잡해서 풀이를 달리해야겠다고 생각했다.

## 풀이
도화지 크기 만큼의 2차원 0배열을 만들어주고  
길이가 10인 정사각형인 점을 이용해 x ~ x+10, y ~ y+10 의 범위를 1로 채워준 뒤 1의 개수를 센다.
```python
import sys
input = sys.stdin.readline
n = int(input())

paper = [[0 for _ in range(101)] for _ in range(101)]

for _ in range(N):
    x, y = map(int, input().split())

    for i in range(x, x+10):
        for j in range(y, y+10):
            paper[i][j] = 1

answer = 0
for i in paper:
    answer += i.count(1)

print(answer)
```