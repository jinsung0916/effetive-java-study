# Effective Java 
## 명명 패턴보다 애너테이션을 사용하라(39)

### **마커 애너테이션**
~~~java
/**
 * 테스트 메서드임을 선언하는 애너테이션
 * 매개변수 없는 정적 메서드 전용이다.
 */
@Retention(RetentionPolicy.RUNTIME) 
@Target(ElementType.METHOD)
public @interface Test {}
~~~

~~~java
for (Method m : testClass.getDeclaredMethods()) {
    if (m.isAnnotationPresent(Test.class)) {
        m.invoke(null);
    }
}
~~~

<br>

### **메타 애너테이션**
* **@Documented:** 문서에도 애너테이션 정보가 표현되게 한다.
* **@Inherited**: 자식클래스가 애너테이션을 상속받을 수 있게 한다.
* **@Repeatable**: 애너테이션을 반복적으로 사용할 수 있게 한다.
* **@Retention(RetentionPolicy**): 애너테이션의 범위를 지정한다.
    * RUNTIME
    * CLASS
    * SOURCE
*  **@Target(ElementType[])**: 애너테이션이 적용될 위치를 선언한다.
    * PACKAGE
    * TYPE
    * CONSTRUCTOR
    * FIELD
    * METHOD
    * ANNOTATION_TYPE
    * LOCAL_VARIABLE
    * PARAMETER
    * TYPE_PARAMETER
    * TYPE_USE

<br>

### **매개변수가 있는 애너테이션**
~~~java
@ExceptionTest(ArithmeticException.class)
public static void m1() {
    int i = 0;
    i = i / i;
}
~~~
~~~java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
~~~
~~~java
for (Method m : testClass.getDeclaredMethods()) {
    if ( m.isAnnotationPresent(Test.class)) {
        try {
            m.invoke(null);
        } catch (InvocationTargetException wrappedEx) {
            Throwable exc = wrappedEx.getCause();
            Class<? extends Throwable> excType =
                m.getAnnotation(ExceptionTest.class).value();
            if (excType.isInstance(exc)) {
                ...
            }
        }
    }
}
~~~

<br>


### **배열 매개변수를 가지는 받는 애너테이션**
~~~java
@ExceptionTest({ IndexOutOfBoundsException.class, NullPointerException.class })
public static void doublyBad() {
   List<String> list = new ArrayList<>();
   
   // 자바 API 명세에 따르면 다음 메서드는 위의 2개를 던질 수 있다.
   list.addAll(5, null);
}
~~~
~~~java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable>[] value();
}
~~~
~~~java
for (Method m : testClass.getDeclaredMethods()) {
    if ( m.isAnnotationPresent(Test.class)) {
        try {
            m.invoke(null);
        } catch (InvocationTargetException wrappedEx) {
            Throwable exc = wrappedEx.getCause();
            Class<? extends Throwable>[] excTypes =
                m.getAnnotation(ExceptionTest.class).value();
            for(Class<? extends Throwable> excType : excTypes) {
                if (excType.isInstance(exc)) {
                    ...
                }            
            }
        }
    }
}
~~~

<br>

### **반복 가능 애너테이션**
~~~java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() { }
~~~
~~~java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
   Class<? extends Throwable> value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}
~~~
~~~java
for (Method m : testClass.getDeclaredMethods()) {
    // isAnnotationPresent 는 컨테이너와 반복 가능 애너테이션을 구분한다.
    if (m.isAnnotationPresent(ExceptionTest.class)
            || m.isAnnotationPresent(ExceptionTestContainer.class)) {
        tests++;
        try {
            m.invoke(null);
        } catch (Throwable wrappedExc) {
            Throwable exc = wrappedExc.getCause();

            // getAnnotationsByType 는 컨테이너와 반복 가능 애너테이션을 구분하지 않는다.
            ExceptionTest[] excTests =
                    m.getAnnotationsByType(ExceptionTest.class);
            for (ExceptionTest excTest : excTests) {
                if (excTest.value().isInstance(exc)) {
                    ...
                }
            }
        }
    }
}
~~~