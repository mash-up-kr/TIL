# 블로킹과 논블로킹

## 논블로킹 소켓

입출력 소켓의 `read()` 논블로킹 모드 메소드의 동작 방식은 다음과 같다.  
`read()` 메소드의 반환값은 소켓에서 읽어들인 바이트 길이다. 만약 클라이언트가 데이터를 아직 전송하지 않았거나 데이터가 수신 버퍼까지 도달하지 않았다면 바이트 길이 0을 반환한다.

## 논블로킹 모드 서버 예제

```java
public class NonBlockingServer {
    private Map<SocketChannel, List<byte[]>> keepDataTrack = new HashMap<>();
    private ByteBuffer buffer = ByteBuffer.allocate(2 * 1024);

    private void startEchoServer() {
        try ( // 1
            Selector selector = Selector.open(); // 2
            ServerSocketChannel serverSocketChannel = ServerSocketChannel.open() // 3
        ) {

            if ((serverSocketChannel.isOpen()) && (selector.isOpen())) { // 4
                serverSocketChannel.configureBlockig(false); // 5
                serverSocketChannel.bind(new InetSocketAddress(8888)); // 6

                serverSockerChannel.register(selector, SelectionKey.OP_ACCEPT); // 7
                System.out.println("Server Listening~");

                while (true) {
                    selector.select(); // 8
                    Iterator<SelectionKey> keys = selector.selectedKeys().iterator(); // 9

                    while (keys.hasNext()) {
                        SelectionKey key = (SelectionKey) keys.next();
                        keys.remove(); // 10

                        if (!key.isValid()) {
                            continue;
                        }

                        if (key.isAcceptable()) { // 11
                            this.acceptOP(key, selector);
                        } else if (key.isReadable()) { // 12
                            this.readOP(key);
                        } else if (key.isWritable()) { // 13
                            this.writeOP(key);
                        }
                    }
                }
            } else {
                System.out.println("[Error] Cannot create a server socket.");
            }
        } catch (IOException ex) {
            System.out.println(ex);
        }
    }

    private void acceptOP(SelectionKey key, Selector selector) throws IOException {
        ServerSocketChannel serverChannel = (ServerSocketChannel) key.channel(); // 14
        SocketChannel socketChannel = serverChannel.accept(); // 15
        socketChannel.configureBlocking(false); // 16

        System.out.println("A client connected : " + socketChannel.getRemoteAddress());

        keepDataTrack.put(socketChannel, new ArrayList<byte[]>());
        socketChannel.register(selector, SelectionKey.OP_READ); // 17
    }

    private void readOP(SelectionKey key) {
        try {
            SocketChannel socketChannel = (SocketChannel) key.channel();
            buffer.clear();
            int numRead = -1;
            try {
                numRead = socketChannel.read(buffer);
            } catch (IOException ex) {
                System.err.println("[Error] Cannot read data");
            }

            if (numRead == -1) {
                this.keepDataTrack.remove(socketChannel);
                System.out.println("Disconnected a client : " + socketChannel.getRemoteAddress());
                socketChannel.close();
                key.cancel();
                return;
            }

            byte[] data = new byte[numRead];
            System.arraycopy(buffer.array(), 0, data, 0, numRead);
            System.out.println(new String(data, "UTF-8") + " from " + socketChannel.getRemoteAddress());
            doEchoJob(key, data);
        } catch (IOException ex) {
            System.out.println(ex);
        }
    }

    private void writeOP(SelectionKey key) throws IOException {
        SocketChannel socketChannel = (SocketChannel) key.channel();

        List<byte[]> channelData = keepDataTrack.get(socketChannel);
        Iterator<byte[]> its = channelData.iterator();

        while (its.hasNext()) {
            byte[] it = its.next();
            its.remove();
            socketChannel.write(ByteBuffer.wrap(it));
        }

        key.interestOps(SelectionKey.OP_READ);
    }

    private void doEchoJob(SelectionKey key, byte[] data) {
        SocketChannel socketChannel = (SocketChannel) key.channel();
        List<byte[]> channelData = keepDataTrack.get(socketChannel);
        channelData.add(data);

        key.interestOps(SelectionKey.OP_WRTIE);
    }

    public static void main(String[] args) {
        NonBlockingServer main = new NonBlockingServer();
        main.startEchoServer();
    }
}
```

