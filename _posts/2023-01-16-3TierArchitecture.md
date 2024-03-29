---
title: "[Spring] Spring Security, Spring Session과 3Tier 아키텍쳐"
excerpt: "3-Tier 아키텍쳐에 대해 알아보자"

categories:
    Spring
tags:
    프레임워크
    백엔드
    SpringBoot
    Security
toc: true
comments: true
use_math: true
---  
<style type = 'text/css'>
    .o{
    font-weight: bold;
    color:orange;
    }
</style>

# 3-Tier 아키텍처  
3-Tier Architecture는 가장 보편적이고 이해하기 쉬운 아키텍처로써 Client-Server 소프트웨어 아키텍처 패턴이다.

다음과 같은 3개의 Layer로 나뉘어져 있다.

- `Presentation Layer`
    
    사용자와의 접점 제공
    
- `Application Layer`
    
    트랜잭션 처리를 위한 비즈니스 로직  제공
    
- `Data Layer`
    
    데이터를 저장하고 조회하는 기능 제공
    

이와 같이 3개의 Layer로 나뉘어진 3-Tier Archietecture로 어플리케이션을 구성함으로써 <span class = "o">프론트엔드</span>, <span class = "o">백엔드 엔지니어의 역할을 분리해 업무의 효율성을 증대</span>시킬 수 있으며 각 계층을 모듈화하여 다른 계층에 미치는 영향 최소화를 통해 <span class = "o">확장이 용이</span>하다는 장점이 있다.

## 3-Tier 아키텍처의 장점

- 계층별 모듈화로 확장성 증대(Scale-Out)
- 프론트엔드와 백엔드의 역할 분리로 업무 효율성 증대

# Http Session

3-Tier 구조의 시스템에서 만약 트래픽의 증가나 이용자 수의 증가와 같은 문제가 발생할 경우 3-Tier 아키텍처의 장점인 확장성을 통해 백엔드 개발자는 Application Layer에서의 Server를 증설할 수 있다.

이렇게 증설된 서버 중 특정 서버에서 문제가 발생할 경우 장애가 발생한 서버에서 Session을 통해 인증을 완료한 사용자의 인증이 해제되는 등 장애가 발생하게 된다.

이와 같은 장애를 해결하기 위해서 Http Session에 대해서 알아보아야 한다.

잘 알다시피, Http는 근본적으로 <span class = "o">Stateless(무상태)</span> 프로토콜이다. 이는 사용자의 요청에 대한 어떠한 정보도 저장하지 않고 그 다음 요청에 지장을 주지 않는다는 의미인데,  상황에 따라 이전 요청에 대한 정보가 필요한 경우가 있다.

사용자의 이전 요청의 저장이 필요한 경우 서버는 사용자의 이전 요청, 인증의 관점으로 보자면 즉 인증된 사용자 정보를 저장하기 위해 Session을 만들고 식별자인 `session-id` 를 통해 클라이언트에게 응답한다.

 (`session-id`는 클라이언트가 웹 브라우저인 경우 보통 Cookie에 저장한다.)  만약 서버에서 장애가 일어난다면, 복제본이 없는 session 정보는 유실되버린다.

반대로 클라이언트는 Http 요청에 `session-id` 를 포함시켜 서버가 클라이언트를 식별할 수 있도록 한다. 

당연한 이야기지만, 세션은 서버 메모리를 사용하기 때문에 너무 많은 세션을 생성하지 않도록 해야한다. 

Http 세션을 이용하면서 서버 장애가 일어났을 때 해당 서버의 세션 데이터가 유실된다는 문제점이 존재했다. 

세션 데이터의 유실을 해결할 수 있는 방법은 여러가지가 존재하겠지만 세션 cluster를 활용해 해결할 수 있다.

## 특징

- 사용자의 이전 요청에 대한 정보를 저장한다.
- `session-id` 식별자를 통해 서버와 클라이언트가 요청, 응답한다.
- 서버에서 장애가 일어날 경우 session 정보는 유실된다.
- 서버 메모리를 이용한다.

# Session Cluster

세션 데이터 유실이라는 문제점은 Session이 서버 메모리 내부에 저장되기 때문에 발생하는 장애다. 반대로 세션 클러스터의 경우 Session을 별도의 외부 스토리지에 저장하게 된다.

세션 클러스터를 이용하면, 특정 서버가 장애를 일으켜 세션 데이터가 유실 되었을 경우 외부, 즉 세션 클러스터에 저장된 세션을 활용해 정상 처리가 가능하게 된다.

또한 세션 클러스터를 이용할 경우 동일한 사용자가 발생시킨 요청은 동일한 WAS에서 처리됨을 보장하는 `Sticky Connection` 제약에서 자유로워 진다.

하지만 세션을 외부에 저장한다는 것은 말 그대로 별도의 외부 스토리지가 필요하다는 것을 의미하며 이는 관리 포인트와 그 비용의 증가가 발생한다는 것을 의미한다.

또한 세션 클러스터를 이용하는 도중 외부 스토리지에 장애가 발생했을 경우 대규모 장애 발생 가능성이 커지게 된다는 단점을 가지고 있다(세션 클러스터가 Single Point Of Failure가 될 가능성이 있다). 이 단점을 해결하기 위해서 세션 클러스터는 다수의 서버로 이루어져야 한다.

## 특징 / 장점

- 세션 데이터를 외부 스토리지에 저장
- 외부 스토리지는 조회 속도를 위해 보통 In-Memory 데이터베이스를 활용
- 세션을 외부스토리지, 세션 클러스터에 저장해 세션 데이터 유실시 큰 문제 없이 정상처리가 가능하다.
- Sticky-Connection 제약에서 자유로움

## 단점

- 관리 포인트의 증가
- 외부 스토리지 장애 발생시 대규모 장애 발생 가능성의 증가