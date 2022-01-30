# Effective Java

## 옵셔널 반환은 신중히 하라(55)

1.  옵셔널을 사용하는 이유 - 옵셔널은 검사 예외와 취지가 비슷하다.
  
    * 검사 예외를 사용하는 예
        ~~~java
        package com.java.effective.item55;

        import java.util.Collection;
        import java.util.Objects;
        import java.util.Optional;

        public class Main {
            public static <E extends Comparable<E>> E max(Collection<E> c) {
                if (c.isEmpty()) {
                    throw new IllegalArgumentException();
                }

                E result = null;

                for (E e : c) {
                    if (result == null || e.compareTo(result) > 0) {
                        result = Objects.requireNonNull(e);
                    }
                }
                return result;
            }
        }
        ~~~
    * 옵셔널을 사용하는 예
        ~~~java
        package com.java.effective.item55;

        import java.util.Collection;
        import java.util.Objects;
        import java.util.Optional;

        public class Main {
            public static <E extends Comparable<E>> Optional<E> maxOptional(Collection<E> c) {
                if (c.isEmpty()) {
                    return Optional.empty();
                }

                E result = null;

                for (E e : c) {
                    if (result == null || e.compareTo(result) > 0) {
                        result = Objects.requireNonNull(e);
                    }
                }
                return Optional.of(result);
                // return Optional.ofNullable(result);
            }

        }
        ~~~

2. 옵셔널을 사용하는 방법
    * 기본값 설정
        ~~~java
        String lastWordInLexicon = max(words).orElse("단어 없음...");
        ~~~
    * 원하는 예외 던지기
        ~~~java
        Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
        ~~~
    * 값이 있다고 가정하고 꺼내기 
        ~~~java
        Element lastNobleGas = max(Elements.NOBLE_GASES).get();
        ~~~
    * Supplier<T>를 인수로 받는 orElseGet을 사용하여 값 초기 설정 비용을 낮출 수 있다.
        ~~~java
        String lastWordInLexicon = max(words).orElseGet(() -> "");
        ~~~ 
    * isPresent는 다른 메소드로 대체될 수 있으니 신중히 사용하라.
        * isPresent
            ~~~java
            if(optional.isPresent()) {
                // do something
            } 
            ~~~  
        * filter, map, flatmap, ifPresent
            ~~~java
            optional.ifPresent(it -> {
                // do something
            });
            ~~~ 
            ~~~java
            streamOfOptionals
                .filter(Optional::ispresent)
                .map(Optional::get)
            
            streamOfOptional
                .flatMap(optional::stream)
            ~~~

3. 옵셔널 반환 시 주의 사항
    * 옵셔널을 반환하는 메소드에서는 절대 null을 반환하지 말자.
    * 컬렉션, 스트릠, 배열은 옵셔널로 감싸면 안된다.
    * optional 은 항상 추가 비용이 발생함을 기억하자(박싱된 기본 타입 사용 금지)

4. 옵셔널 사용 시 주의 사항
    * 옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는 게 적절한 상황은 거의 없으니 절대 사용해서는 안된다.  