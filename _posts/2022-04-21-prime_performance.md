---
title: 
    "알고리즘[소수 판별]세 소수 판별 알고리즘의 성능"
excerpt: 
    흔히 알려진 세 소수 판별법의 각 성능을 확인해보자
category: 
    Algorithm
tags: 
    소수 판별
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
프로그래머스의 `k진수에서 소수 개수 구하기`문제를 해결하다가 문득 실제로 소수 판별 알고리즘의 각 성능차가 얼마나 나는지 궁금해져 확인해보았다.  

### 기본 소수 판별 알고리즘  
간단히 자신을 제외한 수로 나눠떨어지는 수가 있으면 False를 반환하는 알고리즘이다.  
목표 값까지 하나씩 확인하기 때문에 시간복잡도는 $O(N)$이다
```python  
def is_prime(x):
    for i in range(2, x):
        if x % i == 0:
            return False
    return True
```

### 루트(제곱근)를 이용한 소수 판별 알고리즘  
목표 값의 제곱근 까지만 자신을 제외한 수로 나눠떨어지는 수가 있으면 False를 반환한다.  
따라서 시간복잡도는 $O(N^{1/2})$이다.
```python  
def is_prime_route(x):
    import math
    if x == 1:
        return False
    for i in range(2, int(math.sqrt(x)) + 1):
        if x % i == 0:
            return False
    return True
```
### 에라토스테네스의 체 소수 판별 알고리즘  
1부터 목표 값까지 어떤 수(i)를 증가시키며 그 수의 제곱근 까지의 모든 배수를 미리 준비한 목표 값 까지의 배열에서 지운다.  
<p align = "center"><img alt = "Eratosthenes.png" src = "https://upload.wikimedia.org/wikipedia/commons/b/b9/Sieve_of_Eratosthenes_animation.gif"></p>  
<p align ="center"><a href="https://ko.wikipedia.org/wiki/에라토스테네스의_체" style = "font-size : 10px">출처 : wikipedia</a></p>
모든 배수를 찾기 때문에 한 소수 뿐 아니라 다수의 소수를 찾는 문제에 더욱 적절하다. 
시간복잡도는 $O(NloglogN)$이다. 

```python  
def Sieve_of_Eratosthenes(num):
    sieve = [True] * n
    sieve[1] = False
    m = int(n ** 0.5)
    for i in range(2, m + 1):
        if sieve[i] == True:
            for j in range(i+i, n, i): 
                sieve[j] = False

    # 소수 목록 산출
    return [i for i in range(2, n) if sieve[i] == True]
```  

위의 세 방법의 시간 차는 아래와 같이 나왔다.

## 결과  
$10^7$에 대해 나온 시간은 제곱근을 이용한 판별 방법이 가장 빨랐고 기본 판별법, 에라토스테네스의 체를 이용한 판별이 그 뒤를 이었다.  

    is_prime = 4.506111145019531e-05
    is_prime_route = 2.002716064453125e-05
    Sieve of Eratosthenes = 2.568164825439453

판별하는 수를 $10^2$까지 줄였을 때의 결과는 아래와 같이 나왔다.    

    is_prime = 1.7881393432617188e-05
    is_prime_route = 1.7881393432617188e-05
    Sieve of Eratosthenes = 4.100799560546875e-05

비교하려는 수가 커지면 커질수록 제곱근을 이용한 판별법이 가장 우세한 성능을 보였다.  
에라토스테네스의 체를 이용한 판별은 다수의 소수를 구하기 때문에 다소 저조한 성능을 보였는데 이로 인해 목적이 확연하게 쓰여야 함을 확인했다.  

<span class = "o">제곱근을 이용한 소수 판별은 넓은 범위에서의 많지 않은 소수가 필요할 때</span>, <span class = "o">에라토스테네스의 체를 이용한 소수 판별은  다소 좁은 범위에서의 많은 소수가 필요할 때</span> 이용하는 것이 적절할 것 같다.


