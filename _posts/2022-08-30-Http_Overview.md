---
title: "[Web]Http 웹 기술 간단 정리"
excerpt: "SW Academy day 21"

categories:
    Web
tags:
    백엔드
toc: true
comments: true
use_math: true
---

<style type = 'text/css'>
    .o{
    font-weight: bold;
    color:orange;
    }
</style>

# 웹 기술

스프링이 어떻게 웹 기술을 지원하는지 살펴보자 

> *WWW(World Wide Web) 인터넷에 연결된 컴퓨터를 통해 사람들이 정보를 공유할 수 있는 전 세계적인 정보 공간*
> -Wiki

웹의 구성요소

<p align = "center"><img style = "width : 500px; height : 300px;" alt = "web_component.png" src = "../../assets/images/web/web_component.png"></p> 

---

## URI(Uniform Resource Identifier)

URI 구조는 일반적으로 <span class = "o">URI Scheme</span>, <span class = "o">호스트 명</span>, <span class = "o">패스</span>로 이루어진다.

https://www.shopping.naver.com/home/p/index.naver 과 같은 예시에서 <span class = "o">세 개의 구성요소</span>는 아래와 같이 분류된다.

URI 스키마 - `https`

호스트명 - `www.shopping.naver.com`

패스 - `/home/p/index.naver` 

또한 사용자의 인증 정보를 함께 URI에 넣어줄 수 있다.

http://yeoooo:pass@cnu.academy.com:8080/search?q=test&debug=true

위의 예시에서는 아래와 같은 구성요소로 이루어져 있다.

URI Schema - `http`

사용자 - `yeoooo:pass`

호스트명 - `cnu.academy.com`

포트 - `8080`

패스 - `/search`

쿼리 파라미터 - `q=test&debug=true`

---

URI의 패스는 디렉토리와 같이 표기되는데, <span class = "o">현재 자신(파일)의 위치로 부터 특정 파일까지의 디렉토리를 표기</span> 하는 `상대 경로`, <span class = "o"> root 폴더로부터  특정 파일의 디렉토리를 표기</span>하는 `절대 경로`로 표현할 수 있다.

이 때 절대 경로 보다는 상대 경로를 지향해야 하는데 root 폴더로 부터 접근 하는 경우 다른 리소스를 사용할 확률이 커지기 때문이다.

또한 <span class = "o">URI에서는 ASCII 문자만 이용 가능</span>하다.

ASCII 문자에서는 알파벳(`A-Z`, `a-z`) ,숫자(`0-9`), 기호(`-`, `.`, `:`, `~`, `@`, `!`, `&`, `'`, `()`) 만을 사용해야한다.

---

## HTTP(Hyper Text Transfer Protocol)

HTTP는 다음과 같은 의의가 있다.

> 하이퍼 텍스트를 전송하기 위해 만들어진 통신 규약
> 

URI를 통해 식별한 <span class = "o">리소스를 접근하고 조작</span>하기 위해서 HTTP를 사용한다.

HTTP는 Client Server Protocol이기도 한데, 어떤 요청을 보내고 그에 대한 응답을 보내기 위한 통신 규약이라는 의미이다.

즉, HTTP에서는 요청을 보내게 되면 꼭 응답을 보내야하고 이러한 과정은 동기적으로 이뤄진다.

반면 UDP를 이용했을 경우에는 Fire and Forget 패턴, 비동기의 형식으로 요청이나 응답을 보내게 된다.(Client가 응답을 기다리지 않는다.)

**HTTP의 특징** 

- 동기형 프로토콜
- TCP/IP 기반
- 요청/응답형 프로토콜
- 무상태성

## 이외의 프로토콜

### TCP(Transmission Control Protocol)

> *TCP는 근거리 통신망이나 인트라넷, 인터넷에 연결된 컴퓨터에서 실행되는 프로그램 간에 일련의 옥텟을 안정적으로, 순서대로, 에러없이 교환할 수 있게한다. HTTP는 TCP 기반으로 만들어진 프로토콜이다.*


**TCP의 특징**

- 연결지향적(클라이언트와 서버의 연결)
- 3 way handshaking, 4 way handshaking(서로 서버가 통신할 준비가 되어 있는지, 서로 연결해 데이터를 주고 받을 수 있는지 확인하는 절차 4 way handshaking을 통해 그 연결을 해제한다. 하지만 이러한 과정으로 인해 UDP 보다 속도가 떨어진다.)
- 흐름, 혼잡 제어
- 전이중(Full-Duplex), 점대점(Point to Point) qkdtlr

