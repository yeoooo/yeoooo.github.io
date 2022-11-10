---
title: "[Spring] Spring Security 내부 구조 -1"
excerpt: "SecurityContext와 내부 구조"

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


# Spring Security Internals

## 개요
인증, 인가 처리를 하는 SecurityContext는 SecurityContextHolder에 의해 관리된다. 이 SecurityContextHolder는 비동기적으로 스레드풀을 사용하지 않는 이상 서블릿은 thread per request 모델을 따라 <span class = "o">Thread Local</span>을 기본적으로 사용한다.  
먼저 Thred Per Request 모델과 Thread Local을 시작으로 내부 구조에 대해 파악하기로 해본다.

### Thread Per Request  
---

HTTP 요청의 Queue에 적재되면, `ThreadPool` 내의 특정 스레드가 Queue에서 요청을 가져와 처리한다.

이러한 처리는 처음부터 끝까지 동일한 스레드가 맡게되는데, 요청을 끝마친 스레드는 다시 스레드 풀에 반납되게 된다.

위의 말을 정리해서, <span class = "o">한 WAS안에서 처리할 수 있는 최대 HTTP 요청의 개수는 스레드 풀의 크기와 같다</span>고 할 수 있다. 

이렇게 생각해보았을 때 스레드를, 스레드 풀의 크기를 무작정 늘리면 성능이 늘어날 수 있다고 생각하기 쉽지만 Thread Context 스위칭에 의한 오버헤드가 스레드가 늘어난 만큼 커지기 때문에 성능이 선형적으로 증가하지는 않는다.

이러한 점을 고려해 최근 WebFlux와 같은 기술이 스레드의 갯수를 적게 유지하며 HTTP 요청을 동시에 처리할 수 있도록 지원하고 있다. 

### Thread Local  
---

스레드 범위 변수(스레드 로컬)은 동일 스레드 내에서는 언제든 스레드 로컬 변수에 접근할 수 있는 변수를 의미한다(<span class = "o">하나의 스레드에서 자원 공간을 공유하는 방식</span>).

이는 동일한 스레드 내에서 실행되는 컨트롤러, 서비스, 레포지토리 등 어떤 컴포넌트에서도 <span class = "o">명시적인 파라미터 전달 없이 스레드로컬 변수에 접근</span>할 수 있다는 것을 의미한다.

이러한 스레드 로컬을 스레드 풀과 함께 사용할 경우 풀에 반환되기 직전 스레드로컬 변수 값을 반드시 반환해야 하며 그렇지 않을 경우 스레드 로컬이 풀에 반환된 후, 다른 요청을 처리하기 위해 스레드 풀에서 스레드를 호출하게 되고, 이전 요청을 처리해 값이 남아있는 스레드를 참조해 잘못된 동작을 수행할 수 있다.

## Security Context

SecurityContext는 `principal` 이라고 불리는 최근 인증된 유저에 대한 세부 사항을 저장하며 Authenication 객체의 Wrapper 클래스이다. 만약 유저 이름이나 다른 유저 정보를 얻고 싶다면 SecurityContext를 먼저 얻어야 한다. 

## Security Context Holder

> *“Security Context를 담는 그릇”*
> 

Security ContextHolder는 Security Context에 대한 접근을 제공하는 헬퍼클래스이다. 이 클래스를 통해서 코드 어느 부분에서도 SecurityContext에 접근할 수 있다.

`SecruityContextHolder` 의 내부를 살펴보면 `SecurityContextHolderStrategy` 타입의 stragegy 변수에 모든 위임이 일어나고 있는 것을 확인할 수 있다.

```java
public class SecurityContextHolder {
    public static final String MODE_THREADLOCAL = "MODE_THREADLOCAL";
    public static final String MODE_INHERITABLETHREADLOCAL = "MODE_INHERITABLETHREADLOCAL";
    public static final String MODE_GLOBAL = "MODE_GLOBAL";
    public static final String SYSTEM_PROPERTY = "spring.security.strategy";
    private static String strategyName = System.getProperty("spring.security.strategy");
    private static SecurityContextHolderStrategy strategy;
    private static int initializeCount = 0;

    public SecurityContextHolder() {
    }

    private static void initialize() {
        if (!StringUtils.hasText(strategyName)) {
            strategyName = "MODE_THREADLOCAL";
        }

        if (strategyName.equals("MODE_THREADLOCAL")) {
            strategy = new ThreadLocalSecurityContextHolderStrategy();
        } else if (strategyName.equals("MODE_INHERITABLETHREADLOCAL")) {
            strategy = new InheritableThreadLocalSecurityContextHolderStrategy();
        } else if (strategyName.equals("MODE_GLOBAL")) {
            strategy = new GlobalSecurityContextHolderStrategy();
        } else {
            try {
					...
				}
			...
		}
```

이렇듯 대부분의 작업은 SecurityContextHolderStrategy의 구현체 들에서 일어나기 때문에  어떤 작업을 수행하는지 들여다 보기 위해서는 이 인터페이스의 내부를 들여다 보아야한다.

인터페이스 SecurityContextHolderStrategy는 총 세 개의 구현체로 이뤄져 있다.

<p align = "center"><img style = "width : auto; height : auto;" alt = "internal1.png" src = "../../assets/images/spring/internal1.png"></p> 
<p align =  "center" style = "font-size : 10px; color : gray;">
SecurityContextHolderStrategy의 구현체들. 내용이 잘 보이지 않기 때문에 아래에 정리되어 있다.
</p>

- GlobalSecurityContextHolderStrategy
- inheritableThreadLocalContextHolderStrategy
- ThreadLocalContextHolderStrategy

