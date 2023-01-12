---
title: "[Spring] Spring Security와 DB"
excerpt: "Spring과 DB기반 인증처리"

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

# Log4JDBC

Log4JDBC를 통해 쿼리에 따른 로그를 남겨 쿼리가 의도대로 잘 움직이고 있는지 확인할 수 있다.

Log4JDBC는 LogBack 라이브러리를 활용하기 때문에 `logback.xml` 을 활용해 설정이 가능하다.

다음은 logback.xml에서 설정 가능한 속성 값과 예시다.

```xml
<logger name="io.undertow" level="OFF"/>
  <logger name="jdbc.sqltiming" level="INFO"/>
  <logger name="jdbc.audit" level="OFF"/>
  <logger name="jdbc.resultset" level="OFF"/>
  <logger name="jdbc.resultsettable" level="INFO"/>
  <logger name="jdbc.connection" level="OFF"/>
  <logger name="jdbc.sqlonly" level="OFF"/>
```

- `sqltiming`
    
    실행되는 쿼리와 해당 쿼리의 실행시간을 로그로 남긴다. 
    
- `resultsettable`
    
    쿼리 실행 결과를 테이블 형식으로 result set을 로그로 남긴다.
    
- `audit`
    
    ResultSet을 제외한 모든 JDBC 호출 정보를 로그로 남긴다. 많은 양의 로그가 발생되며 특별히 JDBC 문제를 추적해야 할 필요가 있는 경우를 제외하고 사용하지 않는 것이 좋다.
    
- `undertow`
    
    non-blocking IO에 기초한 웹 서버인 undertow에 대한 로그를 남긴다.
    
- `resultset`
    
    ResultSet을 포함한 모든 JDBC 호출 정보를 로그로 남기므로 매우 방대한 양의 로그가 생성된다.
    
- `connection`
    
    수행 도중 열리고 닫히는 연결 내용을 포함한다. 연결 누수 문제를 찾는데 유용하다.
    
- `sqlonly`
    
    쿼리만을 로그로 남기며 PreparedStatement일 경우 관련된 argument 값으로 대체된 쿼리를 로그로 남긴다.
    

# 데이터베이스 기반 인증 처리

Spring Security에서 인증처리의 시작은 `UsernamePasswordAuthneticationFilter` , Remember-me 기능을 사용하는 경우 `Remember-me AuthenicationFilter` 에서 이뤄진다.

`UsernamePasswordAuthenticationFilter` 와`Remember-me AuthenicationFilter` 의 공통점은 AuthenticationManager 인터페이스에 의존하고 있다는 점이다.

```java
public class UsernamePasswordAuthenticationFilter extends AbstractAuthenticationProcessingFilter {

		...

    public UsernamePasswordAuthenticationFilter(AuthenticationManager authenticationManager) {
        super(DEFAULT_ANT_PATH_REQUEST_MATCHER, authenticationManager);
    }
		
		...
}
```

```java
public class RememberMeAuthenticationFilter extends GenericFilterBean implements ApplicationEventPublisherAware {	
...
	public RememberMeAuthenticationFilter(AuthenticationManager authenticationManager, RememberMeServices rememberMeServices) {
	        Assert.notNull(authenticationManager, "authenticationManager cannot be null");
	        Assert.notNull(rememberMeServices, "rememberMeServices cannot be null");
	        this.authenticationManager = authenticationManager;
	        this.rememberMeServices = rememberMeServices;
	    }
	...
}
```

이러한 AuthenticationManager의 내부를 들여다 보면 그 안에는 authentication() 하나만 존재한다. 

```java
public interface AuthenticationManager {

	Authentication authenticate(Authentication authentication) throws AuthenticationException;

}
```

실제로 `authentication()` 을 통해서 위 두 클래스는 인증처리를 하게 되는데, AuthenticationManager 인터페이스의 default 구현체는 `Provider Manager`이다. 

`ProviderManager`는 `AuthenticationProvider`인터페이스를 리스트의 형태로 가지고 있고 해당 리스트에서 실제 인증처리 로직이 구현되어 있다.

