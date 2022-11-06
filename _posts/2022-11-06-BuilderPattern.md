---
title: "[Java] DTO와 빌더패턴"
excerpt: "DTO와 빌더패턴은 왜 써야될까?"

categories:
    java
tags:
    DesignPattern
    
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


# DTO와 빌더패턴

## Data Transfer Object(DTO), 전송객체

위키피디아에서는 DTO에 대해 이렇게 설명한다.

> *데이터 전송 객체는 프로세스 간 데이터를 전달하는 객체이다. 프로세스 간 통신이 일반적으로 원격 인터페이스로 재정렬하면서 이루어지게 되는데 여기에서 각 호출의 비용이 많다는 점을 동기로 하여 이용하게 된다.*
> 

이해하기 쉽지 않지만, 이후에 DTO에 대해 간결하고 쉽게 그 의의에 대해 말한다.

> *DTO는 순수하게 데이터를 저장하고, 데이터에 대한 getter, setter 만을 가져야한다고 한다. 위키피디아에 따르면 DTO는 <span class = "o">어떠한 비즈니스 로직을 가져서는 안되며</span>, 저장, 검색, 직렬화, 역직렬화 로직만을 가져야 한다고 한다.*
> 

아직 제대로 익히지 못했기 때문에 드는 생각이겠지만, 위 설명을 보고 DTO와 Domain이 크게 다르지 않다고 생각들었고 이를 직접 비교했다.

```java
@Entity
@Table(name = "item")
@Getter
@Setter
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private Long id ;

    @ManyToOne
    @JoinColumn(name = "order_item_id", referencedColumnName = "id")
    private OrderItem orderItem;

    private int price;
    private int stockQuantity;

}
```

```java
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@JsonInclude(JsonInclude.Include.NON_NULL)
public class ItemDto {
    private Long id;
    private int price;
    private int stockQuantity;
    private ItemType type;
    private String chef;
    private Integer power;
    private Integer width;
    private Integer height;

}
```

어노테이션을 제외하고는 크게 달라보이지 않는다.

### 왜 DTO?

---

그렇다면 왜 DTO를 사용해야할까? 

DTO는 DB에서 거낸 데이터를 저장하는 Entitiy를 가지고 만드는 일종의 <span class = "o">Wrapper Class</span>이다. 보안상의 문제로 Entity를 Controller와 같은 클라이언트단과 마주하는 계층에 직접 전달하기 보다는 DTO를 통해 데이터를 교환하게 된다. DTO를 사용해야 하는 이유는 아래와 같다.

- Entity의 값이 변하게 되면, Repository 클래스의 flush()가 호출 될 때 그 <span class = "o">값이 DB에 반영</span>되고, 동시에 다른 로직에 영향을 미치게된다.
- Entity 중 Getter만을 사용해 원하는 데이터를 표시하기 어려울 수 있다. 이런 경우 Entriry와 DTO가 분리되어 있다면 DTO에 Presentation을 위한 필드나 로직을 추가하고 Entity에는 변경 사항을 만들지 않아 도메인 모델링을 깨지 않을 수 있다.
- 표현 계층에 필요하지 않은 <span class = "o">도메인 모델의 정보를 외부로 노출시키지 않아</span> 캡슐화를 통해 보안을 강화할 수 있다.
- Entity, 도메인 모델을 계층간 전송에 사용하면 <span class = "o">모델과 뷰가 강하게 결합</span>된다. DTO를 사용하면 결합을 느슨하게 할 수 있다.

## Builder 패턴

> *빌더 패턴은 OOP 환경에서 다양한 객체 생성에 대한 유연한 해결책으로 제공하기 위해 설계된 생성패턴이다. 복잡한 객체의 표현과 생성을 분리해 같은 생성 과정을 통해 다른 표현을 만들어낼 수 있다.* *-WikiPedia*
> 

DTO 를 사용할 때, 생성자 보다는 Builder 패턴을 사용하게 된다.

