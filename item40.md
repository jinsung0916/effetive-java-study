# Effective Java 
## Override 에너테이션을 일관성 있게 사용하라(40)

~~~java
// 잘못된 재정의! - 오버로딩
public class Bigram {
    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }
}
~~~

~~~java
@Override
public boolean equals(Bigram b) {
    return b.first == first && b.second == second;
}

Error:(16, 5) java: method does not override or implement a method from a supertype

@Override
public boolean equals(Object o) {
    if (!(o instanceof Bigram2))
        return false;
    Bigram2 b = (Bigram2) o;
    return b.first == first && b.second == second;
}
~~~