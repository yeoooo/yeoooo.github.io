---
title: "[Project] ARM 서버 배포와 운영 Profile 정리"
excerpt: "A1 ARM 서버용 이미지 빌드와 GHCR 권한 문제"
categories:
    Project
tags:
    백엔드
    Docker
    GHCR
    배포
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

A1 ARM 서버에 배포하기 위해 이미지를 `linux/arm64`로 빌드했다.

처음에는 Docker build만 성공하면 끝날 줄 알았다.  
하지만 실제로는 이미지 push 권한 문제가 더 오래 걸렸다.

## GHCR 권한

GHCR에 로그인은 성공했지만 push에서 권한 오류가 발생했다.

처음에는 token만 있으면 되는 줄 알았다.  
하지만 실제로는 token scope, package 권한, organization 권한이 모두 맞아야 했다.

```text
read:packages
write:packages
organization package access
```

로그인이 성공한다고 해서 push 권한까지 있다는 의미는 아니었다.  
이 부분을 분리해서 봐야 했다.

```text
docker login 성공
!=
docker push 가능
```

## prod profile

운영 profile도 함께 점검했다.

local에서 쓰는 값이 prod까지 섞이면 안 된다.  
특히 Docker Compose 자동 실행이나 SQL debug log는 운영에서는 꺼야 한다.

그래서 prod에서는 필요한 환경변수를 명시적으로 요구하도록 했다.

```text
local: 실행 편의성
prod: 명시적 설정과 보안
```

또 local 전용 prefix가 운영 환경변수 이름에 남아 있으면 어색했다.  
운영에서는 실제 배포 환경에 맞는 이름을 쓰도록 정리했다.

## 정리

처음에는 "컨테이너가 뜨면 배포 준비가 된 것 아닌가?"라고 생각하기 쉽다.  
하지만 운영 배포는 조금 달랐다.

```text
아키텍처에 맞는 platform으로 빌드됐는지
registry 권한이 맞는지
prod profile이 local 설정을 포함하지 않는지
secret이 기본값으로 남아 있지 않은지
```

이런 부분을 함께 확인해야 배포 가능한 상태라고 말할 수 있다.
