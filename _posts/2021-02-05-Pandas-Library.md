---
title: "Pandas"
excerpt: "Pandas 라이브러리 정리"

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

- ### 1차원 자료구조  
Series  
<strong>가장 간단한</strong> 1차원 자료구조이다. <span style =" color : orange">Sequence데이터(배열, 리스트, 딕셔너리)</span>를 입력으로 받아 들인다.  
배열의 각 value에 대응되는 index를 부여할 수 있다

```python
import pandas as pd

test = {
    'year' : [2019, 2020, 2021],
    'price' : [1500, 1600, 1650]
    'yield' : [2000, 1500, 1500]}

df = pd.Series(test)
df
 
```
<img src = "../../assets/images/pandas_series.png">

- ### 2차원 자료구조  
DataFrame  
행과 열이 있는 <strong>테이블 데이터</strong>를 입력으로 받아들인다. <span style = "color : orange">열은 딕셔너리 형의 key, 행은 value</span>로 취한다. 

```python
import pandas as pd

test = {
    'year' : [2019, 2020, 2021],
    'price' : [1500, 1600, 1650]
    'yield' : [2000, 1500, 1500]}

df = pd.DataFrame(test)
df
 
```
<img src = "../../assets/images/pandas_dataframe.png">

- ### 3차원 자료구조  
Panel은 3개의 축을 가지며 한 축의 요소는 데이터 프레임을 가지고 축 2는 데이터프레임의 행, 축 3은 데이터프레임의 열을 가진다고 한다. 개념이 머리에 잘 들어오지 않지만,
패널은 많이 사용되지 않는다고 한다. 

- ### index  
위 자료구조들을 생성해 줄 때 <strong>index</strong> 옵션을 부여 해줄 수 있다. index 옵션을 따로 지정해 주지 않는 경우 default 값을 0으로 지정해준다.

    
- ## 주요 함수  

Series, DataFrame을 만드는 함수는 위에서 다루었으니 다른 쓸만한 함수에 대해서 다루었다.
  
### pd.read_csv('csv file')  
    
  아래 코드를 통해서 csv파일을 읽어주면, Dataframe을 얻을 수 있다.  
  ```python  
  df = pd.read_csv('csv_url')

  ```

### set_index('dataframe col')  

아래와 같이 year를 인덱스로 지정해 줄 경우 해당 열을 index로 사용한다.  

  ```python
  test_df = df.set_index(['year'])
  ```
### df.head(n) 
head함수의 n 변수에 정수 값 인자를 넣어 준다면 위에서 n개 만큼 보여준다.  
head대신 tail이 들어갈 수 있다.  

```python
print(df.head(3))
```  
