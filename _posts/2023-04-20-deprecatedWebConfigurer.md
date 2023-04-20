---
title: "[Spring] WebSecurityConfigurerAdapter 지원 종료"
excerpt: "지원 종료된 Spring Security API에 대해 알아보자."

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

상태: 진행 중

# 개요

Spring Security를 이용해 로그인 기능을 구현하기로 했다.

가장 처음으로 SecurityConfigurer 을 작성해야하는 것을 알고 있었기 때문에, 이를 reference doc과 api doc에서 찾아보는 시간을 가졌다.

reference doc에서는 해당하는 내용을 찾아보기 힘들었다.

이후 이전에 security를 사용했던 프로젝트를 참고해 이를 확인했는데, configurere를 구현하기 위해서 

`WebSecurityConfigurerAdapter`를 extends 한 모습을 볼 수 있었다.

이를 왜 사용해야하는지, 무슨 일을 하는지 문득 궁금해져 api docs에서 확인해봤는데,  해당 내용은 api docs에 존재하지 않았다.

그러고는 지원 종료  되었을 경우를 생각해 Spring 사이트에 쳐봤는데, 

아니나 다를까 다음과 같은 내용을 확인할 수 있었다.

[Spring Security without the WebSecurityConfigurerAdapter](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter)

위 링크의 내용은 다음과 같다.

Spring Secuirty 5.7.0-M2 버전에서는 `WebSecurityConfigurerAdapter` 에 대한 지원을 종료했다. 이는 개발자를 컴포넌트-기반 시큐리티 설정을 장려하기 위함이다.

`WebSecurityConfigurerAdapter` 는 매우 복잡하고 높은 수준 지식을 요구하기 때문에 이를 설정하기 위해서는 복잡한 설정을 해야하는데, 이와 같은 복잡한 설정은 컴포넌트 기반 개발 방식을 저해 한다는 것이다.

  따라서 SpringSecurity는 이에 대한 지원을 종료하고 람다식을 통한 메서드 체인 구성 방식을 통해 컴포넌트 기반 개발을 장려한다. 

Spring Security 에서 제공하는 람다 설정과 기존 설정 샘플은 다음과 같다.

```java
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests((authz) -> authz
                .anyRequest().authenticated()
            )
            .httpBasic(withDefaults());
    }

}
```

```java
@Configuration
public class SecurityConfiguration {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests((authz) -> authz
                .anyRequest().authenticated()
            )
            .httpBasic(withDefaults());
        return http.build();
    }

}
```  
# 결론  
SpringSecurity Configurer를 만들고자 한다면, extends 없이 lambda로 만들어야 한다.