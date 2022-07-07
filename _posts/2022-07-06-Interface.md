---
title: 
    "[JAVA] Abstract Class, Interface 정리"
excerpt: 
    Abstract, Interface에 관한 정리
category: 
    
tags: 
    
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
객체지향 프로그래밍(OOP) 언어인 자바를 사용하면 당연지사 Interface를 마주친다.  
일전에 자바를 공부하며 조금은 알아뒀지만, Spring으로 프로젝트를 진행하게 되면서 사용하는 Interface는 알던 것과 조금 다르게 느껴졌고 이에 대해서 정리하려고 한다.  

Interface를 알아보기전에, 먼저 Abstract class(추상 클래스)가 먼저 떠올랐다. 둘 모두 추상화를 담당하는데 그 차이가 무엇인지 부터 알아보기로 했다.  

## Abstract Class  
추상 클래스를 Javadocs에서 찾아보면 가장먼저 이렇게 언급된다.  

    An abstract class is a class that is declared abstract—it may or may not include abstract methods. Abstract classes cannot be instantiated, but they can be subclassed.  
    추상 클래스는 abstract가 정의된 추상 메서드를 포함하기도 포함하지 않기도 한 클래스이며 이는 인스턴스화될 수 없지만 서브클래스화 될 수 있다.  

서브클래스화 될 수 있다는 의미는 어떤 클래스에 상속할 수 있다는 의미를 가진다.  

추상 클래스에 대해서 블로그나 여타 책으로 접하다보면 공통된 속성이나 기능을 공유하기 위해 추상체를 사용한다고 하는데, 최근에 조금 더 와닿는 설명을 들었다.  

"상속은 공통의 기능의 관점이 아니라 <span class = "o">추상과 구체의 관점</span> 으로 봐야한다." 라는 설명이었다. (자바를 이미 접해보고 다시 접한 뒤에 듣는 설명이라 더 와닿았지만 처음 접하는 사람이라면 생소한 설명일 것 같긴 하다.)  

여튼 abstract 클래스는 다른 구체 클래스에 상속되어지고 그 구체 클래스가 추상 클래스를 구현할 수 있다.
```java  
public abstract class abstractClass {
    public int integer = 1;
    public abstract void voidmethod(); //추상 메서드는 생략될 수 있다.
}
```  
추상 메서드란 구현체(body)가 없는 메서드를 뜻하며 이를 구현하기 위해서는 추상 클래스를 상속받아 구현할 수 있다.  
```java  
public class implement_abstract extends abstractClass{
    @Override
    public void abstractMethod() {
        System.out.println("Hello Abstract");
    }
}  
```  

## Interface  
소프트웨어 공학에서는 종류가 다른 프로그래머 그룹이 어떻게 서로 소통할 것인지 설명하는 계약을 하는 것이 중요한 많은 상황이 있는데, 이 계약에 서로 다른 그룹들이 <span class = "o">계약</span>하고 그에 동의함으로써 서로 어떻게 코드가 작성되고 있는지 모르면서도 작성될 수 있어야하고 이 때 인터페이스가 그 <span class = "o"> 계약서</span>와 같은 역할을 한다고 한다.  

따라서 Javadocs에서 설명하기를, 인터페이스는 서로 코드를 어떻게 작성할 것인지에 대한 계약서라고 한다.  

인터페이스는 아래와 같이 추상 메서드로만 이루어진다.
```java  
public interface testInterface {

    public int sum(int x, int y);
    public String result(int total);
    ...

}  
```  

다른 말로 메서드의 타입과 이름, 들어갈 변수 정도만 적어놓는 "이렇게 구현이 되어야 한다" 와 같은 일종의 계약서이자 명세서 역할을 하는 것이다.  
  
이러한 명세서를 가지고 개발을 하게되면 다음과 같은 이점이 주어진다.  

     1. 표준화
     1. 다형성
     2. 결합도의 저하

여기서 다형성이란 말 그대로 여러가지의 타입이 될 수 있다는 의미이다.  

아래의 예시를 통해서 다형성을 쉽게 파악할 수 있다.
```java  

public class Main {
    public static void main(String[] args) {

        interface Fruit {
            void getName();
        }

        class Banana implements Fruit {
            @Override
            public void getName() {
                System.out.println("바나나!");
            }
        }
        class Apple implements Fruit{
                @Override
                public void getName() {
                    System.out.println("사과!");
                }
            }
            Fruit banana = new Banana();
            Fruit apple = new Apple();

            banana.getName();
            apple.getName();
    }
}
=========================
바나나!
사과!
```  
인터페이스 `Fruit`을 구체화 시키는 `Apple` 과 `Banana`는 타입 `Fruit`으로 묶일 수 있다.  
즉, `Fruit` 타입은 `Apple` 타입도, `Banana` 타입도 될 수 있는 것이다.  

이러한 인터페이스와 다형성을 이용하게 되면 원하는 메서드를 인터페이스에 추가해주고, 타입이 바뀌거나 기능이 추가되더라도 `Fruit`이라는 인터페이스를 사용하고 있었기 때문에 큰 문제없이 유지보수, 기능을 확장할 수 있게 된다.  

하지만 인터페이스는 모든 추상메서드 구현을 강제한다는 단점아닌 단점이 존재했고, 이를 해결하기 위해서 java8 이후로 default 메서드가 등장하게 된다.  
### default method  
default 메서드는 구현부를 지니는 메서드이다.  
```java  
public class Main {
    public static void main(String[] args) {

        interface Fruit{
            default void getFruitByColor(String color){ //default 메서드, 구현부를 지닌다.
                if(color == "Yellow"){
                    System.out.println("바나나!");
                }
                else{
                    System.out.println("사과!");
                }
            }
        }
        class Banana implements Fruit {
        }
        class Apple implements Fruit{
        }
        Fruit banana = new Banana();
        Fruit apple = new Apple();

        banana.getFruitByColor("Yellow");
        apple.getFruitByColor("Red");
    }
}
==================================
바나나!
사과!
```
어떤 메서드가 같은 기능을 수행하거나 각자 구현될 필요 없다면 default 선언을 통해 구현해줄 수 있고, 이는 구현체 클래스에서 따로 구현하지 않아도 된다.  

### static method  
또한 같은 이유로 java8 이후로 interface 내에서 static 선언이 가능해졌다.  
```java  
public class Main {
    public static void main(String[] args) {

        interface Fruit{
            static void getFruitByColor(String color){ //static 메서드, 구현부를 지닌다.
                if(color == "Yellow"){
                    System.out.println("바나나!");
                }
                else{
                    System.out.println("사과!");
                }
            }
        }
        class Banana implements Fruit {
        }
        class Apple implements Fruit{
        }
        Fruit banana = new Banana();
        Fruit apple = new Apple();

        Fruit.getFruitByColor("Yellow");
        Fruit.getFruitByColor("Red");
    }
}  
==================================
바나나!
사과!
```  
default method와 마찬가지로 구현부를 가지는 static 메서드지만 둘은 다르다.  
default method의 경우 <span class = "o">@Override를 통해 이후 다시 구현, 재정의 해줄 수 있지만</span> static의 경우 <span class = "o">구현, 재정의를 할 수 없다.</span>  

이 점을 잘 구분해 사용해야할 것 같다.

## 참고 문서  
-추상클래스  
[JavaDocs](https://docs.oracle.com/javase/tutorial/java/IandI/abstract.html)  

-인터페이스  
[JavaDocs](https://docs.oracle.com/javase/tutorial/java/IandI/createinterface.html)  
  
-다형성  
[https://2ham-s.tistory.com/103](https://2ham-s.tistory.com/103)

