---
title: "[Spring] Spring Security 내부 구조 2"
excerpt: "세션 처리"

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

# 세션 처리

세션을 처리하는 필터는 `SecurityContextPersistenceFilter` 와 `SessionManagementFilter`크게 두 가지로 존재한다.

## SecurityContextPersistenceFilter

SecurityContextPersistenceFiltersms FilterChain에서 인증과 관련된 필터 중 최상단에 위치하고 있다.

이름에서 유추할 수 있듯 <span class = "o">사용자의 최근 인증된 사용자의 정보와 세부사항을 저장하고 있는 SecurityContext를 영속화</span>하는 필터이다.

SecurityContextPersistenceFilter는 별도의 변경이 없다면 SecurityContextRepository의 구현체의 기본 값인 HttpSessionSecurityContextRepository를 통해 HttpSession의 Attribute에 SecurityContext를 영속화한다.

또한 요청의 세션에서 저장되어 있는 SecurityContext를 불러와 SecurityContextHolder에 저장해 SecurityContext에 접근할 수 있도록 돕는다.

## SessionManagementFilter

SessionManagementFilter는 `session-fixation-protection` 기능을 통해 session fixation 을 방지한다.

먼저 Session Fixation Attack에 대해서 알아보자

### Session Fixation Attack

---

Session Fixation은 Session Hijacking의 기법 중 하나로 로그인 시 발급 받은 세션 ID가 로그인 전/후로 모두 동일하게 사용되는 점을 이용해 악의적인 사용자가 타겟의 세션을 하이재킹해 정상적인 사용자로 위장해 접근하는 행위이다.

세션 고정 공격은 아래와 같은 시나리오로 이뤄진다.

1. Attacker는 먼저 취약 웹 어플리케이션에서 Session ID를 발급받는다.
2. User 에게 탈취한 Session ID를 포함시킨 링크, 이메일 등을 통해 취약 웹사이트에 로그인하도록 유도한다.
3. User가 해당 링크를 통해 로그인 정보를 입력해 로그인한다.
4. Attacker는 새로고침을 통해 인증정보를 얻어내 일반 User와 같이 행동한다.

해당 과정을 알기 쉽게 그림으로 표현하면 아래와 같다.

<p align = "center"><img style = "width : auto; height : auto;" alt = "internal5.png" src = "../../assets/images/spring/internal5.png"></p> 
<p align =  "center" style = "font-size : 10px">
<a href = "https://secureteam.co.uk/articles/web-application-security-articles/understanding-session-fixation-attacks/">
출처 :https://secureteam.co.uk/articles/web-application-security-articles/understanding-session-fixation-attacks/ 
</a>
</p>

Spring Security에서는 4가지 옵션을 통해 session-fixation-attack을 로그인 전의 세션ID와 로그인 후의 세션ID를 다르게 제공함으로써 session fixation attack을 방지한다.

- `none`
    
    세션을 그대로 유지(세션 ID 노출)
    
- `newSession`
    
    새로운 세션을 만들고 기존 데이터는 복제하지 않는다.
    
- `migrateSession`
    
    새로운 세션을 만들고 기존 데이터들을 새 세션으로 이주시킨다.
    
- `changeSession`
    
    새로운 세션을 만들지는 않지만, session-fixation 공격을 방어한다.(servlet 3.1 이상에서 지원하며 Spring Security에서 기본 전략으로 취한다)
    

Session Fixation Attack 방지와 더불어 Session에 관한 Spring Security 설정은 아래와 같이 설정할 수 있다.

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfigure extends WebSecurityConfigurerAdapter {

  ...
	@Override
	  protected void configure(HttpSecurity http) throws Exception {
	    http
	      ...
				.and()
				.sessionManagement()
				.sessionFixation().changeSessionId()//session-fixation-protection의 
																						//전략으로 changeSession을 차용
				.sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
																						//session Creation Policy의
																						//전략으로 IF_REQUIRED를 차용
				.invalidSessionURL("/")//유효하지 않은 SessionID가 감지되었을 때 이동시킬 URL
				.maximumSessions(1)//동일 Session 내에 최대로 로그인 가능한 수
					.maxSessionPreventLogin(false)//최대 세션에 도달했을 때 로그인을 허용할지 여부
					.
				...
			}
	...
}
```

### Session Creation Policy

세션 생성 전략을 어떻게 취할 것인가 에대한 옵션이며 이는 SecurityContextRepository에 영향을 미친다. 

`SecurityContextRepository`의 구현체인 `HttpSessionSecurityRepository`에서 세션 생성과 연관있는 메서드인 `createNewSessionIfAllowed()`는 `Session Creation Policy`의 영향을 받아 세션을 어떻게 생성할 것인지 결정한다.

- `IF_REQUIRED(default)`
    
    필요시 생성
    
- `NEVER`
    
    Spring Security에서는 세션을 생성하지않으나 세션이 존재하면 사용
    
- `STATELESS`
    
    세션을 완전히 사용하지 않음(JWT 인증이 사용되는 REST API 서비스에 적합)
    
- `ALWAYS`
    
    항상 세션을 사용
    

## Reference


SecurityContextPersistenceFilter

[https://velog.io/@yaho1024/Spring-Security-SecurityContextPersistenceFilter-알아보기](https://velog.io/@yaho1024/Spring-Security-SecurityContextPersistenceFilter-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)

SessionFixation

[https://exhibitlove.tistory.com/318](https://exhibitlove.tistory.com/318)

[https://secureteam.co.uk/articles/web-application-security-articles/understanding-session-fixation-attacks/](https://secureteam.co.uk/articles/web-application-security-articles/understanding-session-fixation-attacks/)