그렇다면 AuthenticationProvider 인터페이스는 어떤 구현체를 가질까 ?

<p align = "center"><img style = "width : auto; height : 300px;" alt = "secWithDB0.jpg" src = "../../assets/images/spring/secWithDB0.jpg"></p> 
<p align =  "center" style = "font-size : 10px; color : gray;">
AuthenticationProvider의 구현체.
</p>


 `AuthenticationProvider`는 위의 사진과 같이 많은 구현체를 가지지만 이 중 `DaoAuthenticationProvider`를 기본 구현체로 가진다. 

## DaoAuthenticationProvider  
DaoAuthenticationProvider는 `UsernamePasswordAuthenticationToken` 타입의 인증 요청을 처리한다.

이 클래스에서는 사용자를 조회하는`retrieveUser()->loadUserByName()` 를 통해서 핵심적인 역할을 한다. 

## UserDetailService

`loadUserByName()`을 지니고 있는 인터페이스는 UserDetailService이다.

<p align = "center"><img style = "width : auto; height : 300px;" alt = "secWithDB1.png" src = "../../assets/images/spring/secWithDB1.jpg"></p> 
<p align =  "center" style = "font-size : 10px; color : gray;">
UserDetailService 인터페이스의 구현체.
</p>


이 중 `InMemoryUserDetailsManager` 는 사용자 계정 정보를 메모리에 저장하는 방식으로 동작하며 이는 기본 UserDetailService의 기본 구현체이다.

하지만 데이터베이스 기반 인증 처리에는 InmemoryUserDetailManager가 아닌 아래와 같이 `JdbcDaoImpl` 구현체를 빈으로 등록해 DaoAuthenticationProvider에서 사용할 수 있도록 처리해야한다.

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfigure extends WebSecurityConfigurerAdapter {	
	...
	@Bean
	public UserDetailsService userDetailsService(DataSource dataSource) {
  JdbcDaoImpl jdbcDao = new JdbcDaoImpl();
  jdbcDao.setDataSource(dataSource);
  return jdbcDao;
	}
	...
}

```

### JdbcDaoImpl

JdbcDaoImpl 클래스는 수행 목적에 따라 3개의 SQL 쿼리를 정의하고 있다

```java
// @formatter:off
public static final StringDEF_USERS_BY_USERNAME_QUERY= "select username,password,enabled "
      + "from users "
      + "where username = ?";
// @formatter:on

// @formatter:off
public static final StringDEF_AUTHORITIES_BY_USERNAME_QUERY= "select username,authority "
      + "from authorities "
      + "where username = ?";
// @formatter:on

// @formatter:off
	public static final String DEF_GROUP_AUTHORITIES_BY_USERNAME_QUERY = "select g.id, g.group_name, ga.authority "
			+ "from groups g, group_members gm, group_authorities ga "
			+ "where gm.username = ? " + "and g.id = ga.group_id " + "and g.id = gm.group_id";
	// @formatter:on
