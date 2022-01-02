# Effective Java 
## Clone 재정의는 주의해서 진행하라(13)

<br>

### **Clonable 인터페이스는 안티패턴** - 인터페이스의 행동 호환성이 깨진다.
```java
class Object {
    // Cloneable 인터페이스를 구현하지 않은 객체가 clone 을 호출할 경우 CloneableNotSupportException 을 발생시킨다.
    protected native Object clone() throws CloneNotSupportedException;
}

class MyClass {
    public static void main(String[] args) throws CloneNotSupportedException {
        Myclass instance = new MyClass();
        intance.clone(); // throw CloneNotSupportException   
    }
}
```
행동 호환성
>타입의 이름 사이에 개념적으로 어떤 연관성이 있다고 하더라도 행동에 연관성이 없다면 is-a관계를 사용하지 말아야 한다.
```java
class Bird {
    public void fly() {...}
}

class Penguin extends Bird {
    @Override
    public void fly(){
        throw new UnsupportedOperationException();
    }
}
```

<br>

### 어쩔 수 없이 Cloneable 인터페이스를 사용해야 한다면?

1. Clonable 인터페이스를 구현하는 모든 클래스는 clone 을 재정의해야 한다.
    > By convention, the returned object should be obtained by calling super.clone. If a class and all of its superclasses (except Object) obey this convention, it will be the case that x.clone().getClass() == x.getClass().
2. 생성자를 호출하지 않고도 객체를 생성할 수 있기 때문에 생성자와 동일한 규약을 준수해야 한다.

    > 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.
    ```java
    // 클래스에 정의된 모든 필드가 기본 타입이거나 불편 객체를 참조하는 경우
    @Overide 
    public PhoneNumber clone() {
        try {
            return (PhoneNumber) super.clone();
        } catch(CloneNotSupprotedException e) {
            throw new AssertionError(); // 일어날 수 없는 일이다.
        }
    }

    // 클래스가 가변 객체를 참조하는 경우(1)
    public Stack() {
        private Object[] element; // 가변 객체
        private int size = 0;
        private static final int DEFAULT_INITIAL_CAPACITY = 16;

        ...  

        @Overide
        public Stack clone() {
            try {
                Stack result = (Stack) super.clone();
                result.element = elements.clone(); // 배열의 clone() 매서드는 
                return result;
            } catch (CloneNotSupportedException e) {
                throw new AssertionError();
            }
        }
    }
    
    // 클래스가 가변 객체를 참조하는 경우(2)
    public class HashTable implements Cloneable {
        private Entry[] buckets = ...;

        private static class Entry {
            final Object key;
            Object value;
            Entry next;

            public Entry(Object key, Object value, Entry next) {
                this.key = key;
                this.value = value;
                this.next = next;
            }
            
            // 1. 재귀적 방법
            Entry deepCopy() {
                return new Entry(key, value, next == null ? null : next.deepCopy());
            }

            // 2. StackOverflow 를 피하기 위해 반복자를 이용
            Entry deepCopy() {
                Entry result = new Entry(key, value, next);
                for (Entry p = result; p.next != null; p = p.next)
                    p.next = new Entry(p.next.key, p.next.value, p.next.next);
                return result;
            }
        }

        @Override
        protected HashTable clone() {

            try {
                HashTable result = (HashTable) super.clone();
                result.buckets = new Entry[buckets.length];
                for (int i = 0; i < buckets.length; i++) {
                    if (buckets[i] != null)
                        result.buckets[i] = buckets[i].deepCopy();
                }
                return result;
            } catch (CloneNotSupportedException e) {
                throw new AssertionError();
            }
        }
    }

    ```
    Cloneable을 구현하는 모든 클래스는 **clone() 메소드를 재정의**해야하고, **접근 제한자는 public**으로 구현하여 모든 패키지에서 접근이 가능하게 해야 한다. 또한 **반환 타입은 클래스 자신으로 변경**하고 가변 객체가 존재할 시 **가변 객체도 복제될 수 있는 로직을 구성**하여 재정의해야 한다.

3. 상속용 클래스는 Cloneable 을 구현해서는 안된다.
    ```java
    // 하위 클래스에서 재정의하지 못하게 clone 을 퇴회시킨다.
    @Override
    protected final Object clone() throws CloneNotSupportException {
        throw new CloneNotSupportException();
    }
    ```
4. 스레드 안전 클래스를 작성할 때는 clone 매서드 를 동기화 해 주어야 한다.
    ```java
    class Object {
        // clone 의 기본 구현는 다중 스레드 환경이 고려되지 않음.
        protected native Object clone() throws CloneNotSupportedException;
    }

    class synchronized MyClass implement Cloneable {
        public MyClass clone() {...}
    }
    ```
    
<br>

### 따라서, 복사 생성자 혹은 복사 팩터리를 사용하라
> Clonable 을 이미 구현한 클래스를 확장하는 경우가 아니라면, 복사 생성자 혹은 복사 팩터리를 사용하라

```java
// 복사 생성자
public Yum(Yum yum)(...);

// 복사 팩터리
public static Yum newinstance(...);

public <K, V> static Map<K, V> newInstance() {
    // upcasting
    return new HashMap<K, V>();
    return new TreeMap<K, V>();
}

public MyClass clone() throw CloneNotSupprotedException {
    // downcasting
    return (MyClass) super.clone();
}
```