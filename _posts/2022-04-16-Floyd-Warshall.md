---
title: 
    "알고리즘[최단경로] 플로이드 워셜(Floyd-Washall)"
excerpt: 
    모든 노드에서 모든 노드로의 최단경로를 찾는 플로이드 워셜
category: 
    Algorithm
tags: 
    최단경로
toc: 
    true
comments: 
    true
use_math: 
    true
    #for use, there must be dollar sign on both side of contents
---

<style type = 'text/css'>
    .o{
    font-weight: bold;
    color:orange;
    }
</style>

## 개요  
이전 포스트에서 올렸던 임의의 노드에서 특정 지점 까지의 최단 경로를 구하는 경우에 사용할 수 있는 다익스트라 알고리즘과는 다르게 플로이드 워셜 최단거리 알고리즘은 <span class = "o">모든 지점에서 다른 모든 지점까지의 최단 경로를 모두 구하는 알고리즘</span>이다.  

큰 과정 자체는 다익스트라와 비슷하지만. <span class = "o">방문하지 않은 노드 중 최단 거리를 갖는 노드는 찾지 않는다.</span> 이로 인해 노드 개수 N일 때 N의 연산을 하며 단계마다 현재 노드를 거쳐가는 모든 경로를  고려하며 $O(N^2)$의 연산을 거친다.  
즉 플로이드 워셜 알고리즘의 시간 복잡도는 $O(N) * O(N^2) = O(N^3)$이 된다.  

좀 더 쉽게 과정을 말하자면 <span class = "o">플로이드 워셜 알고리즘은 노드 A에서 노드 B까지 가는데에 필요한 비용을 A에서 다른 노드 C로 거친 뒤 C노드에서 B노드로 가는 비용이 더 적다면 이를 테이블에 갱신해주는 알고리즘</span>이다.  

이를 점화식으로 표현하면 $D_{ab} = min(D_{ab}, D_{ac}+D_{cb})$가 되는데, 이처럼 점화식에 따라서 테이블을 갱신해 나아가기 때문에 플로이드 워셜 알고리즘은 다이나믹 프로그래밍으로 분류된다. 

  
플로이드 워셜 알고리즘의 과정은 다음과 같다.  

    1. 최단 거리 테이블 초기화(2차원 리스트)
    2. 자기 자신으로 가는 비용을 0으로 초기화
    3. 현재 경로를 거쳐가는 모든 경로를 탐색

위의 과정에 대한 소스코드이다.  

```python  
INF = int(1e9)

n,m = map(int, input().split())#노드 개수, 간선의 개수
graph = [[INF] * (n+1) for _ in range(n+1)]

for i in range(1, n+1):#자기 자신까지의 거리 초기화
    for j in range(1, n+1):
        if i == j:
            graph[i][j] = 0

for _ in range(m):#a에서 b로가는 비용 c 정보를 그래프에 입력
    a, b, c = map(int, input().split())
    graph[a][b] = c

for k in range(1, n+1):#a에서 b로 바로 가는 것보다 연결된 다른 노드를 통해서 가는 것이 빠르다면 
    for a in range(1, n+1):#그것이 최단 경로가 된다.
        for b in range(1, n+1):
            graph[a][b] = min(graph[a][b], graph[a][k] + graph[k][b])

print("========")
for i in range(1,n+1):
    for j in range(1,n+1):
        if graph[i][j] == INF:
            print("can't reach", end=" ")
        else:
            print(graph[i][j], end= " ")
    print()
```  

위에서 말했던 것 처럼 _모든 노드로 부터 모든 노드까지의 경로의 최소 비용_ 을 테이블에 갱신 해주어야하기 때문에 다익스트라 최단경로 알고리즘과 다르게 플로이드 워셜 알고리즘은 최소 비용 테이블을 <span class = "o">2차원 리스트</span>로 설정하게 된다.

