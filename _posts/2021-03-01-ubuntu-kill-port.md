---
title: "Linux(Ubuntu) 포트 닫기"
excerpt: "우분투 간단 정리"

categories:
    Linux
tags:
   리눅스
   우분투
   로컬호스트
   포트
toc: true
comments: true
---
## 개요  
리눅스 터미널에서는 특정 포트를 확인하고 닫을 수 있다.  lms서버와 jupyter를 함께 열려고 했는데, jupyter를 먼저 열고 lms서버에 연결 하려고 하니 lms서버에 연결이 안되는 문제가 발생했고 확인해보니 jupyter에 연결된 8888포트가 닫히지 않아 발생하는 문제였다. 이를 위해서 특정 포트를 닫기 위해 검색하여 해결했다.  (물론 리부팅을 하는 방법으로 해결할 수 있지만, 매번 그렇게 하기는 너무 번거로웠다.)

## 패키지 설치  
포트를 확인하고 닫기 위해서는 먼저 패키지가 설치 되어 있어야 한다. 설치 되어있지 않다면 아래 코드를 통해서 설치하도록 한다.  

> sudo apt install net-tools  


## 포트 확인  
>netstat -nap \| grep port  

<strong><span style ="color : orange">port</span></strong>에 직접적으로 port가 아닌 닫고 싶은 포트(<strong><span style ="color : orange">숫자</span></strong>)를 입력해줘야 한다.  

## 포트 사용 프로그램 닫기(죽이기)  
> fuser -k -n tcp port  

위의 주의사항과 같이 port에 숫자를 넣어줘야 한다.  

<span style = "font-size: 12px">출처 : [https://blog.naver.com/kletgdgo/90126796684](https://blog.naver.com/kletgdgo/90126796684)</span>
