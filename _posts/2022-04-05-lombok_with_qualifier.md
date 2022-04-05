---
title: 
    "[Spring] @RequiredArgsConstructor(lombok)와 @Qualifier(Spring)을 함께 사용하기"
excerpt: 
    lombok을 이용하는 프로젝트에서 @Qualifier를 사용하기
category: 
    Spring
tags: 
    Spring
    라이브러리
toc: 
    true

comments: 
    true

use_math: 
    false
---

<style type = 'text/css'>
    .o{
    font-weight: bold;
    color:orange;
    }
</style>
  
## 개요  
최근 강의를 들으면서 `생성자`와 `getter`, `setter`를 직접 구현하지 않아도 이를 알아서 만들어주는 `lombok` 라이브러리에 대해 알게 되었다. lombok은 생성자의 변수에서 `final`을 사용하는 경우 `@RequiredArgsConstructor`, 사용하지 않는 경우 `@NoArgsConstructor`를 이용해 생성자를 만들어준다.  

강의 내용을 따라가면서 <span class ="o">타입이 중복되는 빈 문제를 해결하기 위한 방안인 @Qualifier</span>를 사용하게 되었는데, 이를 lombok과 함께 사용하고 싶어 아래와 같이 작성했으나 오류가 났다.  
```java
@@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService{

    private final MemberRepository memberRepository;
    private final @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy;
    //NoUniqueBeanDefinitionException 발생
}
```
  
## 원인 및 해결  
lombok은 AnnotationProcessor를 통해서 @RequiredArgsConstructor를 인지해 생성자를 만든다.  
이 때 변수 타입과 변수 명은 인식하지만 <span class = "o">@Qualifier과 같은 어노테이션은 따로 인식할 수 없는 것이 원인</span>이 되었다.  
  
이를 해결하기 위해서는 @Qualifier를 이용하지 않고 <span class ="o">먼저 타입을 매칭시키고 이에 맞는 조건이 존재하지 않으면 이름을 통해 매칭시키는 @Autowired</span>의 작동 과정을 생각해 변수의 이름을 직접 바꿔줄 수 있지만, 이름을 변경시키는 방법으로는 번거로워질 가능성이 있기 때문에 사용하고 싶지않았다.  

이 때 lombok에 따로 설정값을 추가하여 @RequiredArgsConstructor와 @Qualifier를 사용할 수 있는 방법에 대해 알게 되었다.  
  
방법은 아래와 같다.  
  
    1. 프로젝트의 최상위 디렉토리에 lombok.config파일을 생성한다.  
    2. 생성한 config 파일에 lombok.copyablAannotations += org.springframew ork.beans.factory.annotation.Qualifier 를 추가한다.
    3. gradle cache를 삭제한다.  

이와 같은 방법을 이용해 문제없이 lombok과 @Qualifier을 함께 이용할 수 있게 되었다.  

gradle cache는 Intellij 기준으로 아래의 방법으로 삭제할 수 있다.  
1. <span class = "o">Intellij - File - Invalidate Caches...</span> 선택
    <p align = "center"><img alt = "gradle_cache.png" src = "../../assets/images/spring/gradle_cache.png"></p>  
2. <span class = "o">Clear file system cache and Local History</span> 체크 후 <span class = "o">Invalidate and Restart</span> 선택
    <p align = "center"><img alt = "gradle_cache2.png" src = "../../assets/images/spring/gradle_cache2.png"></p> 

