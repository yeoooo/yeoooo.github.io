---
title: "[Spring] Spring Security란?"
excerpt: "Spring Security과 사용 방법에 대해 간단히 알아보자"

categories:
    Spring
tags:
    SpringBoot
    백엔드
    Security
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

# Spring Security

> Spring기반의 어플리케이션을 보호할 수 있는 표준 기술. *Spring Security는* *사용자화 할 수 있는 강력한 접근 제어, 인증 프레임 워크이며 자바 어플리케이션에 권한 부여와 인증을 제공하는 것에  집중한다. -Spring Security Docs*
> 

Spring Security는 권한과 보안에 관한 스프링 프레임워크의 프로젝트이다.

Spring Security에 대해서 먼저 알아가기 전에, 어떤 것들이 웹 어플리케이션의 <span class = "o">주요 보안 위협 요소</span>에 대해 알아본다.

---

### 웹 어플리케이션의 주요 보안 위협 요소

#### 인증(Authentication)의 절차 미비

- <span class = "o">인가(Authorization)</span>과 함께 보안 관련 핵심 개념 중 하나
- 사용자의 신원을 확인하는 과정(로그인 기능과 관련)
- 어플리케이션 보안의 영역
    - <span class = "o">인증 영역</span>
        
        사용자의 개인 정보를 확인, 수정 가능한 영역
        
    - <span class = "o">익명 영역</span>
        
        사용자의 민감 정보를 노출하지 않아야 하고 시스템의 상태를 변경하거나 데이터를 관리할 수 있는 기능을 제공하지 않아야하는 영역
        
    

#### 인가(Authorization) 처리의 미비

- 사용자에 따라 <span class = "o">적절한 권한과 그에 맞는 특정 기능 수행</span>이 가능하고 데이터 접근이 허용되어야 한다.
- 적절치 못한 권한과 그에 따른 기능 수행은 개인 정보, 민감 데이터 유츨 등 과 같은 보안사고의 발생 가능성이 높아진다.

#### 크레덴셜(Credential) 보안

- <span class = "o">민감정보</span>(연락처, 결제 정보, 비밀번호 등)는 항상 <span class = "o">최우선적으로 보호</span>되어야 한다.
- 따라서 민감정보는 일반 텍스트로 저장되지 않고 <span class = "o">암호화</span>되어야 한다.

#### 전송 레이어 보안

- <span class = "o">https 프로토콜(SSL 보호)의 적용</span>이 필요
    - (Secure Sockets Layer)SSL
        
        디지털 인증서로 불린다.
        
        브라우저와 서버 사이의 암호화된 연결을 수립하는데 사용되는 표준 기술
        
    - (Transport Layer Secure)TLS
        
        SSL의 상위 버전
        
- TLS/SSL 이 적용된 웹은 https 프로토콜이 적용된다.


## Spring Security가 제공하는 기능

- Spring 기반 어플리케이션에 쉽게 적용 가능하며 적은 노력으로 각 상황에 보안을 적용 가능
- 사용자 인증 및 인가 처리
- 비교적 쉬운 사용자화
- 다양한 확장 기능과 자연스러운 추상화(세션 클러스터 기능, Oauth 등)

## 사용 방법

먼저 Spring Security를 사용하기 위해서는 그에 관련한 Configuration 클래스가 필요하다.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

