---
title: 
    "[JAVA] StringBuilder와 StringBuffer, String"
excerpt: 
    StringBuilder와 StringBuffer 그리고 String에 관한 정리
category: 
    java
tags: 
    java
    javaLibrary
toc: 
    true
comments: 
    true
use_math: 
    false
    #for use, there must be dollar sign on both side of contents
---

<style type = 'text/css'>
    .o{
    font-weight: bold;
    color:orange;
    }
</style>

## 개요  
sql문을 입력하거나, 예외를 던지거나, 또는 그 자체를 표현하기 위해서도 String은 자바 곳곳에서 사용된다.  
하지만 이를 python에서와 같이 + 연산자를 사용하고 또는 이와 다르게 new 연산자를 이용해 사용하는 것은 <span class = "o">올바르지 못한 사용 방법</span>이라는 말을 듣게 되었다.  

왜 올바르지 못한 사용 방법인지, 또 이에 대한 방안인 StringBuffer와 StringBuilder에 대해 알아보았다.

먼저 흔히 사용되는 StringBuilder, StringBuffer를 알아가기 전에 String에 대해서 알아보았다.  

## String  
[javaDocs](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html)가 언급하는 String의 가장 큰 특징은 아래와 같다.  
```  
String은 상수이며 이들의 value는 만들어지고나서 그 값이 바뀌지 않는다.
```
이와 같이 String은 상수라고 언급이 되어있다.  
String이 상수라면 `+`와 같은 연산자로 값이 바뀌지지 않아야하지만 값이 바뀐다.  

```java
String text = "123";
text += "456";

System.out.println(text1);
=========================
123456
```  

이것이 가능한 이유는 `+`연산자가 사용되면 <span class = "o">내부적으로 StringBuilder 인스턴스가 생성</span>된 후 연산, <span class = "o">String 객체를 반환</span>하기 때문이다(JVM의 영역인 것 같다). 또한 이렇게 되면 "123", "456", "123456" 과 같이 세 개의 문자열이 각각 메모리에 저장되어 효율이 떨어지게 된다.

하지만 이렇게 만들어진 변수는 같은 값을 가진 상수와 비교하면 결과는 예상과 다르다.  
```java
String text1 = "123";
text1 += "456";

System.out.println(text1 == "123456");
==========================
false
```
상수인 `"123456"`과 내부적으로 생성된 `StringBuilder`를 통해 반환된 String객체의 주소 값이 다르기 때문에 위와 같은 결과가 나오게 된다.  
그렇다면 상수형문자열 `"123456"`과 String객체`text1`은 각각 어디에 저장되는 걸까?  
  
결론부터 말하자면 상수형 문자열 `"123456"`<span class = "o">은 JVM이 관리하는 Heap 영역 내의 Stringpool</span>, 참조형 문자열 객체 `text1`은 JVM이 관리하는 <span class = "o">Heap의 별도 영역</span>에 저장된다.  
이 때 Stringpool에 들어가있는 문자열과 같은 값을 갖는 변수를 선언하면 변수는 이전에 이미 상수형 문자열이 있던 메모리의 주소를 참조하게 된다.  
이는 String을 아래의 방법으로 만들었을 경우이다.  

```java  
String s = "this is String";
```  
이와 다른 방법인 new를 사용해 문자열을 아래외 같이 만든다면,
```java
String s = new String("this is String");
```  
Heap내의 별도 공간에 저장되는 것이다.  

### 불변성

지금까지의 과정으로 어느 정도 유추가 가능하겠지만 String은 왜 Stringpool에 저장, 즉 불변하게 정의해놨을까?  
    
    1. 캐싱을 이용한 효과적인 메모리 사용  
    2. 보안  
    3. 스레드의 안전성  

아래와 같은 코드가 있다고 가정해보자.  

```java  
for(int i = 0; i < 300; i++){
    String hello = "Hello!";
    System.out.println(hello);
}
```  
hello라는 변수에 "Hello!"라는 문자열이 할당되어 있고, 이는 for문에 의해 300번 반복된다. 하지만 "Hello!"와 같은 문자열 객체 자체는 단 한 번 생성되어 300번 참조된다.(변수 hello 자체는 300번 생성되고 사라진다.)  

이 때 참조되는 문자열 "Hello!" 는 <span class = "o">String pool에 들어가 있어서 객체를 따로 생성하지 않고, 따라서 오버헤드가 발생하지 않아 메모리를 효율적으로 사용</span>할 수 있게 된다.  

또한 불변성에 의해 한번 정해진 문자열을 바꾸거나, 여러 스레드가 동시에 참조한다고 해도 그 값이 충돌을 일으키지 않기 때문에 보안, 스레드의 안전성까지 챙길 수 있다.  

이러한 불변성의 이점을 챙기면서 문자열을 동적으로 이용하기 위해서 StringBuilder와 StringBuffer가 사용된다.  

## StringBuilder, StringBuffer  

[StringBuilder](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StringBuffer.html)는 변경 가능한 문자열이며 StringBuffer와 호환되는 API를 제공하나, <span class = "o">동기화(Synchronization)</span>를 보장하지 않는다.  

이와 반대로 [StringBuffer](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StringBuffer.html)는 String과 다르게, StringBuilder와 같게 변경 가능한 문자열이지만 동기화(Synchronization)을 지원한다.  

요청을 보내고, 대기하는 동기화가 존재하지 않는 <span class = "o">StringBuilder가 일반적인 단일 스레드 환경에서 뛰어난 성능</span>을 보이고 이 때문에 javadocs에서는  

    Where possible, it is recommended that this class be used in preference to StringBuffer as it will be faster under most implementations.  

와 같이 가능한 StringBuilder를 사용하는 것을 추천한다.  

하지만 이와 반대로 <span class = "o">StringBuffer는 동기화로 멀티 스레드 환경에서도 안전하게 동작</span>한다는 장점이 있다.  



## 참고  
- String 객체와 문자열 상수  
[https://www.codelatte.io/courses/java_programming_basic/PP95VV3NZJM71V14](https://www.codelatte.io/courses/java_programming_basic/PP95VV3NZJM71V14)  
- JVM  
[https://docs.oracle.com/javase/specs/jvms/se11/html/jvms-4.html#jvms-4.4](https://docs.oracle.com/javase/specs/jvms/se11/html/jvms-4.html#jvms-4.4)  
- StringBuffer
[https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StringBuffer.html](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StringBuffer.html)  
- StringBuilder  
[https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StringBuilder.html](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StringBuilder.html)  
- String이 상수로 정의된 이유
[https://stackoverflow.com/questions/2068804/why-is-the-string-class-declared-final-in-java](https://stackoverflow.com/questions/2068804/why-is-the-string-class-declared-final-in-java)  
- immutable  
[https://www.mindprod.com/jgloss/immutable.html](https://www.mindprod.com/jgloss/immutable.html)  
- ConstantPool, StringPool  
[https://blog.breakingthat.com/2018/12/21/java-constant-pool과-string-pool/](https://blog.breakingthat.com/2018/12/21/java-constant-pool과-string-pool/)  