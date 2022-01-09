# Effective Java 
## 태그 달린 클래스보다는 클래스 계층구조를 활용하라(23)

<br>

### **태그 달린 클래스 vs 클래스 계층 구조**
1. 태그 달린 클래스
    ~~~java
    class Figure {
        enum Shape { RECTANGLE, CIRCLE }

        private final Shape shape;

        private double length;
        private double width;

        private double radius;

        // constructors...

        public double area() {
            switch(shape) {
                case RECTANGLE:
                    return length * width;
                case CIRCLE:
                    return Math.PI * (radius * radius);
                default:
                    ...
            }
        }

    }
    ~~~
2. 클래스 계층 구조
    ~~~java
    abstract class Figure { public abstract double area(); }

    class Rectangle extends Figure {}

    class Circle extends Figure {}
    ~~~

<br>

### **태그 달린 클래스의 단점**
> 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다.
1. 클래스 응집도가 낮다.
2. 클래스 결합도는 높다.

<br>

### **클래스 계층 구조의 장점**
1. 클래스 응집도가 높다.
2. 클래스 결합도는 낮다.
3. *개방 폐쇄 원칙을 준수한다.*

<br>

#### **(참고) Mybatis 사용 시 enum type 을 통해 다형성을 구현하기도 한다.**
~~~java
enum Shape { 
    RECTANGLE{
        public double area(Figure figure) {
            return figure.getLength() * figure.getWidth();
        } 
    }, 
    CIRCLE{
        public double area(Figure figure) {
            double redius = figure.getRadius();
            return  Math.PI * (radius * radius);
        } 
    };

    public abstract double area(Figure figure);
}

class Figure {

    private final Shape shape;

    ...

    public double area() {
        return shape.area(this);
    }
}
~~~