---
title: "NLP 전처리"
excerpt: "노이즈 제거"

categories:
    NLP
tags:
   전처리
   노이즈 제거
toc: true
comments: true
---
## 개요  
Aiffel에서 GoingDeeper노드로 들어갔다. 그동안 스터디에 친하게 지내지 않았던 수학을 다시 접하게 되어 겪게된 적응 시간 탓에 포스팅을 하지 못했다. 한번씩 읽어보기는 했지만, 제대로 읽고 이해할 필요가 있다고 생각되어 딥러닝의 시작 전처리부터 포스팅을 시작한다.  

## 노이즈 ?  
노이즈는 말그대로 잡음과 같이 <span style = " color : orange">학습을 방해하는 요소</span>가 되겠다.  
따라서 원활하고 좋은 결과를 이끌어내기 위한 학습을 하려면 <span style = " color : orange">노이즈가 적은 데이터를 이용하거나  
 노이즈를 적절히 제거</span>해주어야 한다.  
  
노이즈의 유형으로는 다음과 같다.  

1. 불완전한 문장으로 구성된 대화  
> A : "아니아니" "오늘이 아니고" "내일이지"  
> 
>B : "그럼 오늘 사야겠네. 내일 필요하니까?"
  
2. 과도하게 짧거나 긴 문장의 길이  
> A : "ㅋㅋㅋㅋ","ㄴㄴ","ㅇㅇ
>
> A : "이 편지는 영국에서부터 시작되어 ... "
  
3. 문장에서의 시간 간격이 너무 긴 채팅 데이터  
> A : "나 급하게 돈이 필요한데 혹시.."  
> B : "...."  
> B : "아 못 봤어 혹시 지금도 필요해?"  

4. 바람직하지 않은 문장(오타, 욕설이 많은 문장)  
> A : "쒸프트까 안빠찐따::"  
> A : 욕설로 이루어진 문장 등  

------------------------  

위의 유형들은 모두 노이즈이지만 좀 더 대표적이고 근본적인 유형의 노이즈가 존재한다.  


### 1. 문장부호  
> input -> "uhm, of course"  
> output -> ["uhm,","of","course"]

사람은 uhm,이 uhm과 ,이 한 단어가 아니라는 것을 알지만 컴퓨터와 같은 기계는 이를 인식할 수 없다.  
이는 노이즈로써 처리해주어야 할 것이다.  

바람직하게 단어가 나눠진다면 아래와 같이 나눠져야 할 것 이다.  

>"uhm"  
>","  
>"of"  
>"course"  

### 2. 대소문자  
> Banana와 banana를 다른 단어라고 인식하는 문제  

이는 어렵지 않게 처리해줄 수 있다. 모든 문자를 소문자나 대문자로 만들어준다면 같은 단어로 인식할 수 있을 것이다. (Python에서 내장함수인 lower와 upper를 통하면 간단하다!)  
  
### 3. 특수문자  
>input ->he is ten-year-old boy  
>output -> ["he","is","ten-year-old","boy"]  

특수문자까지 한 단어로 인식하는 이 유형은 정규표현식으로 특수문자를 제거하는 방법으로 해결할 수 있다. 정규표현식은 다른 포스트에서 간단하게 정리할 예정이다.  


