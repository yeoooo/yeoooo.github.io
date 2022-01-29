---
title: "빈의 초기화와 정리(삭제), 커스텀화"
excerpt: "5주차 스프링 스터디 - 스프링파전"

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
벌써 스프링 스터디를 시작한지 6주차(0주차 포함, 챕터6)가 되었다.  

이번 5주차에서는 빈의 초기화와 정리 메서드를 정리하려고한다.  

지금까지 의존 관계 주입, 빈 정의에 대해서 배웠고 그 과정에서 빈과 컨테이너의 생명주기에 대해서도 어느정도 알게 되었다.  
  
### 컨테이너의 생명주기  
Spring 컨테이너는 아래와 같은 생명주기를 가진다.  
>생성 -> 빈 설정 -> 사용 -> 소멸  

### 빈의 생명주기  
빈은 아래와 같은 생명주기를 가진다.  
>객체 생성 -> 초기화 -> 사용 -> 소멸  
  
이러한 과정 속에서 `초기화`와 `소멸`에 대해서 알아보려고 한다.  
  
### 초기화  
빈은 `초기화`를 필요로 한다.  
부끄럽지만, 왜 초기화가 필요로 되는가 라는 생각이 먼저 들었다.  
따라서 간단히 초기화에 대해 조금 생각해보기로 했다.  

```java  
init_int = 1;
```  

초기화는 어떤 `변수` <span class ="o">의 초기값을 설정해주는 것</span>이며 이는 다른 프로그램이나 운영체제 등에서 같은 변수를 사용할 수 있기 때문에 지금 사용할 값을 `초기값`으로 지정하는 것과 같다고 생각했다.  
  
이는 빈에서도 마찬가지이며 어떤 빈을 이용하기 위해서 그 빈에 `프로퍼티`를 통한 초기 설정 값을 설정해주어 사용할 수 있는 상태로 만들어 주는 것 이다.  

## 빈의 초기화, 정리 로직의 커스텀화  
  
컨테이너의 생명주기에 따라 스프링 컨테이너는 빈 클래스의 생성자를 호출해서 빈을 생성하고, 이후 컨테이너가 빈을 초기화 한다.  
하지만 <span class = "o">초기화 이전에 어떤 </span>`로직`(파일을 열거나 데이터 베이스 연결 등)<span class = "o">을 실행</span>하고 싶다면 초기화 메서드를 `<bean>` 엘리먼트의 `init-method` 속성의 값으로 사용해 이를 커스텀화 할 수 있다.  
이는 스프링 컨테이너가 제거되기 전에 어떤 `로직`을 실행하고 싶을 때 도 마찬가지다. 정리 로직은 `destroy-method` 속성을 통해 커스텀화 할 수 있다.  
  
```java  
public void initDbConnect(){
    connection = DbConnection.getInstance();
}
public void releaseConnect(){
    connection.releaseConnection();
}

```  

```xml  
<bean id = "myDao" class = "example.spring" init-method = "initDbConnect" destroy-method = "releaseDbConnection">
```  
  
