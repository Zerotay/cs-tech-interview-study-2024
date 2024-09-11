# Adapter Pattern

> 클래스를 바로 사용할 수 없는 경우가 있음. (다른 곳에서 개발했다거나, 수정할 수 없을 때)
> 중간에서 변환 역할을 해주는 클래스가 필요 > 어댑터 패턴

- **사용 방법** : 상속
- 호환되지 않은 인터페이스를 사용하는 클라이언트 그대로 활용 가능.
- 향후 인터페이스가 바뀌더라도, 변경 내역은 어댑터에 캡슐화 되므로 클라이언트가 바뀔 필요가 없음. 

> 예시 )
> 아이폰의 이어폰을 생각해보자.
> 가장 흔한 이어폰 잭을 아이폰에 사용하려면, 잭 자체가 맞지 않는다.
> 따라서 우리는 어댑터를 따로 구매해서 연결해야 이런 이어폰들을 사용할 수 있다. 

### 💡 주요 구성 요소
- 타겟 (Target) : 클라이언트가 직접적으로 호출하는 인터페이스
- 어댑티 (Adaptee) : 아직 호환되지 않은 기존 클래스 (또는 인터페이스)
- 어댑터 (Adapter) : 타겟 인터페이스를 구현하여 클라이언트 요청을 어댑티로 전달하는 클래스 
- 클라이언트 (Client) : 특정 작업을 요청하는 클래스

