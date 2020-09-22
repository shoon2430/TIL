# AOP(Aspect Oriented Programming)

**AOP**는 **Aspect Oriented Programming**의 약자로 **관점 지향 프로그래밍**이라고 불린다. 관점 지향은 쉽게 말해 **어떤 로직을 기준으로 핵심적인 관점, 부가적인 관점으로 나누어서 보고 그 관점을 기준으로 각각 모듈화하겠다는 것이다**. 여기서 모듈화란 어떤 공통된 로직이나 기능을 하나의 단위로 묶는 것을 말한다.

![AOP](/images/java/spring_AOP.png)

예로들어 핵심적인 관점은 결국 우리가 적용하고자 하는 핵심 비즈니스 로직이 된다. 또한 부가적인 관점은 핵심 로직을 실행하기 위해서 행해지는 데이터베이스 연결, 로깅, 파일 입출력 등을 예로 들 수 있다.

### 예시

Event를 발생시키는 createEvent 메서드와 pulbicEvent메서드가 있을 때, 이 두메서드의 실행 시간을 알고싶다고 가정하자.

```java
package hello.hellospring.aop;

@Service
public class SimpleEventService implements EventService {

	@Override
	public void createEvent() {
		try {
			Thread.sleep(1000);
		} catch (Exception e) {
			e.printStackTrace();
		}
		System.out.println("Create Event!!");
	}

	@Override
	public void pulbicEvent() {
		try {
			Thread.sleep(2000);
		} catch (Exception e) {
			e.printStackTrace();
		}
		System.out.println("Pulbic Event!!");
	}
}
```

그리고 실행시간을 구할수 있도록 메서드를 수정한다.

```java
//package hello.hellospring.aop;
public void createEvent() {
    long begin = System.currentTimeMillis();
    try {
        Thread.sleep(1000);
    } catch (Exception e) {
        e.printStackTrace();
    }
    System.out.println("Create Event!!");
    System.out.println(System.currentTimeMillis()-begin);
}
```

> 이렇게 수정을 할 경우에는 시간을 구할 수는 있지만, 기존의 소스코드를 수정해야하는점이 있다. 기존의 코드를 변경하지않고 실행시간을 구하기위해 **프록시 패턴**을이용해보자

![프록시 패턴](/images/java/proxy_pattren.png)

```java
package hello.hellospring.aop;

@Primary //같은타입의 빈이 여러가지 있다면 그중 하나를 선택하는 방법
@Service
public class ProxySimpleEventService implements EventService {

	@Autowired
	EventService simpleEventService;

	@Override
	public void createEvent() {
		long begin = System.currentTimeMillis();
		simpleEventService.createEvent();
		System.out.println(System.currentTimeMillis()-begin);
	}

	@Override
	public void pulbicEvent() {
		long begin = System.currentTimeMillis();
		simpleEventService.pulbicEvent();
		System.out.println(System.currentTimeMillis()-begin);
	}
}
```

> 기존 소스코드는 변경하지 않은 상태에서 simpleEventService를 주입받고, 프록시 객체의 메서드는 실행 시 실제 객체의 메서드에 위임하도록 하고, 그사이에 시간을 측정할 수 있도록 한다.

하지만 우리가 측정해야 하는 메서드들이 100개, 1000개씩 많아지게 된다면 이러한 방식으로 프록시 객체를 만들어 확인하는 데에는 시간이 오래 걸릴 것이다. 그래서 스프링에서 제공하는 AOP를 이용하여 더 많은 메서드들을 확인할 수 있도록 수정해보자

### 스프링 AOP

- **Aspect** : 위에서 설명한 흩어진 관심사를 모듈화 한 것. 주로 부가기능을 모듈화함.

- **Target** : Aspect를 적용하는 곳 (클래스, 메서드 .. )

- **Advice** : 실질적으로 어떤 일을 해야할 지에 대한 것, 실질적인 부가기능을 담은 구현체

- **JointPoint** : Advice가 적용될 위치, 끼어들 수 있는 지점. 메서드 진입 지점, 생성자 호출 시점, 필드에서 값을 꺼내올 때 등 다양한 시점에 적용가능

- **PointCut** : JointPoint의 상세한 스펙을 정의한 것. `TestMethod`란 메서드의 진입 시점에 호출할 것'과 같이 더욱 구체적으로 Advice가 실행될 지점을 정할 수 있음

#### 예시

스프링 **AOP**를 사용하기 위해서 의존성을 등록한다.

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

다음에는 아래와 같이 **@Aspect** 어노테이션을 붙여 이 클래스가 **Aspect**를 나타내는 클래스라는 것을 명시하고 **@Component**를 붙여 스프링 빈으로 등록한다.

