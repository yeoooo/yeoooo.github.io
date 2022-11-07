---
title: "[Spring] API 문서화"
excerpt: "문서화에 대해 알아보자"

categories:
    Spring
tags:
    SpringBoot
    백엔드
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


# API 문서화

> *데이터를 주고 받기 위한 방법과 그 규격*
> 

API를 통해 다른 플랫폼끼리 통신을 해 데이터를 주고받게 되며 이는 정말 많은 곳에서 사용된다.

이러한 API를 사용하기 위해서는 제작자 이외의 사람이 사용하기 위해 설명서가 필요하게되고,  제작자는 그 설명서를 만들어야한다.

하지만 어플리케이션에 기능이 추가되고, 복잡해짐에 따라 일일이 그 기능들을 명세하기에는 무리가 있다.

이러한 OAS(Open API Specification)를 지원해주는 프레임워크가 존재하며 크게 `Swagger`와 `Spring Rest Restdocs`를 이용한다.

<p align = "center">
<p align = "center"><img style = "width : 500px; height : 300px;" alt = "docs1.png" src = "../../assets/images/spring/docs1.png"></p> 

<p align =  "center" style = "font-size : 10px; color : gray;">
SwaggerDocs
</p>
<p align = "center"><img style = "width : 500px; height : 300px;" alt = "docs2.png" src = "../../assets/images/spring/docs2.png"></p> 

<p align =  "center" style = "font-size : 10px; color : gray;">
Spring REST Docs
</p>
</p>

## Swagger


> *Swagger는 개발자가 REST 웹 서비스를 설계, 빌드. 문서화, 소비 하는 일을 도와주는 대형 도구 생태계의 지원을 받는 오픈 소스 소프트웨어 프레임워크이다. -Wikipedia*
> 

위키에서 언급한 것 과 같이 사실 Swagger는 OAS(Open API Specification)을 제외하고도 REST에 관련된 많은 작업을 도와준다.

<p align = "center"><img style = "width : 500px; height : 300px;" alt = "docs3.png" src = "../../assets/images/spring/docs3.png"></p> 

Swagger의 홈페이지를 들어가보면, Swagger의 기능에 대해 설명한다.

### 기능

---

- API Design(설계)
    
    명세된 기준에 따라 API 를 디자인하고 모델링한다.
    
- API Build(빌드)
    
    API를 위한 안정적이고 재사용 가능한 코드를 대부분의 언어에서 빌드할 수 있다.
    
- API Documentation(문서화)
    
    상호작용하는 API 문서를 통해 개발자의 경험을 증진시킨다.
    

- API Testing(테스팅)
    
    간단하고 기능적인 테스트를 오버헤드 없이 작성한 API에서 수행한다.
    

- Standardize(표준화)
    
    API 아키텍쳐에 적용되는 API 작성 가이드라인을 설정하고 강제한다.
    

### 장점

---

- 현행화(API가 변경될 때 마다 적용)
- Parameter, 응답 정보, 예제, Spec 정보 전달의 용이성
- 실제 사용되는 Parameter로 테스트 가능

Swagger는 위와 같은 장점이 존재한다.  그렇다면 좀 더 간단하게 , Swagger는 왜 사용할까 ?

- 적용의 용이성
    
    `Spring REST Docs` 는 테스트 코드를 필히 작성해야하며 작성된 테스트 코드는 성공적으로 작동해야한다.
    
    하지만 이와 달리 Swagger는 간단한 코드를 통해 만들 수 있다.
    
- 테스트 UI 제공
    
    `Swagger`는 `Spring REST Docs` 에서 테스트와 병행해야 하는 것과 달리  문서 화면에서 API바로 테스트할 수 있다.
    

지금까지 Swagger가 무엇이고, 왜 사용하는지에 대해 알아보았다.  그렇다면 그와 비교되는 Spring REST Docs는 어떨까?

## Spring REST Docs

> *Spring REST Docs는 REST한 서비스를 문서화하는데 도움을준다. `Asciidoctor`로 직접 작성한 문서와 와 `Spring MVC Test`로 자동으로 생성된 코드 조각 (snippets)을 결합한다.  이러한 접근은 `Swagger` 로 생성한 문서의 한계로부터 자유롭게 한다.                                                                                                                                                                                                                                                                                                   - Spring REST Docs*
> 

다시 말해, Spring REST Docs는 Spring MVC Test와 Asciidoctor를 이용한 OAS 프레임워크이다.

Spring REST Docs를 사용하기 위해서는 필수적으로 `테스트도구`, `테스트 유틸리티 클래스`, `표현 언어`가 필요하다.

사용할 수 있는 도구들로는 아래와 같다.

| 테스트 도구 | 테스트 유틸리티 클래스 | 언어 |
| --- | --- | --- |
| JUnit | MockMVC(@WebMvcTest) | MarkDown |
| Spock | RestAssured(@SpringBootTest) | AsciiDoc |

## 테스트도구

### JUnit

- 장점
    - 라이브러리를 추가하지 않아도 된다.
    - 컴파일 시점에 오류를 잡기 쉽다.
