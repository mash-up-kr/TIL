### 동기화

- **공유 자원에 대한 스레드들의 접근 문제** = **Critical Section(임계 구역)** 에 대한 스레드들의 접근 문제를 해결하기 위해 동기화를 해야한다.

- 동기화의 핵심 요소
    - 동기화를 이루기 위해서는 Wait와 Signaling이라는 개념을 이용해야한다. (자바에서는 각각 wait와 notify를 의미)

    - **Wait** 
    
        1. 동기화된 지역이 동작하기 전에 이 구간을 들어가도 되는지 확인한다.

        2. 조건이 충족되어 들어가도 된다면 Unsafe Region에 존재하는 변수 따위를 이용하러 들어간다.

        3. 조건이 충족되지 않는다면 계속해서 기다린다.(조건이 충족 될 때 까지 Unsafe Region에 들어갈 수 없다.)
    
    - **Signaling**

        - 동기화된 지역에서 모든 일을 마치고 Wait하고 있는 스레드를 깨워주어 들어갈 수 있도록 한다.

    - 이러한 핵심 요소를 쓰는 동기화 방식은 크게 Mutex, Semaphore 두가지가 존재한다.

<br>

### Mutex

- 뮤텍스는 Unsafe Region을 동기화된 지역으로 만들기 위해 Critical Section을 설정한다.
    
- 하나의 스레드만이 Critical Section에서 행동할 수 있다.

- lock을 가지고 있을 때만 공유 데이터에 접근이 가능하다.
    
- 화장실에 갈 때 키를 가진 사람만이 갈 수 있다. 일을 다 본 후에는 키를 반납하고, 대기큐에 들어가 있는 그 다음 사람이 갈 수 있다.

<br>

### Semaphor

- 세마포어 또한 동기화된 지역을 만들기 위해 한정된 스레드를 Critical Section에 출입 시킨다.

- 동시에 리소스에 접근 할 수 있는 **허용 가능한 counter의 개수**를 가지고 있다.

- 병원에 있는 어느 한 병실에 5명(counter)까지 들어 갈 수 있다고 한다면, 5명까지 들어 갈 때 counting을 하고 5명 이후에는 밖에서 기다려야 한다. 한 사람이 나오게 되면 다음 사람이 들어 갈 수 있게 된다.

- count수에 따라서 1개이면 binary semaphore, 여러 개이면 counting semaphore

- binary semaphore는 뮤텍스와 개념적으로 같다.

<br>

### Mutex vs Semaphor

- 공유자원에 대한 접근권한, Lock이라는 키를 한 개만 가지고 있는 것은 뮤텍스(mutex)이다.

- Lock이라는 키를 여러 개 가질 수 있는 것은 세마포어(Semaphore)이다.

<br>

### Monitor

- Mutex와 Condition value를 가지고 있는 동기화 메커니즘이다.

- condition value란 wait와 signal을 의미한다.(자바에서는 wait와 notify)

- 이때 자바에서 모든 객체는 Object 클래스를 상속 받는다. 
    - 이 Object 클래스에는 wait(), notifyAll(), notify() 메소드를 가지고 있는데 이게 바로 Condition Variables 역할이다. 

    - 따라서 모든 자바 객체는 Monitor를 가지고 있다는 것을 알 수 있다.

- 자바의 Synchronized 키워드

- mutex의 상위 호환 버전

<br>

## 참고자료

https://slenderankle.tistory.com/197

https://www.crocus.co.kr/1261

https://www.crocus.co.kr/563


