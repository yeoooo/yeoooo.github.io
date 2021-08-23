---
title: "파이썬 recursion limit"
excerpt: "재귀 한도 설정"

categories:
    Python
tags: 
    library

comments: true
---
백준 1012 - 유기농배추 문제를 DFS로 풀이하던 도중에 recursion Error를 접했다.  
오류에 대해선 이미 알고있었기 때문에 탈출 조건을 잘못 설정해줬는지 생각해보다가  

파이썬에서는 재귀에 대한 한도가 있다는 것을 떠올려서  
이를 찾아 보았고 이 한도를 아래 코드를 통해서 늘여줄 수 있었다.  
```python
import sys

sys.setrecursionlimit(10**6)

```  
이유를 찾지 못해서 뇌가 작동 정지할 참이었는데 다행이었다 :)
