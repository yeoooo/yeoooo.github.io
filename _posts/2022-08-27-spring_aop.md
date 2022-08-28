---
title: "[Spring] Spring AOP"
excerpt: "SW Academy day 20"

categories:
    Spring
tags:
    프레임워크
    백엔드
toc: true
comments: true
use_math: true
---

# Aspect-Oriented-Programming(AOP,관점 지향 프로그래밍)

> AOP?  
> 횡단 관심사의 분리를 허용함으로써 모듈성을 증가시키는 것이 목적인 프로그래밍 패러다임 코드 그 자체를 수정하지 않고 기존의 코드에 추가 동작(어드바이스)을 추가함으로써 수행하며, "함수의 이름이 'set'으로 시작하면 모든 함수 호출을 기록한다"와 같이 어느 코드가 포인트컷 사양을 통해 수정되는지를 따로 지정한다. 이를 통해 기능의 코드 핵심부를 어수선하게 채우지 않고도 비즈니스 로직에 핵심적이지 않은 동작들을 프로로그램에 추가할 수 있게 한다. -Wiki


AOP에서는 <span class = "o">관점</span>이 트랜잭션 매니지먼트나 로깅, 보안과 같이 여러 타입과 객체를 가로지르는 관심사에 대한 모듈화를 가능하게 하는데, Spring AOP는 이를 스프링 어플리케이션에서 사용할 수 있게 한다.  

<p align = "center"><img style = "width : 500px; height : 300px;" alt = "aop_concept.png" src = "../../assets/images/spring/aop_concept.png"></p>  

<p align =  "center" style = "font-size : 10px">
<a href = "https://www.codejava.net/frameworks/spring/understanding-spring-aop">
출처 : https://www.codejava.net/frameworks/spring/understanding-spring-aop
</a>
</p>  

위의 사진을 보면 알 수 있듯 Business, Presentation, Data Access Layer들을 관통하는 관심사들이 횡단(Cross Cutting) 관심사들이다.  
AOP는 동적으로 플러그와 같은 구성을 사용해 횡단 관심사를 실제 로직의 전, 후 주위에 추가할 수 있는 방법을 제공한다.  
## 특징
- Proxy 패턴 기반의 AOP 구현체
- 스프링 빈에만 AOP를 적용할 수 있다.
- 모든 AOP 기능을 지원하기 보다는 중복코드, 프록시 클래스 작성의 번거로움, 객체들 간의 관계 복잡도가 증가하는 등과 같이 스프링 IoC와 연동하는 엔터프라이즈 애플리케이션에서의 가장 흔한 문제에 대한 해결책을 지원하는 것이 목적

## AOP 주요 용어

- `타겟(Target)`
    - 핵심 기능을 담고있는 모듈, 부가기능을 부여할 대상
- `조인 포인트(Join Point)`
    - Advice가 적용될 수 있는 위치
    - Target 객체가 구현한 인터페이스의 모든 메서드
