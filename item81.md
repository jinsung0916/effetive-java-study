# Effective Java

## wait와 notify보다는 동시성 유틸리티를 애용하라(81)

> wait와 notify는 올바르게 사용하기가 아주 까다로우니 고수준 동시성 유틸리티를 이용하자.
 
1. 실행자 프레임워크
2. 동시성 컬렉션
   * List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 추가한 것이다. 높은 동시성을 위해 동기화를 내부에서 수행한다. 동시성을 무력화하는 것이 불가능하며, 외부에서 락(Lock)을 걸면 오히려 속도가 더 느려진다.
   * 상태 의존적 매서드를 활용하라.
        ~~~java
        private static final ConcurrentMap<String, String> map =
        new ConcurrentHashMap<>();

        public static String intern(String s) {
            String result = map.get(s);
            if (result == null) {
                result = map.putIfAbsent(s, s);
                if (result == null) {
                    result = s;
                }
            }
            return result;
        }
        ~~~
    * 동기화한 컬렉션보다 동시성 컬렉션을 사용해야 한다. 
       * SynchronizedMap vs ConcurrentMap  
    * 동기화 컬렉션의 예: ConcurretMap, BlockingQueue ...
3. 동기화 장치
    * 스레드가 다른 스레드를 기다릴 수 있게 하여 서로의 작업을 조율할 수 있도록 해준다.
    * CountDownLatch, Semaphore, CyclicBarrier, Exchanger, Phaser
        ~~~java
        /**
         * CountDownLatch
         */
        public static long time(Executor executor, int concurrency,
                            Runnable action) throws InterruptedException {
            CountDownLatch ready = new CountDownLatch(concurrency);
            CountDownLatch start = new CountDownLatch(1);
            CountDownLatch done = new CountDownLatch(concurrency);

            for (int i = 0; i < concurrency; i++) {
                executor.execute(() -> {
                    // 타이머에게 준비가 됐음을 알린다.
                    ready.countDown();
                    try {
                        // 모든 작업자 스레드가 준비될 때까지 기다린다.
                        start.await();
                        action.run();
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    } finally {
                        // 타이머에게 작업을 마쳤음을 알린다.
                        done.countDown();
                    }
                });
            }

            ready.await(); // 모든 작업자가 준비될 때까지 기다린다.
            long startNanos = System.nanoTime();
            start.countDown(); // 작업자들을 깨운다.
            done.await(); // 모든 작업자가 일을 끝마치기를 기다린다.
            return System.nanoTime() - startNanos;
        }
        ~~~
        ~~~java
        /**
         * Phaser
         */
        void runTasks(List<Runnable> tasks) {
            Phaser startingGate = new Phaser(1); // "1" to register self     
            // create and start threads
            for (Runnable task : tasks) {
                startingGate.register();
                new Thread(() -> {
                    startingGate.arriveAndAwaitAdvance();
                    task.run();
                }).start();
            }
            // deregister self to allow threads to proceed     
            startingGate.arriveAndDeregister();
        }
        ~~~
4. wait와 notify 메서드
    * wait는 항상 while문 내부에서 호출하도록 해야 한다.
        ~~~java
        //반복문은 wait 호출 전후로 조건이 만족하는지를 검사하는 역할을 한다. 대기 전에 조건을 검사하여 조건이 충족되었다면 wait를 건너뛰게 한 것은 응답 불가 상태를 예방하는 조치다.

        synchronized (obj) {
            while (조건이 충족되지 않았다) {
                obj.wait(); // 락을 놓고, 깨어나면 다시 잡는다.
            }

            ... // 조건이 충족됐을 때의 동작을 수행한다.
        }
        ~~~ 
    * 조건이 만족되지 않아도 스레드가 깨어날 수 있는 경우
      * notify를 호출하여 대기 중인 스레드가 깨어나는 사이에 다른 스레드가 락을 거는 경우
      * 조건이 만족되지 않았지만 실수 혹은 악의적으로 notify를 호출하는 경우
      * 대기 중인 스레드 중 일부만 조건을 충족해도 notifyAll로 모든 스레드를 깨우는 경우
      * 대기 중인 스레드가 드물게 notify 없이 깨어나는 경우. 허위 각성(spurious wakeup)이라고 한다.
    * 일반적으로 notify보다는 notifyAll을 사용하는 것이 안전하다. 깨어나야 하는 모든 스레드가 깨어남을 보장할 수 있으니 항상 정확한 결과를 얻을 수 있다.
