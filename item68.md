# Effective Java

## 일반적으로 통용되는 명명 규칙을 따르라(68)
1. 철자 규칙
    * 패키지와 모듈 이름은 각 요소를 점(.)으로 구분하여 게층적으로 짓는다.
        > edu.cmu, com.google, org.eff
    * 패키지 이름 다음에는 패키지를 설명하는 요소로 이뤄진다. 
        * 이름은 8자 이하의 짧은 단어로 짓고, 여러 단어로 이루어져 있다면 약어를 활용한다.
            > util, awt
        * 많은 기능을 제공하는 경우, 요소 계층을 분할해도 무방하다.
            > java.util.concurrent.atomic 
    * 클래스와 인터페이스의 이름은 하나 이상의 단어로 이루어지며, 대문자로 시작한다.
        > Stream, FutureTask, LinkedHashMap, HttpClient
    * 매서드와 필드 이름은 첫 글자를 소문자로 쓴다는 점을 제외하면 클래스 명명 규칙과 같다.
        > remove, groupingBy, getCrc 
    * 상수는 모두 대문자로 쓰고, 단어 사이는 밑줄로 구분한다.
        > MIN_VALUE, NEGATIVE_INFINITY
    * 지역변수에는 약어를 사용해도 무방하다.
        > i, denom, hostNum
    * 타입 매개변수는 보통 한 문자로 표현한다.
        > T(임의의 타입), E(컬렉션 원소의 타입), K, V(맵의 키와 값), R(매서드의 반환 타입)

2. 문법 규칙
   * 클래스 
       * 객체를 생성할 수 있는 클래스
           > Thread, PriorityQueue, ChessPiece (단수 명사나 명사구) 
       * 객체를 생성할 수 없는 클래스
           > Collectors, Collections (복수형 명사)  
       * 인터페이스
           > Collection, Comparator (클래스 이름과 동일하게), Runnable, Iterable, Accessible (-able 또는 -ible)  
       * 애너테이션 
           > BindingAnnotation, Inject, ImplementedBy, Singleton     
   * 매서드
       * 동사, 동사구
            > append, drawImage 
       * is_, has_ - boolean 을 반환하는 경우
            > isDigit, isProbablePrime, isEmpty, isEnabled, hasSibilings 
       * get, set - non-boolean 을 반환하는 경우
            > getAttribute, setAttribute 
       * toType - 객체의 타입을 변환하는 경우
            > toString, toArray  
       * asType - 객체의 내용을 다른 뷰로 보여주는 경우
            > asList
       * typeValue - 객체의 값을 기본 타입으로 반환하는 경우
            > intValue
       * 정적 팩터리 매서드
            > from, of, instance, getInstance, newInstance, getType, newType  
   * 필드
       * boolean 
           > initialized, composite  
       * 명사나 명사구
           > height, digit, bodyStyle 
  