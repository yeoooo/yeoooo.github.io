---
title: "[Project] Path 기반 라우팅에서 Subdomain 라우팅으로 변경"
excerpt: "prefix 라우팅에서 Host 기반 라우팅으로 전환하며 확인한 것들"
categories:
    Project
tags:
    백엔드
    Gateway
    Nginx
    DNS
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

라우팅 기준을 path에서 subdomain으로 바꿨다.

초기에는 path prefix를 기준으로 라우팅했다.

```text
/auth/**
/service/**
```

하지만 운영 환경에서는 subdomain 기준이 더 명확해 보였다.

```text
auth.example.com -> 인증 서버
api.example.com  -> API 서버
```

## Host 기반 라우팅

gateway는 `Host` 헤더를 보고 내부 서비스로 라우팅하게 했다.

이때 중요한 것은 Nginx가 원래 요청의 `Host`를 유지해서 gateway로 넘겨야 한다는 점이다.

```nginx
proxy_set_header Host $host;
```

Nginx는 모든 요청을 gateway로 넘기고, gateway가 Host 기준으로 내부 서비스에 전달한다.

```text
Client
-> Nginx
-> Gateway
-> Internal Service
```

## index.html 문제

한 번은 특정 도메인으로 Swagger UI에 접근했는데 프론트엔드 `index.html`이 반환되는 문제가 있었다.

처음에는 내부 서비스가 잘못 뜬 줄 알았다.  
하지만 다시 보니 Nginx의 `server_name`과 gateway 라우팅 기준이 맞지 않았던 문제였다.

즉 요청이 내가 생각한 server block으로 들어가지 않았고, 다른 location 규칙을 타고 있었다.

## DNS 전파

DNS도 확인해야 했다.

어떤 DNS 서버에서는 도메인이 조회되지 않았고, 다른 DNS 서버에서는 이미 조회됐다.  
이런 경우 코드나 서버 설정 문제가 아니라 DNS 전파나 resolver 차이일 수 있다.

```text
DNS
-> Nginx server_name
-> Host header
-> Gateway route
-> Internal service
```

이 전체 흐름이 맞아야 요청이 제대로 전달된다.

## 정리

subdomain 라우팅은 path prefix보다 깔끔하지만, 확인해야 할 계층이 더 많다.

특히 `Host` 헤더가 유지되지 않으면 gateway는 어떤 서비스로 보내야 할지 판단할 수 없다.  
앞으로 운영 라우팅을 볼 때는 DNS부터 gateway route까지 한 번에 봐야겠다.
