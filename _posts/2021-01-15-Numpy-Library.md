---
title: "Numpy"
excerpt: "Numpy 정리"

categories:
    Python
tags:
   파이썬
   라이브러리
toc: true
comments: true
---

    

예전에 '__전산언어학__'이라는 강의를 들었는데 그 강의에서는 파이썬을 통해 코퍼스를 분석하는 강의였다.
  또한 당연하게도 파이썬에서의 여러 모듈과 라이브러리를  이용했는데 <strong><font size = 6>하 나 도</font></strong> 그 명령어들이 무슨 의미인지 모르는 채로 공부하다가 처참한  성적을 받은 기억이있다.  
뒤로 머신러닝을 공부하게 되면서 다시 파이썬을 접했고, 밑바닥 부터 시작하는 마당에  라이브러리부터 차근차근 정리하며 가자는 생각이 들었다.  
  
서론이 길었다. 이제부터 Numpy에 대해 정리할 것이다.  
    
# Numpy란  
---------
머신러닝을 공부하다보면 Numpy는 피할 수 없다.  
이 Numpy가 하는 일은 일반적으로 코드에서 구현하기 힘든 행렬이나 다차원 배열을 쉽게 처리할 수 있도록 지원한다.  이러한 <strong><span style="color:orange; font-size : 30px;">데이터 구조</span></strong> 이외에도 <strong><span style="color:orange; font-size : 30px;">수치 계산</span></strong>   을 효율적으로 처리해준다! 
  
  
# Numpy의 함수 정리 노트
   ----------------------------------------- 

   ## <span style = "font-size : 30px">np.arange(start, stop,(step))</span>  
   np.arange()는 정수 값 start부터 stop 까지의 수를 array 형태로 반환한다.  
   step의 명시는 선택적이지만 지정해주지 않는다면 default 값으로 1을 갖는다.   
   이 메서드의 반환 타입은 array이다.
   ```python
   np.arange(10)
   >>> array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
   np.arange(0,10,3)
   >>> array([0, 3, 6, 9])
   ``` 

## <span style = "font-size : 30px">np.zeros()</span>

  기본 행렬을 생성해주는 함수 중 하나.  
  원소 모두 0을 가진채로 생성해준다!  
  
  ```python  
  >>> np.zeros((3,3))  
    
  array([0, 0, 0],
        [0, 0, 0]
        [0, 0, 0])
  ```

##  <span style = "font-size : 30px">np.full()</span>  
 
  np.zeros()와 같이 기본 행렬 생성 함수.  
  원소를 원하는 값으로 채워줄 수 있다.  

  ```python  
  >>> np.full((3,3),16)  
    
  array([16, 16, 16],
        [16, 16, 16]
        [16, 16, 16])
  ```  

##  <span style = "font-size : 30px">np.eye()</span>  
     

  위 두 함수와 같이 기본 행렬 생성 함수.
  원소를 원하는 값으로 채워줄 수 있다.  
  이 때 인자로 들어가는 값은 행렬의 x, y 값이며 주대각선을 0으로, 나머지는 0인 행렬을 생성한다.  
  ```python  
  >>> np.eye(3)  
    
  array([1, 0, 0],
        [0, 1, 0]
        [0, 0, 1])
  ``` 
##  <span style = "font-size : 30px">np.array()</span>  
 
  np.array는 list를 ndarray로 변환해준다. 

  ```python  
  >>> list = [1,2,3,4]
  >>> arr = np.array(list)
  >>> arr   
    
  array([1,2,3,4])
  ```  
##  <span style = "font-size : 30px">np.shape()</span>  
 
  해당 ndarray의 차원을 보여준다. 

  ```python  
  >>> some_data = ([1, 2, 3, 4, 5])  
  >>> some_data = np.array(some_data) #ndarray만 쓸 수 있다
  >>> some_data.shape  

  (5,)
  ```  
##  <span style = "font-size : 30px">np.reshape()</span>  
 
  np.reshape()는 행렬의 차원을 바꿔줄 수 있다.   

  ```python  
  >>> sample_array = ([1, 2, 3],
               [4, 5, 6],
               [7, 8, 9])   
  >>> sample_array = np.array(sample_array)
  >>> sample_array.reshape(-1) #이렇게 1 행으로 만들어 줄 수 있다. 제일 많이 썼던 것 같다.
array([1, 2, 3, 4, 5, 6, 7, 8, 9])
    
  ```  
  이러한 주요 함수를 제외하고도 많은 함수가 있는데, 이는 딥러닝 공부를 이어가면서 함께 늘려나갈 생각이다!


  
  
  
  