- 단점
    - 동적 테스트 작성이 불편하다.

### Spock

- 장점
    - BDD(Behavior Driven  Development) 스타일로 직관적이다.
    - 동적 테스트를 하기에 용이하다.
- 단점
    - 라이브러리를 추가해야한다.
    - 컴파일 시점에 오류를 찾기가 어렵다.

## 테스트 유틸리티 클래스

### MockMVC(@WebMVCTest)

- 특징
    - @WebMVCTest로 수행(Controller Layer만 따로 테스트하기에 속도가 빠르다)이 가능하다.

### Rest Assured(@SpringBootTest)

- 장점
    - BDD 스타일로 직관적이다.
- 단점
    - 별도의 구성 없이는 @SpringBootTest로 수행해야한다.(전체 컨텍스트를 로드하게 돼 빈을 주입하기에 속도가 느려진다)
    

전체 컨텍스트를 로드해야하는 통합테스트를 수행하는 것이 아니라면, API 명세에는 MockMVC가 적절하다.

## 언어

### AsciiDoctor

Asciidoctor는 Ruby로 만들어진 AsciiDoc을 document model로 파싱하고 이를 다른 형태로 Converting해 결과로 내는 text processor 이다.

AsciiDoctor의 반환물은 다음과 같다.

- HTML5
- DocBook 5

- manual pages
- PDF
- EPUB 등

AsciiDoctor는 또한 Extension(확장), Converters(변환), Build Plugins(플러그인 빌드)와 AsciiDoc의 Author(작성), publish(배포)를 도와줄 도구에 대한 생태계를 갖는다.

계속 깊이 들어왔지만, 이해를 위해서 AsciiDoc에 관해서도 알아보자

### AsciiDoc?

AsciiDoc은 간단한 기호와 태그 등으로 문서 편집을 쉽고 보기좋게 만들어주는 경량 마크업 언어에 속한다. 

위 내용을 보았을 때 한 가지 언어가 떠오를텐데, MarkDown 또한 경량 마크업 언어의 한 종류이다.

사용법은 [https://asciidoctor.org/docs/asciidoc-writers-guide/#its-just-text-mate](https://asciidoctor.org/docs/asciidoc-writers-guide/#its-just-text-mate에서)  에서 자세하게 설명해준다.(AsciiDoctor만이 아닌 OAS에 대한 정리이기 때문에 생략한다)

정리하자면 AsciiDoc은 경량마크업언어 AsciiDoctor는 프로세서가 된다.

한편 Spring REST Docs에서는 AsciiDoc 뿐만 아니라 MarkDown 언어도 사용 가능하다.

MarkDown의 문법이 편하다는 사실은 익히 알지만, include가 되지 않는다는 단점이 있어 AsciiDoc이 선호되는 추세이다.

지금까지 Spring REST API에 사용되는 도구들에 대해 알아보았다.

그 중 내용을 조합해보면 `JUnit`, `MockMVC`, `AsciiDoc` 을 사용했을 때 최소한의 라이브러리로 구현할 수 있게 된다.

다시 Swagger와 Spring REST Docs의 비교로 돌아와서, Spring REST Docs의 특징은 다음과 같다.

### 특징
---

- 장점
    - Production Code에 영향이 없다.
    - 테스트 코드가 성공해야 문서 작성이 가능하다(잘못된 API 문서가 작성될 경우를 방지한다)
- 단점
    - 테스트 코드를 꼭 작성해야 한다.
    - 문서를 위한 테스트 코드를 관리해야한다.

## 한눈에 보기  
Spring REST Docs 와 Swagger의 장단점을 정리해 비교하자면 아래 표와 같다.

| OAS | Spring REST Docs | Swagger |
| --- | --- | --- |
| 장점 | Production Code에 영향이 없다. | 문서상에 API 테스트가 가능하다. |
|  | 테스트 코드가 성공해야 문서 작성이 가능하다(잘못된 API 문서가 작성될 경우를 방지한다) | 테스트 코드가 필요없다.  (== 적용이 쉽다) |
| 단점 | 테스트 코드를 꼭 작성해야 한다.(== 적용이 불편하다) | 프로덕션 코드에 관계 없고 테스트와 관련된 어노테이션이 추가된다.  |
|  | 문서를 위한 테스트 코드를 관리해야한다. | 프로덕션 코드에 추가되어 직관성이 떨어지고 유지보수가 어려워진다. |

## References.

[https://tecoble.techcourse.co.kr/post/2020-08-31-spring-swagger/](https://tecoble.techcourse.co.kr/post/2020-08-31-spring-swagger/)[https://velog.io/@monkeydugi/Spring-Rest-Docs-채택-이유](https://velog.io/@monkeydugi/Spring-Rest-Docs-%EC%B1%84%ED%83%9D-%EC%9D%B4%EC%9C%A0)

[https://techblog.woowahan.com/2597/](https://techblog.woowahan.com/2597/)

 [https://asciidoctor.org](https://asciidoctor.org/)

[https://galid1.tistory.com/641](https://galid1.tistory.com/641)