# Effective Java 
## Comparable을 구현할지 고려하라(14)

```java
public interface Comparable<T> {

    public int compareTo(T o);

}
```
```java
package java.util;

/**
 * Package private supporting class for {@link Comparator}.
 */
class Comparators {
    private Comparators() {
        throw new AssertionError("no instances");
    }

    /**
     * Compares {@link Comparable} objects in natural order.
     *
     * @see Comparable
     */
    enum NaturalOrderComparator implements Comparator<Comparable<Object>> {
        INSTANCE;

        @Override
        public int compare(Comparable<Object> c1, Comparable<Object> c2) {
            return c1.compareTo(c2);
        }

    }
}
```
<br>

### `compareTo` 매서드의 일반 규약
1. 대칭성
    * `x.compareTo(y) < 0` 일 때 `y.compareTo(x) > 0` 이다.
2. 추이성
    * `x.compareTo(y) < 0` 이고 `y.compareTo(z) < 0` 이라면 `x.compareTo(z) < 0` 이다.
3. 반사성
    * `(x.compareTo(y) == 0) == (x.compareTo(y))`
4. equals 메서드와 일관되게 작성하라
    ```java
    // 정렬된 컬렉션의 경우 동치성 비교를 compareTo로 한다.
    class TreeMap {
            final Entry<K,V> getEntry(Object key) {
                // Offload comparator-based version for sake of performance
                if (comparator != null)
                    return getEntryUsingComparator(key);
                if (key == null)
                    throw new NullPointerException();
                @SuppressWarnings("unchecked")
                    Comparable<? super K> k = (Comparable<? super K>) key;
                Entry<K,V> p = root;
                while (p != null) {
                    int cmp = k.compareTo(p.key);
                    if (cmp < 0)
                        p = p.left;
                    else if (cmp > 0)
                        p = p.right;
                    else
                        return p;
                }
                return null;
        }
    }
    ```

<br>

### `compareTo` 매서드 작성 방법

* `compareTo`의 작성 요령은 `equals`와 비슷하나 몇가지 차이점이 있다.
    1. `Comparable`은 제네릭 인터페이스이기 때문에 `equals`에서 행했던 type 확인 작업과 casting이 필요없다.
    2. null을 인자로 받는다면 단순하게 `NullPointerException`을 발생시키면 된다.
    3. 객체 참조 필드를 비교한다면 `compareTo` 매서드를 재귀적으로 호출한다.

<br>

### `compareTo` 매서드 작성 시 주의할 점
1. primitive type 을 비교할 때 관계 연산자를 사용하지 말고 정적 매서드인 `compare()` 을 사용하자 
    ```java
    // before
    static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
        @Override
        public int compare(Object o1, Object o2) {
            return o1.hashCode() - o2.hashCode();
        }
    }

    // after
    static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
        @Override
        public int compare(Object o1, Object o2) {
            return Integer.compare(o1.hashCode(), o2.hashCode());
        }
    }
    ```
2. 기존 클래스를 확장한 구체 클래스에 새로운 컴포넌트를 추가했다면 compareTo 규약을 지킬 방법이 없다
    (출처: https://javabom.tistory.com/10).
    ```java
    public class Point implements Comparable<Point> {
        protected Integer x;
        protected Integer y;

        public Point(Integer x, Integer y) {
            this.x = x;
            this.y = y;
        }

        @Override
        public int compareTo(Point point) {
            int result = Integer.compare(x, point.x);
            if (result == 0) {
                return Integer.compare(y, point.y);
            }
            return result;
        }
    }

    class ColorPoint extends Point implements Comparable<Point> {

        private Integer color;

        public ColorPoint(Integer x, Integer y, Integer color) {
            super(x, y);
            this.color = color;
        }

        @Override
        public int compareTo(Point point) {
            int result = super.compareTo(point);
            if (result == 0) {
                return Integer.compare(color, ((ColorPoint) point).color); // 잘못된 구현
            }
            return result;
        }
    }

    @DisplayName("잘못된 다운 캐스팅으로 ClassCastException 이 발생한다.")
    @Test
    void test() {
        Point point = new Point(1, 2);
        ColorPoint colorPoint = new ColorPoint(1, 2, 3);

        assertThat(point.compareTo(colorPoint)).isEqualTo(0);
        assertThatThrownBy(() -> colorPoint.compareTo(point))
                .isInstanceOf(ClassCastException.class);
    }
    ```

    ```java
    public class Point implements Comparable<Point> {
        private int x;
        private int y;

        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }

        @Override
        public int compareTo(Point point) {
            int result = Integer.compare(x, point.x);
            if (result == 0) {
                return Integer.compare(y, point.y);
            }
            return result;
        }
    }

    class ColorPoint implements Comparable<ColorPoint> {
        private Point point;
        private int color;

        public ColorPoint(Point point, int color) {
            this.point = point;
            this.color = color;
        }

        public Point asPoint() {
            return point;
        }

        @Override
        public int compareTo(ColorPoint colorPoint) {
            int result = point.compareTo(colorPoint.point);
            if (result == 0) {
                return Integer.compare(color, colorPoint.color);
            }
            return result;
        }
    }

    @DisplayName("구체 클래스인 Point 의 일반 규약을 지킬 수 있다.")
    @Test
    void test() {
        Point point = new Point(1, 3);
        ColorPoint colorPoint = new ColorPoint(new Point(1, 2), 1);

        assertThat(point.compareTo(colorPoint.asPoint())).isEqualTo(1);
        assertThat(colorPoint.asPoint().compareTo(point)).isEqualTo(-1);
    }
    ```
<br>

### `Comparator` 생성 메서드
 * Comparator 생성 메서드 패턴을 이용한다면 보다 읽기 쉬운 코드가 될 수 있으나 성능의 저하가 뒤따르니 유의한다.
    ```java
    // before
    public int compareTo(PhoneNumber pn) {
        int result = Short.compare(areaCode, pn.areaCode);
        if (result == 0)  {
            result = Short.compare(prefix, pn.prefix);
            if (result == 0)
                result = Short.compare(lineNum, pn.lineNum);
        }
        return result;
    }
        
    // after
    private static final Comparator<PhoneNumber> COMPARATOR =
            Comparator.comparingInt((PhoneNumber pn) -> pn.areaCode)
                    .thenComparingInt(pn -> pn.prefix)
                    .thenComparingInt(pn -> pn.lineNum);

    public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this, pn);
    }
    ```