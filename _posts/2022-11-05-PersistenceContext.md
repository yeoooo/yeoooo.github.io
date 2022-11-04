---
title: "[JPA] 영속성 컨텍스트와 엔티티 매핑"
excerpt: "SW Academy 개인 프로젝트"

categories:
    JPA
tags:
    프레임워크
    백엔드
    SpringBoot
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


## Persistence Context, 영속성 컨텍스트

- JPA의 핵심 요소
- 엔티티를 영속화(영구 저장, Persistence Context내에서 관리)하는 환경이라는 의미
- 엔티티를 Persistence Context에 보관하고 관리한다.

### 특징

- **식별자**  
     영속성 컨텍스트는 key-value로 엔티티를 관리하기 때문에 식별자 값을 반드시 가져야한다.
    
- **영속성 컨텍스트와 DB**  
    JPA는 트랜잭션을 커밋하는 순간 영속성 컨텍스트에 새로 저장된 엔티티를 DB에 반영한다.  
    `flush()`를 통해 영속성 컨텍스트 내의 변경 사항을 DB와 동기화(변경 사항을 DB에 적용) 한다. 
    
- **1차 캐시**
  > 흐름  
       1. 최조 조회시 1차 캐시에 엔티티는 존재하지 않는다.  
       2. save() 가 호출되면 엔티티를 1차 캐시와 DB에 보관한다.  
       3. 1차 캐시에 보관된 결과를 반환한다.  
       4. 같은 엔티티가 조회됐을 때 1차 캐시에 같은 엔티티가 있다면 데이터베이스를 조회하지 않고 1차캐시의 엔티티를 그대로 반환한다.
    
    - 영속성 컨텍스트 내부에 <span class = "o">엔티티를 보관</span>하는 장소
    - key/value 형태를 통해 성능을 크게 높여주지는 않지만, JPA의 동작 메커니즘(<span class = "o">변경감지</span> 등)을 가능하게 한다.
    - 같은 엔티티에 대한 <span class = "o">객체의 동일성</span>을 보장한다(a == b).
    - <span class = "o">트랜잭션 단위로 작동</span>되며 한 트랜잭션 단위에서만 유효하다.  
<br>
    
- <span class = "o">객체의 동일성 보장</span>  
    1차 캐시를 이용함으로써 동일성을 보장받을 수 있다.
    
- **트랜잭션을 지원하는 쓰기 지연**  
    SQL 버퍼에 쿼리를 담은 후, 영속성 컨텍스트의 명령에 따라 DB에 전송되기 때문에 <span class = "o">한 번의 INSERT</span>만 발생한다. 이는 DB와의 통신 회수를 줄여주며 성능상 이점을 가져다준다.
    
- <span class = "o">변경 감지</span>  
    1차 캐시에 DB에서 처음 불러온 엔티티의 <span class = "o">스냅샷</span>(JPA는 엔티티를 영속성 컨텍스트에 보관할 때 최초 상태를 복사해 저장하며 이를 스냅샷이라고 한다.)을 저장하고 현재 내용과 차이점이 있다면 UPDATE SQL을 자동으로 수행한다.
    
- <span class = "o">지연로딩</span>  
    엔티티 조회 시점이 아닌 엔티티 내 연관관계를 참조할 때 해당 연관관계에 대한 SQL이 질의되는 기능  
    불필요한 조인 처리를 해 연관관계에 있는 <span class = "o">데이터까지 한 번에 조회하지 않기 때문</span>에 동작 부담을 줄일 수 있다.
    

## Entity

Entity는 RDB의 Table과 매핑되는 객체이다. 

- `@Entity` 로 엔티티임을 명시할 수 있다
- `@Table(name = ?)` 을 통해 RDB에 저장될 테이블명을 명시할 수 있다. 지정해주지 않을 경우 Class 명을 따라간다.
- `@Id` 를 통해 Primary Key를 명시해줄 수 있다.

### 생명주기

---

<p align = "center"><img style = "width : 500px; height : 300px;" alt = "lifecycle.png" src = "../../assets/images/spring/persistence_lifecycle.png"></p> 

- **영속** (managed)
    
    영속성 컨텍스트에 저장된 상태
    
- **비영속** (new / transient)
    
    영속성 컨텍스트와 전혀 관계가 없는 상태
    

- **준영속** (detached)
    
    영속성 컨텍스트에 저장되어 있다가 분리된 상태
    
    - 특징
        - <span class = "o">비영속 상태와 근사</span>
            
            영속성 컨텍스트가 관리하지 않기 때문에 제공하는 어떤 기능도 동작하지 않음
            
        - <span class = "o">식별값 보유</span>
            
            비영속 상태에서는 식별값이 없을 수 있으나 영속 상태 였던 준영속 상태는 식별값을 보유
            
        - <span class = "o">지연 로딩 불가</span>
            
            영속성 컨텍스트가 관리하지 않는 상태이기 때문에 지연 로딩 불가
            
- **삭제** (removed)
    
    삭제된 상태
    

---

1\. 객체가 Entity Manager가 호출한 persist()에 의해 영속화된다.  

2\. 영속화된 객체는 Persistence Context에 의해 관리되며 이는 (`detach()`/`clear()`/`close()`,`remove()`)에 따라 준영속, 비영속 상태로 바뀌게 된다.  

