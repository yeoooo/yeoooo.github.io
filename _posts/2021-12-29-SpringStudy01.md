---
title: "생성자 기반 DI와 세터 기반 DI"
excerpt: "1주차 스프링 스터디 - 스프링파전"

categories:
    Spring
tags:
    프레임워크
    백엔드
    배워서바로쓰는스프링프레임워크
toc: true
comments: true
use_math: false
---

## 개요  
첫 주차 간단히 스프링이 무엇인지 알아보며 진행한 OT에서 DI(Dependency Injection)이 무엇인지 알아보았다.  
첫번째 스터디에서는 챕터 1 스프링에 대한 소개를, 이번 주 차에서는 챕터 2 스프링의 기초에 대해 각자 발표했다.  
이 중 다른 것 보다 좀 더 생각해볼 필요가 있는 소주제를 포스팅하기로 했다.  
  
## DI(Dependency Injection) - 세터와 생성자
스프링에서는 DI를 통해서 객체와 객체간 의존 관계를 설정해줄 수 있다.  
이를 설정하는 방법에는 먼저 setter메서드를 통한 방법과 생성자 인수를 통한 방법이 존재한다.  

### 세터(수정자) 기반 DI  
세터 기반 DI는 bean 설정 안에서 \<property\>엘리먼트를 통해서 빈 의존 관계를 설정할 수 있다. 
  
java  
```java  
public class PersonalBankingService{
    private JmsMessageSender jmsMessageSender;

    public void setJmsMessageSender(JJmsMessageSender jmsMessageSender){ #setter
        this.jmsMessageSender = jmsMessageSender;
    }
    ...
}
```  
빈 정의 xml
```xml
<bean id = "personalBankingService" class = "PersonalBankingService"> #빈 정의
    <property name = "jmsMessageSender" ref = "jmsMessageSender"/> #속성과 참조 빈. ref 속성은 name 속성에 대응하는 (자바빈 스타일 세터 메서드/자바 내의 세터 메서드)에 전달
</bean>

<bean id = "jmsMessageSender" class "JmsMessageSender">
</bean>
```  

### 생성자(constructor) 기반 DI  
생성자 기반 DI는 생성자의 인수를 통해서 빈의 의존 관계를 빈 클래스에 전달한다.  

java
```java
public class PersonalBankingService{
    private JmsMessageSender jmsMessageSender;
    private EmailMessageSender emailMessageSender;

    public PersonalBankingService(JmsMessageSender jmsMessagesSender,
    EmailMessageSender emailMessageSender) #의존 관계를 빈 클래스의 생성자 인수로 입력한다.
    
    this.jmsMessageSender = jmsMessageSender;
    this.emailMessageSender = emailMessageSender;
}
```
java에서 정의한 빈 클래스는 xml에서 <constructor-arg> 엘리먼트로 의존 관계를 제공한다.  

빈 정의 xml
```xml  
<bean id = "personalBankingService" class = "PersonalBankingService">
    <constructor-arg index = "0" ref = "jmsMessageSender"/>
    <constructor-arg index = "1" ref = "emailMessageSender"/>
</bean>

<bean id = "jmsMessageSender" class = "JmsMessageSender">
...
</bean>

<bean id = "emailMessageSender" class = "EmailMessageSender">
...
</bean>
```
생성자 기반 DI에서는 <constructor-arg> 엘리먼트를 통해서 빈 클래스의 인스턴스의 생성자에 전달할 인수를 지정할 수 있다.  
여기서 속성 index는 생성자 인수의 순서를 보여주며 배열과 비슷하게 '0'이 첫번째 인수로 지정된다.  

이 때 주의할 점은 생성자 인수 사이에 서로 <span style = "color : orange">상속</span>관계가 없다면 index 속성은 지정하지 않는다. 즉, 타입이 다르다던지, 모종의 이유로 클래스 별로 서로 구분 가능한 객체라면 index 속성을 설정할 필요가 없다.  

\<constructor-arg>엘리먼트의 ref속성은 세터 기반 DI 에서의 <property> 엘리먼트의 ref와 같이 빈에 참조를 전달하는 기능을한다.  

추가 적으로 찾아본 내용인데, 생성자(Constructor) DI, 세터기반 DI 뿐 아니라 Autowired 어노테이션을 사용하는 필드 기반 주입도 존재한다.  

### 필드 기반 DI  
  
```java
public class PersonalBankingService{
    @Autowired #Autowired어노테이션을 활용해준다.
    JmsMessageSender jmsMessageSender;
    @Autowired
    EmailMessageSender emialMessageSender;
    ...
}  
```
Autowiring을 활용한 필드 기반 DI는 설정파일을 사용하지 않고 DI 컨테이너에서 Bean을 주입받을 수 있다.  
Autowiring은 또 타입, 이름 방식으로 나뉘어 지는데 더 깊게 알아보지는 않으려고 한다(이유는 아래 추가적인 내용에 작성되어있다.).  

### 세터 기반 DI & 생성자 기반 DI  
위에서 언급했던 세터 기반 DI와 생성자 기반 DI를 함께 혼용하여 사용할 수도 있다.  
왜 이렇게 혼용해서 써야할까 라는 의문이 먼저 들었는데, 이를 통해 두 엘리먼트(\<constructor-arg>, \<property>)안에 <span style = "color : orange">value 속성</span>을 이용해 빈에 필요한 <span style = "color : orange">설정 정보(간단한 String 값)</span>을 전달할 수 있다.  

java  
```java
public class PersonalBankingService{
    private JmsMessageSender jmsMessageSender;
    private EmailMessageSender emailMessageSender;

    public PersonalBankingService(JmsMessageSender jmsMessageSender){#constructor
        this.jmsMessageSender = jmsMessageSender;
    }
    public void setEmailMessageSender(EmailMessageSender emailMessageSender){#setter
        this.emailMessageSender = emailMessageSender;
    }
}
```  

빈 정의 xml  

```xml  
<bean id = "dataSource" class = "PersonalBankingService">
    <constructor-arg index = "0" ref = "jmsMessageSender" />#constructor, value 속성을 추가해 원하는 설정값을 넘길 수 있다.  
                                                            #<property name = "host" value = "smtp.gmail.com">
    <property name = "emailMessageSender" ref = "emailMessageSender"/>#setter
</bean>
```  

### 추가적인 내용  
요즘은 필드 주입을 사용하지 않는다고 한다.  
생성자 기반 DI는 <span style = "color : orange">필수 종속성</span>에, 세터 기반 DI는 <span style = "color : orange">선택적 종속성</span>에 사용하는 것이 좋고  
이를 사용했을 때 얻을 수 있는 이점이 <span style = "color : orange">NullPointExceptio 방지</span>와 <span style = "color : orange">final키워드의 사용 가능</span>, <span style = "color : orange">순환 참조를 앱 구동시 검출할 수 있는 점</span>이 있으며 이 때문에 필드 기반 DI는 지양하고, 생성자 기반 DI를 자주 사용하는 것이 좋다고 한다.  
이는 [Spring Core](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-beanfactory)의 공식사이트의 내용이며 자세한 내용은 원문을 읽어보는 것이 좋겠다.

## 출처  
블로그 : [[Spring/Core] DI(의존성 주입)은 생성자 주입을 사용해라](https://lee1535.tistory.com/117)
