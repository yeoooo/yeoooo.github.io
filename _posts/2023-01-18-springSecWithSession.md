---
title: "[Spring] Spring Security, Spring Session과 Repository"
excerpt: "SpringSession과 Repository"

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

# Spring Session

Spring Session은 세션 클러스터를 구현하는데 필요한 기능을 추상화된 API를 제공함으로써 실제 Session을 저장하는 스토리지에 의존적인 구현을 방지해준다.

Spring Session에서는 아래와 같은 스토리지를 활용할 수 있다.

- JDBC
- Apache Geode
- MongoDB

- Hazelcast(In-Memory)
- Redis(In-Memory)

`Hazelcase` 와 `Redis` 에 특별히 In-memory를 표시한 이유는 현업에서 세션 클러스터를 구현할 때 Http 요청 때 마다 입출력의 빈도가 높은 Session 객체의 특성을 고려해 우수한 성능을 보이는 In-Memory 스토리지 엔진을 활용한다면 전반적인 처리 성능이 향상되기 때문에 In-memory 스토리지 엔진을 선호하기 때문이다.

Spring Session은 앞서 언급했듯 잘 추상화된 API이다. 따라서 먼저 Spring Session JDBC에 대해 알아보고 In-Memory 스토리지 엔진 기반 스토리지로의 Migration 까지 알아보자.

# Spring Session JDBC

JDBC를 활용한 Spring Session 구현을 하기 위해서는 먼저 h2를 활용해 JDBC에 맞는 의존성을 추가해주어야 한다. 

## 의존성 추가

- <span class = "o">maven(pom.xml)</span>
    
    ```xml
    <dependency>
      <groupId>org.springframework.session</groupId>
      <artifactId>spring-session-jdbc</artifactId>
    </dependency>
    ```
    
- <span class = "o">gradle(gradle.build)</span>
    
    ```yaml
    dependencies {
    	implementation 'org.springframework.session:spring-session-jdbc'
    }
    ```
    

## YML 설정

### DDL SQL 추가

아래와 같이 Session에 관련된 속성을 관리하는 테이블을 추가해준다. 

```yaml
sql:
	init.schema-location:classpath:org/springframework/session/jdbc/schema-h2.sql
```

### Spring Session 관련 설정

`spring.session` 에는 사용하는 스토리지에 맞는 이름을 설정해주고 `spring.session.{Storage}` 에는 스토리지에 관련된 설정을 한다.

참고로 jdbc에 관련된 설정은 아래의 사진과 같다.

<p align = "center"><img style = "width : auto; height : auto;" alt = "springSecSession.png" src = "../../assets/images/spring/springSecSession.png"></p> 

```yaml
spring:	
	session:
	  store-type:jdbc
	jdbc:
	    initialize-schema:never
```

### @EnableJdbcHttpSession 적용

마지막으로 MVC관련 java 설정 class에서 @EnableJdbcHttpSession을 적용해 jdbc 기반 Spring Session을 활성화할 수 있다.

이후로 h2 콘솔에 들어가 테이블을 확인하면 `SPRING_SESSION`, `SPRING_SESSION_ATTRIBUTES`와 같은 Spring Session 관련 설정을 데이터로 들고있는 테이블을 확인할 수 있다.

# Session Repository, Session Repository Filter

## Session Repository

Spring Session은 핵심적으로 SessionRepository 인터페이스와 SessionRepositoryFilter 에 의해 작동한다.

Session Repository는 Session의 CRUD 처리에 대한 책임을 가지며 스토리지의 종류에 따라 다양한 구현체를 제공한다.

- MapSessionRepository
    
    `In-Memory Map` 기반으로써 별도의 의존 라이브러리가 필요하지 않다.
    
- RedisIndexedSessionRepository
    
    `redis` 기반으로 `@EnableRedisHttpSession`을 통해 생성된다.
    
- JdbcIndexedSessionRepository
    
    `Jdbc` 기반으로 `@EnableJdbcHttpSession`을 통해 생성된다.
    

## Session Repository Filter

httpServletRequest, httpServletResponse 클래스를 SessionRepositoryRequestWrapper, SessionRepositoryResponseWrapper로 교체해 Spring Session이 세션과 관련된 동작과 처리를 하도록 구현되어있다. 

이는 Spring Security가 세션 처리에서 세션의 요청, 응답과 그 타입에 대해 자세히 알 필요 없이 추상화된 객체를 잘 처리하도록 도와준다. 

즉 어떤 스토리지에 의존적이지 않은 세션 처리를 하도록 돕는다고 할 수 있다.