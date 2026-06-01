---
title: "[Project] Object Storage 접근 권한 정리"
excerpt: "SDK 인증 방식과 Instance Principal을 고민한 내용"
categories:
    Project
tags:
    백엔드
    Cloud
    ObjectStorage
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

Object Storage 접근 권한을 정리했다.

SDK를 사용할 예정이라 API Key 파일을 서버에 둘 수도 있었다.  
하지만 운영 서버에 key 파일을 직접 두는 방식은 유출 위험과 관리 부담이 있다.

그래서 Instance Principal 방식을 고려했다.

## Instance Principal

Instance Principal은 서버에 별도 credential 파일을 두지 않고, VM 자체에 권한을 부여하는 방식이다.

구조는 다음과 같다.

```text
VM
-> Instance Principal
-> Dynamic Group
-> IAM Policy
-> Object Storage Bucket
```

즉 특정 VM을 Dynamic Group에 포함시키고, 그 Dynamic Group에 Object Storage 접근 권한을 주는 방식이다.

처음에는 compartment, dynamic group, policy 같은 용어가 낯설었다.  
하지만 결국 리소스를 묶고, 그 묶음에 권한을 주는 구조라고 이해했다.

## Bastion 고민

Bastion도 함께 고민했다.

private subnet의 VM에 접근하려면 public IP가 없기 때문에 중간 경로가 필요하다.  
다만 이미 gateway VM을 bastion처럼 사용하고 있었기 때문에 OCI Managed Bastion을 당장 추가하지는 않기로 했다.

현재 구조는 다음과 같다.

```text
내 PC 또는 runner
-> gateway VM
-> private VM
```

이 구조는 단순하고 이미 구성되어 있다는 장점이 있다.  
하지만 gateway VM에 내부 접근 권한이 모이기 때문에, 이 VM의 보안은 더 신경 써야 한다.

## 정리

운영 환경에서는 credential 파일을 서버에 직접 두는 방식보다 VM 자체에 권한을 부여하는 방식이 더 깔끔해 보인다.

다만 권한이 VM 단위로 열리는 만큼, 어떤 VM이 어떤 bucket에 접근할 수 있는지 명확히 관리해야 한다.
