## 자바8 Defualt Method

- 자바8 이전에는 인터페이스에서 정의한 메소드는 모두 그 인터페이스를 구현하는 클래스에서 필수로 구현해주어야했다.

- 자바8에서 `default method`를 사용하면 메서드의 구현을 포함하는 인터페이스를 정의할 수 있다.

    ```java
    public interface List<E> extends Collection<E>{
        default void sort(Comparator<? super E> c) {
            Object[] a = this.toArray();
            Arrays.sort(a, (Comparator) c);
            ListIterator<E> i = this.listIterator();
            for (Object e : a) {
                i.next();
                i.set((E) e);
            }
        }
    }
    ```

    - **인터페이스에서 이미 구현**을 했으니 그 인터페이스를 구현하는 클래스는 **구현을 추가적으로 할 필요가 없다.**

    - 기존 인터페이스를 구현하는 클래스는 자동으로 인터페이스의 디폴트 메서드를 상속받는다.

    - 구현하는 클래스에서 당연히 `default method`를 `override`할 수 있다.

- 장점
    - 기존 자바 API의 호환성을 유지하면서 라이브러리를 변경할 수 있다.
    - 자바 API의 인터페이스에 디폴트 메서드를 추가해도 해당 인터페이스를 구현하는 클래스들이 해당 메소드를 구현해 줄 필요가 없다.
    - **선택형 메소드**
        - 인터페이스를 구현할 때 필요없는 추상메소드여도 반드시 구현을 해줬어야하므로 빈 구현을 했다면, `default method`를 걸어놓고 인터페이스에서 구현을 해주면 implements하는 클래스에서 쓸데없는 빈 구현의 코드가 사라진다.
     


## 자바8 interface vs abstract class

- `default method`가 추가되면서 인터페이스와 추상클래스간의 기능적인 차이가 줄었다.

- 자바8 부터는 인터페이스도 default 키워드를 통해 구현체를 가질 수 있는데 왜 추상클래스를 사용해야 하는지?

- interface와 abstract class의 **공통점**은?
    - 추상클래스와 인터페이스는 인스턴스화 하는 것이 불가능
    - 구현부가 있는 메소드와 없는 메소드 모두 가질 수 있다

- 그렇다면 **차이**는?

    - 인터페이스에서 모든 변수는 기본적으로 `public static final`
    - 인터페이스에서 모든 메소드는 `public abstract`
    - 추상클래스에서는 **static 이나 final 이 아닌 필드를 지정할 수 있고, public, protected, private 메소드를 가질 수 있다.**
    - 추상클래스는 어쨌든 클래스이므로 **한 개만 상속**가능
    - 인터페이스는 **다중구현**가능

- 언제 인터페이스와 추상클래스를 구분해서 사용하는 것이 좋을까?

    - **추상클래스**
        - **관련성이 높은 클래스 간에 코드를 공유**하고 싶은 경우
        - 추상클래스를 상속받은 클래스들이 공통으로 가지는 메소드와 필드가 많은 경우
        -  public 이외의 접근제어자(protected, private) 사용이 필요한 경우
        - on-static, non-final 필드 선언이 필요한 경우
        - 템플릿 메소드 패턴의 경우
            - 템플릿 메소드 내부에서만 호출되어야 할 메소드들이 인터페이스를 사용한다면 public 제어자에 의해 의도치 않은 사용처에서 호출될 위험이 있지 않을까
    
    - **인터페이스**
        - **구현클래스들 간에 관련성이 없는 경우**/ 서로 관련성이 없는 클래스들이 인터페이스를 구현하게 되는 경우 ex) `Comparable`
        - **다중구현**을 허용하고 싶은 경우
        - 구현이 되어있는지 신경쓰고 싶지 않은 경우?

<br>

## 참고자료

https://asfirstalways.tistory.com/353

https://yaboong.github.io/java/2018/09/25/interface-vs-abstract-in-java8/

https://yaboong.github.io/design-pattern/2018/09/27/template-method-pattern/