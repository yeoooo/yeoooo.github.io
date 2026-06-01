---
title: "[Project] Self-hosted Runner 기반 CI/CD 정리"
excerpt: "private subnet 배포를 위해 gateway VM에 runner를 둔 이유"
categories:
    Project
tags:
    백엔드
    GitHubActions
    CI/CD
    Docker
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

CI/CD 파이프라인을 정리했다.

처음에는 GitHub Actions에서 private subnet의 VM으로 직접 SSH 접속하면 된다고 생각했다.  
하지만 private VM은 외부에서 접근할 수 없다.

키가 있어도 네트워크 경로가 없으면 접속할 수 없다.  
즉 문제는 SSH key가 아니라 <span class = "o">네트워크 경로</span>였다.

## ProxyJump 고민

처음에는 ProxyJump도 고려했다.

구조상 gateway VM은 public IP를 가지고 있고, 내부 서비스 VM은 private IP만 가지고 있었기 때문이다.

```text
GitHub Actions
-> gateway VM
-> private VM
```

하지만 GitHub-hosted runner의 IP 대역을 열어줘야 하는 문제가 있었다.  
운영 보안상 계속 열어두기 애매했다.

## self-hosted runner

결국 gateway VM에 self-hosted runner를 두는 방향으로 바꿨다.

```text
GitHub-hosted runner
-> test
-> build
-> image push

gateway VM self-hosted runner
-> image pull
-> container recreate
-> private VM에 SSH로 배포 명령 전달
```

처음에는 1CPU / 2GB VM에서 runner를 돌리는 게 벅차지 않을까 걱정했다.  
하지만 빌드까지 돌리는 것이 아니라 배포 명령만 실행한다면 가능하다고 판단했다.

## 빌드 속도

빌드 속도도 개선했다.

기존에는 Docker build 안에서 Gradle build를 수행했다.

```dockerfile
RUN ./gradlew bootJar
```

이 방식은 Docker build 때마다 Gradle 의존성과 빌드가 반복될 수 있었다.

그래서 GitHub Actions에서 먼저 jar를 만들고, Docker image에는 jar만 복사하는 runtime-only 방식으로 바꿨다.

```text
Gradle build
-> jar 생성
-> Docker image build
```

Gradle cache와 Docker Buildx cache도 함께 적용했다.

## 정리

배포는 단순히 명령어를 실행하는 것이 아니라, 네트워크 구조와 권한 구조를 같이 봐야 했다.

특히 private subnet에 있는 서버는 외부에서 바로 접근할 수 없기 때문에, 배포 명령이 어디에서 실행되는지가 중요하다.

이번 구조에서는 gateway VM의 runner가 그 역할을 맡게 됐다.
