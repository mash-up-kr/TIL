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
        try (
            Selector selector = Selector.open();
            ServerSocketChannel serverSocketChannel = ServerSocketChannel.open()
        ) {

            if ((serverSocketChannel.isOpen()) && (selector.isOpen())) {
                serverSocketChannel.configureBlockig(false);
                serverSocketChannel.bind(new InetSocketAddress(8888))
            };

            serverSockerChannel.register(selector, SelectionKey.OP_ACCEPT);
            System.out.println("Server Listening~");

            while (true) {
                selector.select();
                Iterator<SelectionKey> keys = selector.selectedKeys().iterator();

                while (keys.hasNext()) {
                    SelectionKey key = (SelectionKey) keys.next();
                    keys.remove();

                    if (!key.isValid()) {
                        continue;
                    }

                    if (key.isAcceptable()) {
                        this.acceptOP(key, selector);
                    } else if (key.isReadable()) {
                        this.readOP(key);
                    } else if (key.isWritable()) {
                        this.writeOP(key);
                    }
                }
            }
        } else {
            
        }
    }
}
```

# 참고 도서

* 자바 네트워크 소녀 네티