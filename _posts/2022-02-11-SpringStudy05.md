---
title: "자바 기반의 빈 설정"
excerpt: "6주차 스프링 스터디 - 스프링파전"

categories:
    Spring
tags:
    프레임워크
    백엔드
    배워서바로쓰는스프링프레임워크
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

## 개요  
이전 까지는 xml을 통한 빈, 컨테이너 정의에 대해서 배우고 어노테이션에 대해 배웠다. 이번 주차에서는 직접 xml에 정의하지 않고 자바에 어노테이션을 이용해 빈과 컨테이너를 정의하는 <span class = "o">자바 기반 컨테이너 설정</span>을 배운다.  
책에 따르면, 빈을 정의할 때는 xml 정의와 자바 어노테이션 정의 중 취향에 맞게 고르면 된다고 한다.  

## 스프링 빈 설정  
### @Configuration
스프링 빈을 설정하기 위해서는 `@Configuration`과 `@Bean`을 이용하면 된다.  
먼저 클래스에 `@Configuration`을 설정하면 그 클래스 안에는 `@Bean`을 설정한 <span class = "o">메서드</span>가 1개 이상있어야한다.  
그리고 그 <span class ="o">메서드</span>는  빈 인스턴스를 생성하여 반환한다. 이 때 `스프링 컨테이너`는 `@Bean`을 설정한 메서드가 반환하는 빈 인스턴스를 관리한다.  
아래에 예시가 제시되어있다.  

```java  
@Configuration
public class BankAppConfiguration {
    
    @Bean(name = "accountStatementService")
    public AccountStatementService accountStatementService() {
        AccountStatementServiceImpl accountStatementServiceImpl = new AccountStatementServiceImpl();
        accountStatementServiceImpl.setAccountStatementDao(accountStatementDao());
        return accountStatementServiceImpl;
    }
    ...
}
```  
이 때 위의 예시는 xml에서의 다음 예시 빈 정의와 같은 효과를 낸다.  

```xml  
<bean id = "accountStatementService" 
    class = "sample.spring.chapter.07.bankapp.service.accountStatementServiceImpl"/>
```  

이렇게 `@Configuatrion`을 설정한 클래스를 이용하기 위해서는 `CGLIB`라는 라이브러리가 필요한데, `@Configuration`을 설정한 클래스를 확장해 `@Bean`을 설정한 메서드에 행동을 추가한다.  
하지만 스프링3.2 부터 `CGLIB`는 spring-core JAR에 함께 패키징되기 시작해 <span class = "o">의존 관계를 명시해줄 필요는 없다.</span> 하지만 CGLIB가 하위클래스를 만들 수 있어야 하기 때문에 아래와 같은 조건을 가지고 `@Configuration`을 설정해줘야 한다.  

> + @Configuration을 설정한 클래스를 final로 정의해서는 안된다.  
> + 해당 클래스에는 반드시 인수가 없는 생성자가 제공되어야 한다.  

### @Bean  
`@Bean`어노테이션에는 `name`, `autowire`, `initMethod`, `destoryMethod`와 같은 속성이 이용 가능하다.  
또한 `@Bean`을 설정한 메서드에는  

> + 지연초기화를 설정하는`@Lazy`,  
>
> + 암시적 의존 관계를 맺는`@DependsOn`(암시적 의존 관계를 맺는 빈 들에 대해 명시하여 의존 관계를 해결하기 때문에 명시적 의존관계라고 생각되긴 한다.),  
>
> + Autowire시에 우선관계를 설정해주는 `@Primary`,  
>
> + 빈 스코프를 설정해줄 수 있는 `@Scope`  

위와 같은 것들을 아래와 같이 덧붙여 설정해줄 수 있다.  

```java  
... 

@Bean(name = "customerRegistrationService")
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public CustomerRegistrationService customerRegistrationService() {
    return new CustomerRegistrationServiceImpl();
	}  
...
```  
### 빈 의존 관계 주입  
`@Bean`을 통해서 빈을 만들 수 있음을 위에서 확인할 수 있었다.  
이렇게 만들어진 빈 들은 아래의 방법을 통해서 `@Bean` 메서드가 생성하는 빈의 의존관계를 만족시킬 수 있다.  
  
> + 명시적으로 의존 관계를 생성하고 반환하는 `@Bean` 메서드를 <span class ="o"> 호출해서 </span> 의존 관계를 얻어온다.  
> + `@Bean` 메서드 인수로 의존 관계를 지정해준다. 이 때 의존 관계에 대한 책임은 <span class = "o">스프링 컨테이너</span>가 의존 관계를 메서드 인수로 공급(주입)한다.  
> + 빈 클래스에서 `@AutoWired`, `@Inject`, `@Resource`등을 사용해 의존 관계를 자동 연결한다.  

아래는 호출을 통해서 의존 관계를 얻어오는 방법의 예시이다.  
```java  
package sample.spring.chapter07.bankapp;
...
@Configuration
public class BankAppConfiguration{
    @Bean(name = "accountStatementService")
    public AccountStatementService accountStatementService(){
        AccountStatementServiceImpl accountStatementSerice(){
            AccountStatementServiceImpl accountStatementServiceImpl = new  AccountStatementServiceImpl();
            accountStatementServiceImpl.setAcountStatementDao(accountStatementDao()); // accountStatementDao()를 호출해 빈 의존 관계를 얻어온다.
            // 생략되어있지만 accountStatementDao에는 @Bean이 설정되어있다.
            ...
        }
    }
}
```  
다음은 `@Bean` 메서드를 인수로 전달해 의존 관계를 지정해주는 방법에 관한 예시이다.  
```java  
...
@Configuration
public class BankAppConfiguration{
    @Bean(name = "accountStatementService")
    public AccountStatementService accountStatementService(AccountStatementDao accountStatementDao){//인수로서 accountStatementDao를 전달함으로써 의존관계를 설정한다.
        AccountStatementServiceImpl accountStatementSerice(){
            AccountStatementServiceImpl accountStatementServiceImpl = new  AccountStatementServiceImpl();
            accountStatementServiceimpl.setAccountStatementDao(accountStatementDao);
            return accountStatementServiceImpl;
        }
        ...
    }
    ...
}  
```

다음은 `@Autowired` 을 사용하여 의존 관계를 자동연결하는 방법이다.  
아래에서는 `@Autowired` 어노테이션은 `fixedDepositDao` 메서드가 생성한 `FixedDepositDao`빈을 자동 연결한다.
```java  
@Configuration
public class BankAppConfiguration{
    @Bean(name = "fixedDepositService")
    public FixedDepositService fixedDepositService(FixedDepositDao fixedDepositDao){
        return new FixedDepositServiceImpl();
    }
    ...
}
...
}  


...
public class FixedDepositServiceImpl implements FixedDepositService{
    @Autowired
    private FixedDepositDao fixedDepositDao;
}
```  