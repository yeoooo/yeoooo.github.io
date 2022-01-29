---
title: "싱글턴과 프로토타입 스코프 빈의 의존 관계"
excerpt: "3주차 스프링 스터디 - 스프링파전"

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
4장에서는  _property_ 엘리먼트와 _constructor-arg_ 엘리먼트의 _ref_ 속성을 통해 빈을 설정하는 것 이외에도 _내부 빈_ 을 통해서 빈을 설정할 수 있다고 먼저 설명한다.  
또한 빈 정의들 간의 _암시적 관계_ 와 이를 명시적 관계로 바꿔줄 수 있는 _depends-on_ 속성에 대해서도 언급했다.  

이 들은 모두 의존 관계를 설정해주는 일 이외의 개발 과정과 비슷하다고 생각되었기 때문에 좀 더 이해가 필요했던 싱글턴과 프로토타입 스코프 빈 간의 의존 관계에 대해서 정리하기로 했다.  

## 싱글턴 스코프 빈과 프로토타입 스코프 빈  
해당 내용은 챕터 2에서 자세하게 설명되어있다.  
원래 따로 포스팅을 하려고 했는데, 게으름과 쓸만한 내용인가 고민하느라 챕터4까지 포스팅을 미루다가 이참에 이곳에 정리하려 한다. 

싱글턴과 프로토타입을 먼저 언급하기 이전에 `스코프`의 의미가 아리송했다.  

스코프는 __빈이 생성된 컨테이너로부터 존재할 수 있는 범위__ 를 뜻한다. 이를 검색했을 때 자바스크립트의 스코프가 많이 검색되었는데 이는 개발 전역에서 비슷한 의미로 통용되기 때문인 것 같다. 

### 싱글턴 스코프 빈  
싱글턴 스코프 빈은 아래의 뜻을 가진 싱글턴 패턴이 적용된 빈이다.

    전역 변수를 사용하지 않고 객체를 하나만 생성하여 생성된 객체를 어디에서든지 참조할 수 있도록 하는 패턴  

싱글턴 빈은 다음과 같은 특징을 가진다.  

>1. 이 싱글턴 빈의 인스턴스는 그 인스턴스의 스프링 컨테이너가 생성될 때 <span class = "o">함께</span> 생성되며 그 스프링 컨테이너가 파괴될 때 <span class = "o">함께</span> 파괴된다.  
>  
>2. 스프링 컨테이너는 싱글턴 스코프 빈의 인스턴스를 <span class = "o">단 하나</span>만 만들고 해당 빈에 의존하는 모든 빈이 그 싱글턴 빈의 인스턴스를 공유한다.  
>
>3. 싱글턴 빈은 빈을 정의할 때 `scope` 속성을 따로 설정해주지 않는다면 default 값이다. 즉 `scope` 속성이 언급되지 않은 모든 빈은 싱글턴 빈이다.  
  
### 프로토타입 스코프 빈
프로토 타입 패턴 또한 디자인 패턴에 존재한다. 프로토 타입의 본 뜻은 원형, 초기 형태 등을 가리킨다.  
즉 프로토 타입 패턴은 어떠한 인스턴스의 원형이 되어 이에 대한 복제를 쉽게 하기 위한 패턴이다.  

프로토타입 빈은 이러한 패턴이 적용된 빈이며 다음과 같은 특징을 가진다.  
  
>1. 프로토 타입 빈을 생성하는 스프링 컨테이너는 항상 프로토타입 스코프 빈의 <span class ="o">새로운 인스턴스</span>를 반환한다.  
>
>2. 프로토타입 빈은 항상 `지연 초기화(lazy-init)`된다.  
>
>3. 프로토타입 빈을 생성하기 위해서는 `scope` 속성을 사용해야한다.  

특징 3에서 언급했듯 프로토 타입 빈을 생성하기 위해서는 `scope` 속성을 아래와 같이 명시해주어야 한다.  

```xml  
<bean id = "Abean" class = "example.spring.prototype" scope = "prototype">
```  
-------------
#### 지연초기화 (lazy-init)  
스프링 컨테이너에서는 <span class ="o">싱글턴 빈을 처음 요청받았을 때</span> 인스턴스를 생성하도록 설정할 수 있다.  
이를 `지연 초기화(lazy-init)`라고 하며 그 사용법은 아래와 같다.  
  
