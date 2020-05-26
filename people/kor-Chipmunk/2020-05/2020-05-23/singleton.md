# 디자인패턴 - 싱글톤

## 목차

1. Eager Initialization (Early Loading)
2. Static Block Initialization
3. Lazy Initialization
4. Thread Safety
5. Double-Checked Locking
6. Bill Pugh Solution
7. Reflection 을 이용해 싱글톤 무력화시키기
8. Enum 싱글톤
9. 싱글톤과 직렬화
10. 싱글톤 패턴의 실제 사용 예

## 싱글톤 패턴

전체 애플리케이션 안에서 단 하나의 인스턴스만 생성되도록 강제하는 디자인 패턴이다. 인스턴스 생성은 외부에서 이루어져선 안된다. 접근 제어자를 통해 싱글톤 패턴으로 정의된 클래스 내부에서만 생성되어야 한다.

## 1. Eager Initialization (Early Loading)

`EagerSingle` 클래스가 로드될 때 `EagerSigleton` 인스턴스가 생성된다. 가장 간단한 방법이지만, 애플리케이션에서 한 번도 사용하지 않더라도 인스턴스는 항상 생성된다는 단점이 있다.

```java
public class EagerSingleton {
    private static EagerSingleton instance = new EagerSingleton();
    
    // private constructor
    private EagerSingleton() {
    }
    
    public static EagerSingleton getInstance() {
        return instance;
    }
}
```

static 초기화가 필요한 경우 클래스가 로드된다. 로드된 클래스는 계속해서 메모리에 남을 수 있다.

static 접근 제어자로 `EagerSingleton` 클래스는 항상 로드된다. `instance` static 변수에 `EagerSingleton` 인스턴스가 생성되어 할당한다.

위 코드 만으로 `EagerSingleton.getInstance()` 메소드를 호출하지 않는 경우 인스턴스가 생성되지 않는다. `EagerSingleton` 클래스에 `getInstance()` 메소드 하나만 존재하기 때문이다. `EagetSingleton` 클래스가 사용되는 경우 `getInstance()` 메소드 하나 밖에 없기에 `getInstance()` 호출 외에는 `EagerSingleton` 클래스가 사용될 수 없다. 사용되지 않는 클래스는 로드되지 않는다. 따라서 클래스가 로드되지 않으면 static 초기화도 진행되지 않는다.

만약 `EagerSingleton` 클래스에 다른 static 메소드가 존재하고, 이 다른 메소드가 `getInstance()` 메소드가 호출되기 전에 어딘가에서 호출된다면, `getInstance()`를 호출하지 않아도 `EagerSingleton` 인스턴스는 생성된다. 다른 static 메소드로 `EagerSingleton` 클래스가 로드되기 때문이다.

클래스 로딩을 확인하기 위해 static으로 선언된 임의의 메소드를 만든다. `main` 메소드에 `EagerSingleton.~~~()` 메소드를 호출한다. 이 때, `getInstance()` 메소드는 호출하지 않는다. JVM 옵션으로 `-verbose:class`를 입력하고 실행하면 로딩되는 클래스들을 모두 출력한다. `EagerSingleton` 클래스가 로딩되는 것을 확인할 수 있다.

```
[Loaded singleton.EagerSingleton from file:~]
```

## 2. Static Block Initialization

Eager Initialization과 유사하지만, 인스턴스가 static bloc 내에서 만들어지고 예외 처리까지 할 수 있다.

```java
public class StaticBlockSingleton {
    private static StaticBlockSingleton instance;
    private StaticBlockSingleton(){}
    
    //static block initialization for exception handling
    static{
        try{
            instance = new StaticBlockSingleton();
        }catch(Exception e){
            throw new RuntimeException("Exception occured in creating singleton instance");
        }
    }
    public static StaticBlockSingleton getInstance(){
        return instance;
    }
}
```

`new` 연산으로 새로운 메모리를 힙에 할당하지 못한다면 `OutOfMemoryError` 예외를 내려준다.

## 3. Lazy Initialization

Eager Initialization의 단점을 보완한다. 생성자는 역시 `private` 접근 제어자로 클래스 내부에서만 호출할 수 이싿. 객체 생성은 static 메소드인 `getInstance()` 메소드를 이용해야만 한다.

```java
public class LazyInitializedSingleton {
    private static LazyInitializedSingleton instance;
    
    private LazyInitializedSingleton() {}
    
    public static LazyInitializedSingleton getInstance(){
        if(Objects.isNull(instance)) {
            instance = new LazyInitializedSingleton();
        }
        return instance;
    }
}
```

`getInstance()` 메소드에서 변수 `instance`가 `null`인 경우에만 인스턴스를 새로 생성하므로 싱글톤 패턴에 해당된다. `getInstance()` 메소드를 호출 해야만 인스턴스가 생성 되기에 Eager Initialization의 단점을 보완했다.

그러나 새로운 문제점이 있는데 바로 스레드에 안전하지 않다. 멀티 스레드 환경에서 동시에 `getInstance()`를 호출하고 동시에 `instance` 변수의 `null` 값을 검사하는 상황이라면, 여러 개의 인스턴스가 생성되는 꼴이 된다.

## 참고 사이트

1. [https://yaboong.github.io/design-pattern/2018/09/28/thread-safe-singleton-patterns/]()
2. [https://dzone.com/articles/all-about-the-singleton]()
3. [https://stackoverflow.com/questions/4785343/what-if-new-fails]()