### 💡 종류
#### 1. Object Adapter (객체 어댑터)
![img](https://velog.velcdn.com/images/octo__/post/83e9108b-5b07-46d5-a620-7479ae44e70e/image.png)
- **이미 존재하는 객체의 인터페이스를 변경**
- 클라이언트의 요청을 어댑터를 통해 어댑티에 요청을 전달한다. 
#### 2. Class Adapter (클래스 어댑터)
![img](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F252CFB4F575EB62D0A)
- **클래스 자체의 인터페이스 변경**
- 어댑터를 만들 때 타겟과 어댑티 모두의 서브 클래스로 만든다. 
- 클래스 어댑터를 사용하기 위해서는 **다중 상속**이 필요한데, 자바에서는 다중 상속이 불가능하다.

### 💡 실제 개발에서의 어댑터 패턴 구현 방법
#### 1) 자바 라이브러리 또는 프레임워크에서의 활용
##### 📌 I/O 라이브러리 - InputStreamReader 
- 바이트 스트림을 문자 스트림으로 변환하는 어댑터 역할 
- InputStream 을 받아서 Reader 를 제공함으로써 바이트를 문자로 변환하는 과정을 캡슐화 함. 
- **사용자 입력은 결국 바이트 스트림으로 입력된다**
##### 📌 Spring Framwork - MVC - HandlerAdapter
- HandlerAdapter 인터페이스는 다양한 종류의 핸들러를 동일한 방식으로 처리하는데 활용됨. 
- 클라이언트의 요청이 들어오면 `DispathcerServlet` 은 적절한 컨트롤러를 찾고, 그 컨트롤러를 처리할 수 있는 `HandlerAdapter` 를 선택. 
- 선택된 `HandlerAdpater`가 실제 컨트롤러 메서드를 호출하여 요청을 처리하고, 그 결과를 `ModelAndView` 로 반환. 
##### 📌 업체에서 제공한 클래스가 기존 시스템에 맞지 않으면 ? 
- 기존 시스템을 수정할 것이 아니라, 어댑터를 활용해 유연하게 해결하자. 

### 💡 어댑터 패턴 구현 방법
1. 패턴을 적용하고자 하는 인터페이스 식별
2. 어댑터 클래스 작성
3. 클라이언트 코드에서 호출
```java
// 타겟 인터페이스
public interface Target {
	void doSomething();
}

// 실제 사용하려는 클래스
public class Adaptee {
	public void performAction() {
		System.out.println("Action performed!");
	}
}

public class Adapter implements Target {
	private final Adaptee adaptee;
	
	public Adapter(Adaptee adpatee) {
		this.adpatee = adaptee;
	}
	
	@Override
	public void doSomething() {
		adaptee.performAction();
	}
}
```

```java
public class Client {
	public static void main(String[] args) {
		Adaptee adaptee = new Adaptee();
		Target target = new Adapter(adaptee);
		target.doSomething(); // Output : "Action performed!"
	}
}
```

#### ✔ Duck, Turkey 예제로 이해하기
> Duck, Turkey 인터페이스 생성
> 만약 Duck 객체가 부족해서 Turkey 객체를 대신 사용해야 한다면 ?
> 두 객체는 인터페이스가 다르므로, 바로 Turkey 객체를 사용하는 것은 불가능.
> 따라서 Turkey 어댑터를 생성해서 활용해야 한다. 

- `Duck.java`
```java
public interface Duck {
	public void quack();
	public void fly();
}
```

- `MallardDuck.java`
```java
public class MallardDuck implements Duck {  

    @Override  
    public void quack() {  
        System.out.println("Duck duck!");  
    }  
  
    @Override  
    public void fly() {  
        System.out.println("I'm flying!");  
    }  
}
```

- `Turkey.java`
```java
public interface Turkey {
	public void gobble();
	public void fly();
}
```

- `WildTurkey.java`
```java
public class WildTurkey implements Turkey {

	@Override
	public void gobble() {
		System.out.println("Gobble gobble");
	}
	
	@Override
	public void fly() {
		System.out.println("I'm flying a short distance");
		}
}
```

- `TurkeyAdapter.java`
```java
public class TurkeyAdapter implements Duck {
	
	Turkey turkey;
	
	public TurkeyAdapter(Turkey turkey) {
		this.turkey = turkey;
	}
	
	@Override
	public void quack() {
		turkey.gobble();
	}
	
	@Override
	public void fly() {
		turkey.fly();
	}
}
```

- `DuckTest.java`
```java
public class DuckTest {
	public static void main(String[] args) {
		
		MallardDuck duck = new MallardDuck();
		WildTurkey turkey = new WildTurkey();
		Duck turkeyAdapter = new TurkeyAdapter(turkey);
		
		System.out.println("The turkey says ...");
		turkey.gobble();
		turkey.fly();
		
		System.out.println("The Duck says ...");
		testDuck(duck);
		
		System.out.println("The TurkeyAdapter says...");
		testDuck(turkeyAdapter);
	}
	
	public static void testDuck(Duck duck) {
		
		duck.quack();
		duck.fly();
	}
}
```

### 💡 장점
- 기존 클래스를 수정하지 않고 클라이언트에서 새로운 인터페이스를 사용할 수 있음. 
- 기존의 코드를 재사용하고 코드 중복을 줄여주는 데 도움이 됨.
- 클래스 간의 결합도를 줄여주어, 소스 코드 변경이 필요할 때 쉽게 수정할 수 있음. 

### 💡 단점
- 어댑터 클래스를 추가로 작성해야 하기 때문에 소스 코드가 늘어나게 됨. 
- 이는 코드의 복잡성을 증가시키고, 유지 보수를 어렵게 만들 수도 있음. 
- 어댑터가 중간에 데이터를 변환하는 과정에서 추가적인 처리 시간과 오버 헤드가 발생할 수 있음. 

### 💡 어댑터 패턴이 필요한 경우
- 어댑터 패턴은 추가 코드를 작성해야 하고, 오버헤드가 발생할 수도 있기 때문에 무분별한 사용은 권장하지 않음. 
- 따라서 호환되지 않은 인터페이스를 가진 클래스들이 함께 작동해야 하는 경우 
- 이미 존재하는 클래스의 인터페이스가 요구 사항과 맞지 않는 경우
- 기존 클래스에 원하는 인터페이스가 없는 경우 

> 예를 들어, 서드 파티 라이브러리나 API를 사용하는데 그 인터페이스가 어플리케이션 코드와 잘 맞지 않는 경우, 어댑터 패턴을 사용해 서드파티 라이브러리 및 API 내부 구현에 영향을 받지 않으면서, 프로젝트에 필요한 인터페이스를 생성할 수 있음. 

---
**Q. 그니까 어댑터 패턴을 쓰는 건 이해가 가는데, 그 가정한 상황이 언제 발생하는지 이해가 안갑니다. Duck 이랑 Turkey 는 엄연히 다른 객체인데, 왜 Duck 객체가 모자르다고 Turkey 객체를 사용하는 일이 생기나요 ?**

A. 좋은 질문입니다. Duck과 Turkey가 서로 다른 객체인데, 왜 Turkey를 Duck의 대체로 사용하는 상황이 발생할 수 있는지에 대한 이해를 돕기 위해, 몇 가지 구체적인 상황을 상상해 보겠습니다.

### 1. 외부 시스템과의 통합

**상황**: 당신의 애플리케이션은 여러 종류의 객체를 처리하는 시스템을 개발하고 있습니다. 이 시스템에서는 `Duck` 객체를 필요로 하지만, 외부 시스템에서 제공하는 객체가 `Turkey`밖에 없을 때가 있습니다.

- **예시**: 당신의 애플리케이션이 다양한 동물 소리와 행동을 시뮬레이션하는 게임을 만들고 있다고 가정해 봅시다. 게임의 코드에서 `Duck` 객체를 사용하여 오리의 소리와 비행을 시뮬레이션하고 싶습니다. 그러나 외부에서 제공하는 API나 라이브러리에서는 `Turkey` 객체만 제공하고 있습니다. 이 경우, `Turkey`를 `Duck`처럼 사용할 수 있도록 어댑터를 만들어야 합니다.

### 2. 레거시 코드와의 호환성

**상황**: 기존의 레거시 시스템에서는 `Duck` 객체를 사용하는 코드가 많이 작성되어 있습니다. 새롭게 도입된 시스템에서는 `Turkey` 객체를 사용하는데, 이 두 시스템이 서로 상호작용해야 할 때가 있습니다.

- **예시**: 기존의 사내 애플리케이션에서 `Duck` 인터페이스를 구현한 객체들이 많이 사용되고 있습니다. 새로 도입된 시스템에서 `Turkey` 객체를 사용하게 되었는데, 기존 시스템과 호환성을 유지하기 위해 `Turkey`를 `Duck`으로 변환할 필요가 있습니다. 이때 어댑터 패턴을 사용하여 `Turkey`를 `Duck`처럼 사용할 수 있습니다.

### 3. 새로운 기능 추가

**상황**: 기존 시스템에서 `Duck` 인터페이스를 사용하는 기능이 잘 작동하고 있지만, 새로운 기능으로 `Turkey` 객체를 통합해야 하는 상황입니다.

- **예시**: 애플리케이션이 오리와 관련된 다양한 기능을 처리하고 있는데, 새로운 요구 사항으로 인해 칠면조의 소리와 행동도 시뮬레이션해야 합니다. 기존 코드에서 `Duck` 인터페이스를 사용하고 있지만, `Turkey` 객체를 추가적으로 사용해야 하므로 어댑터 패턴을 통해 기존 시스템과 호환되도록 해야 합니다.

### 4. 코드 재사용과 유연성

**상황**: 코드의 유연성과 재사용성을 높이기 위해, 다양한 동물 객체를 처리할 수 있는 공통 인터페이스를 정의하고 싶습니다. 이때 기존 코드에서 `Duck` 인터페이스를 사용하고 있으며, 새로운 동물인 `Turkey`도 동일한 방식으로 처리하고 싶습니다.

- **예시**: 동물 시뮬레이션 소프트웨어에서 다양한 동물 객체를 다루는 공통 인터페이스를 사용하고 있습니다. 새로운 동물인 `Turkey`가 추가되었는데, 기존의 `Duck` 인터페이스와 동일한 방식으로 처리하기 위해 어댑터를 사용하여 `Turkey`를 `Duck`처럼 사용할 수 있습니다.

### 요약

- **Duck과 Turkey의 차이**: Duck과 Turkey는 서로 다른 인터페이스를 가지지만, 특정 상황에서는 두 객체 간의 호환성을 필요로 할 수 있습니다. 
- **어댑터 패턴의 필요성**: Duck 인터페이스를 사용하는 기존 시스템에서 Turkey 객체를 통합해야 하는 경우, 어댑터 패턴을 사용하여 Turkey 객체를 Duck처럼 사용할 수 있게 합니다.
- **상황의 예시**: 외부 시스템과의 통합, 레거시 코드와의 호환성, 새로운 기능 추가, 코드의 유연성 및 재사용성 확보 등이 이러한 상황을 설명할 수 있는 예입니다.

어댑터 패턴은 서로 다른 인터페이스를 가진 객체들 간의 호환성 문제를 해결하기 위해 사용됩니다. 이를 통해 다양한 객체들을 일관된 방식으로 처리할 수 있게 됩니다.

---
https://yozm.wishket.com/magazine/detail/2077/
https://gyoogle.dev/blog/design-pattern/Adapter%20Pattern.html