---

### UDP(User Datagram Protocol)

> *UDP의 전송 방식은 너무 단순해 서비스의 신뢰성이 낮고 순서를 보장하지 않으며 데이터를 통보 없이 누락하기 때문에 오류의 검사와 수정이 필요 없는 애플리케이션에서 수행될 것을 가정한다. 따라서 신뢰성보다는 연속성이 중요한 DNS, IPTV, VolP, 온라인 게임, Streaming 등에서 이용된다.*

**UDP의 특징**

- 비연결형 서비스로 데이터그램 방식을 제공
- 정보를 주고 받을 때 비동기형식으로 동작
- UDP헤더의 CheckSum 필드를 통해 최소한의 오류만을 검출
- 낮은 신뢰성
- TCP 보다 빠른 속도

<p align =  "center" style = "font-size : 10px">
<a href = "https://mangkyu.tistory.com/15">
출처 : https://mangkyu.tistory.com/15
</a>
</p>
---

웹서버는 클라이언트가 HTTP를 통해 URL를 호스트 서버에 요청하고, 요청을 처리한 호스트 서버가 HTTP를 통해 HTML 문서를 클라이언트에 응답하는 프로세스를 거친다.

<p align = "center"><img style = "width : 500px; height : 300px;" alt = "web_req&resp.png" src = "../../assets/images/web/web_req&resp.png"></p> 

<p align =  "center" style = "font-size : 10px">
<a href = "https://velog.io/@sujeong/2-웹의-동작-HTTP-프로토콜-이해">
출처 : https://velog.io/@sujeong/2-웹의-동작-HTTP-프로토콜-이해
</a>
</p>

---

### HTTP의 요청

HTTP의 요청은 다음과 같이 구성되어 있다.

<p align = "center"><img style = "width : 500px; height : 300px;" alt = "web_httpreq.png" src = "../../assets/images/web/web_httpreq.png"></p> 

- (start-line)메서드: 클라이언트가 서버에 요청할 동작(POST, PUT, GET, DELETE, HEAD, OPTIONS, TRACE, CONNECT, PATCH)
- (start-line)프로토콜: 사용되는 프로토콜, 버전
- 헤더: 클라이언트 자체에 대한 자세한 정보
- empty-line : 헤더와 본문을 구별
- 바디: 해당 요청을 전송하는 데이터

### Http 메서드  


| 메서드 | 의미 |
| --- | --- |
| GET | 리소스 요청 |
| POST | 서브 리소스의 작성, 리소스 데이터 추가 등 |
| PUT | 리소스 갱신, 리소스 작성 |
| DELETE | 리소스 삭제 |
| HEAD | 리소스 헤더 취득(본문 포함 X) |
| OPTIONS | 리소스가 지원하는 메서드 취득 |
| TRACE | 목적 리소스의 경로를 따라 loop-back 테스트 |
| CONNECT | 리소스로 식별되는 서버로의 연결 |
| PATCH | 리소스의 한 부분을 수정 |  
{: .table-center}

<p align =  "center" style = "font-size : 10px">
<a href = "https://developer.mozilla.org/ko/docs/Web/HTTP/Methods">
출처 : https://developer.mozilla.org/ko/docs/Web/HTTP/Methods
</a>
</p>

---

### HTTP의 응답

http의 응답은 다음과 같이 구성되어 있다.

<p align = "center"><img style = "width : 500px; height : 300px;" alt = "web_httpresp.png" src = "../../assets/images/web/web_httpresp.png"></p> 

- (start-line)프로토콜: 사용 프로토콜과 그 버전
- (start-line)상태코드: 요청에 대한 응답 상태
- (start-line)상태메시지: 상태코드와 함께 전달되는 데이터

### 상태코드

상태코드는 100번~505번 대 까지 존재하며 주로 개략적인 내용은 다음과 같다.

| 상태코드 | 내용 |
| --- | --- |
| 100-109 | 메세지 정보 |
| 200-206 | 요청 성공 |
| 300-305 | Rediretion |
| 400-415 | 클라이언트 에러 |
| 500-505 | 서버 에러 |

상태코드는 [이곳](https://developer.mozilla.org/ko/docs/Web/HTTP/Status)에서 자세히 확인할 수 있다.

**요청, 응답 모두 body(본문)의 내용은 생략될 수 있다.**

<p align =  "center" style = "font-size : 10px">
<a href = "https://velog.io/@bining/HTTP-메세지-요청request과-응답response">
출처 : https://velog.io/@bining/HTTP-메세지-요청request과-응답response
</a>
</p>


