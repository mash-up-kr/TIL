# 블로킹과 논블로킹

소켓의 동작 방식은 블로킹 모드와 논블로킹 모드로 나뉜다. 블로킹은 요청한 작업이 성공하거나 에러가 발생하기 전까지 응답을 반환하지 않는 경우를 뜻한다. 반면 논블로킹은 요청한 작업의 성공 여부와는 상관없이 결과를 반환하는 것을 말한다. 요청의 응답값에 따라 성공, 에러 여부를 판단한다.

JDK 1.3 까지 블로키 방식의 I/O만 지원했다. JDK 1.4 이후에 NIO (New I/O) 명칭의 논블로킹 I/O API가 추가됐다. 새로운 입출력이라는 이름으로, 소켓도 입출력 채널의 하나로 NIO API를 사용할 수 있다. 따라서 NIO API로 블로킹과 논블로킹 소켓 사용이 가능하다.

## 블로킹 소켓

블로킹 소켓은 `ServerSocket`, `Socket` 클래스를 사용한다.  
논블로킹 소켓은 `ServerSockerChannel`, `SocketChannel` 클래스를 사용한다.

```java
public class BlockingServer {
    public static void main(String[] args) throws Exception {
        BlockingServer server = new BlockingServer();
        server.run();
    }

    private void run() throws IOException {
        ServerSocket server = new ServerSocket(8888);
        System.out.println("Listening~");

        while (true) {
            Socket sock = server.accept();
            System.out.println("Client connected.");

            OutputStream out = sock.getOutputStream();
            InputStream in = sock.getInputStream();

            while (true) {
                try {
                    int request = in.read();
                    out.write(request);
                }
                catch (IOException e) {
                    break;
                }
            }
        }
    }
}
```

다음 `telnet` 서비스로 윈도우 커맨드라인 도구에서 접속해볼 수 있다.

```
telnet localhost 8888
```

서버에 접속 시 `accept()` 메소드 반환이 완료되어 새 클라이언트 소켓을 생성한다.

`in.read()` 메소드에서는 클라이언트로부터 데이터를 수신받는다. 수신될 때 까지 기다리며 메인 쓰레드가 블로킹된다. 메소드의 처리가 완료될 때 까지 반환하지 않고 기다리는 것을 `블로킹` 방식이라 한다.


## 블로킹 소켓 단점

* 입출력 시 스레드의 블로킹이 발생하기 때문에 동시에 여러 클라이언트 처리를 못 한다.
  * 연결된 클라이언트 별로 스레드를 할당하는 방법이 있다. 새 클라이언트가 접속할 시 스레드를 할당하고 그 스레드에 해당 클라이언트의 I/O 처리를 담당한다.
  * 단, 서버 소켓의 병목 지점은 `accept()` 메소드다. 단위 시간에 하나의 연결만을 처리하는 블로킹 함수이므로 동시에 접속했을 때 대기 시간이 있을 수 있다.
  * 스레드 수가 증가하면 힙 메모리가 부족해지는 OOM (Out of Memory) 오류가 발생한다. 이 때, 서버는 더 이상 운영이 불가능하다.
  * 서버에서 생성되는 스레드 수를 제한하는 `스레드 풀링(Thread Pooling)` 기법을 사용하기도 한다. 스레드 풀에서 쓸 수 있는 스레드를 가져와 클라이언트 소켓의 I/O를 담당하는 방식이다. 동시 접속자가 스레드 수에 의존한다는 문제점을 가지고 있다.
  * 스레드 풀의 크기를 자바 힙 메모리가 허용하는 최대 한도에 도달할 때 까지 늘리는 것이 합당한지 두 가지 관점에서 생각해 볼 필요가 있다.
    1. 자바의 가비지 컬렉션 관점이다. 하드웨어의 메모리가 늘어남에 따라 가비지 컬렉션이 처리해야 할 객체가 늘어난다면, 작업을 완료하기 위해 스레드들의 멈추는 시간이 길어진다. 그 때 서버의 기능이 동작하지 않는 이슈가 발생할 수 있다.
    2. 운영체제의 컨텍스트 스위칭 관점이다. 컨텍스트 스위칭은 한 프로세스에서 수행되는 스레드들이 CPU 자원을 차지하기 위해 자신의 상태를 변경하는 작업이다. 서버 프로세스의 수 많은 스레드들이 CPU 자원을 가져오기 위해 경쟁한다. 이 과정에서 자원을 소모하므로 실제 작업에 쓸 CPU 자원이 적어진다.

따라서 블로킹 소켓을 사용한 서버는 충분히 많은 동시접속 사용자를 유연하게 수용하지 못한다.

# 참고 도서

* 자바 네트워크 소녀 네티