...

}
```

Configuration 클래스는 `WebSecurityConfigurerAdapter`를  상속 받아야 하며 `@EnableWebSecurity`

를 적용해야한다.

`WebSecurityConfigurerAdapter` 를 상속받게 되면  `configure()` 메서드를 오버라이드할 수 있게 된다.

Spring Security의 설정은 해당 메서드 안에서 일어나게 된다.

```java
@Override
    protected void configure(HttpSecurity http) throws Exception {
        /*
		 csrf 토큰 활성화시 사용
		 쿠키를 생성할 때 HttpOnly 태그를 사용하면 클라이언트 스크립트가 보호된 쿠키에 액세스하는 위험을 줄일 수 있으므로 쿠키의 보안을 강화할 수 있다.
		*/
        //http.csrf().csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())

        http.csrf().disable()	// csrf 토큰을 비활성화
                .authorizeRequests() // 요청 URL에 따라 접근 권한을 설정
                .antMatchers("/boards/write/**","boards/free/post").auth
...
}
```

하지만 이전에  `configure()` 를 오버라이드 하려고 하면 3 종류의 메서드가 발견된다.

<p align = "center"><img style = "width : auto; height : auto;" alt = "security1.png" src = "../../assets/images/spring/security1.png"></p> 

이는 각각 인자로 `AuthenticationManagerBuilder`, `HttpSecurity`, `WebSecurity` 타입의 변수를 받는다.

세 타입을 인자로 받는 configure 메서드의 각 설명은 아래와 같다.

- `AuthenticationManagerBuilder`
    
    Spring Security를 이용해 만든 웹 어플리케이션에서 사용할 기본 유저를 생성할 수 있다. 
    
    - auth.inMemoryAuthentication
    - auth.JdbcAuthentication
- `HttpSecurity`
    
    <span class = "o">세부적인 웹 보안기능 설정</span>을 처리할 수 있는 API를 제공
    
- `WebSecurity`
    
    필터 체인 관련 <span class = "o">전역 설정을 처리</span>할 수 있는 API 제공
    
    `ignoring()`을 통해 필터 체인을 적용하고 싶지 않은 리소스에 대해 설정할 수 있다.
    
    - `ignoring()`
        
        일반적으로 정적 리소스(html, css, js 등 Spring Security를 통해 처리하지 않아도 되는 자원)을 AntPattern의 형식으로 설정할 수 있다.
        
        불필요한 서버 자원 낭비를 방지할 수 있음.
        

### Filter chain?

---

Spring Security에서는 보안과 관련된 필터를 체인처럼 엮어서 놓는다. 모든 요청은 필터 체인을 반드시 거쳐야 서블릿에 도달하게 된다.

이 과정에서 `DelegatingFilterProxy` 와 `FilterChainProxy` 를 사용하게 된다. 

- `DelegatingFilterProxy`
    
    서블릿 컨테이너와 <span class = "o">스프링 컨테이너(어플리케이션 컨텍스트) 사이의 링크를 제공</span>하는 ServletFilter이다.
    
    스프링 시큐리티는 DelegatingFilterProxy 필터를 만들어, 메인 필터 체인에 끼워넣고, 그 아래 다시 SecurityFilterChain 그룹을 등록한다. <span class = "o">각 SecurityFilterChain 은 url 패턴에 따라 적용</span>시킬 수 있다.
    
- `FilterChainProxy`
    
    springSecurityFilterChain의 이름으로 생성되는 필터 빈이며 <span class = "o">DelegatingFilterProxy로부터 요청을 위임 받고 실체로 보안을 처리</span>한다.
    
    스프링 시큐리티 초기화 시 생성되는 필터들을 관리하고 제어한다.
    

### Thymleaf, Spring Security


Thymeleaf는 Spring Security와 관련된 기능을 제공한다.

주로 <span class = "o">인증 사용자의 정보를 가져오는데 사용</span>할 수 있다.

Depedency는 thymeleaf와 다르게 추가적으로 설정해주어야 한다.

```yaml
<dependency>
      <groupId>org.thymeleaf.extras</groupId>
      <artifactId>thymeleaf-extras-springsecurity5</artifactId>
</dependency>
```
<p align =  "center" style = "font-size : 10px; color : gray;">
Maven의 경우
</p>

```yaml

implementation 'org.thymeleaf.extras:thymeleaf-extras-springsecurity5'
```  

<p align =  "center" style = "font-size : 10px; color : gray;">
Gradle의 경우
</p>

의존성을 추가해준 후 사용하기 위해서 기존에 사용해왔던 thymleaf 사용법에 추가적으로 xmlns 부분을 추가해주어야 한다.

```java
<html lang="ko" xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
```

위 내용을 추가해주면 사용법은 기존 thymeleaf 사용법과 크게 다르지 않다.

```html
<div th:text="${#authentication.name}">
  The value of the "name" property of the authentication object should appear here.
</div>
```

## Reference. 

[https://www.digicert.com/kr/what-is-ssl-tls-and-https](https://www.digicert.com/kr/what-is-ssl-tls-and-https)

[https://devfunny.tistory.com/615](https://devfunny.tistory.com/615)

[https://docs.spring.io/spring-security/site/docs/3.0.x/reference/security-filter-chain.html](https://docs.spring.io/spring-security/site/docs/3.0.x/reference/security-filter-chain.html)

[https://uchupura.tistory.com/24](https://uchupura.tistory.com/24)