2.1.  
`detach()`: Persistence Context에서 더이상 해당 엔티티를 관리하지 않는다. (식별자는 존재한다.)  
`clear()` : 모든 엔티티를 더이상 관리하지 않는다.  
`close()` : 모든 엔티티를 더이상 관리하지 않고 Persistence Context를 종료시킨다.

2.1.1  
준영속 상태의 엔티티는 <span class = "o">merge()를 통해 다시 영속</span>되어질 수 있다.

2.1.2   
비영속 상태의 엔티티는 <span class = "o">persist()를 통해 다시 영속</span>되어질 수 있다. 비영속 상태에서  <span class = "o">flush()가 실행되면 DB와 동기화하여 DB내의 데이터를 삭제</span>한다.

3\.  
영속화된 객체는 flush()를 통해서 <span class = "o">DB와 동기화</span>를 시작한다. 

## EntityManager

- EntityManager는 엔티티에 CRUD 작업과 더불어 엔티티와 관련된 모든 일을 처리한다.
- 멀티 스레드 환경에서 안전하지 않다. 따라서 여러 Thread에서 동시 접근할 경우 동시성 이슈가 발생할 수 있다.
- 트랜잭션을 시작할 때 커넥션을 획득한다.

## Entity Manager Factory

- 엔티티를 관리하는 EntityManager를 생산하는 공장
- Thread Safe(멀티 스레드 환경에서도 안전하다.)

## Entity Mapping

### 데이터 베이스 스키마 생성

JPA와 hibernate를 이용하면 DB스키마(DDL)를 자동으로 생성해줄 수 있다.

```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: create-drop
                    //create
                    //update
                    //validate
                    //none
```

ddl-auto의 옵션으로는 위의 예시와 같이 설정할 수 있다.

- `create` : 기존 테이블을 삭제하고 새로 테이블을 생성한다. (DROP + CREATE)
- `create-drop` : 어플리케이션 종료시 생성한 DDL을 제거한다. (DROP + CREATE + DROP)
- `update`: 테이블, 엔티티 매핑 정보를 비교해 변경사항을 수정한다.
- `validate`: 테이블, 엔티티 매핑정보를 비교해서 차이가 있으면 어플리케이션을 실행하지 않는다.
- `none`: 자동 생성 기능을 생성하지 않는다.

### DDL 옵션

Entity 클래스 내에서 설정할 수 있는 DDL 옵션은 다음과 같다.

굳이 사용하지 않아도 되지만, 직접 개발하지 않은 개발자가 Entity를 확인함으로써 어떻게 구성되어있는지 가시적으로 표현함으로써 권장된다.

**Class level**  

---

- `@Entity`
    - `name`: 엔티티의 이름을 명시할 수 있다.
- `@Table`
    - `name` :테이블의 이름을 명시할 수 있다.

**Field level**  

---

- `@Column`: 영속화된 엔티티나 속성에 대해 매핑된 컬럼을 특정할 수 있다.
    - 옵션
        - `columnDefinition` : 컬럼에 대한 DDL을 생설할 때 사용된 SQL fragment를 명시할 수 있다.
        - `insertable` : 엔티티를 저장할 때 필드도 함께 저장한다. false일 경우 일기 전용으로 사용한다.
        - `length` : 컬럼의 길이를 명시할 수 있다.
        - `name` : 컬럼의 이름을 명시할 수 있다.
        - `nullable` : DB의 컬럼에 null값이 들어갈 수 있는지 명시할 수 있다.
        - `precision` : 10진수 컬럼으로 명시할 수 있다.
        - `scale`: 10진수 컬럼에 대한 범위를 명시할 수 있다.
        - `table`: 해당 컬럼을 포함하는 테이블의 이름을 명시할 수 있다.
        - `unique` : 해당 컬럼이 unique 한 key인지 명시할 수 있다.
        - `updatable` : 엔티티를 수정할 때 필드도 함께 저장한다. false일 경우 일기 전용으로 사용한다.
- `@Id`: 엔티티의 Primary key를 특정할 수 있다.
- `@GeneratedValue`: Primary key의 생성 전략을 제공할 수 있다.
    - 옵션
        - strategy: 생성 전략
            - GenerationType.`Auto` : persistence provider가 특정 DB에 대한 적절한 전략을 선택할 수 있게 한다.
            - GenerationType.`IDENTITY`: 기본키 생성을 데이터베이스에 위임한다. persist()시점에 INSERT 쿼리가 수행된다.
            - GenerationType.`SEQUENCE` : DB가 자동으로 숫자를 생성한다.
            - GenerationType.`TABLE`: 키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략
            - @GeneratedValue를 입력하지 않을경우 영속화 전에 애플리케이션에 직접 값을 할당한다.
        - generator: primary key 제너레이터의 이름  
  
## Ref
[https://velog.io/@bread_dd/JPA는-왜-지연-로딩을-사용할까](https://velog.io/@bread_dd/JPA%EB%8A%94-%EC%99%9C-%EC%A7%80%EC%97%B0-%EB%A1%9C%EB%94%A9%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%A0%EA%B9%8C)

[https://logical-code.tistory.com/140](https://logical-code.tistory.com/140)
