---
title: "[JPA] 매핑과 프록시"
excerpt: "연관관계와 프록시에 대한 정리"

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


## 연관관계 매핑

### 연관관계?


RDB(관계형 데이터베이스) 연관관계를 설정할 수 있다.

연관관계란 논리적으로 연관이 있는 두 테이블 사이의 연결, 참조를 의미하는데, **객체끼리의 연관관계는 참조(주소)로 연관관계**를 가지며 **테이블끼리의 연관관계는 FK를 통해 연관관계**를 가진다.

연관관계 매핑을 할 때는 아래의 <span class = "o">3가지 요소</span>들을 고려해야한다.

#### 방향

- 단방향
    
    `회원 → 주문`과 같이 한 방향으로만 참조 관계를 갖는 것을 단방향 관계라고 한다.
    
    - 코드 예시
        
        ```java
        @Getter
        class Member{
        	private long id;
        	private List<Order> orders;
        }
        
        @Getter
        class Order{
        	private String id;
        }
        
        Member member = new Member();
        Order order = member.getOrders(); //회원 -> 주문의 참조관계를 가진다.
        // order.getMember() 참조관계를 가질 수 있는 메서드가 존재하지 않는다. 
        // 따라서 회원 <- 주문의 참조관계는 가지지 않는다. == 단방향 관계
        ```
        
- 양방향
    
   `회원 → 주문, 회원 ← 주문`과 같이 서로 참조 관계를 갖는 것을 양방향 관계라고 한다.
    
    - 코드 예시
        
        ```java
        @Getter
        class Member{
        	private long id;
        	private List<Order> orders;
        }
        
        @Getter
        class Order{
        	private String id;
        	private Member orderedMember;
        }
        
        Member member = new Member();
        Order order = new Order();
        
        Member member = order.getOrderedMember();
        Order order = member.getOrders();
        //양쪽 모두 참조 관계를 가지는 양방향 관계를 가진다.
        ```
        

#### 다중성

테이블들은 서로 한개, 또는 N개의 연관관계를 가질 수 있다.

- `1 : N`
    
    1명의 회원은 N개의 주문을 할 수 있다. == 회원과 주문은 1:N(일대다)의 연관관계를 가진다.
    
- `N : 1`
    
    N명의 사람은 1개의 팀에 속할 수 있다.  == 사람과 팀은 N:1(다대일)의 연관관계를 가진다.
    
- `N : N`
    
    N명의 회원은 N개의 상품을 구매할 수 있다. == 회원과 상품은 N:N(다대다)의 연관관계를 가진다.
    

#### 연관관계 주인

- 두 객체가 **양방향 관계를 갖기 위해서**는 연관 관계의 <span class = "o">주인이 존재</span>해야 한다.
- 연관관계의 주인은 <span class = "o">FK(외래키)를 가지는 쪽</span>(1:N, N:1 관계에서 N이 주인이 된다)이 된다.
- 연관관계의 주인이 된 객체는 FK를 관리(등록, 삭제, 수정)한다.
- 연관관계의 주인이 아닌 객체는 FK 조회 외의 작업은 할 수 없다.
- `@mappedBy` 를 통해 연관관계의 주인임을 명시해줄 수 있다.


## 고급 매핑

JPA는 RDB의 테이블과 매핑된 Entity를 자바의 객체처럼 사용할 수 있도록 여러가지 고급 매핑 전략을 제공한다.

### 상속, @Inheritance

Mysql과 같은 RDB 자체는 상속을 지원하지 않는다. 하지만 JPA의 `@Inheritance` 를 이용해 DB에 상속과 같은 기능을 구현할 수 있다.

`@Inheritance` 를 사용하기 위해서는 조상이 될 클래스가 **abstract class**이어야 한다.

- 옵션
    - strategy
        - `JOINED` : 조상 클래스와 자손 클래스를 Join해 객체를 조합한다.(DB 내에 자손클래스에 대한 테이블이 생성된다.)
            
            <p align = "center"><img style = "width : 250px; height : 150px;" alt = "rel1.png" src = "../../assets/images/spring/rel_1.png"></p> 
            <p align = "center"><img style = "width : 250px; height : 150px;" alt = "rel2.png" src = "../../assets/images/spring/rel_2.png">
            <img style = "width : 250px; height : 150px;" alt = "rel3.png" src = "../../assets/images/spring/rel_3.png"></p> 
        
        조상클래스는 ITEM, 자손클래스는 FOOD, CAR, FURNITURE이다. 자손클래스에 관한 테이블이 생성되었다.
        
        - `SINGLE_TABLE` : DB내 조상 객체(Entity)에 DTYPE(구분자) 컬럼을 생성하고 자손 클래스의 이름을 넣는다. 추가적으로 테이블이 생성되지 않는다. (<span class = "o">대개 선호되는 방식</span> → 관리해야할 테이블이 불 필요하게 늘어나지 않는다.)
            
            
            <p align = "center"><img style = "width : 250px; height : 150px;" alt = "rel4.png" src = "../../assets/images/spring/rel_4.png"></p>
            
            <p align = "center"><img style = "width : 500px; height : 150px;" alt = "rel5.png" src = "../../assets/images/spring/rel_5.png"></p> 
            
        
        - `TABLE_PER_CLASS` : 추상 클래스인 조상 클래스가 생성되지 않고, 그 속성을 자손 클래스에 전달한다. (DB 내에 조상 클래스에 대한 테이블이 생성되지 **않는다**.)
            
            
            <p align = "center"><img style = "width : 250px; height : 150px;" alt = "rel6.png" src = "../../assets/images/spring/rel_6.png">
            <img style = "width : 250px; height : 150px;" alt = "rel7.png" src = "../../assets/images/spring/rel_7.png"></p> 
            

