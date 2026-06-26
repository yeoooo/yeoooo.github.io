---
title: "[CS] Https 요청의 여정(SSL/TLS)"
excerpt: "브라우저에서 HTTPS 주소를 입력했을 때 DNS, TCP, TLS를 거쳐 서버까지 도달하는 과정"
categories:
    CS
tags:
    네트워크
    HTTP
    HTTPS
    SSL
    TLS
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

# 개요

[https://yeoooo.github.io](https://yeoooo.github.io) 를 주소창에 입력하고, 엔터를 치는 순간 무슨 일이 벌어질까?

이 여정에 대해서 생각해보지 않았다면, 그저 ‘해당 주소로 입장한다’ 정도의 답변이 떠오를 것이다.

실제로는 더 복잡한 일들이 순식간에 일어나는데, 이 문서에서는 그 과정을 되도록 쉽게 풀어보고자 한다.

# 전제

전제는 다음과 같다.

- 요청할 주소 - [https://yeoooo.github.io](https://yeoooo.github.io) (또는 임의의 도메인)
- 환경은 실제 github 호스팅과 다르다고 가정
    - DNS 는 Route53을 활용한다고 가정한다.
        - 국내로는 후이즈, 가비아 등 기업이 존재한다.
    - Edge Proxy(시스템 내 가장 앞단에 존재하는 프록시)은 Cloudfront를 활용한다고 가정한다.
     이하 서버
        - AWS ALB 도 같은 작업을 수행할 수 있다.
        - CA를 의미하지 않는다.

# 여정

실제로 주소창에 주소를 적고, 요청을 하게 되면 다음과 같은 여정이 기다리고 있다.

![HTTPS 요청의 전체 여정](/assets/images/web/https-request-journey.png)

1. DNS
주소 문자열은 사람이 이해하기 쉬운 형태다. 실제 요청은 IP를 통해서만 가능하다.
    1. 브라우저에 주소를 입력하면, 브라우저는 가장 먼저 브라우저 캐시를 확인한다.
    2. 브라우저에 없는 경우 운영체제의 DNS 캐시를 확인한다.
    3. 운영체제의 DNS 캐시에도 존재하지 않는 경우 DNS Resolver(KT, SKT .. etc)에 질의한다.
        1. DNS resolver 에 캐시가 존재하는 경우 반환한다.
        2. DNS Resolver에 캐시가 존재하지 않는 경우 Root → TLD → Authoriatative 순으로 조회한다.
    4. DNS Resolver는 결과를 캐싱하고 브라우저에게 IP를 반환한다.
2. TCP 연결 - 3 Way Handshake
    1. 클라이언트 (브라우저 등) 는 서버에  TCP 연결을 요청한다.(SYN)
    2. 서버는 연결 가능한 상태임을 알리며 클라이언트 또한 연결 가능한 상태인지 응답을 기다린다. (SYN + ACK)
    3. 클라이언트는 연결 가능한 상태임을 서버에 알린다.(ACK)
3. <a href="#tls"><span class="o">TLS</span></a> (서버 인증 및 Session Key 생성)
4. HTTP를 Session Key로 암호화, 복호화하여 통신

## TLS

인증서에는 서버의 공개키가 포함되어 있으며, 브라우저는 이를 Root CA의 공개키로 검증한 후 신뢰하게 된다.

1. ClientHello
    - TLS 버전
    - Cipher Suite
    - Client Random
2. ServerHello
    - 선택된 Cipher Suite
    - Server Random
3. Certificate
    - 서버 인증서 전송
    - 서버 공개키 포함
4. 브라우저가 인증서를 검증
    - Root CA 공개키(브라우저, 또는 클라이언트가 이미 가지고 있는 공개키)로 전자서명 검증
    - 검증 성공 시 서버 공개키를 신뢰
5. TLS 1.2 / TLS 1.3 에 따라 다른 Session Key(AES Key) 생성
    - 최근 대부분의 TLS 는 1.3 을 사용한다.
    서버의 개인키가 한번이라도 노출 되는 경우 과거 모든 데이터의 복호화가 가능한 단점 때문에 순방향 비밀성이 보장되지 않기 때문이다.

### TLS 1.2(RSA)

1. 클라이언트(브라우저 등)가 Pre-Master Secret 생성
2. 클라이언트가 Pre-Master Secret을 서버 공개키로 암호화하여 ClientKeyExchange로 전송
3. 서버는 서버 개인키로 복호화하여 Pre-Master Secret 획득
4. 서버와 클라이언트 양쪽 모두
Client Random + Server Random + Pre-Master Secret을 통해 Master Secret 생성
5. Master Secret을 통해 Session Key 생성

**서버의 개인키가 한번이라도 노출된다면 과거 네트워크의 정보들이 모두 복호화 가능하다는 치명적인 단점이 존재했다.**

### TLS 1.3(<span class="o">ECDHE</span>)

1. 클라이언트와 서버는 임시 개인키/공개키를 생성한다.
2. 클라이언트와 서버는 서로 임시 공개키를 교환한다.
3. 각자의 조합을 통해 AES 대칭키(Session Key)를 만들어 낸다
    1. 서버 - 서버 임시 개인키 + 클라이언트 임시 공개키
    2. 클라이언트 - 클라이언트 임시 개인키 + 서버 임시 공개키

**이렇게 만들어진 세션 키는 그 과정에서 네트워크를 통해 전송될 필요가 없으며 또한 네트워크  왕복이 감소되어 연결지연성이 개선된다는 장점이 존재한다.**

# 정리

브라우저에서 https://yeoooo.github.io을 입력하면

1. DNS를 통해 IP를 찾는다.

2. TCP 연결을 수립한다.

3. TLS Handshake를 통해 서버를 인증하고 Session Key를 생성한다.

4. 이후 모든 HTTP 데이터는 Session Key로 암호화되어 전송된다.
