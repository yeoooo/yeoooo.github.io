---
title: "Matplotlib"
excerpt: "Matplotlib 정리"

categories:
    Python_Library
tags:
   파이썬
   라이브러리
toc: true
comments: true
---

## 개요  
matplotlib은 데이터의 그래프를 그리는데에 __요긴하게__ 사용될 수 있다.  
함수를 그려주는 라이브러리로는 <span style = "color : orange">MATLAB</span> 또한 존재하는 것으로 알고있는데, 꼭 MATLAB을 써야 하는 상황이 아니라면 matplotlib을 사용한다는 의견이 대다수 였다.  
  
matplotlib을 사용하기 위해서는 다음과 같이 import를 해줘야한다.  
```python
from matplotlib import pyplot as plt
#or
import matplotlib.pyplot as plt
#이쪽이 훨씬 간편하다
```
import에 있어서 바로 'import matplotlib'가 아니라 matplotlib.pyplot과 같이 사용하는 것이 잊어버리기 쉬울 것 같아 정리해 놓았다.  
  
import하는 방법을 알아보았으니 바로 주요 함수에 대해 정리했다.  

## 주요 함수  
### <span style = "color : orange">plt.plot(x,y)</span>  
 plt.plot은 정수 x, y를 받아들여 x축과 y축을 그린다.  
 list 형을 하나만 넣어준다면 이 안의 값들을
 y값으로 그려내고, x값으로는 자동으로 이에  맞는 x값 [0, 1, 2, 3] 값을 생성한다. 

```python
plt.plot([1,2,3,4])
```  
<img src = "../../assets/images/matplotlib/matplotlib_plot1.png">  

다음은 x 값을 추가해준 코드이다.  
```python
plt.plot([1, 2, 3, 5, 6], [1, 3, 5, 10, 18])
```  
<img src = "../../assets/images/matplotlib/matplotlib_plot2.png">  
  
이해를 돕기 위해 반대로도 적용해 보았다.  
```python
plt.plot([1, 3, 5, 10, 18],[1, 2, 3, 5, 6])
```  
<img src = "../../assets/images/matplotlib/matplotlib_plot3.png">  

#### <span style = "color : orange">Graph colors, line style, markers</span>  
x,y에 대한 리스트 이외로 세번째 인자를 주서 다른 옵션을 줄 수 있는데, 그 인자의 종류로는 아래가 있다.  

<strong><span style= "font-size :15px">색상</span></strong>  
  
|Colors|Factor|  
|:---:|:---:|  
|<span style = "color:blue">blue</span>|'b'|  
|green|<span style = "color:green">'g'</span>|  
|red|<span style = "color:red">'r'</span>|  
|cyan|<span style = "color:cyan">'c'</span>|  
|magenta|<span style = "color:magenta">'m'</span>|  
|yellow|<span style = "color:yellow">'y'</span>|  
|black|<span style = "color:black">'k'</span>|  
|white|<span style = "color:white">'w'</span>|  
그래프의 모양을 실선이 아닌 다른 형식으로도 나타낼 수 있다. 그 중 세번째 인자 값으로 'bo'를 준다면 파란색 원형 마커로 표시된다.
```python
plt.plot([1, 2, 3, 5, 6], [1, 3, 5, 10, 18], 'bo')
```  
### <span style = "color : orange">plt.ylabel('labelname')/ plt.xlabel('labelname')</span>  

xlabel, ylabel로 각 축의 이름을 설정해 줄 수 있다.  
```python
plt.plot([1, 2, 3, 5, 6], [1, 3, 5, 10, 18])
plt.xlabel('X-Axis')
plt.ylabel('Y-Axis')
plt.show()
```
<img src = "../../assets/images/matplotlib/matplotlib_x_ylabel.png">  


