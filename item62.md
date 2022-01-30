# Effective Java

## 다른 타입이 적절하다면 문자열 사용을 피하라(62)
> 이번 장에서는 문자열을 쓰지 말아야 할 사례를 다룬다.

1. 문자열은 다른 값 타입을 대신하기에 적합하지 않다.
   * 입력받을 데이터가 진짜 문자열일때만 String으로 받는게 좋다.
   * 수치형이라면 int, float, BigInteger 으로 변환해야 한다.
   * 예/아니오라면 열거타입이나 boolean으로 변환해야 한다.
   * 적절한 타입이 없다면 새로 작성하라.
2. 문자열은 열거 타입을 대신하기에 적합하지 않다.
    * 상수는 문자열 대신 열거타입으로 표현하자(item34).
3. 문자열은 혼합 타입을 대신하기에 적합하지 않다.
   ~~~java
    // 따라하지 말것!
    String compoundKey = className = "#" + i.next();
    ~~~
    * 요소를 개별로 접근하려면 문자열을 파싱해야 해서 느리고, 귀찮고, 오류 가능성도 커진다. 적절한 equals, toString, compareTo 메서드를 제공할 수 없으며, String이 제공하는 기능에만 의존해야 한다.
    * 차라리 전용 클래스를 새로 만드는 편이 낫다. 이런 클래스는 보통 private 정적 멤버 클래스로 선언해야 한다.

4. 문자열은 권한을 표현하기에 적합하지 않다.
   * 잘못된 예 - 문자열을 사용해 권한을 구분하였다. - 클라이언트 간 키 중복 문제 발생
        ~~~java
        public class ThreadLocal { 
                private ThreadLocal() {} 

                public static void set(String key, Object value);

                public static Object get(String key); 
            }
        ~~~
   * Key 클래스로 권한을 구분했다.
        ~~~java
        public class ThreadLocal {
                private ThreadLocal() {} 
                public static class Key {}
        
                public static Key getKey() { return new Key(); } 
                public static void set(Key key, Object value);
                public static Object get(Key key); 
            }
        ~~~
    * 리팩터링하여 Key를 ThreadLocal로 변경
        ~~~java
        public final class ThreadLocal { 
            public ThreadLocal();
             public void set(Object value); 
             public Object get(); 
        }   
        ~~~
    * 매개변수화하여 타입안전성 확보
        ~~~java
        public final class ThreadLocal<T> { 
            public ThreadLocal();
             public void set(T value); 
             public T get(); 
        }   
        ~~~