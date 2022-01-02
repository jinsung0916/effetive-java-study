#Effective Java

##지역변수의 범위를 최소화하라(57)

지역변수의 범위를 최소화 해야 하는 이유는
* 소스코드 가독성, 유지보수성은 높아지고 
* 오류 가능성은 낮아진다.

~~또한 소스코드의 추상화 수준이 너무 낮다는 위험 신호이기도 하다.~~
```java
// before - 낮은 추상화 수준
public void temp() {
	int num = 0
	while(...) {
		// Do something
	}
	
	if(...) {
		// Do something
	}
	
	return num;
}

// after - 높은 추상화 수준
public void temp() {
	int num = 0
	int whileNum = handleWhile(num);
	int ifNum = handleIf(whileNum);
	return ifNum;
}
```

<br>

지역변수의 범위를 최소화 하기 위해서는
1. 가장 처음 쓰일 때 선언한다.
2. 거의 모든 지역변수는 선언과 동시에 초기화한다.
3. 매서드를 작게 유지하고 한 가지 기능에 집중한다.

<br>

######가장 처음 쓰일 때 선언한다.
소스코드 가독성을 높이는 기본적인 접근이다.이하 생략!

<br>

######거의 모든 지역변수는 선언과 동시에 초기화한다.
1. 가장 좋은 방법 - forEach 문
	```java
	// Stream 을 제외하면 추상화 수준이 가장 높은 방법
	for(Element e : c) {
		// Do something
	}
	```
2. 차선책 - 전통적인 for 문
	```java
	// forEach 문 보다 추상화 수준이 떨어진다.
	for(int i = 0, n = expensiveComputation(); i < n; i++) {
		// Do something
	}

	// 주의! - 아래와 같이 코딩하면 안된다.
	for(int i = 0; i < expensiveComputation(); i++) {
		// Do something
	}
	```
3. while 문
	```java
	Iterator<Element> i;
	// 반드시 블록 외부 변수를 참조할 수 밖에 없다.
	while(i.hasNext()) {
		// Do something
	}

	Iterator<Element> i2;
	while(i.next()) { // 버그!	
	} 
	```

~~덧붙여서, 불변 변수를 선언하여 지역변수의 범위를 강제할 수 있다.~~
```java
public void myFunc() {
	final int constVal = 1;
	
	constValue = 2; // 에러!
	
	// 새로운 변수를 선언해야 한다.  
}
```

<br>

######매서드를 작게 유지하고 한 가지 기능에 집중한다.
~~클래스 혹은 매서드의 결합도를 측정하는 방법 = 지역변수가 파편화 되어 있는가?~~
```java
// before - 각각의 변수를 사용하는 로직이 하나의 메소드에 섞여있다.
public void myFunc() {
	int a;
	int b;
	int c;
	
	// Do something with A
	// Do something with B
	// Do something with C
}

// after
public void funcA() {int a;}
public void funcB() {int b;}
public void funcC() {int c;}
```

```java
// before- 각각의 인스턴스 변수를 사용하는 매서드들이 하나의 클래스에 섞여있다.
class MyClass {
	int a;
	int b;
	int c;
	
	public void funcA() {
		// Do something with a
	}
	
	public void funcB() {
		// Do something with b
	}
	
	public void funcC() {
		// Do something with c
	}
}

// after
class MyClassA() {
	int a;
}

class MyClassB() {
	int b;
}

class MyClassC() {
	int c;
}
```