```java
package hello.hellospring.aop;

@Component
@Aspect
public class PerfAspect {
    // 해야할일 : 메서드 실행시간 측정
	@Around("execution(* hello.hellospring..*.EventService.*(..))")
	public Object logPerf(ProceedingJoinPoint pip) throws Throwable {
		long begin = System.currentTimeMillis();
		Object retVal = pip.proceed(); // 메서드 호출 자체를 감쌈
		System.out.println(System.currentTimeMillis()-begin);
		return retVal;
	}
}
```

- **Around** 어노테이션은 타겟 메서드를 감싸서 특정 **Advice**를 실행한다는 의미

- **execution(\* hello.hellospring..\*.EventService.\*(..))**는 **hello.hellospring** 아래의 패키지 경로의 **E ventService** 객체의 모든 메서드에 이 **Aspect**를 적용하겠다는 의미

```java
// Create Event!!
// 1016
// Pulbic Event!!
// 2001
// Delete Event!!
// 0
```

이렇게 패키지 경로를 이용해서 AOP를 적용함으로써 이전의 문제를 해결 할 수 있다.

하지만 해당서비스의 모든 메서드에 적용된다는 점이 있기때문에, 몇몇 메서드에는 적용하지 않고싶은 경우에는 **annotation**을 이용하여 AOP를 적용하여 해결할 수 있다.

### annotation을 이용한 AOP 적용 방법

annotation 등록

```java
package hello.hellospring.aop;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.CLASS)
public @interface PerLogging {}
```

> **@Target**은 적용될 타입
>
> **RetentionPolicy** 이어노테이션정보를 얼마나 유지할것인가? 기본값은 CLASS, **CLASS**인경우에는 해당 어노테이션정보가 byte코드까지 남아있지만, **SOURCE**로 변경시 컴파일 후 사라진다

annotation을 이용하여 AOP 적용

```java
package hello.hellospring.aop;

@Component
@Aspect
public class PerfAspect {
    @Around("@annotation(PerLogging)")
	public Object logPerf(ProceedingJoinPoint pip) throws Throwable {
		long begin = System.currentTimeMillis();
		Object retVal = pip.proceed();
		System.out.println(System.currentTimeMillis()-begin);
		return retVal;
	}
}
```

기존의 Service에서 createEvent(), pulbicEvent() 메서드에만 시간을 측정하고싶다.

```java
package hello.hellospring.aop;

@Service
public class SimpleEventService implements EventService {

	@PerLogging //어노테이션 적용
	@Override
	public void createEvent() {
		try {
			Thread.sleep(1000);
		} catch (Exception e) {
			e.printStackTrace();
		}
		System.out.println("Create Event!!");
	}

	@PerLogging //어노테이션 적용
	@Override
	public void pulbicEvent() {
		try {
			Thread.sleep(2000);
		} catch (Exception e) {
			e.printStackTrace();
		}
		System.out.println("Pulbic Event!!");
	}

	@Override
	public void deleteEvent() {
		System.out.println("Delete Event!!");

	}
}
```

```java
// Create Event!!
// 1016
// Pulbic Event!!
// 2001
// Delete Event!!
// 0
```

위와 같은 방법으로 AOP를 적용하여 내가 원하는 메서드에 **@PerLogging** 어노테이션을 붙여 **Aspect**가 적용되어 원하는 메서드만 수행시간을 측정 할 수있게 되었다.

마지막으로 또다른 방법으로는 스프링 빈의 모든메서드에 적용할 수 있는 방법도 있다.

```java
package hello.hellospring.aop;

@Component
@Aspect
public class PerfAspect {
    @Around("bean(simpleEventService)")
	public Object logPerf(ProceedingJoinPoint pip) throws Throwable {
		long begin = System.currentTimeMillis();
		Object retVal = pip.proceed();
		System.out.println(System.currentTimeMillis()-begin);
		return retVal;
	}
}
```

이 밖에도 **@Around** 외에 타겟 메서드의 **Aspect** 실행 시점을 지정할 수 있는 어노테이션이 있다.

- **@Before (이전)** : 어드바이스 타겟 메소드가 호출되기 전에 어드바이스 기능을 수행
- **@After (이후)** : 타겟 메소드의 결과에 관계없이(즉 성공, 예외 관계없이) 타겟 메소드가 완료 되면 어드바이스 기능을 수행
- **@AfterReturning (정상적 반환 이후)** : 타겟 메소드가 성공적으로 결과값을 반환 후에 어드바이스 기능을 수행
- **@AfterThrowing (예외 발생 이후)** : 타겟 메소드가 수행 중 예외를 던지게 되면 어드바이스 기능을 수행
- **@Around (메소드 실행 전후)** : 어드바이스가 타겟 메소드를 감싸서 타겟 메소드 호출전과 후에 어드바이스 기능을 수행
