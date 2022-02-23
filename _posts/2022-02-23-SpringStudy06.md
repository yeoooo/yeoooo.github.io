---
title: "자바 기반의 컨테이너 설정"
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
이전 스프링 스터디에서는 자바 기반의 빈 설정에 대해서 알아봤다.  
<span class = "o">자바 기반 컨테이너 설정</span>과 <span class = "o">XML기반 설정</span> 중 무엇을 택해야 하는가는 책에서 취향에 따라 방법을 선택하면 된다고한다.(뭐가 특별히 더 좋고 그런 것은 없는 것 같다.)  
하지만 주의할 점으로는 <span class = "o"> 한 프로젝트 안에서는 한 접근 방법만 사용하는 것을 권장</span>한다고 한다.  
  
## 스프링 컨테이너 설정  
빈의 소스로 `@Confiuration`을 설정한 클래스를 사용하려면 `AnnotationConfigApplicationContext`의 인스턴스를 만들어서 스프링 컨테이너를 표현해야한다.  
### @Configuration  
---------------------
```java  
public static void main(String args[]) throws Exception{
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(BankAppConfiguration.class);
    ...
}
```

또한 `@Component`나 `JSR330`의 `@Named`를 설정한 클래스나 `@Bean` 메서드를 통해 정의한 빈 클래스를 `AnnotationConfigApplicationContext`의 <span class = "o">생성자의 인수</span>로써 넘길 수 있다.  

configClass JVM 프로퍼티를 이용해 어떤 `@Configuration` 클래스를 `AnnotationConfigApplicationContext`에 등록할지 결정해줄 수 있다.

```java  
...
if(context.getEnvironment().getProperty("configClass").eqaulsIgnoreCase("myConfig")){
    context.register(MyConfig.class);
else{
    context.register(YourConfig.class);
    }
}
context.refresh();
...
```  

`getEnvironment()`는 `Profile`, 즉 <span class ="o">어떤 환경에 필요한 설정을 얻어올 수 있다</span>.  
`context` 객체로부터 `Profile`(아무런 Profile을 적용하지 않았다면 default 프로파일이 적용된다.)이 적용된 어떤`Environment`객체를 얻어내고
이로부터 JVM 프로퍼티가 `getProperty()`를 통해 configClass의 프로퍼티 값을 얻어온다.  
이 때 프로퍼티 값이 myConfig라면 `AnnotationConfigApplicationContext`에 `MyConfig`가 추가되게 되고,  
아니라면 `YourConfig`가 추가되게 된다.  

마지막으로 `refresh()`를 통해서 `AnnotationConfigApplicationContext` 인스턴스가 등록된 클래스의 소유권을 확보하도록 만들어줘야 한다.  

### scan  
--------------------
  
지금까지와 같이 명시적으로 `@Configuration` 클래스를 추가하는 방식 말고도 `scan()`을 통해서 패키지를 지정해 컴포넌트를 스캔할 수 있다.  
`scan`메서드는 `<component-scan>`엘리먼트와 같은 역할을 하는데,  
`@Component`나 `@Named`를 설정한 클래스를 찾아서 스프링 컨테이너에 등록해준다.  
```java
context.scan("sample.spring", "com.sample")
```  
위와 같은 상황에서는 두 패키지와 그 하위에 있는 모든 패키지에 있는 `@Component`가 설정되어있는 클래스를 검색한다.  
`@Component`클래스를 찾게되면 `AnnotationConfiogApplicationContext`인스턴스에 클래스를 추가해준다.  


