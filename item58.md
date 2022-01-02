#Effective Java

##전통적인 for 문 보다는 for-each 문을 사용하라(58)

> 가능한 모든 곳에서 for 문이 아닌 for-each 문을 사용하자(p.350).

<br>

forEach 문을 써야 하는 이유는
* 전통적인 for 문에 비해 추상화 수준이 높다.
	```java
	for(int i = elements.iterator(); i.hasNext();) {
		Element e = i.next();
	}

	for(Element e: elements) {
		// Do something
	}
	```
* 컬렉션과 배열을 모두 처리할 수 있다.
	```java
	// 전통적인 for 문의 경우 타입에 따라 인터페이스가 달라진다.
	Element[] elements;
	for(int i = 0; i < elements.length; i++) {
		// Do something
	}
	
	List<Element> elements;
	for(int i = 0; i < elements.size(); i++) {
		// Do something
	}
	```


<br>

forEach 문을 쓸 수 없는 경우는
* 파괴적인 필터링
	```java
	List<Elements> elements;
	elements.remove(getTargetIndex(elements));
	
	public int getTargetIndex(List<Elements> elements) {
		for(int i = 0; i < elements.size(); i++) {
			if(...) {
				return i;
			}
		}
	}
	
	// 위의 방식이 싫다면 removeIf() 를 사용하자
	elements.removeIf(this::somePredicate)
	
	public boolean somePrecdicate(element e) {
		// Return boolean value
	}
	```
* 변형
	```java
	List<Elements> elements;
	for(int i = 0; i < elements.size(); i++) {
		if(...) {
			Element e = element.get(i)
			// Set value of e
		}
	}
	```
	*컬렉션을 변형할 경우에는 새로운 컬렉션을 생성하자*
	```java
	List<Element> newElements 
		= elements.stream()
			.map(...)
			.collect(Collectors.toList())
	```
* 병렬 반복(중첩된 for 문?)

<br>

**Iterable 인터페이스를 구현하는 습관을 기르자!**
```java
public interface Iterable<E> {
	Iterator<E> iterator();
}
```

