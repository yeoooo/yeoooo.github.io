---
title: "[Spring] 프로필 이미지와 캐시데이터"
excerpt: "프로필 이미지가 안바껴요"

categories:
    Spring
tags:
   
toc: true
comments: true
---
<style type = 'text/css'>
    .o{
    font-weight: bold;
    color:orange;
    }
</style>

# 프로필 이미지 구현

프로필 이미지를 업데이트 받아 이를 바로 띄워주려고 하니 `404 에러`가 떴다.

이를 해결하기 위해서 쿼리 스트링으로 난수값 추가, html meta 태그 내 cache-control 등을 추가했으나 크게 의미 있지는 않았다.

해당 문제를 해결하며 다음과 같은 과정을 거쳤다.

## 1. 문제 인식 및 정의

프로필 이미지를 <span class = "o">파일로 저장 시켰으나 다시 해당 페이지로 접근할 때는 업데이트된 이미지가 아닌 그 전의 이미지를 불러왔다</span>.

새로 생성된 이미지를 불러오고자 하니 `404` 에러를 띄웠다.

또한 이 문제는 <span class = "o">서버가 기동 되고 있는 동안 생성된 파일에 대해서만 적용되었고, 서버를 재기동했을 때는 온전히 그 전의 파일로 취급되어 잘 불러와졌다</span>.

## 2. **검색 및 시도**

먼저 위와 같은 양상을 생각해 보았을 때, 자주 사용되는 페이지나 리소스, 최근에 사용된 것들에 대해 캐싱한다는 사실이 떠올랐고 이가 유력하다고 판단되어 중점적으로 해결하고자 했다.

이에 대한 대안으로는 총 3가지 정도였다.

1. ~~쿼리 스트링에 유니크 값 추가~~
    -  자바 스크립트에서 Math.random 을 통해 난수를 추가해 매번 새로운 요청으로 인식하게 했으나 스프링 부트 서버는 크게 연연하지 않는 느낌이었다.
2. ~~html 요청 헤더 값 추가~~
    -  html 내 meta 태그에 cache-control : no-cache 추가해 캐싱을 우회하려 했으나 이는 왜인지 알지 못한채로 실패로 끝났다. 이유는 아래에서 밝혀냈다.
3. ~~WebMVCConfigure 설정 추가~~
    - WebMVCConfigure에 캐시 정책 추가를 통해 해결하려했는데, 분명 요청 위치와 실제 위치를 잘 매핑해주었음에도 돌아가지 않았다.



**<center>공통적으로 개발자 도구를 통해서 헤더값을 확인해보았을 때,</center>**  
**<center>모두 Cache-Control : no-cache 와 같은 값이 들어가지 않았다.</center>**

## 3. 생각 및 해결

곰곰히 생각해보니 요청은 해당 페이지에서 들어온다기 보다는 페이지 안에서의 img 태그에서 이뤄진다는 생각이 들었다. 

헤더 값을 추가할 수 있는 옵션들을 찾아보다가 다른 옵션을 건드리지 않고 컨트롤러의 매핑 함수의 매개변수 중 `HttpResponseServlet`을 이용할 수 있음을 알게 되었다.  
이를 img > src 값에 매핑하기로 했다. (이 값을 토대로 요청이 들어올 것이라 추측했기 때문이다)

따라서 다음과 같은 코드를 추가했다.

```java
@GetMapping("/img/profiles/{filename}")
    public ResponseEntity<byte[]> getImage(@PathVariable String filename) throws IOException {
        String directoryPath = "src/main/resources/static/img/profiles";
        Path filePath = Paths.get(directoryPath, filename);

        // 이미지 파일 읽기
        byte[] imageBytes = Files.readAllBytes(filePath);

        // Cache-Control 헤더 설정
        CacheControl cacheControl = CacheControl.maxAge(0, TimeUnit.SECONDS).noCache().mustRevalidate();
        return ResponseEntity.ok()
                .contentType(MediaType.IMAGE_JPEG)
                .cacheControl(cacheControl)
                .body(imageBytes);
    }
```

ResponseEntity를 이용하면 어떤 뷰가 반환 값이 될 필요가 없었고, 이전 과정과 같이 파일을 읽되 헤더 값에 `.cacheControl` 을 통해서 추가해줄 수도 있었다.