### BaseEntity, @MappedSuperclass

---

객체의 입장에서 공통 매핑 정보(속성)이 필요할 때 사용할 수 있다.(ex. id, name 등과 같은 공통 속성에 이용될 수 있다)

Entity자체는 아니므로 DB에 저장되지 않는다. 

```java
@MappedSuperclass
@Getter
@Setter
public class BaseEntity {
    @Column(name = "created_by")
    private String created_by;
    @Column(name ="created_at", columnDefinition = "TIMESTAMP")
    private LocalDateTime created_at;
}

@Entity
@Table(name = "orders")
@Getter
@Setter
public class Order extends BaseEntity{
    @Id
    @Column(name = "id")
    private String uuid;
    private String memo;
    @Enumerated(value = EnumType.STRING)
    private OrderStatus orderStatus;
    @Column(name= "order_datetime", columnDefinition = "TIMESTAMP")
    private LocalDateTime orderDatetime;

    //fk
    @Column(name = "member_id", insertable = false, updatable = false)
    private Long memberId;

    @ManyToOne //2022-08-10_yeoooo : 회원과 주문은 N:1연관관계
    @JoinColumn(name = "member_id", referencedColumnName = "id")
    private Member member;
}

Order order = new Order();
order.setUuid(UUID.randomUUID().toString());
order.setMemo("no contents");
order.setOrderDatetime(LocalDateTime.now());
order.setCreated_by("yeoooo");
order.setCreated_at(LocalDateTime.now());

save(order)
```

위와 같은 코드를 실행시켰을 때 DB에서 ORDER 테이블을 조회한다면 다음과 같은 결과가 나온다.

<p align = "center"><img style = "width : 800px; height : 150px;" alt = "rel8.png" src = "../../assets/images/spring/rel_8.png"></p>

### 식별자 클래스

---

JPA에서 식별자를 둘 이상 사용하기 위해서는 별도의 식별자 클래스를 만들어야 한다.

```java
public class Member{
	@Id
	private String id_a;
	@Id
	private String id_b;
}
```

JPA는 영속성 컨텍스트에 엔티티를 보관할 때, `@EqualAndHashCode` 를 이용해 동등성 비교를 한다. 

하지만 위 클래스와 같이 식별자가 두 개 이상 존재한다면 복합키를 식별하지 못해 runtime error를 발생시킨다.

JPA는 이러한 복합키를 관리하기 위해 두 가지 전략을 제공한다.

#### @IdClass

`@IdClass` 의 값에 어떤 클래스에서 해당 IdClass를 이용할 것인지 명시하고, IdClass의 키 값을 이용할 수 있다. 

```java
@IdClass(CombKey.class)
public class Member{
	@Id
	private String id_a;
	@Id
	private String id_b;
}

@EqualsAndHashCode
@NoArgsConstructor
@AllArgsConstructor
public class CombKey implements Serializable{
	private String id_a;
	private String id_b;//IdClass와 같은 key 필드를 가지고 있어야한다.
}

Parent parent = new Parent();
parent.setId_a("a");
parent.setId_b("b");
```

이렇게 `@IdClass` 를 이용하기 위해서는 사용할 클래스에서 다음과 같은 요건을 충족시켜야 한다.

- 식별자 클래스는 `public` 이어야 한다.
- `Serializable` 인터페이스를 구현해야 한다.
- `EqualsAndHashcode`를 구현해야한다.
- 기본 생성자가 있어야한다.

조회 시에 em.find()의 인자 값으로 지정한 복합키를 모두 넣어주면 된다.

#### @Embedded

`@IdClass`와 같이 복합키를 매핑해줄 수  있다.

```java

@EqualsAndHashCode
@NoArgsConstructor
@AllArgsConstructor
@Embeddable
public class ParentId implements Serializable {
    private String Id_A;
    private String Id_B;
}

@Entity
@Getter
@Setter
public class Parent {
    @EmbeddedId
    private ParentId id;
}

Parent parent = new Parent();
parent.setId(new ParentId("a", "b"));
```

기본 생성자와 private, EqualsAndHashCode, Serializable과 같은 요건은 @IdClass와 같이 충족시켜야 하지만 복합키로 사용할 클래스에 `@Embeddable` , 복합키를 사용할 클래스의 Key 필드에 `@EmbeddedId` 를 선언해 사용할 수 있다.

 복합키의 타입을 `@Embeddable`이 선언된 클래스로 해 가시성을 높여주고 복합키를 따로 설정해주어야 한다는 불편함을 줄여줘 선호된다.

