---
title: "[Spring] Spring Security Architecture에 대한 정리"
excerpt: "그리고 FilterChaninProxy"

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
# Spring Security Architecture

## 거시적 관점

웹 요청을 가로챈 후 사용자를 인증하고, 인증된 사용자가 적절한 권한을 지니고 있는지 확인

Spring Security Architecture는 크게 아래 두 핵심 모듈로 이뤄져있다.

- `Authentication Manager`
    - 사용자의 인증과 관련된 처리를 담당
- `Access Decision Manager`
    - 사용자가 적절한 권한을 가지고 있는지 확인

## 미시적 관점

서버로 들어오는 모든 요청은 서블릿을 거쳐 들어오게 된다.

요청은 서블릿으로 전달되기 전에 서블릿 필터(Delegating Filter Proxy Filter)를 거치게된다.

- 서블릿 필터
    
    웹 요청을 가로채 전처리, 후처리를 수행하거나 요청 자체를 리다이렉트 한다.
    


<p align = "center"><img style = "width : auto; height : auto;" alt = "secArchitecture.png" src = "../../assets/images/spring/secArchitecture.png"></p> 


<p align = "center"><img style = "width : auto; height : auto;" alt = "secArchitecture1.png" src = "../../assets/images/spring/secArchitecture1.png"></p> 
<p align =  "center" style = "font-size : 10px; color : gray;">
 요청을 받아 보안 처리를 하는 과정.
 </p>

1. 웹 요청을 수신한 서블릿 컨테이너는 받은 요청을 먼저 DelegatingFilterProxy로 전달한다. 
2. 요청을 받은 DelegatingFilterProxy는 실제 보안 처리를 수행할 FilterChainProxy을 지정한다.

FilterChainProxy를 구성하고 있는 필터의 수는 적지 않다.  

<details>
<summary>이렇게나 많다😨</summary>
<div markdown="1">    
 - [ForceEagerSessionCreationFilter](https://docs.spring.io/spring-security/reference/servlet/authentication/session-management.html#session-mgmt-force-session-creation)
 - ChannelProcessingFilter
 - WebAsyncManagerIntegrationFilter
 - SecurityContextPersistenceFilter
 - HeaderWriterFilter
 - CorsFilter
 - CsrfFilter
 - LogoutFilter
 - OAuth2AuthorizationRequestRedirectFilter
 - Saml2WebSsoAuthenticationRequestFilter
 - X509AuthenticationFilter
 - AbstractPreAuthenticatedProcessingFilter
 - CasAuthenticationFilter
 - OAuth2LoginAuthenticationFilter
 - Saml2WebSsoAuthenticationFilter
 - [UsernamePasswordAuthenticationFilter](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/form.html#servlet-authentication-usernamepasswordauthenticationfilter)
 - OpenIDAuthenticationFilter
 - DefaultLoginPageGeneratingFilter
 - DefaultLogoutPageGeneratingFilter
 - ConcurrentSessionFilter
 - [DigestAuthenticationFilter](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/digest.html#servlet-authentication-digest)
 - BearerTokenAuthenticationFilter
 - [BasicAuthenticationFilter](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/basic.html#servlet-authentication-basic)
 - RequestCacheAwareFilter
 - SecurityContextHolderAwareRequestFilter
 - JaasApiIntegrationFilter
 - RememberMeAuthenticationFilter
 - AnonymousAuthenticationFilter
 - OAuth2AuthorizationCodeGrantFilter
 - SessionManagementFilter
 - [ExceptionTranslationFilter](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-exceptiontranslationfilter)
 - [FilterSecurityInterceptor](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-requests.html#servlet-authorization-filtersecurityinterceptor)
 - SwitchUserFilter
 - [Architecture](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-security-filters)
</div>    
</details>  
<br/>

이 중 Spring Security Docs([https://docs.spring.io/spring-security/site/docs/3.0.x/reference/security-filter-chain.html](https://docs.spring.io/spring-security/site/docs/3.0.x/reference/security-filter-chain.html))에서 언급하는 주요한 필터는 다음과 같다.

필터링은 필터 이름에 적혀있는 숫자 순서대로 수행되며 이 순서는 **매우 중요**하다. 

### 주요 필터

1. <span class = "o">ChannelProcessingFilter</span>
    - 다른 프로토콜로 리다이렉트 해야할 가능성이 있기 때문에 첫번째.
    - 웹 요청이 어떤 프로토콜로 (http 또는 https) 전달되어야 하는지 처리
2. <span class = "o">SecurityContextPersistenceFilter</span>
    - 요청의 시작 부분에서 `Security Context`가 `SecurityContextHolder`에 셋업되어야 하고 요청이 끝날 때 `SecurityContext`의 변화가 `HttpSession`에  복사될 수 있게 하기 위해(다음 웹 요청에 사용할 준비를 하기 위해) 두번째.
    - `SecurityContextRepository`를 통해 `SecurityContext`를 Load/Save 처리
3. <span class = "o">LogoutFilter</span>
    
     로그아웃 URL로 요청을 감시하여 매칭되는 요청이 있으면 해당 사용자를 로그아웃 시킴
    
4. <span class = "o">UsernamePasswordAuthenticationFilter</span>
    
    ID/비밀번호 기반 Form 인증 요청 URL(기본값: /login) 을 감시하여 사용자를 인증함
    
5. <span class = "o">DefaultLoginPageGeneratingFilter</span>
    
    로그인을 수행하는데 필요한 HTML을 생성함
    
6. <span class = "o">RequestCacheAwareFilter</span>
    
    로그인 성공 이후 인증 요청에 의해 가로채어진 사용자의 원래 요청으로 이동하기 위해 사용됨
    
7. <span class = "o">SecurityContextHolderAwareRequestFilter</span>
    
    서블릿 3 API 지원을 위해 `HttpServletRequest`를 `HttpServletRequestWrapper` 하위 클래스로 감쌈
    
8. <span class = "o">RemeberMeAuthenticationFilter</span>
    
    요청의 일부로 remeber-me 쿠키 제공 여부를 확인하고, 쿠키가 있으면 사용자 인증을 시도
    
9. <span class = "o">ExceptionTranslationFilter</span>
    
    해당  인증 필터에 도달할때까지 사용자가 아직 인증되지 않았다면, 익명 사용자로 처리하도록 함
    
10. <span class = "o">FilterSecurityInterceptor</span>
    
    접근 권한 확인을 위해 요청을 AccessDecisionManager로 위임