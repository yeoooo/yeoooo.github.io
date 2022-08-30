---
title: "[Web]Web Server와 WAS, Servlet 간단 정리"
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

# Web Server, WAS(Web Application Server)

## Web Server

웹서버는 Http나 Http에서 지원하는 프로토콜 사용하는 서버를 의미한다.

<span class = "o">정적 컨텐츠</span>만을 처리한다.

## WAS

Web Server와 다르게 정적 컨텐츠 뿐 아니라 <span class = "o">동적 컨텐츠</span> 또한 처리한다.

WAS는 WebServer(WAS의 앞단에 위치하게 된다.)와 Web Container를 포함하며 Web Container는 대게 DB와 연결된다.
<p align = "center"><img style = "width : 500px; height : 300px;" alt = "web_WAS.png" src = "../../assets/images/web/web_WAS.png"></p> 

클라이언트로부터 요청을 받으면 WebServer가 Web Container의 **Servlet**을 구동시킨다.

Web Container는 DB로부터 데이터를 받아 템플릿 엔진에 뿌려준다.
---

## Servlet

위키에서 언급하는 Jakrta Servlet의 의의는 다음과 같다.

> *http 요청에 대해 특정한 기능을 수행하고 html문서를 생성하는 등 응답 처리를 하는 서버 의 역할을 하는 자바 소프트웨어 컴포넌트*
> -Wiki

<span class = "o">Servlet은 WAS안에 존재</span>한다. 따라서 OServlet을 이용하기 위해서 WAS 또한 이용되어야 한다.

자바에서 Servlet은 <span class = "o">인터페이스 형태로 제공</span>되며 이를 구현하는 형식으로 사용된다.

Servlet을 통해 요청과 응답의 헤더, 메서드 등 파싱을 직접하지 않고 VIEW에 데이터를 전송하는 것 또한 Servlet에게 맡길 수 있다.

WAS를 통한 Http의 요청은 다음과 같이 이뤄진다.

<p align = "center"><img style = "width : 500px; height : 300px;" alt = "web_WASreq.png" src = "../../assets/images/web/web_WASreq.png"></p> 

1. 먼저 웹서버에서 클라이언트의 요청을 받는다.
2. 웹 서버는 프록시 웹 서버를 통해 WAS를 요청한다.
3. WAS에서는 내부적으로 여러 요청에 대한 처리를 위해 (멀티)스레드를 생성한다.
4. 스레드는 서블릿의 service 메서드를 요청한다.(**하나의 스레드가 하나의 서블릿을 생성하지 않는다.**)
5. service()는 http 메서드에 따른 메서드를 실행시킨다.
6. 응답을 처리한다.

서블릿에 대한 자세한 내용은 [이곳](https://learning.oreilly.com/library/view/head-first-servlets/9780596516680/ch04s01.html) 에 있다.

### 서블릿의 LifeCycle

서블릿은 init(), service(), destroy()의 생명주기를 가진다. 생명주기는 다음의 구상도와 같다.

<p align = "center"><img style = "width : 500px; height : 300px;" alt = "web_servletLifecycle.png" src = "../../assets/images/web/web_servletLifecycle.png"></p> 

1. 요청이 오면 Servlet 클래스를 불러와 요청에 대한 인스턴스를 생성한다.(첫 한번)
2. 서버는 init()을 호출해 Servlet을 초기화한다.(첫 한번)
3. service() 메소드를 호출해 servlet이 http 요청 메서등에 대한 브라우저의 요청을 처리한다.
4. 서버는 destroy()를 호출해 servlet을 제거한다.(마지막 한번)