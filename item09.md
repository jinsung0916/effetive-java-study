# Effective Java
## try-finally 보다는 try-with-resources를 사용하라(9)

<br>

### `try-with-resources` 란?
~~~java
InputStream in = new FileInputStream(src);
try {
    // do something
} finally {
    in.close();
}

try(InputStream in = new FileInputStream(src);) {
    // do something
}
~~~

<br>

### `try-with-resources` 를 사용해야 하는 이유

1. 코드가 단순해진다.
    ```java
    // before - 여러 개의 리소스를 처리해야 하는 경우 코드가 지저분해진다.
    static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
            } finally {
                out.close();
            }
        } finally {
            in.close();
        }
    }

    // after - 코드가 단순해진다.
    static void copy(String src, String dst) throws IOException {
        try (InputStream in = new FileInputStream(src);
            OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
        }
    }

    // catch 문을 사용할 수 도 있다.
    public static String firstLineOfFile(String path) throw IOException {
        try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            return br.readLine();
        } catch (Exception e) {
            return defaultVal;
        }
    }
    ```
2. 예외 처리
    * `try` 블록과 `finally` 블록에서 모두 예외가 발생할 수 있는데,
    두 번째 예외가 첫 번째 예외를 완전히 집어삼켜 버린다.
    그러면 스택 추적 내역에 첫 번째 예외에 관한 정보는 남지 않게 되어, 실제 시스템에서의 디버깅을 어렵게 한다.
    ```java
    // before
	public static void main(String[] args) {
		MyResource myResource = new MyResource();
		try {
			myResource.doSomething();
		} finally {
            // 여기서 예외가 발생할 경우, try 블록에서 발생한 예외를 잡아먹는다.
			myResource.close();
		}
	}

    // after
    public static void main(String[] args) {
		try (MyResource myResource = new MyResource()){
			myResource.doSomething();
		}
	}
    ```

<br>

### `AutoClosable` 인터페이스를 구현하자

```java
package java.lang;

public interface AutoCloseable {

    void close() throws Exception;

}
```

```java
class FileInputStream extends InputStream {
    public void close() throws IOException {
        if (closed) {
            return;
        }
        synchronized (closeLock) {
            if (closed) {
                return;
            }
            closed = true;
        }

        FileChannel fc = channel;
        if (fc != null) {
            // possible race with getChannel(), benign since
            // FileChannel.close is final and idempotent
            fc.close();
        }

        fd.closeAll(new Closeable() {
            public void close() throws IOException {
               fd.close();
           }
        });
    }
}
```