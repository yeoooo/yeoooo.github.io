---
title: "Spectrogram"
excerpt: "Spectrogram 정리"

categories:
    NLP
tags:
    언어학
    음성처리
    딥러닝
toc: true
use_math: true
comments : true
---
## 개요  
전공으로 배웠던 언어학이 혹여나 조금 더 쉽지 않을까 하고 선택한 발표주제 스펙트로그램. 관련 정보를 찾아보자마자 실수였던걸 깨닳았다. 그래도 알아둬야 하는 정보라고 생각했기 때문에 전에 보았던 강의 노트를 다시 보고 희미하게나마 남아있는 전공 지식을 되짚어 보기로했다.  

## Spectrgram(스펙트로그램)

위키피디아에서는 스펙트로그램을 다음과 같이 말한다.  
>``  
스펙트로그램(Spectrogram)은 소리나 파동을 시각화하여 파악하기 위한 도구로, 파형(waveform)과 스펙트럼(spectrum)의 특징이 조합되어 있다.  
``  
  
위와 같이 스펙트로그램은 한마디로 소리를 시각화한 그래프와 같다.  
여기에 NLP를 함께 생각해보면 __음성__ 을 시각화한 것이 스펙트로그램이라고 생각하면 되겠다.  

아래는 "nineteenth century"를 발음한 스펙트로그램이다.  
<img alt = "Spectrogram.jpg" src = "../../assets/images/spectrogram/Spectrogram.png">  
  
그렇다면 스펙트로그램은 어떤 데이터를 그래프에 그릴까?  
위의 사진을 보면 x축으로는 Time, y축으로는 Frequency, 색에 따른 진폭이 적혀있다. 시간에 따른 주파수를 시각화 했다는 것이다. 

시간에 따른 주파수를 표시한 그래프, 스펙트로그램에서는 어떤 정보를 얻을 수 있을까?  

스펙트로그램에서는 기식, 파열, 진동, VOT(VoiceOnsetTime),포먼트 등을 파악할 수 있다.  

이는 무엇을 의미하냐면, 스펙트로 그램을 통해서 어떤 것을 발음했는지 알게 된다는 의미이다.  

그 때 교수님께서는 스펙트로그램을 설명하시면서 스펙트로그램 분석을 잘 하는 학자들은 모양을 보고 완벽하게는 아니더라도 어떤 음인지 파악할 수 있다고 하셨다.(파열음, 마찰음, 파쇄음, ... )  

이 스펙트로그램을 그리기 위해서는 원래 존재하는 파형에 시간에 따른 축으로 변화시키기 위해 Short-Time-Fourier-Transform을 거쳐주면 된다.(STFT는 학부 급에서는 알 필요없고 쓰는 법 정도만 알면된다고 하셨다.. 따로 다뤄야할 만큼 복잡한 내용이기도 하다.)


## Formant(포먼트)  
사람의 목소리에서 나오는 스펙트럼을 보면  

<img alt = "spectrum.jpg" src = "../../assets/images/spectrogram/spectrum.jpg">
<center><span style = "font-size : 12px "> 스펙트럼(x축이 주파수, y축이 진폭이다.)</span></center>  
<br>
  

위 이미지는 어떤 것을 발음 하는게 아니기 때문에 관련은 없지만, 특정 부분에 peak가 존재한다. 이 peak에 해당하는 주파수가 바로 포먼트이고, 곧 성도의 공명주파수라고 한다.  


이 포먼트들의 구조는 __말소리를 인식하거나, 구분 하는데에 중요한 역할을 한다.__  
포먼트는 사람이 가지고 있는 성도의 구조, 혀, 입술 등 조음기관의 생김새에 따라 주파수가 달라지고 이에따라 음색이 달라지기 때문이다.  (어떤 소리든 간에 포먼트를 지니고 있는데, 성인 남성은 평균 120Hz, 여성은 225Hz 정도)

이 포먼트를 통해서 음성인식도, 음성을 만들어 내는 것도 가능하지 않을까? 하는 생각이 들었다.  

(여담이지만, 수업을 들을 당시에 포먼트를 통해서 어느 정도의 포먼트가 듣기 좋은 목소리 일거라고 말씀하셨었다. 어느 정도였는지는 기억나지 않는다 ㅎㅎ)  

## 참고  
[https://ko.wikipedia.org/wiki/%EC%8A%A4%ED%8E%99%ED%8A%B8%EB%A1%9C%EA%B7%B8%EB%9E%A8](https://ko.wikipedia.org/wiki/%EC%8A%A4%ED%8E%99%ED%8A%B8%EB%A1%9C%EA%B7%B8%EB%9E%A8)  

[http://dongascience.donga.com/news.php?idx=22378](http://dongascience.donga.com/news.php?idx=22378)  

[https://tech.kakaoenterprise.com/66](https://tech.kakaoenterprise.com/66)
