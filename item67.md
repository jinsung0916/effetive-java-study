# Effective Java

## 최적화는 신중히 하라(67)
1. 빠른 프로그램보다는 좋은 프로그램을 작성하라.
    * 좋은 설계는 변경에 유연한 구조를 의미한다
    * 성능같은 구현 상의 문제는 나중에 최적화해서 해결할 수 있지만, 아키텍처의 결함은 시스템 전체를 다시 작성하지 않고는 해결하기 불가능하다.
   
<br>

2. 성능을 제한하는 설계를 피하라(변경하기 어려운 설계에서는 성능을 생각하라).
   * 컴포넌트끼리, 또는 외부 시스템과의 소통방식은 변경하기 어렵기 때문에 신중하게 설계해야 한다.   
   * API, 네트워크 프로토콜, 영구 저장용 데이터 포맷 등등...

<br>

3. API를 설계할 떄 성능에 주는ㄴ 영향을 고려하라.
    * 객체의 불변성을 보장하라 (아이템 50)
    * 상속보다 컴포지션을 활용하라 (아이템 18)
    * 인터페이스를 활용해 참조하라 (아이템 64)
    * 그러나, 성능을 위해 API를 왜곡하는 건 지양해야 한다.

<br>    

4. 각각의 최적화 시도 전후로 성능을 측정하라.
    * 프로그램에서 시간을 잡아먹는 부분을 추측하기가 어렵다.
    * 프로파일링 도구를 활용하면 최적화 노력을 어디에 집중해야 할지 찾는데 도움을 준다.