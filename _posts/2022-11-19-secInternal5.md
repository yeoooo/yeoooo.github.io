---
title: "[Spring] Spring Security 내부 구조 5"
excerpt: "인증처리 필터"

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

# AbstractAuthenticationProcessingFilter

인증에 대한 처리를 맡는 필터는 `AbstractAuthenticationProcessingFilter`로authenticationManager 속성이 설정되어 있어야 한다. 또한 구현하고 있는 클래스에 의해 생성된 인증 요청 토큰을 처리하도록 요구된다.

`AbstractAuthenticationFilter`는 요청이 `setRequiresAuthenticationRequestMatcher`(RequestMather)와 매칭되면 요청을 가로채 인증 수행을 시도한다. 인증은 서브클래스에 의해 구현되야 하는 `attemptAuthentication()`에 의해 수행된다.

## AbstractAuthenticationProcessingFilter - 인증 성공, (AuthenticationSuccessHandler)

인증이 성공했을 경우, 결과 인증 객체는 이전 필터에 의해 생성된 것으로 보장된 현재 스레드의 Security Context에 위치하게 된다. 성공적인 로그인 후에 적절한 목적지로 리다이렉트를 취하기 위해 설정된 `AuthenticationSuccessHandler`가 호출된다. 

이 핸들러의 기본 행동은 `ExceptionTranslationFilter`에 의해 설정된 `SavedRequestAwareAuthenticationSuccess` 에 구현되어 있으며 `DefaultSavedRequst`를 사용하고 유저를 포함한 URL을 리다이렉트 한다.  

아니면 webapp의 루트인 `/` 로 리다이렉트 한다. 기본 행동은 해당 클래스의 다르게 설정된 인스턴스를 주입하거나 다른 구현을 사용해 customize될 수 있다.

## AbstractAuthenticationProcessingFilter - 인증 실패, AuthenticationFailureHandler

인증이 실패하면 실패정보가 클라이언트에 전달되도록 설정되어 있는 `AuthenticationFailureHandler`에 위임한다.

AuthenticationFailureHandler는 추상 클래스로써 실패의 원인에 따라 그 구현 클래스가 달라지게 된다. 기본 구현은 `SimpleUrlAuthenticationFailureHandler`로 401 에러를 클라이언트에 보낸다. 또한 실패시 대체 URL을 전송하도록 설정되어 있으며 이곳에 원하는 동작을 주입할 수 있다.

## Event Publication  
인증이 성공적이면 InteractiveAuthenticationSuccessEvent 가  ApplicationContext에 발행된다. 인증이 성공적이지 않으면 어떠한 이벤트도 발행되지 않으며 이는 AuthenticationManager에 기록되기 때문이다.

### Event?
---

Event는 Spring의 빈과 빈 사이에 존재하는 데이터를 전달하는 방법 중 하나로써 Spring 내부에 존재하는 메거니즘이다. 

일반적으로 DI를 통해 데이터를 전달하는 메커니즘과 다르게 한 빈에서 설정한 이벤트를 ApplicationContext에 넘겨주고 이를 B빈에서 Listener를 통해 받아 처리하는 메커니즘을 가지고 있다.

Event의 데이터는 Event 모델에 담아서 처리하게 된다.

Spring Security의 Event는 `AuthenticationEventPublisher` 클래스를 통해 일어난다.

```java
package org.springframework.security.authentication;

public interface AuthenticationEventPublisher {

	void publishAuthenticationSuccess(Authentication authentication);

	void publishAuthenticationFailure(AuthenticationException exception, Authentication authentication);

}
```

publishAuthenticationSuccess와 publishAuthenticationFailure 메서드는 모두 `ProvideManager`클래스에서 호출된다.  이들은 이벤트 모델로써 컴포넌트간의 결합도를 낮추는데 도움을 준다.

하지만 이벤트 모델을 사용할 때는 이벤트 모델이 동기적이라는 점을 기억해두어야 한다.  이벤트를 구독하는 리스너의 처리 지연은 이벤트를 발생시킨 요청의 응답 지연에 직접적인 영향을 끼치기 때문이다.

```java
@Component
public class CustomAuthenticationEventHandler {
    private final Logger log = LoggerFactory.getLogger(getClass());

    @EventListener
    public void handleAuthenticationSuccessEvent(AuthenticationSuccessEvent event) {
				try{
				Theread.sleep(5000L);//로그인 처리 자체가 5초 지연된다.
				}catch(InterruptedException e){
				}
        Authentication authentication = event.getAuthentication();
        log.info("Successful authentication result: {}", authentication.getPrincipal());
    }

    @EventListener
    public void handleAuthenticationFailureEvent(AbstractAuthenticationFailureEvent event) {
        Exception e = event.getException();
        Authentication authentication = event.getAuthentication();
        log.warn("Unsuccessful authentication result: {} ", authentication, e);
    }
}
```

```java
@EnambleAsync
@Configuration
publci class WebMvcConfigure implements WebMvcConfigurer{
	@Override
	public void addViewControllers(ViewControllerRegistry reg){
		...
	}

@Component
public class CustomAuthenticationEventHandler {
    private final Logger log = LoggerFactory.getLogger(getClass());

		@Async
    @EventListener
    public void handleAuthenticationSuccessEvent(AuthenticationSuccessEvent event) {
				try{
				Theread.sleep(5000L);//로그인 처리 자체가 5초 지연된다.
				}catch(InterruptedException e){
				}
        Authentication authentication = event.getAuthentication();
        log.info("Successful authentication result: {}", authentication.getPrincipal());
    }

    @EventListener
    public void handleAuthenticationFailureEvent(AbstractAuthenticationFailureEvent event) {
        Exception e = event.getException();
        Authentication authentication = event.getAuthentication();
        log.warn("Unsuccessful authentication result: {} ", authentication, e);
    }
}
```

로그인 성공 시 사용자에게 이메일을 발송해야하는 시스템이 존재하는 경우에서 SNS에 알림을 띄워야 하는 요구사항이 늘어났을 때를 예로, 이벤트 모델을 사용하지 않는 경우 이메일 발송 수행을 맡는 클래스에서 SNS 알림 기능을 추가함으로써 이전에 존재하던 클래스를 수정해야한다. 이전에 존재하던 코드를 수정하는 것은 결합도가 높고 확장에 닫혀 있다는 것을 의미한다.

하지만 이벤트 모델을 사용한다면 로그인 성공 이벤트를 구독하는 리스너를 추가함으로써 쉽게 SNS 알림 기능 추가가 가능하다. 

## Reference

- AbstractAuthenticationProcessingFilter
    
    [https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/authentication/AbstractAuthenticationProcessingFilter.html](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/authentication/AbstractAuthenticationProcessingFilter.html)
    
- Event
    
    [https://sabarada.tistory.com/184](https://sabarada.tistory.com/184)