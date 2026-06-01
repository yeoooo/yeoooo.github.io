---
title: "[Project] JWT 공개 식별자와 Gateway 인증 구조"
excerpt: "내부 id를 숨기고 gateway가 token 검증을 담당하도록 변경하기"
categories:
    Project
tags:
    백엔드
    JWT
    Gateway
    Security
toc: true
comments: true
use_math: false
---

<style type = 'text/css'>
    .o{
    font-weight: bold;
    color:orange;
    }
</style>

## 개요

JWT에 내부 DB id가 노출되는 부분을 정리했다.

처음에는 내부 account id를 그대로 사용해도 큰 문제가 없어 보였다.  
하지만 외부 token이나 API 경로에 내부 id가 드러나는 것은 피하는 게 맞다고 판단했다.

## 공개 식별자

JWT에는 내부 id 대신 공개 식별자를 담도록 변경했다.

```text
sub = globalId
globalId = 외부 공개 식별자
```

내부 DB에서는 bigint id를 계속 사용할 수 있다.  
하지만 API 경계 밖으로 나가는 값은 공개 식별자를 사용하도록 했다.

이후 인증이 필요한 API에서 이 값을 쉽게 꺼낼 수 있도록 별도 어노테이션도 만들었다.

```java
@GlobalId String globalId
```

컨트롤러마다 token을 직접 파싱하지 않아도 되기 때문에 사용하기 편했다.

## 인증 책임

작업 중 다시 의문이 생겼다.

> 원래 인증 작업 자체는 gateway가 하기로 하지 않았나?

맞다. 원래 설계대로라면 gateway가 token을 검증하고, 내부 서비스는 gateway가 전달한 인증 컨텍스트를 신뢰하는 구조가 맞다.

문제는 gateway가 token을 제대로 검증하지 못하고 있었다는 점이다.  
인증 서버는 비대칭키 기반으로 token을 발급하고 있었는데, gateway는 기존 단일 secret 방식으로 검증하려 했다.

그래서 구조를 다시 맞췄다.

```text
1. gateway가 JWT에서 project 식별자 확인
2. 인증 서버에서 해당 project의 public key 조회
3. public key로 JWT 검증
4. 내부 서비스에 인증 컨텍스트 전달
```

이 과정을 거치며 gateway와 내부 서비스의 책임을 다시 나눴다.

```text
Gateway: 인증 판단
Internal Service: 비즈니스 처리
```

## Swagger 401

Swagger에서 token을 넣었는데도 401이 나는 문제도 있었다.

원인 중 하나는 Swagger Authorize 입력창에 `Bearer`까지 같이 넣으면서 실제 헤더가 다음처럼 만들어지는 경우였다.

```text
Authorization: Bearer Bearer eyJ...
```

작은 문제처럼 보였지만, 실제 테스트에서는 충분히 발생할 수 있는 입력 방식이었다.

## 정리

이번 작업을 통해 인증 구조에서 중요한 것은 token을 발급하는 것만이 아니라, <span class = "o">어디에서 검증할지</span>를 명확히 정하는 것이라고 느꼈다.

gateway가 인증을 담당한다면, 내부 서비스는 중복 검증보다는 gateway가 전달한 컨텍스트를 기준으로 동작하는 편이 더 자연스럽다.
