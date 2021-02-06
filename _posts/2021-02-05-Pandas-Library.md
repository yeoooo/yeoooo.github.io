---
title: "Os"
excerpt: "Os 정리"

categories:
    Python_Library
tags:
    파이썬
    라이브러리
toc: true
comments : true
---
## 개요
pandas는 <strong>데이터 분석</strong>을 위해 사용되는 패키지 이다.
pandas는 크게 세가지의 자료구조를 지원한다.  

- 1차원 자료구조  
Series  
<strong>가장 간단한</strong> 1차원 자료구조이다. <span style =" color : blue">Sequence데이터(배열, 리스트)</span>를 입력으로 받아 들인다.  
```python
import pandas as pd

test = {
    'year' : [2019, 2020, 2021],
    'price' : ['1500', '1600', '1650']
    'yield' : [2000, 1500, 1500]}

df = pd.DataFrame(test)
```

- 2차원 자료구조  
DataFrame  
행과 열이 있는 <strong>테이블 데이터</strong>를 입력으로 받아들인다. 열은 딕셔너리 형의 key, 행은 value로 취한다.   

- 3차원 자료구조  
Panel
  
