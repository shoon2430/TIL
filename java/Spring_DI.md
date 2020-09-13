# DI(dependency injection)

## DI 란 무엇인가?

**의존성 주입**이란 **외부에서 의존 객체**를 생성하여 넘겨주는 것

> DI(의존성 주입)은 프로그래밍에서 구성요소간의 의존 관계가 소스코드 내부가 아닌 외부의 설정파일 등을 통해 정의되게 하는 디자인 패턴 중의 하나이다. - 위키백과

예를들어 그림과 같이 A Class가 B Class를 의존할 때 B Object를 A가 직접 생성하지 않고 외부에서 생성하여 넘겨주면 의존성을 주입했다고 한다.

![DI](/images/java/java_DI.png)

## DI가 왜 필요할까?

1. 의존성 파라미터를 생성자에 작성하지않아도 되기 때문에 코드를 줄일 수 있다. 또한 Interface에 구현체를 쉽게 교체할 수 있다. 상황이 변할 때 마다 유용하게 적용시킬 수 있게 된다.

2. Interface에 구현체를 쉽게 교체하면서 상황에 따라 적절한 행동을 정의할 수 있습니다. 따라서 유닛 테스트 시 간결한 구현을 가능하게 도와줍니다.

## 간단한 예제로 알아보는 DI

Coffee를 만드는데 필요한 CoffeeMachine을 인터페이스로 구현합니다. makeCoffee()는 커피를 만들어내는 함수입니다.

```java
public interface CoffeeMachine {
	void makeCoffee();
}
```

그리고 CoffeeMachine를 사용해 커피를 만드는 CoffeeMaker 클래스를 구현해 봅시다.

```java
public class CoffeeMaker {
	private final CoffeeMachine coffeeMachine;

    public CoffeeMaker(CoffeeMachine coffeeMachine){
        this.coffeeMachine = coffeeMachine;
    }

	public void brew(){
        System.out.println("=== 커피 제작 ===");
		coffeeMachine.makeCoffee();
    }
}
```

그리고 CoffeeMachine의 구현체인 2종류의 커피머신 클래스를 만들면 다음과 같습니다.

```java
public class StarbucksCoffeeMachine implements CoffeeMachine {

	@Override
	public void makeCoffee() {
		System.out.println("스타벅스 커피 제작");
	}
}
```

```java
public class NespressoCoffeeMachine implements CoffeeMachine {

	@Override
	public void makeCoffee() {
		System.out.println("네스프레소 커피 제작");
	}
}
```

### DI를 사용하지 않는 상태

DI를 사용하지 않은 상태에서 메인함수에서 커피를 만들어 보겠습니다.

```java
public static void main(String[] args) {
    CoffeeMachine coffeeMachine = new StarbucksCoffeeMachine();
    CoffeeMaker coffeeMaker = new CoffeeMaker(coffeeMachine);
    coffeeMaker.brew();
}
```

> 커피를 내리기 위해서는 CoffeeMaker 객체 사용자가 StarbucksCoffeeMachine라는 인터페이스 구현체를 알아야 커피를 내릴수 있습니다. 커피를 마시는 사람은 어떤 CoffeeMachine를 사용하는지 알고 싶어 하지 않습니다.

### DI를 활용한 상태

DI는 CoffeeMaker 사용자가 의존성을 모르는 상태(어떤 CoffeeMachine 를 쓰는지 모르는 상태)에서도 커피를 내릴수 있도록 해줍니다. DI로 CoffeeMaker 사용자 대신에 의존성을 주입해주는 Injection 이라는 클래스를 생성합니다.

```java
public class Injection {
    // 지금은 스타벅스 커피머신을 사용한다고 가정한다.
    public static CoffeeMachine provideCoffeeMachine(){
        return new StarbucksCoffeeMachine();
    }

    public static CoffeeMaker provideCoffeeMaker(){
        return new CoffeeMaker(provideCoffeeMachine());
    }
}
```

> Injection 클래스는 DI **의존성 주입**을 해주는 설정 파일로 가정하고, 현재는 이 설정 파일인 Injection에서 **인터페이스 구현체**를 **리턴**해주고 있다.

DI 를 구현한 상태에서 CoffeeMaker 사용자는 어떻게 커피를 내리는지 확인해보자.

```java
public static void main(String[] args) {
	CoffeeMaker coffeeMaker = new CoffeeMaker(Injection.provideCoffeeMachine());
	coffeeMaker.brew();
}
```

> Injection 클래스에서 provideCoffeeMachine을 제공받도록 하여, 사용자는 StarbucksCoffeeMachine를 사용해야하는지 NespressoCoffeeMachine를 사용해야하는지 알 필요가 없다. 단지 provideCoffeeMachine 만 알고 쓰게 된다.

### 정리

갑자기 `StarbucksCoffeeMachine` 말고 `NespressoCoffeeMachine` 를 사용해야하는 상황이 왔을 때 DI 가 구현된 코드에서 CoffeeMaker관리자는 사용자에게 아무 얘기 없이 Injection 설정만 `NespressoCoffeeMachine `로 바꿔주면 됩다.

그러나 DI 가 구현되어 있지 않다면 CoffeeMaker 관리자는 사용자에게 커피 내릴때 `StarbucksCoffeeMachine`객체 말고 `NespressoCoffeeMachine`객체를 사용해야 한다고 요청해야하는 번거로운 일이 생기게됩니다.

이처럼 DI 를 이용하면 의존관계가 외부에 있기에 사용하는데 있어서 유연하게 사용할 수 있게 됩니다.
