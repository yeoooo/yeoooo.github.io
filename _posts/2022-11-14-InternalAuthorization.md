---
title: "[Spring] Spring Security 내부 구조 3"
excerpt: "인가 처리"

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

---
# Authorization
Authorization은 <span class = "o">권한이 부여된 사용자들만 특정 기능 또는 데이터에 접근을 허용</span>하는 기능이다. 

인가 처리는 두 가지 작업으로 구분된다.

- **인증된 사용자와 권한의 매핑**
    
     ex) `ROLE_USER`, `ROLE_ADMIN`, `ROLE_ANONYMOUS`
    
- **보호되는 리소스에 대한 권한 확인**
    
    관리자 권한을 가진 사용자만 관리자 페이지에 접근할 수 있도록 해야한다.
    

이와 같은 인가 처리 수행은 Spring Security의 FilterChain 상에서 가장 마지막 단에 존재하는 `FilterSecurityInterceptor` 에서 수행된다.

## FilterSecurityInterceptor


FilterSecurityInterceptor 는 FilterChain의 가장 마지막에 위치하며 리소스에서 요구하는 권한과 User의 권한을 비교해 접근 가능 여부를 결정한다. 

이 때의 실질적인 권한 비교와 접근 가능 여부를 결정하는 것은`accessDecisionManager`이다.

FileSecuirtyInterceptor 필터가 호출되는 시점에서 사용자는 인증이 완료되며 Authentication 인터페이스의 getAuthorities() 메소드를 통해 인증된 사용자의 권한 목록을 가져올 수 있게 된다.

이 때  인증 완료는 익명 사용자에게도 해당하며 익명 사용자는 ROLE_ANONYMOUS 권한을 갖게된다.

보호되는 리소스에서 요구하는 권한 정보는 `SecurityMetadataSource` 인터페이스를 통해 `ConfigAttribute` 타입으로 가져오게 된다.

### AccessDecisionManager

---

실질적인 권한 비교와 접근 가능 여부를 결정하는 AccessDecisionManager 인터페이스는 `FileSecurityInterceptor` 클래스의 조상 클래스인 `AbstractSecurityInterceptor`  클래스 내부에 존재한다.

구현체로는 아래와 같이 3개의 구현체를 가진다.

- AffirmativeBased(default)
    
    AccessDecisionVoter가 승인하면 이전에 거부된 내용과 관계없이 접근 승인
    
- ConsensusBased
    
    다수의 AccessDecisionVoter가 승인하면 접근 승인
    
- UnanimousBased
    
    모든 AccessDecisionVoter가 만장일치로 승인해야 접근 승인
    

위 3가지 구현체들은 모두 `org.springframework.security.access.vote` 에 속하며 이 들의 추상 클래스인 AbstractAcessDecisionManager는 AccesDecisionVoters객체들 을 리스트 형태로 가진다.

이 decisionVoters는 구현체 클래스 내부의 decide()의 내부 로직을 통해 접근 여부에 대한 결정을 내린다.

```java
public class AffirmativeBased extends AbstractAccessDecisionManager {

	...

	@Override
	@SuppressWarnings({ "rawtypes", "unchecked" })
	public void decide(Authentication authentication, Object object, Collection<ConfigAttribute> configAttributes)
			throws AccessDeniedException {
		int deny = 0;
		for (AccessDecisionVoter voter : getDecisionVoters()) {
			int result = voter.vote(authentication, object, configAttributes);
			switch (result) {
			case AccessDecisionVoter.ACCESS_GRANTED:
				return;
			case AccessDecisionVoter.ACCESS_DENIED:
				deny++;
				break;
			default:
				break;
			}
		}
	...
}
```

### AccessDecisionVoter

AccessDecisionManager는 AccessDecisionVoter의 객체들을 리스트로 가진다고 했다.

권한을 통해 접근 여부를 결정하는 AccessDecisionVoter는 인터페이스로서 그 추상체로는 AbstractAclVoter를 가지며 구현체로는 아래를 가진다.

#### AccessDecisionVoter 구현체

- `AuthenticatedVoter`
    
    인증을 받은 경우 그 인증의 종류가 어떤 종류인지 판단한다. 이제 막 인증을 받고 접근한 사용자와 RememberMe 토큰을 통해 들어온 사용자, 익명 사용자를 구분하기 위해 사용한다.
    
- `Jsr250Voter`
    
    자바 공통 어노테이션인 JSR-250 설정 속성에 대한 Voter
    
- `PreInvocationAuthorizationAdviceVoter`
    
    @PreFilter 및 @PreAuthorize 주석에서 생성된 PreInvocationAuthorizationAdvice 구현을 사용하여 작업을 수행하는 유권자.
    
- `RoleHierarchyVoter`
    
    권한간 계층을 가질 경우에 Role Voter만으로 표현하기 어렵기 때문에 이 점을 보안한 Role Voter의 확장버전.
    
    웹 시큐리티에서 사용하는 기본 구현체로써 ROLE_XXXX와 같은 역할이 매치하는지 확인한다.
    
- `RoleVoter`
    
    권한에 이름을 부여하고 이 이름을 이용해 access 권한을 부여한다.
    
- `WebExpressionVoter`
    
    SpEL 표현식을 사용해 접근 승인 여부에 대한 규칙을 지정할 수 있다.
    
    DefaultWebSercurityExpressionHandler.createSecurityExpressionRoot()에서 WebSecurityExpressionRoot객체를 생성한다.
    
    SecurityExpressionRoot에서는 아래와 같이 SpEL 표현식에서 사용할 수 있는 다양한 메소드를 제공한다.
    
  - Spel 표현식에서 사용할 수 있는 다양한 메소드  
        
    | Expression | Description  | Example |
    | --- | --- | --- |
    | halspAddress(ipAddress) | 요청 IP 주소가 특정 IP 주소나 특정 대역에 해당하는지 확인 | hasIpAddress(ipAddress) |
    | hasRole(String orole) | 사용자가 특정 Role을 가지고 있는지 확인 | access="hasRole('USER')” |
    | hasAnyRole(String… roles) | 사용자가 주어진 Role 목록 중 매칭되는 Role을 가지고 있는지 확인 | access="hasAnyRole('USER', 'ADMIN')" |
    | hasAuthority(String authority) | 사용자가 특정 권한을 가지고 있는지 확인 | access="hasAuthority('ROLE_USER')" |
    | hasAnyAuthority(String… authorities) | 사용자가 주어진 권한 목록 중 매칭되는 권한을 가지고 있는지 확인 | access="hasAnyAuthority('ROLE_USER', 'ROLE_ADMIN')” |
    | permitAll | 모든 사용자에 대해 접근 허용 | ccess="permitAll” |
    | denyAll | 모든 사용자에 대해 접근 거부 | access="denyAll” |
    | isAnonymous() | 사용자가 익명 사용자인지 확인 | access="isAnonymous()” |
    | isRememberMe() | 사용자가 remember-me를 통해 인증되었는지 확인 | access="isRememberMe() or hasRole('USER')” |
    | isAuthenticated() | 사용자가 인증되었는지 확인 | access="isAuthenticated()” |
    | isFullyAuthenticated() | 사용자가 익명 사용자가 아니고, remember-me 인증 사용자도 아닌지 확인 | access="isFullyAuthenticated() and hasRole('USER')” |  


## Reference.

WebExpressionVoter

[https://ugo04.tistory.com/171](https://ugo04.tistory.com/171)