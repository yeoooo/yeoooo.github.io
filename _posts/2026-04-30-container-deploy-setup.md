---
title: "[Project] 컨테이너 기반 배포 구조 정리"
excerpt: "VM에 소스코드를 올릴지, image registry를 쓸지 고민한 흐름"
categories:
    Project
tags:
    백엔드
    Docker
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

배포 방식을 고민했다.

처음에는 Docker로 배포하려면 소스 파일도 VM으로 옮겨야 하는지 궁금했다.  
VM에서 직접 빌드하면 단순하긴 하지만, 운영 서버에 소스 코드와 빌드 도구가 모두 남는다.

운영 관점에서는 이 방식이 그다지 좋아 보이지 않았다.

## 이미지 기반 배포

결국 image registry를 사용하는 방향으로 정리했다.

```text
CI 또는 로컬
-> Docker image build
-> Registry push
-> VM에서 pull
-> container 실행
```

이렇게 하면 VM에는 소스 코드가 필요 없다.  
VM은 이미지를 받아 실행하기만 하면 된다.

Docker Hub도 사용할 수 있지만, GitHub 저장소와 붙여 쓰기 좋은 GHCR을 기준으로 잡았다.

## local profile

API 서버도 local profile을 분리했다.

로컬에서는 DB, 인증 서버, gateway 등 필요한 컨테이너를 함께 올리고 싶었다.  
하지만 운영에서는 이런 보조 컨테이너를 애플리케이션이 자동으로 띄우면 안 된다.

그래서 local과 prod를 분리했다.

```text
local: 개발 편의성 중심
prod: 명시적 환경변수와 외부 인프라 중심
```

## 설정 충돌

이 과정에서 여러 설정 문제가 이어졌다.

```text
DB 이름 누락
datasource 설정 누락
Swagger 401
profile별 설정 충돌
```

특히 Swagger 401은 API 서버 문제가 아니었다.  
gateway local 설정이 최신 설정을 덮어쓰고 있었던 것이 원인이었다.

프로파일별 설정이 많아질수록 내가 수정한 설정이 실제 런타임에서 적용되고 있는지 확인해야 한다.

## 정리

컨테이너 기반 배포는 단순히 Dockerfile을 만드는 것으로 끝나지 않는다.

```text
어디서 빌드할지
어디에 저장할지
VM은 무엇만 담당할지
local과 prod 설정을 어떻게 나눌지
```

이 부분을 함께 정리해야 실제로 운영 가능한 배포 흐름이 된다.