- `포인트 컷(PointCut)`
    - 어드바이스를 적용할 타겟의 메서드를 선별하는 표현식
    - 포인트컷 표현식은 execution으로 시작하고 메서드의 Signature를 비교하는 방법을 주로 이용
    - 형태는 @(Around/Before/After/…/)(”execution(접근지정자 반환타입 클래스(디렉토리 포함) 메서드(인자)”의 형태로 이뤄져 있으며 접근지정자를 제외하고는 와일드카드로써 `..` 와 `*` 가 사용가능 하다.
    - `..` 의 경우 하위 디렉토리를 포함하거나 인자의 개수를 특정할 수 없을 때 사용
    - `*` 의 경우 클래스나 메서드, 타입 등을 특정할 수 없을 때 사용
    
- `애스펙트 (Aspect)`
    - 어떠한 기능을 어떠한 메서드에 제공할지 정해진 것(Advice + Pointcut = Aspect)
    - Spring에서는 Aspect를 빈으로 등록해 사용(스프링 AOP는 빈으로 등록한 객체에 대해서만 동작한다.)
- `어드바이스 (Advice)`  
    - 구상도  
    <p align = "center">
    <img style = "width : 500px; height : 300px;" alt = "aop_advice.png" src = "../../assets/images/spring/aop_advice.png"/>
    </p> 
    <p align =  "center" style = "font-size : 10px">
    <a href = "https://www.codejava.net/frameworks/spring/understanding-spring-aop">
    출처 : https://www.codejava.net/frameworks/spring/understanding-spring-aop
    </a>
    </p>
    - Advice는 Target의 특정 Joinpoint에 제공할 (부가)기능
    - Advice에는 `@Before` , `@After`, `@Around`, `@AfterReturning`, `@AfterThrowing`이 존재  
        
- 위빙 (Weaving)
    - Target의 Join Point에 Advice를 적용하는 과정

## AOP 적용 방법

AOP를 적용할 때는 세가지의 방법이 존재한다. 

- 컴파일 시점  
    AspectJ 제공 AOP 방식, 컴파일 하기 전에 공통 관심사 코드를 삽입하는 방식으로 동작  
- 클래스 로딩 시점  
    클래스를 로딩할 때 ByteCode에 부가 기능을 삽입
- 런타임 시점  
  Spring 제공 AOP 방식, 소스 코드 내부 객체의 Proxy를 만들어내 실행 순서를 변경  

스프링 AOP 는 두 가지의 방식으로 AOP를 제공한다.

- <span class = "o">JDK dynamic proxies</span>(인터페이스 기반)
- <span class = "o">CGLib Proxy</span>(클래스 기반)

Logger를 AOP로써 target에 적용하는 예시

```java
@Aspect
@Component
//2022-07-29_yeoooo : Aspect는 내부에 Component를 포함하고 있지 않다. 따라서 Componenet를 추가해주어야
public class LoggingAspect {
    public static final Logger log = LoggerFactory.getLogger(LoggingAspect.class);

    /**
     * Advice
     */
    @Around("org.prgrms.kdt.aop.CommonPointcut.servicePublicMethodPointcut()")
    public Object log(ProceedingJoinPoint joinPoint) throws Throwable{
        log.info("Before method called. {}", joinPoint.getSignature().toString());
         Object result = joinPoint.proceed();
         log.info("After method called with result => {}", result);
         return result;
    }
}
```

 아래와 같이 PCD(Pointcut Designator)를 따로 메서드로서 정하고 사용할 수 있다.

```java
public class CommonPointcut {
    @Pointcut("execution(public * org.prgrms.kdt..*Service.*(..))")
    public void servicePublicMethodPointcut() {
    }

    @Pointcut("execution(* org.prgrms.kdt..*Repository.*(..))")
    public void repositoryMethodPointcut() {
    }

    @Pointcut("execution( * org.prgrms.kdt..*Repository.insert(..))")
    public void repositoryInsertMethodPointcut() {
    }
}
```

## AOP Annotation

AOP는 어노테이션을 통해서 구현할 수 도 있다.

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface TrackTime {

}
```

위와 같이 애너테이션을 만들어주고

```java
@Aspect
@Component
//2022-07-29_yeoooo : Aspect는 내부에 Component를 포함하고 있지 않다. 따라서 Componenet를 추가해주어야함.
public class LoggingAspect {
    public static final Logger log = LoggerFactory.getLogger(LoggingAspect.class);

    /**
     * Advice
     */
    @Around("@annotation(org.prgrms.kdt.aop.TrackTime)")
    public Object log(ProceedingJoinPoint joinPoint) throws Throwable{
        log.info("Before method called. {}", joinPoint.getSignature().toString());
        long startTime = System.nanoTime();
        long endTime = System.nanoTime() - startTime;
        Object result = joinPoint.proceed();
        log.info("After method called with result => {} and time taken {} nano sec", result, endTime);
        return result;
    }
}
```

아래와 같이 @Around안에 애너테이션과 그 상세 경로를 넣는다.
이후로 이 어드바이스를 적용할 타겟에 애너테이션을 작성해준다. 

```java
@Override
@TrackTime
public Voucher insert(Voucher voucher) {
    storage.put(voucher.getVoucherId(), voucher);
    return voucher;
}
```

### Reference.
---  
Spring AOP  
[Spring AOP : https://howtodoinjava.com/spring-aop-tutorial/](https://howtodoinjava.com/spring-aop-tutorial/)  

[Spring AOP : https://engkimbs.tistory.com/746](https://engkimbs.tistory.com/746)  
Pointcut 표현식의 자세한 이용방법  
[Spring AOP in Spring Certification : https://mossgreen.github.io/Spring-Certification-Spring-AOP/](https://mossgreen.github.io/Spring-Certification-Spring-AOP/)  

[Join Points and Pointcuts : https://www.eclipse.org/aspectj/doc/released/progguide/language-joinPoints.html](https://www.eclipse.org/aspectj/doc/released/progguide/language-joinPoints.html)
        