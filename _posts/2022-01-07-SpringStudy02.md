---
title: "빈 정의 상속"
excerpt: "2주차 스프링 스터디 - 스프링파전"

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

## 개요   
자바에는 객체지향언어의 특성으로써 같은 설정, 속성, 함수 등 어떤 데이터를 공유한다면 클래스간 '상속'을 사용할 수 있다.  
이는 스프링의 빈에서도 가능하며 빈 정의를 더욱 간결화 시킬 수 있다.  

## 빈 정의의 상속, 부모 빈 정의가 abstract인 경우
상속을 통해서 상속하거나 상속 받을 수 있는 정보는 아래와 같다.  

    1. <property>  
    2. <construct-arg>  
    3. 메서드 오버라이드  
    4. 초기화, 정리 메서드  
    5. 팩토리 메서드  

상속은 아래와 같이 \<parent> 속성을 이용하여 그 부모 빈(=상위 빈)을 지정해줄 수 있다.  
```xml  
<bean id = "aBean" parent = "aBeanTemplate">
```  
상속 예제를 통해서 더 자세히 보도록 한다.  
```xml  
<bean id = "dbOps" class = "sample.spring.someapp.utils.dbOps"/>
<bean id = "abBeanTemplate">
    <property name = "dbOps" ref = "dbOps"/>
</bean>

<bean id = "aBean" class = "sample.spring.someapp.dao.aBeanImpl">
    <property name = "dbOps" ref = "dbOps">
</bean>

<bean id = "bBean" class = "sample.spring.someapp.dao.bBeanImpl"/>
    <property name = "dbOps" ref= "dbOps">
</bean>
```
위 xml 빈 정의에서 aBean과 bBean은 dbOps인스턴스를 참조하는 dbOps 프로퍼티가 정의되어있다.  
즉 같은 속성을 공유하고 있는 것인데, 이에 상속을 사용하면 아래와 같이 수정할 수 있다.  
```xml
<bean id = "dbOps" class = "sample.spring.someapp.utils.dbOps"/>

<bean id = "abBeanTemplate" abstract = "true">
    <property name = "dbOps" ref = "dbOps"/>
</bean>

<bean id = "aBean" parent = "abBeanTemplate" class = "sample.spring.someapp.dao.aBeanImpl"/>
<bean id = "bBean" parent = "abBeanTemplate" class = "sample.spring.someapp.dao.bBeanImpl"/>
```  
aBean과 bBean은 parent속성을 통해서 abBeanTemplate를 부모빈으로서 속성을 상속받는다.  
이 때 abBeanTemplate는 property엘리먼트에 의해 dbOps 의존 관계가 정의되어 있으므로 aBean과 bBean이 dbOps에 대한 의존관계를 상속 받을 수 있다.  

### 빈 정의에서의 abstract
이 떄 추가로 abBeanTemplate에는 abstract 속성이 존재하는데 이를 true로 만들어주면 이 빈은 <span style = "color : orange">추상 빈</span>이 된다. 그리고 추상 빈은 스프링 컨테이너에서 그 빈 정의에 대한 빈을 <span style = "color : orange">생성하지 않는다</span>. 이는 추상 빈에 의존하는 빈을 정의할 수 없다는 뜻이 될 수 있다.  

부모 빈 정의인 abBeanTemplate에는 class 속성이 지정되어 있지 않다.  
class가 지정되어 있지 않다고해서 그 null로서 지정되어있는 것이 아니라 aBean, bBean 즉 자식 빈 정의가 부모 빈 정의의 class 속성을 지정한다.  
이렇게 되면, 원치 않는 결과를 얻을 수 있기 때문에 class 속성을 지정하지 않는 빈은 추상 빈으로 만들어 주어야 그 빈 인스턴스가 생성되지 않는다.  

## 부모 빈 정의가 abstract가 아닌 경우  
아래는 부모 빈 정의가 abstract가 아니고, 자식 빈에 추가 의존 관계를 정의하는 상속 예제이다.  
```xml  
<bean id = "serviceTemplate" class = "sample.spring.someapp.base.ServiceTemplate">
    <property name = "a" ref = "a"/>
    <property name = "b" ref = "b"/>
    <property name = "c" ref = "c"/>
</bean>

<bean id = "aService" class = "...aServiceImpl" parent = "serviceTemplate">
    <property name = "daoA" ref = "daoA"/>
</bean>

<bean id = "bService" class = "...bServiceImpl" parent = "serviceTemplate">
    <property name = "daoB" ref = "daoB"/>
</bean>

<bean id = "aController" class = "...aControllerImpl">
    <property name = "serviceTemplate" ref = "serviceTemplate"/>
</bean>
```  
serviceTemplate 빈 정의는 aService 빈 정의와 bService 빈 정의를 자식 빈으로 두는 둘의 부모 빈 정의의다.  
이 때 serviceTemplate는 abstract가 아니며 class 속성은 ServiceTemplate 클래스로 지정되어있다.  
부모 빈 정의가 abstract인 경우에서의 예제와 달리 이번 예제에서는 각 aService와 bService 빈이 aDao와 bDao를 정의한다.  

이 때 자식 빈 정의는 부모 빈 저의의 프로퍼티를 상속 받기 때문에 a,b,c 프로퍼티에 대한 <span style = "color : orange">세터 메서드가 반드시 정의되어있어야</span> 한다.  
이 예제를 통해서 <span style= "color : orange">부모 빈 정의가 꼭 추상일 필요는 없으며 자식 빈 정의에서 프로퍼티가 추가적으로 정의 될 수 있음을 확인할 수 있다.</span> 
아래는 bServiceImpl에 대한 코드이다.

```java  
public class bServiceImpl extends ServiceTemplate implements bService{
    private BDao bDao;
    
    public void setBDao(BDao bDao){
        this.bDao = bDao;
    }
    @Override
    public someClass someMethod(){
        return bDao.someMethod();
    }
}
```
## 팩토리 메서드 설정 상속  
처음 언급되었듯, 빈 상속을 통해서 팩토리 메서드 정의 또한 부모 빈 정의에서 상속받을 수 있다.  
아래는 ControllerFactory 클래스 예제이다.  
```java  
public class ControllerFactory{
    public Object getController(String controllerName){
        Object controller = null;
        if ("fixedDepositController".equalsIgnoreCase(controllerName)){
            controller = new FixedDepositControllerImpl();
        }
        if ("personalBankingController".equalsIgnoreCase(controllerName)){
            controller = new PersonalBankingControllerImpl();
        }
        return controller
    }
}
```  
팩토리메서드로써 인수에 원하는 인수를 넣는다면 그에 맞게 인스턴스를 반환하는 모습을 보이고있다.  
아래는 이를 상속하는 xml 예제이다  
```xml  
<bean id = "controllerFactory" class = "sample.spring.someapp.controller.ControllerFactory"/>
<bean id = "controllerTemplate" factory-bean = "controllerFactory" factory-method = "getController" abstract = "true"/>
<bean id = "fixedDepositController" parent = "controllerTemplate">
    <contructor-arg index = "0" value = "fixedDepositContorller"/>
    <property name = "fixedDepositService" ref = "fixedDepositService"/>
</bean>

<bean id = "personalBankingController" parent = "controllerTemplate">
    <constructor-arg index = "0" value = "personalBankingController"/>
    <property name = "personalBankingService" ref= "personalBankingService"/>
</bean>
```  
다른 상속들과 큰 흐름은 비슷하지만, factory-bean 속성을 이용해 어떤 팩토리에서 factory-method로 지정된 factory-method를 이용해 빈 인스턴스를 생성하라고 지정되어있다.  
