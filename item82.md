# Effective Java

## 스레드 안전성 수준을 문서화하라(82)

> 멀티스레드 환경에서도 API를 안전하게 사용하게 하려면 클레스가 지원하는 스레드 안전성 수준을 정확히 명시해야 한다.

1. 스레드 안전성 수준
    * 불변(immutable)
      * 상수와 같아서 외부 동기화가 필요 없다.
      * String, Long, BigInteger...
    * 무조건적인 스레드 안전(unconditionally thread-safe)
      * 해당 클래스의 인스턴스는 수정될 수 있지만 내부에서도 충실히 동기화하여 별도의 외부 동기화없이 동시에 사용해도 안전하다.
      * AtomicLong, ConcurrentHashMap...
    * 조건부 스레드 안전(conditionally thread-safe)
      * 무조건적인 스레드 안전성과 같지만 일부 메서드는 동시에 사용하려면 외부 동기화가 필요하다.
      * Collections.synchronized 래퍼 메서드가 반환한 컬렉션(예를 들어, Collections.synchronizedList().iterator())
    * 스레드 안전하지 않음(not thread-safe)
      * 해당 클래스의 인스턴스는 수정될 수 있으며 동시에 사용하려면 각각의 메서드 호출을 클라이언트가 선택한 외부 동기화 로직으로 감싸야 한다.
      * ArrayList, HashMap...
    * 스레드 적대적(thread-hostile)
      * 외부 동기화로 감싸더라도 멀티스레드 환경에서 안전하지 않다.

2. 동기화에 대한 문서화
    * 조건부 스레드 안전한 클래스는 주의하여 문서화해야 한다.
        ~~~java
        /**
        * It is imperative that the user manually synchronize on the returned
        * map when iterating over any of its collection views
        * 반환된 맵의 콜렉션 뷰를 순회할 때 반드시 그 맵으로 수동 동기화하라
        * 
        *  Map m = Collections.synchronizedMap(new HashMap());
        *      ...
        *  Set s = m.keySet();  // Needn't be in synchronized block
        *      ...
        *  synchronized (m) {  // Synchronizing on m, not s!
        *      Iterator i = s.iterator(); // Must be in synchronized block
        *      while (i.hasNext())
        *          foo(i.next());
        *  }
        */
        ~~~
3. 외부에 공개된 lock
    * 외부에 공개된 락을 사용할 경우 유연한 코드를 만들 수 있지만, 클라이언트가  서비스 거부 공격(denial-of-service attack)을 수행할 수 있다.
    * 아래와 같이 비공개 락 객체를 사용한다.
        ~~~java
        private final Object lock = new Object();

            public void someMethod() {
                synchronized(lock) {
                    // do something
                }
            }
        ~~~