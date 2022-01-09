# Effective Java 
## 맴버 클래스는 되도록 static으로 만들라(24)

<br>

### **중첩 클래스 패턴 4가지**
1. 정적 멤버 클래스
    * 해당 클래스는 다른 클래스 안에 선언되고, 바깥 클래스의 private 멤버에도 접근할 수 있다는 점을 제외하고는 일반 클래스와 동일하다.

    > 바깥 클래스가 표현하는 객체의 한 부분(구성요소)을 나타낼 때 쓴다.
    ~~~java
    public interface Map<K, V> {
    
        interface Entry<K, V> {}

    }

    public abstract class AbstractMap<K,V> implements Map<K,V> {
    
        public static class SimpleEntry<K,V> implements Entry<K,V>, java.io.Serializable {}

    }
    ~~~
    ~~~java
    public class EnumMap<K extends Enum<K>, V> extends AbstractMap<K, V>
    implements java.io.Serializable, Cloneable {
    
        ...

        public Object[] toArray() {
            return fillEntryArray(new Object[size]);
        }

        private Object[] fillEntryArray(Object[] a) {
            int j = 0;
            for (int i = 0; i < vals.length; i++)
                if (vals[i] != null)
                    a[j++] = new AbstractMap.SimpleEntry<>(
                        keyUniverse[i], unmaskNull(vals[i]));
            return a;
        }

    }
    ~~~
2. 비정적 멤버 클래스
    * 비정적 멤버 클래스의 경우 바깥 클래스의 인스턴스와 암묵적으로 연결된다. 이로 인해, 비정적 멤버 클래스의 인스턴스 메소드에서 this를 통해 외부 클래스의 메소드를 호출하거나 외부 클래스를 참조할 수 있다.

    > 비정적 멤버 클래스는 어뎁터를 정의할 때 자주 쓰인다.
    ~~~java
    public class EnumMap<K extends Enum<K>, V> extends AbstractMap<K, V>
    implements java.io.Serializable, Cloneable {

        public Set<K> keySet() {
            Set<K> ks = keySet;
            if (ks == null) {
                ks = new KeySet();
                keySet = ks;
            }
            return ks;
        }

        private class KeySet extends AbstractSet<K> {

            public Iterator<K> iterator() {
                return new KeyIterator();
            }
            public int size() {
                return size;
            }
            public boolean contains(Object o) {
                return containsKey(o);
            }
            public boolean remove(Object o) {
                int oldSize = size;
                EnumMap.this.remove(o);
                return size != oldSize;
            }
            public void clear() {
                EnumMap.this.clear();
            }
        }
    }
    ~~~
3. 익명 클래스
    * 익명 클래스는 이름이 없는 클래스이고 외부 클래스의 멤버 클래스도 아니다. 사용되는 시점에서 선언과 동시에 인스턴스가 만들어진다. 또한, 익명 클래스가 상위 타입(자기 자신 혹은 부모)에 상속한 멤버 외에는 호출할 수가 없다.

    ~~~java
    public static void main(String[] args) {
        List<String> words = Arrays.asList("kim", "taeng", "mad", "play");

        Collections.sort(words, new Comparator<String>() {
            public int compare(String s1, String s2) {
                return Integer.compare(s1.length(), s2.length());
            }
        });
    }
    ~~~
    ~~~java
    public static void main(String[] args) {
        List<String> words = Arrays.asList("kim", "taeng", "mad", "play");

        Collections.sort(words,
                (s1, s2) -> Integer.compare(s1.length(), s2.length()));
    }
    ~~~
     (출처: https://madplay.github.io/post/prefer-lambdas-to-anonymous-classes)
4. 지역 클래스
    * 지역 클래스는 네 가지 중첩 클래스 중 가장 드물게 사용되고 있다.
    * 지역 클래스의 경우 지역 변수를 선언할 수 있는 곳이라면 어디든 선언이 가능하고 유효 범위(메소드 종료 시 메모리에 사라짐)도 지역 변수와 동일하다.
    ~~~java
    public class LocalExample {
        private int number;

        public LocalExample(int number) {
            this.number = number;
        }

        public void foo() {
            // 지역변수처럼 선언해서 사용할 수 있다.
            class LocalClass {
                private String name;

                public LocalClass(String name) {
                    this.name = name;
                }

                public void print() {
                    // 비정적 문맥에선 바깥 인스턴스를 참조 할 수 있다.
                    System.out.println(number + name);
                }

            }

            LocalClass localClass = new LocalClass("local");

            localClass.print();
        }
    }
    ~~~
    (출처: https://javabom.tistory.com/46)

    ### **정적 멤버 클래스를 사용해야 하는 이유**
    1. 바깥 인스턴스 없이 생성할 수 있다.
    1. 비정적 멤버 클래스를 사용할 경우 메모리 누수가 발생할 수 있다.


