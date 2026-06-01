---
title: "[Project] 로컬 실행 환경과 Docker Compose 정리"
excerpt: ".env, init.sql, compose.yaml을 정리하며 겪은 문제"
categories:
    Project
tags:
    백엔드
    Docker
    SpringBoot
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

로컬 실행 환경을 정리했다.

처음에는 기본 데이터를 `init.sql`로 넣으려 했다.  
하지만 환경마다 달라지는 값까지 SQL에 고정하는 것은 애매했다.

로컬에서는 편할 수 있지만, 환경이 바뀔 때마다 SQL 파일을 수정해야 하고, 민감한 값이 파일에 남을 수도 있다.

## .env 처리

그래서 환경변수를 사용하기로 했다.

처음에는 자바 코드에서 `.env`를 직접 읽는 로더를 만들었다.  
하지만 다시 생각해보니 굳이 그럴 필요가 없었다.

로컬 실행이 `./gradlew bootRun` 기준이라면 Gradle에서 `.env`를 읽어 주입하는 편이 더 자연스러웠다.

```text
.env
-> Gradle bootRun
-> Spring Boot application
```

이렇게 정리하니 애플리케이션 코드에 로컬 실행을 위한 부가 코드가 들어가지 않아도 됐다.

## gateway 요청 검증

내부 서비스는 gateway를 거친 요청만 받도록 하고 싶었다.

단순히 아래처럼 boolean header를 두는 것은 부족하다.

```text
X-From-Gateway: true
```

이 값은 누구나 흉내낼 수 있기 때문이다.

그래서 gateway가 내부 요청을 보낼 때 공유 시크릿을 넣고, 내부 서비스는 그 값을 검증하도록 했다.

```text
Client
-> Gateway
-> Internal Service
```

완벽한 보안 장치는 아니지만, 내부 API를 외부에서 직접 찌르는 흐름은 어느 정도 막을 수 있다.

## compose.yaml

Docker Compose 구성도 많이 고민했다.

처음에는 DB 전용 compose와 전체 compose를 분리했다.  
하지만 로컬에서는 필요한 컨테이너를 한 번에 올리는 편이 더 편했다.

그래서 하나의 `compose.yaml` 안에서 DB와 gateway를 함께 실행하도록 정리했다.

이때 Spring Boot Docker Compose가 MySQL의 host port mapping을 찾지 못하는 문제가 있었다.  
결국 포트 바인딩을 명시적으로 IPv4 localhost에 묶어 해결했다.

```yaml
127.0.0.1:3306:3306
```

## 정리

처음에는 파일을 나누는 것이 더 깔끔해 보였다.  
하지만 실제 로컬 사용성까지 생각하면 한 파일에서 필요한 컨테이너를 같이 올리는 편이 더 나은 경우도 있었다.

이번 정리를 통해 로컬 환경에서도 <span class = "o">편의성과 명확한 책임 분리 사이의 균형</span>을 계속 봐야 한다고 느꼈다.
