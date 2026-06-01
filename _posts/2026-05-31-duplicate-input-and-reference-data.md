---
title: "[Project] 중복 입력 처리와 기준 데이터 스냅샷 정책"
excerpt: "JPA flush 순서와 기준 데이터, 직접 입력을 함께 다루는 방식"
categories:
    Project
tags:
    백엔드
    JPA
    API
    설계
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

항목 배정 로직에서 중복 입력을 테스트했다.

같은 항목에 같은 대상이 중복으로 들어오면 DB unique key에 걸릴 수 있었다.  
겉으로 보기에는 기존 데이터를 삭제하고 다시 넣는 흐름이라 문제가 없어 보였다.

하지만 JPA flush 순서 때문에 insert가 delete보다 먼저 실행될 수 있었다.

## 중복 입력 처리

기존 흐름은 대략 다음과 같았다.

```text
기존 배정 삭제
새 배정 저장
```

하지만 실제 SQL 실행 순서가 항상 내가 생각한 순서와 같지는 않았다.  
그래서 기존 배정을 삭제한 뒤 `flush()`를 강제했다.

```text
delete existing shares
flush
insert new shares
```

이후 중복 입력 테스트를 추가했다.  
이런 문제는 코드만 보면 놓치기 쉬운데, 실제 중복 요청을 넣어보니 바로 드러났다.

## 빈 배열 처리

또 빈 배열을 보내면 기존 배정을 모두 제거할 수 있도록 했다.

```json
{
  "items": [
    {
      "itemId": "itm_xxx",
      "shares": []
    }
  ]
}
```

처음에는 빈 배열을 잘못된 요청으로 볼 수도 있다고 생각했다.  
하지만 사용자가 기존 배정을 제거하려는 명확한 의도로 볼 수 있기 때문에 허용하는 쪽이 더 자연스러웠다.

## 기준 데이터와 직접 입력

이후 기준 데이터와 직접 입력 데이터를 함께 다루는 흐름도 고민했다.

처음에는 기준 데이터 ID가 들어오면 이름과 값을 따로 받지 않는 게 맞다고 생각했다.  
하지만 다시 생각해보니 사용자가 실제로 입력한 값을 스냅샷으로 남기고 싶을 수 있었다.

그래서 정책을 바꿨다.

```text
기준 데이터 ID 있음 + name/price 없음
-> 기준 데이터 값 사용

기준 데이터 ID 있음 + name/price 둘 다 있음
-> 기준 데이터 연결은 유지하고 입력값을 스냅샷으로 저장

기준 데이터 ID 있음 + name 또는 price 하나만 있음
-> 400

기준 데이터 ID 없음
-> name/price 직접 입력 필수
```

이 방식이면 기준 데이터와 연결도 유지하면서, 실제 입력 당시의 값도 보존할 수 있다.

## 관리자 API

관리자용 기준 데이터 관리 API도 추가했다.

```http
GET    /v1/admin/resources
POST   /v1/admin/resources
PUT    /v1/admin/resources/{resourceId}
DELETE /v1/admin/resources/{resourceId}
```

Swagger에는 `[ADMIN 전용]` 표시를 넣었다.  
삭제는 물리 삭제가 아니라 상태값을 변경하는 방식으로 처리했다.

## 정리

이번 작업에서는 단순한 CRUD보다 입력 정책을 정하는 일이 더 중요했다.

```text
중복 입력을 어떻게 볼 것인지
빈 배열을 삭제 의도로 볼 것인지
기준 데이터와 사용자 입력을 어떻게 함께 남길 것인지
```

이런 기준을 명확히 해두어야 API를 사용하는 쪽에서도 예측 가능한 방식으로 동작할 수 있다.
