---
title: "[Spring] Spring Security 내부 구조 4"
excerpt: "HeaderWriterFilter에 대한 정리"

categories:
    Spring
tags:
    프레임워크
    백엔드
    SpringBoot
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


## HeaderWriterFilter

`HeaderWriterFilter`는 커스터마이징이 크게 필요없는 필터로 Http Response Header에 보안 관련 헤더를 추가한다.

관련 이슈에 대해 기본적인 방어 기능만 제공하며 브라우저마다 다르게 동작할 수 있기 때문에 유의해야 한다.

HeaderWriterFilter는 headerWriter들을 List형태로 가지고 있으며 그 내부에는 다음과 같은 HeaderWriter 객체들이 존재한다.

- `XContentTypeOptionsHeaderWriter`
    - `MIME Sniffing` 공격에 대한 방어
    - 지정된 MIME 형식 이외의 다른 용도로 사용하고자 하는 것을 차단한다.
    
### MIME Sniffing

MIME 타입이 없을 때, 혹은 클라이언트가 타입이 잘못 설정되었다고 판단한 어떤 다른 경우 브라우저들은 MIME 스니핑을 시도할 수 있는데, 이는 리소스를 훑어보고 정확한 MIME 타입(Request Content Type)을 추측 해내는 것이다. 이는 XSS 공격에 악용될 여지가 있다.
    
- MIME는 ASCII가 아닌 문자 인코딩을 이용해 영어가 아닌 다른 언어로 된 전자 우편을 보낼 수 있는 방식을 정의한다.  
  
- `XXssProtectionHeaderWriter`
    - 브라우저에 내장된 XSS 필터를 활성화 한다.
        
### XSS

웹 상에서 가장 기초적인 취약점 공격 방법의 일종으로 악의적인 사용자가 공격하려는 사이트에 스크립트를 넣는 기법을 의미한다.
        
- 일반적으로 브라우저에 XSS 공격을 방어하기 위한 필터링 기능이 내장되어 있다.
- 물론 해당 필터로 XSS 공격을 완벽하게 방어하지는 못하나 XSS 공격의 보호에 많은 도움이 된다.
- `CacheControlHeadersWriter`
    - 캐시를 사용하지 않도록 설정할 수 있다.
    - 브라우저 캐시 설정에 따라 사용자가 인증 후 방문한 페이지를 로그아웃한 후 캐시 된 페이지를 악의적인 사용자가 볼 가능성이 있다.  
  
- `XFrameOptionsHeaderWriter`
    - Clickjacking 공격 방어
        
        웹 사용자가 자신이 클릭하고 있다고 인지하는 것과 다른 어떤 것을 클릭하게 속이는 악의적인 기법
    
    보통 사용자의 인식 없이 실행될 수 있는 임베디드 코드나 스크립의 형태를 가지고 있다.
    
- `HstsHeaderWriter`
    - Http 대신 Https만을 사용해 통신해야함을 브라우저에 알리는 역할을 한다.

## CsrfFilter

### Csrf(Cross-site request forgery)

- 사용자가 자신의 의지와는 무관하게 공격자가 의도한 행위를 특정 웹사이트에 요청하게 하는 공격
- <span class = "o">위조 요청을 전송하는 서비스에 사용자가 로그인</span>한 상태로 <span class = "o">해커가 만든 피싱 사이트에 접속한다는 조건</span>을 충족하면 사용자의 권한을 도용해 중요 기능을 실행하는 것이 가능해진다.
- Request의 referrer를 확인해 domain이 일치하는지 확인하는 Referrer 검증을 통해 방지할 수 있다.
- `CSRF Token`을 활용해 방지할 수 있다.
    
### Csrf Token 활용 검증

1. 사용자의 세션에 임의의 난수 값을 저장한 후 사용자의 요청마다 해당 난수 값을 포함 시켜 전송한다.
2. 리소르를 변경해야하는 요청(PUT,POST,DELETE)을 받을 때마다 사용자의 세션에 저장된 토큰 값과 요청 파라미터에 전달되는 토큰 값이 일치하는지 검증한다.
   - 브라우저가 아닌 클라이언트에서 사용하는 서비스의 경우 CSRF보호를 비활성화할 수 있다.

## BasicAuthenticationFilter

> BasicAuthentication을 처리하기 위한 필터

### Basic Authentication
Http 요청 헤더에 username과 password를 base64로 인코딩해 포함하는 것으로
username과 password를 그대로 노출하기 때문에 <span class = "o">HTTPS 프로토콜과 함께 사용</span> 해야 한다.

## WebAsyncManagerIntegrationFilter

WebAsyncManagerIntegrationFilter는 ThreadLocal 변수를 이용하는 SecurityContext는 다른 스레드에서 참조할 수 없었던 것과 다르게 MVC Async Request가 처리될 때 <span class = "o">쓰레드간 SecurityContext를 공유할 수 있게 한다</span>.

이를 사용하기 위해서는 ThreadLocal이 아닌 부모 스레드가 만든 자식스레드가 부모 스레드에 대한 참조를 할 수 있게 해주는 `INHERITABLE ThreadLocal` 모드를 사용해야하는데, 이는 그다지 권장할만한 방법이 아니다.

그 이유는 아래와 같다.

    1. Pooling 처리된 TaskExecutor와 함께 사용할 경우 ThreadLocal의 clear가 제대로 처리되지 않아 문제가 될 수 있다.
    2. Pooling 되지 않는 taskExecutor(ex. SimpleContextHolder)와 함께 사용해야 한다.

### DelegatingSecurityContextAsyncTaskExecutor

위 문제를 해결하면서 Security Context를 공유하기 위해 DelegatingSecurityContextAsyncTaskExecutor를 사용할 수 있다.

해당 Executor는 내부적으로 Runnable을 delegatingSecurityContextRunnable 타입으로 Wrapping 처리하고 DelegatingSecurityContextRunnable 객체 생성자에서 SecurityContextHolder.getContext() 메서드를 호출해 SecurityContext 참조를 획득한다.

## Reference.

MIME Sniffing    
[https://developer.mozilla.org/ko/docs/Web/HTTP/Basics_of_HTTP/MIME_types](https://developer.mozilla.org/ko/docs/Web/HTTP/Basics_of_HTTP/MIME_types)