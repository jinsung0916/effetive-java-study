# Effective Java
## equals는 일반 규약을 지켜 재정의하라(10)

<br>

### `equals()` 를 재정의 하지 말아야 하는 경우
* `Object.equals()`로 해결되는 경우
    ```java
    class Object {
        public boolean equals(Object obj) {
            return (this == obj);
        }
    }
    ```
    1. 개별 인스턴스가 본질적으로 고유한 경우
    2. 인스턴스의 논리적 동치성(logical equality)을 검사할 일이 없는 경우.
    3. 상위 클래스에서 재정의한 equals()가 하위 클래스에도 딱 들어 맞는 경우.
    4. 클래스가 private이거나 package-private이고 equals() 메서드를 호출할 일이 없는 경우.
        ```java
        @Override
        public boolean equals(Object o) {
            throw new AssertionError();	//호출 금지!
        }
        ```

### `equals()` 를 재정의 해야 하는 경우
* `Object.equals()`로 해결되지 않는 경우, 주로 Value Object 에 해당된다. (String, Integer...)

### `equals()` 재정의 시 지켜야 할 규약
* **반사성(relfexivity)** : null이 아닌 모든 참조 x에 대해, x.equals(x)는 true
* **대칭성(symmetry)** : null이 아닌 모든 참조 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true다
* **추이성(transitivity)** : null이 아닌 모든 참조 x,y,z에 대해 x.equals(y), y.equals(z)가 true라면, x.equals(z)도 true다
* **일관성(consistency)** : null이 아닌 모든 참조 x,y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다
* **null-아님** : null아닌 모든 참조 x에 대해, x.equals(null)은 false다


### `equals()` 매서드를 구현하는 방법
~~~java
class App {
    
    private String var1 = "str";
    private String var2 = "str";

    @Override
    public boolean equals(Object o) {
        // 1. == 연산자를 사용해 입력이 자기 자신의 참조인지 한다.
        if (this == o) return true;
        // 2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
        if (!(o instanccof App)) return false;
        // 3. 입력을 올바른 타입으로 형변환한다.
        App app = (App) o;
        // 4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.
        return Objects.equals(var1, app.var1) && Objects.equals(var2, app.var2);
    }
}
~~~

### `equals()` 주의사항
1. `Float`, `Double` 의 경우 `compare`함수를 통해 비교하자. `Float.equals` 와 `Double.equals`는 오토박싱을 수반하기 때문에 성능상 좋지않다.
    ~~~java
    class Float {
        public boolean equals(Object obj) {
            return (obj instanceof Float)
               && (floatToIntBits(((Float)obj).value) == floatToIntBits(value));
        }

        public int compareTo(Float anotherFloat) {
            return Float.compare(value, anotherFloat.value);
        }
    }
    ~~~
2. `null`을 허용하는 필드는 `Objects.equals()`를 통해 `nullPointException`을 방지하자.
    ~~~java
    class Objects {
        public static boolean equals(Object a, Object b) {
            return (a == b) || (a != null && a.equals(b));
        }
    }
    ~~~
3. 필드들을 비교할 때, 비교하는 비용이 싼 필드들을 먼저 비교하자(성능 고려).
4. `equals`를 재정의할 땐 `hashcode`도 반드시 재정의하자.