```xml  
<bean id ="lazyABean" class = "example.LazyBean" lazy-init = "true">  
```  

위 예제와 같이 `lazy-init` 속성 값을 `true`로 설정해준다면 스프링 컨테이너는 빈을 처음 요청받고 나서 인스턴스를 초기화한다.    
지연 초기화는 에러를 발견하는 시점에 영향을 주기 때문에 대부분의 애플리케이션에서는 미리 초기화를 시켜주는 것이 낫다고 한다.  

지금까지 설명했던 것과 같이 싱글턴 빈은 스프링 컨테이너가 생성될 때 생성된다. 그에반해 프로토타입 빈은 프로토타입 빈을 얻기 위해 getBean 메서드가 호출될 때 마다 생성된다.  
이 때 싱글턴 빈이 프로토타입 빈에 의존을 하는 경우가 생기거나 반대의 경우가 생기게 된다.  

---------------------
### 싱글턴 - 프로토타입 의존  
싱글턴 빈이 프로토 타입 빈에 의존하는 경우의 예제는 다음과 같다.  
```xml  
<bean id = "singletonA" class = "..." >  
    <constructor-arg name = "prototypeA" ref = "prototypeA">
    <constructor-arg name = "singletonDao" ref = "singletonDao">
</bean>
<bean id = "prototypeA" class = "..." scope = "prototype">
<bean id = "singletonDao" class = "...">
```  
  
ApplicationContext가 생성 될 때 먼저 프로토타입 스코프 `prototypeA`와 싱글턴 스코프 `singletonDao`가 생성되고 마지막으로 싱글턴 스코프 `singletonA`가 생성되게 되는데 스프링 컨테이너는 싱글턴 빈을 단 한 번 생성하기 때문에 singletonA의 의존 관계를 주입할 기회는 단 한 번이 있게 된다.  
이로 인해 컨테이너는 `prototypeA` 인스턴스를 `singletonA`에 한번만 주입하게 되고 이로인해 `singletonA` 빈은 <span class ="o">생애주기 동안에 동일한 `prototypeA` 빈의 인스턴스에 대한 참조를 유지</span>하게 된다.  

------------------  

아래의 접근 방법을 사용하면 이와 같은 의존관계에서 프로토타입 스코프 의존 관계의 새 인스턴스를 얻을 수 있게 된다.  

>1. 싱글턴 빈 클래스의 ApplicationContextAware 인터페이스 구현
>2. 스프링 beans 스키마의 <lookup-method>엘리먼트의 사용
>3. 스프링 beans 스키마의 <replace-method>엘리먼트의 사용  

---------------
### 프로토타입 - 싱글턴 의존  
위와 반대로 의존관계를 갖는 경우에는 다소 간단하게 느껴졌다.  
  
```xml  
<bean id = "prototypeA" class = "..." scope = "prototype">
    <constructor-arg name = "singletonA" ref = "singletonA">
    <constructor-arg name = "singletonDao" ref = "singletonDao">
</bean>
<bean id = "prototypeB" class = "..." scope = "prototype">  
<bean id = "singletonDao" class = "...">
```  
스프링 컨테이너는 먼저 `singletonDao`와 `prototypeB`를 생성한 후 `prototypeA`를 생성한다. 
싱글턴 빈은 ApplicationContext 인스턴스가 생성될 때 단 한 번 생성되는 반면 프로토타입 스코프 빈들은 ApplcationContext의 getBean메서드를 호출한 때 마다 출력되게 되고 스프링컨테이너는 `prototypeA`와 `prototypeB` 인스턴스를 <span class ="o">매번 새로 만들게</span>된다.  

추가적으로 위의 예제를 통해서 프로토타입 빈이 다른 프로토타입 빈에 의존한다면 스프링 컨테이너에 처음 요청한 프로토타입을 요청할 때마다 <span class ="o">새로운 두 프로토타입 인스턴스를 생성</span>한다는 것을 알 수 있었다.
gd