세 구현체 중 기본값으로 사용되는 구현체는 `ThreadLocalContextHolderStrategy` 이며 이름에서 유추할 수 있듯, ThreadLocal 변수를 사용한다.

앞서 말한 ThreadLcoal 변수는 이를 사용하고 전 스레드로컬 변수 값을 반드시 반환되어야 한다. 이점은 FilterProxy의 내부를 통해서 살펴볼 수 있다.

```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        boolean clearContext = request.getAttribute(FILTER_APPLIED) == null;
        ...
         finally {
              SecurityContextHolder.clearContext();//컨텍스트를 비워 ThreadLocal이 변수가 잘못된 값을 참조치 않도록 한다.
              request.removeAttribute(FILTER_APPLIED);
        ...
        }
    }
```

갑작스럽게 FilterProxy의 내부를 확인하는게 뜬금 없다고 생각될 수 있지만, 앞서 봤던 SecurityContextHolder는 이전에 언급했듯 ThreadLocalContextHolderStrategy가 구현하는 인터페이스이다. 

```java
public class SecurityContextHolder {
	...
	private static SecurityContextHolderStrategy strategy;
	...

	public static void clearContext() {
			strategy.clearContext();
		}
  ...
}
```

즉, SecurityContextHolder의 구현체 중 ThreadLocal 변수를 사용하는 ThreadLocalContextHolderStrategy는 SecurityContextHolder의 `clearContext()`를 통해 사용된 ThreadLocal 변수가 잘못된 값을 참조하는 것을 방지한다는 의미이다.   

### Authentication

Authentication(인증)은 사용자가 주장하는 본인이 맞는지 확인하는 절차를 의미한다.

Authentication 클래스는 사용자를 표현하는 인증 토큰 인터페이스로 인증된 주체를 표현하는 `principal`

과 그 사용자의 권한을 의미하는 `GrantAuthority`를 의미한다. Authentication은 인터페이스로서 여러 구현체들을 가지고 있다.

- Authentication의 모든 구현체
    - `AbstractAuthenticationToken`
        
        인증 객체를 위한 기본 클래스
        
    - `AnonymousAuthenticationToken`
        
        익명 사용자를 표현하는 인증 객체
        
    - `JaasAuthenticationToken`
        
        사용자가 로그인한 Jaas(자바에서 제공하는 보안 프레임워크) Context를 전달하는 UsernamePasswordAuthenticationToken의 확장
        
    - `PreAuthentcatedAuthenticationToken`
        
        사전 인증된 인증을 위한 Authentication 구현체
        
    - `RememberMeAuthenticationToken`
        
        remember-me 기반 Authentication 인터페이스 구현체
        
    - `RunAsUserToken`
        
        RunAsManagerImpl 클래스를 지원하는 immutable한 Authentication 구현체
        
    - `TestingAuthenticationToken`
        
        단위 테스트 중 사용하도록 설계된 Authentication 구현체
        
    - `UsernamePasswordAuthenticationToken`
        
        로그인 아이디/비밀번호 기반 Authentication 구현체
        

#### 인증처리

---

Spring Security의 전체적인 인증처리는 아래의 그림과 같이 흘러간다.

<p align = "center"><img style = "width : auto; height : auto;" alt = "internal2.png" src = "../../assets/images/spring/internal2.png"></p> 
<p align =  "center" style = "font-size : 10px; color : gray;">
Spring Security3.0 -Dmitry Noskov
</p>


인증 처리과정을 따라 주요한 개념인 uthenticationProcessingFilter, AuthenticationManager, AuthenticationProvider에 대해서 알아본다.

- `AuthenticationProcessingFilter`(AbstractAuthenticationProcessingFilter)
    - AbstractAuthenticationProcessingFilter의 구현체는 UsernamePasswordAuthenticationFilter 하나 존재한다.
    - 먼저 들어온 요청에 대해 처리하기 위해 인증되지 않은 사용자의 `Authentication`을 생성한다.
        
        (권한을 가지고있지 않고 인증 여부는 false 값을 갖고 있게 된다.)
        
    
- `AuthencticationManager`
    - ProvideManager를 기본 구현체로 가지는 인터페이스다.
    - ProviderManger는 AuthenticationProvider를 리스트 형태로 가진다.

- `AuthenticationProvider`
    - AuthenticationProvider 중 에서 DAOAuthenticationProvider는 인증처리를 수행한다.
        
        <p align = "center"><img style = "width : auto; height : auto;" alt = "internal3.png" src = "../../assets/images/spring/internal3.png"></p> 
        <p align =  "center" style = "font-size : 10px; color : gray;">
        AuthenticationProvider는 많은 수의 구현체를 가진다.
        </p>
        
        AuthenticationProvider는 많은 수의 구현체를 가진다.
        
    - 인증이 완료되면 인증이 완료된 Authentication 객체를 만들고 GrantedAuthority에서 권한을 적절히 매핑한 후 인증을 완료한다.

위 개념들을 구상도로 표현하면 아래와 같다.

<p align = "center"><img style = "width : auto; height : auto;" alt = "internal4.png" src = "../../assets/images/spring/internal4.png"></p> 

## Reference

Spring AOP
    
[https://www.javacodegeeks.com/2018/02/securitycontext-securitycontextholder-spring-security.html](https://www.javacodegeeks.com/2018/02/securitycontext-securitycontextholder-spring-security.html)

Security Internal 

[https://github.com/sombrero104/springSecurityForm](https://github.com/sombrero104/springSecurityForm)