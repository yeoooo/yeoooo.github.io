---
title: "Data preprocessing(데이터 전처리)"
excerpt: "Preprocessing(전처리) 정리"

categories:
    deep_learning
tags:
    파이썬
    딥러닝
toc: true
comments : true
---

# 개요  
딥러닝을 하기 위해서 데이터 전처리는 <strong><span style = "color : orange; font-size : 17px">꼭 필요한 작업</span></strong>이다.  
전처리가 이루어지지 않았을 때 모델이 학습을 수행하는데에 많은 에러를 발생시킬 수 있고, 모델에서 데이터를 통해 학습을 수행할 때 <strong><span style = "color : orange; font-size : 17px">성능에서의 차이</span></strong>또한 보여주기 때문이다.  
이러한 이유 때문에  
           " 데이터 분석의 8할은 