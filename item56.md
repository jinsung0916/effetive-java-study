# Effective Java

## 공개된 API 요소에는 항상 문서화 주석을 작성하라
> 여러분의 API를 올바로 문서화하려면 고개된 모든 클래스, 인터페이스, 매서드, 필드 선언에 문서화 주석을 달아야 한다.

* @param, @return, @throws
    * 모든 매개변수에 @param 태그를 달아야 한다.
    * 반환 타입이 void 가 아니라면 @return 태그를 달야야 한다.
    * 발생할 수 있는 모든 예외에 @throws 태그를 달야아 한다. 
    ~~~java
    /**
    * Returns the element at the specified position in this list.
    *
    * <p>This method is <i>not</i> guaranteed to run in constant
    * time. In some implementations it may run in time proportional
    * to the element position.
    *
    * @param  index index of element to return; must be
    *         non-negative and less than the size of this list
    * @return the element at the specified position in this list
    * @throws IndexOutOfBoundsException if the index is out of range
    *         ({@code index < 0 || index >= this.size()})
    */
    E get(int index) {
        return null;
    }
    ~~~
* 요약설명
    * 첫 문장은 주로 요약 설명이다.
    * 한 클래스 혹은 한 인터페이스 안에 요약설명이 중복되면 안된다(ex. 오버로딩 된 매서드).
    * 마침표에 주의해야 한다.
        * 잘못된 예
            ~~~java
            /**
            * A suspect, such as Colonel Mustard or Mrs. Peacock. 
            */
            ~~~
        * 올바른 예 
            ~~~java
            /**
            * A suspect, such as Colonel Mustard or {@literal Mrs. Peacock}.
            */

            /**
            * {@summary A suspect, such as Colonel Mustard or Mrs. Peacock.}
            */
            ~~~
* {@code ...}
    * 태그로 감싼 내용을 코드용 폰트로 렌더링한다
    * 태그로 감싼 내용에 포함된 HTML요소나 다른 자바독 태그를 무시한다
    * \<pre>{@code ...코드... }\</pre> 와 같이 사용하면 여러줄로 된 코드도 작성 가능하다.
* @implSpec
    * 해당 메서드와 하위 클래스 사이의 계약을 설명한다.
    * 하위 클래스들이 그 메서드를 상속하거나 super키워드를 이용해 호출할 때 그 메서드가 어떻게 동작하는지를 명확히 인지하고 사용하도록 해야한다.
    ~~~java
    /**
    * Returns true if this collection is empty.
    * 
    * @implSpec
    * This implementation returns {@code this.size == 0}.
    *
    * @return true if this collection is emplty
    */
    public boolean isEmpty() { ... }
    ~~~
* {@literal ...}
    * HTML 마크업이나 자바독 태그를 무시하게 해준다.
    ~~~java
    /**
     * A geometric series converges if {@literal |r| < 1}.
     */
    ~~~
* {@index}
    * 중요한 용어를 추가로 색인화할 수 있다.
    ~~~java
    /**
    * This method complies with the {@index IEEE 754} standard.
    */
    public void function() {
    }
    ~~~
* 제네릭 문서화
    * 모든 타입 매개변수에 주석을 달아야 한다.
    ~~~java
    /**
    * An object that maps keys to values.  A map cannot contain duplicate keys; 
    * ...
    * @param <K> the type of keys maintained by this map
    * @param <V> the type of mapped values
    */
    public interface Map<K,V> { ... }
    ~~~
* 열거 타입 문서화
    * 상수들, 열거 타입 자체, public메서드에도 주석을 달아야 한다
    ~~~java
    /**
    * An instrument section of a symphony orchestra.
    */
    public enum OrchestraSection {
        /** Woodwinds, such as flute, clarinet, and oboe. */
        WOODWIND,

        /** Brass instruments, such as french horn and trumpet. */
        BRASS,

        /** Percussion instruments, such as timpani and cymbals. */
        PERCUSSION,

        /** Stringed instruments, such as violin and cello. */
        STRING;
    }
    ~~~
* 애너테이선 타입 문서화
    * 타입 자체, 멤버들에도 모두 주석을 달아야 한다.
    ~~~java
    /**
    * Indicates that the annotated method is a test method that
    * must throw the designated exception to pass.
    */
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface ExceptionTest {
        /**
        * The exception that the annotated test method must throw
        * in order to pass. (The test is permitted to throw any
        * subtype of the type described by this class object.)
        */
        Class<? extends Throwable> value();
    }
    ~~~