데이터를 읽고 쓰는 로직이 완전히 분리됐다. 구조적으로 소켓에서 읽은 데이터를 바로 소켓 스트림에 보낼 수 없다.  
모든 이벤트에 공유하는 데이터 객체를 생성하고 그 객체로 각 소켓 채널로 데이터를 전송한다.

1. Java 1.7 기능으로 try 블록이 끝날 때 선언된 자원을 자동으로 해제해준다. 자바 1.6 이하 버전에서는 `finally()` 구문으로 자원을 직접 해제해야 했다.
2. `Selector`는 자바 NIO 컴포넌트 중 하나다. 등록된 채널에 변경 사항이 일어났는지 감지하고 감지된 채널에 접근하게 해준다.
3. 논블로킹 소켓의 서버 소켓 채널을 생성한다. 블로킹 소켓과 다르게 소켓 채널을 먼저 생성하고 사용할 포트를 바인딩한다.
4. `Selector`와 `ServerSocketChannel` 객체가 정상적으로 생성됐는지 확인한다.
5. 소켓 채널의 블로킹 모드의 기본값은 `true`다. 별도로 논블로킹 모드로 설정해줘야만 한다. `ServerSocketChannel` 객체를 논블로킹 모드로 설정했다.
6. 클라이언트 연결을 대기할 포트를 지정한다.
7. `ServerSocketChannel` 객체를 `Selector` 객체에 등록한다. 해당 `Selector` 객체가 감지할 이벤트는 연결 요청에 해당하는 `SelectionKey.OP_ACCEPT` 이벤트다.
8. 등록된 채널에서 변경 사항이 일어났는지 확인한다. `Selector` 객체에 아무런 I/O 이벤트도 발생하지 않으면 스레드는 블로킹된다. 블로킹을 피하고 싶다면 `selectNow` 메소드를 사용한다.
9. `Selector`에 등록된 채널들 중 I/O 이벤트가 발생한 채널 목록을 조회한다.
10. 한 번 감지되면 중복을 막기 위해 조회 목록에서 제거한다.
11. 연결 요청인지 확인한다. 연결 요청 이벤트라면 연결 처리 메소드로 이동한다.
12. 데이터 수신인지 확인한다. 데이터 수신 이벤트라면 데이터 읽기 처리 메소드로 이동한다.
13. 데이터 쓰기 가능인지 확인한다. 데이터 쓰기 가능 이벤트라면 데이터 쓰기 처리 메소드로 이동한다.
14. 연결 요청 이벤트가 발생한 채널은 항상 `ServerSocketChannel` 객체다. 이벤트가 발새한 채널을 `ServerSocketChannel` 객체로 캐스팅한다.
15. `ServerSocketChannel` 객체로 클라이언트의 연결을 수락하고 연결된 소켓 채널을 가져온다.
16. 연결된 클라이언트 소켓 채널을 논블로킹 모드로 설정한다.
17. 클라이언트 소켓 채널을 `Selector`에 등록하여 I/O 이벤트를 감시한다.

## 논블로킹 소켓 특징

1. 더 많은 클라이언트 연결을 수용할 수 있다.
2. 블로킹 소켓에서 논블로킹 소켓으로 고칠 때, 변경해야 할 코드 로직이 매우 많다.
   1. 네티는 사용 소켓의 모드와 상관 없이 개발이 가능하도록 추상화된 전송 API를 제공한다. ( 송수신 로직을 변경하지 않아도 된다. )

# 참고 도서

* 자바 네트워크 소녀 네티