그 이유로는 크게 <span class = "o">가독성</span>, <span class = "o">불변성</span> 이 존재한다.

먼저 생성자와 빌더 패턴으로 객체를 만드는 과정을 비교해보자.

```java
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class OrderDto {
    private String uuid;
    private LocalDateTime orderDateTime;
    private OrderStatus orderStatus;
    private String memo;

    private MemberDto memberDto;
    private List<OrderItemDto> orderItemDtos;
}

OrderDto orderDto = new OrderDto(uuid, "문앞 보관 해주세요",
 LocalDateTime.now(), OrderStatus.OPENED, new MemberDto("lee", "yeoooo", "숲 속")
```

생성자로 만든 OrderDto 객체의 경우 코드가 길지 않고 짧다 .

하지만 인자를 생성자 변수의 순서에 따라 맞추어 넣어줘야하고 이 때 타입 오류가 나기쉽다.

사실, 타입 오류로 미리 오류를 알 수 있다면 다행이지만 동일한 타입일경우 컴파일러에서 오류를 찾아주지 않기 때문에 <span class = "o">더욱 오류를 발견하기 어려워진다</span>.

또한 생성자로 만든 객체는 Setter()의 접근이 가능해지기 때문에 도중애 데이터가 바뀔수도 있는 위험이 있다.

```java
OrderDto orderDto = OrderDto.builder()
        .uuid(uuid)
        .memo("문앞 보관 해주세요.")
        .orderDateTime(LocalDateTime.now())
        .orderStatus(OrderStatus.OPENED)
        .memberDto(
                MemberDto.builder()
                        .name("lee")
                        .nickName("yeoooo")
                        .address("숲 속")
                        .build()
              )
```

그에 비해  Builder 패턴으로 만들어진 객체는 그 코드가 확연히 길다. 

하지만 Builder 패턴을 이용하면  어떤 변수에 어떤 데이터를 넣어 주어야할지 <span class = "o">쉽게 판단</span>할 수 있게 되고 Builder 패턴의 특성상 객체가 <span class = "o">Immutable</span> 하기 때문에 그 데이터를 아무나 바꿀 수 없다는 장점을 갖게 된다.

이러한 Builder 패턴의 장점은 생성자의 인수가 많을 때 더욱 빛을 발하게 된다.

마지막으로 Builder패턴과 생성자를 이용한 객체 생성 방법을 비교해 표로 표현했다.  

## 한눈에 보기  

| 생성 방법 | Builder 패턴 | 생성자 |
| --- | --- | --- |
| 장점 | Immutable하다. | 코드가 간결하다. |
|  | 변수의 이름에 직접 값을 넣을 수 있어 가독성이 좋다. |  |
| 단점 | 코드가 길어진다. | 필드가 Setter 메서드로 인해 변경될 수 있다. |
|  |  | 어떤 자리에 어떤 값을 넣어줘야할 지 인지하고 있어야한다. |  

## Reference.

DTO

[https://velog.io/@ohzzi/Entity-DAO-DTO가-무엇이며-왜-사용할까](https://velog.io/@ohzzi/Entity-DAO-DTO%EA%B0%80-%EB%AC%B4%EC%97%87%EC%9D%B4%EB%A9%B0-%EC%99%9C-%EC%82%AC%EC%9A%A9%ED%95%A0%EA%B9%8C)

[https://hudi.blog/data-transfer-object/](https://hudi.blog/data-transfer-object/)

Builder Pattern

[https://mangkyu.tistory.com/163](https://mangkyu.tistory.com/163)

[https://dev-youngjun.tistory.com/197](https://dev-youngjun.tistory.com/197)

[https://yjna2316.github.io/study/2020/11/06/생성자와-빌더패턴/](https://yjna2316.github.io/study/2020/11/06/%EC%83%9D%EC%84%B1%EC%9E%90%EC%99%80-%EB%B9%8C%EB%8D%94%ED%8C%A8%ED%84%B4/)