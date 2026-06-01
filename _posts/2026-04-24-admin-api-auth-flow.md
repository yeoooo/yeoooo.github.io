---
title: "[Project] 관리자 API 인증 흐름 정리"
excerpt: "정적 secret 기반 관리 API를 로그인 기반 권한 구조로 변경하기"
categories:
    Project
tags:
    백엔드
    SpringBoot
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

내부 관리 API를 어떻게 보호할지 다시 고민했다.

처음에는 특정 secret header 하나만 맞으면 관리 API를 호출할 수 있도록 구성했다.  
구현은 단순했다. 요청 헤더에 정해진 값이 있으면 통과시키면 됐다.

하지만 다시 생각해보니 이 값 하나가 사실상 <span class = "o">마스터 키</span>처럼 동작했다.

## 정적 secret의 문제

처음에는 `X-ADMIN-SECRET`만 있으면 충분하지 않을까 생각했다.  
외부에 노출하지 않고 내부에서만 쓰면 괜찮을 것 같았다.

하지만 이 값이 유출되면 누구나 관리 API를 호출할 수 있다.  
특히 관리 API는 시스템 내부 데이터나 기준 정보를 다루는 경우가 많기 때문에, 단순 secret 하나에 의존하는 구조는 불안했다.

그래서 관리 API는 로그인한 관리자만 호출할 수 있도록 변경했다.

```text
Authorization: Bearer {access-token}
role: ADMIN 또는 SYSTEM
```

이렇게 바꾸면 단순히 secret 값을 아는 것만으로는 호출할 수 없다.  
사용자 인증과 권한 검사를 모두 통과해야 한다.

## role과 신청 상태

관리자 권한 신청 흐름도 함께 고민했다.

처음에는 회원가입 시 `is_staff=true`를 받으면 별도의 role을 줄 수 있지 않을까 생각했다.  
예를 들면 `PENDING_ADMIN` 같은 role을 두는 방식이다.

하지만 곧 애매하다는 생각이 들었다.

role은 현재 권한을 의미해야 한다.  
그런데 `PENDING_ADMIN`은 권한이라기보다 신청 상태에 가깝다.

이 둘을 섞으면 나중에 권한 판단이 지저분해질 수 있다.

```text
USER
ADMIN
SYSTEM
PENDING_ADMIN
```

위처럼 role에 상태가 섞이기 시작하면 `hasRole()` 같은 코드가 점점 애매해질 것 같았다.

## 정리

결국 권한과 신청 상태를 분리했다.

```text
role = USER
staffRequested = true
```

`is_staff=true`는 권한 부여가 아니라 "관리자 권한을 신청했다"는 의미만 갖게 했다.  
실제 권한 상승은 기존 관리자가 승인 API를 호출했을 때만 일어나도록 했다.

이번 작업을 통해 role은 가능한 단순하게 유지하는 것이 좋다고 느꼈다.  
권한과 상태를 구분해두면 이후 정책이 늘어나도 코드가 덜 흔들릴 것 같다.