## 프록시 객체, 연관관계

### 객체 그래프 탐색

객체는 객체 그래프(연관관계 그래프)를 제한없이 탐색할 수 있어야 한다.

(연관관계를 타고 다니며 객체를 참조할 수 있어야한다.) 

하지만 Entity는 RDB와 매핑되어 있어 자유롭게 객체를 탐색하는데 제한이 있는데, JPA는 <span class = "o">프록시 객체를 이용해 연관된 객체를 처음부터 DB에서 조회하지 않고 실제로 사용하는 시점에서 조회</span>할 수 있다.

orders.getMember()와 같은 메서드를 호출하기 이전에는 SELECT 쿼리를 날리지 않고 Proxy 객체와 매핑을 시켜놓고, getMember()와 같은 메서드를 호출했을 때 비로소 쿼리를 날리게 된다.

## Proxy


- 프록시 객체는 처음 사용할 때 단 한번 초기화 된다.
- 초기화 이 후로 프록시 객체를 통해서 실제 엔티티에 접근 가능하다.
- 초기화는 영속성 컨텍스트를 통해서 할 수 있다. 준영속 상태의 프록시를 초기화하면 LazyInitializationException(엔티티 객체가 아닌 프록시 객체이기 때문)예외가 발생한다.

## Loading


Load전략은 `@OneToMany`, `@ManyToOne`, `@ManyToMany` 와 같은 연관관계 어노테이션에서 fetch 옵션을 통해 설정할 수 있다.

```java
@OneToMany(fetch = FetchType.LAZY)
```

- FetchType.`LAZY`(지연 로딩)
    
    연관된 엔티티를 실제로 사용할 때 조회한다.
    
- FetchType.`EAGER`(즉시 로딩)
    
    엔티티 조회 시 연관된 엔티티를 함께 조회한다.
    

아래의 예시 코드를 통해 자세히 알아보자.

```java
@Test
public void proxyTest() throws Exception {
    EntityManager em = emf.createEntityManager();
    Order order = em.find(Order.class, uuid);
    Member member = order.getMember();//EAGER일 경우 이 때 Load

		log.info("MEMBER USE BEFORE IS_LOADED = {}",emf.getPersistenceUnitUtil().isLoaded(member));

    String nickName = member.getNickName();//LAZY일 경우 이 때 LOAD

		log.info("MEMBER USE AFTER IS_LOADED = {}",emf.getPersistenceUnitUtil().isLoaded(member));
		}
================================================================================
Order 클래스 내의 member 필드 로드 설정이 LAZY 인 경우
MEMBER USE BEFORE IS_LOADED = false
MEMBER USE AFTER IS_LOADED = true

Order 클래스 내의 member 필드 로드 설정이 EAGER인 경우
MEMBER USE BEFORE IS_LOADED = true
MEMBER USE AFTER IS_LOADED = true
```

- LAZY의 경우
    
    order.getMember()를 통해 얻어온 객체는 Proxy 객체이고, 이를 member.getNickName()을 통해 사용할 때  Entity객체가 된다.
    
- EAGER의 경우
    
    처음 order.getMember()를 통해 얻어온 객체는 DB와 통신해 가져온 Entity 객체이다.
    

## CASCADE(영속성 전이)


특정 엔티티를 영속 상태로 만듦과 동시에 연관된 엔티티를 영속하기 위해 사용할 수 있다.

CASCADE 옵션은 Load 전략과 마찬가지로 연관 관계 어노테이션의 옵션으로 값을 줄 수 있다.

```java
@OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
```

- 옵션, CascadeType
    - `ALL`
        
        해당 엔티티의 모든 변경에 연관된 엔티티를 영속시킨다.
        
    - `PERSIST`
        
        해당 엔티티가 영속될 때 연관된 엔티티를 영속시킨다.
        
    - `MERGE`
        
        해당 엔티티가 병합될 때 연관된 엔티티를 영속시킨다.
        
    - `REMOVE`
        
        해당 엔티티가 제거될 때 연관된 엔티티를 영속시킨다.
        
    - `REFRESH`
        
        해당 엔티티가 Refresh될 때 연관된 엔티티를 영속시킨다.
        
    - `DETACH`
        
        해당 엔티티가 DETACH(준영속화)될 때 연관된 엔티티를 영속시킨다.
        

## 고아 객체

부모 엔티티가 삭제 되거나 어떠한 상태 변경으로 자식 엔티티와 연관관계가 끊어졌을 때 자식 엔티티를 고아 객체라고 부른다.

이러한 고아객체는 DB내에서 자동으로 삭제되지 않는다. CASCADE 옵션과 같이 연관 관계 어노테이션의 옵션으로 `orphanRemoval` 을 줄 수 있다.

해당 옵션을 true로 설정하면, flush 순간에 RDB에서 삭제한다.

```java
@OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.ALL, orphanRemoval= true)
```

- 옵션
    - `true`
        
        고아객체를 자동으로 제거한다.
        
    - `false`(default)
        
        고아객체를 자동으로 제거하지 않는다.