```

- usersByUsernameQuery
    - 사용자명과 일치하는 하나 이상의 사용자를 조회
    - 조회하는 값은 username:String, password: String, enabled:Boolean의 순서를 가지고 있어야 한다.
    
- authoritiesByUsernameQuery
    - 사용자에게 직접 부여된 하나 이상의 권한을 반환 (Group-based Access Control 미적용시)
    - 조회하는 두 번째 값은 authority:String 이어야 한다.
    
- groupAuthoritiesByUsernameQuery
    - 그룹 멤버십을 통해 사용자에게 승인된 권한을 반환 (Group-based Access Control 적용시)
    - 조회하는 세 번째 값은 authority:String 이어야 한다.
    
---  

# 데이터베이스 기반 인증 고급 설정

데이터베이스 기반 인증 처리를 위해 JdbcDaoImpl 써야 했다.

JdbcDaoImpl의 내부를 보면 아래와 같이 유저 정보를 불러오기 위해 쿼리를 사용했음을 알 수 있다.

즉, JdbcDaoImpl을 사용하기 위해서는 만들어져있는 쿼리를 사용하기에 적합한 테이블을 만들어야 한다. 하지만 JdbcDaoImpl 구현체를 사용하고자 테이블을 이에 맞추는 것은 현실적이지 못하다.

해당 문제는 이전에 JdbcDaoImpl내에 구현되어 있는 쿼리를 Override하고 관련 설정을 바꿔줌으로써 해결할 수 있다.

예시의 스키마는 다음과 같다.

<p align = "center"><img style = "width : auto; height : 300px;" alt = "secWithDB2.jpg" src = "../../assets/images/spring/secWithDB2.jpg"></p> 

```java
@Bean
public UserDetailsService userDetailsService(DataSource dataSource) {
  JdbcDaoImpl jdbcDao = new JdbcDaoImpl();
  jdbcDao.setDataSource(dataSource);
  jdbcDao.setEnableAuthorities(false);
	//Group-based Access Control 활용시 false 
  jdbcDao.setEnableGroups(true);
	//Group-based Access Control 활용시 true
  jdbcDao.setUsersByUsernameQuery(
          "SELECT " +
            "login_id, passwd, true"+
          "FROM " +
            "USERS "+
          "WHERE "+
            "login_id = ?"
  );
  jdbcDao.setGroupAuthoritiesByUsernameQuery(
          "SELECT "+
              "u.login_id, g.name, p.name " +
          "FROM "+
              "users u JOIN groups g ON u.group_id = g.id "+
              "LEFT JOIN group_permission gp ON g.id = gp.group_id "+
              "JOIN permissions p ON p.id = gp.permission_id " +
          "WHERE "+
              "u.login_id = ?"
  );
  return jdbcDao;
}
```

하지만 이렇게 JdbcDaoImpl을 설정하게 되면 

Remember Me 기능을 이용할 때 아래와 같은 문제점이 발생할 수 있다.

`AbstractAuthenticationProcessingFilter` → `sucessfulAuthentication` → `loginSuccess` → `TokenBaseRememberMeServices.onLoginSuccess()` → `UserDetails.loadUserByUsername()` → `delegate`의 UserDetails 참조 불가 → `IllegalStateException()`

이러한 문제점을 해결하기 위해 JdbcUserDetailsManager를 사용할 수 있다.

# JdbcUserDetailsManager

JdbcUserDetailsManager를 이용하기 위해서는 Spring Security 설정 클래스에서 `AuthenticationManagerBuilder`를 변수로 취하는 `configure()` 를 오버라이드 해야한다. 

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
 
}
```

`AuthenticationManagerBuilder` 는 빌더 클래스로서, 친숙히 사용하던 `InMemoryUserDetailsManagerConfigurer` 을 더불어 `JdbcUserDetailsManagerConfigurer` 를 빌더 패턴을 통해 설정할 수 있고, 그 안에 있는 JdbcUsersDetailsManager를 통해서 데이터베이스 기반 인증 처리에 대한 설정을 할 수 있다.

JdbcUserDetailsManager를 통한 고급 설정은 아래와 같이 할 수 있다.

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth.jdbcAuthentication()
    .dataSource(dataSource)
    .usersByUsernameQuery(
      "SELECT " +
        "login_id, passwd, true " +
      "FROM " +
        "USERS " +
      "WHERE " +
        "login_id = ?"
    )
    .groupAuthoritiesByUsername(
      "SELECT " +
        "u.login_id, g.name, p.name " +
      "FROM " +
        "users u JOIN groups g ON u.group_id = g.id " +
        "LEFT JOIN group_permission gp ON g.id = gp.group_id " +
        "JOIN permissions p ON p.id = gp.permission_id " +
      "WHERE " +
        "u.login_id = ?"
    )
    .getUserDetailsService().setEnableAuthorities(false)
  ;
}
```

# Reference.

---

- LogBack 설정
    
    [https://yongho1037.tistory.com/721](https://yongho1037.tistory.com/721)
    
    [https://kafcamus.tistory.com/23](https://kafcamus.tistory.com/23)