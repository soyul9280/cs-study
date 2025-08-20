Spring의 핵심 개념에는 DI(Dependency Injection), IoC(Inversion of Control), AOP(Aspect-Oriented Programming)가 있습니다.  
  
이 글에서는 DI: 의존성 주입의 개념과 필요성, 사용 방법을 다룹니다.

---

## 📌 DI: 의존성 주입

> **의존성이란?**  
> 한 객체가 다른 객체를 사용할 때 의존성이 있다고 합니다.  
> 예를 들어 \`OrderService\`가 \`OrderRepository\`를 사용한다면, \`OrderService\`는 \`OrderRepository\`에 의존합니다.

Spring의 특징 중 [제어의 역전](https://syleeblog.tistory.com/64)에 대해 공부할 때

객체의 생성과 의존성 관리를 개발자가 아닌 스프링 프레임워크의 IoC 컨테이너가 담당하고 있다고 했습니다.

의존성 주입은 IoC를 구현하는 방법 중 하나로,

**필요한 의존 객체를 직접 생성하지 않고 외부에서 주입받아 사용**하는 방식입니다.

주로 애플리케이션 구동 시점에 컨테이너가 객체를 생성하고, 의존성을 연결해 줍니다.  
이를 통해 객체 간 결합도를 낮추고, 유연성과 테스트 용이성을 높일 수 있습니다.

---

## 💉 Spring에서 의존성을 주입하는 방법

### 1\. 생성자 주입 (Constructor Injection)

의존성을 생성자를 통해 주입 받습니다.

```
@Service
public class Service {
	private final Repository repository;
    
    public Service(Repository repository) {
    	this.repository = repository;
    }
}
```

-   불변성 보장
    -   필드에 final로 선언하여 의존성을 객체 생성 시점에 1회만 주입받고, 이후에는 변경할 수 없게 합니다.
    -   필수 의존성을 항상 초기화된 상태로 일관성 있게 유지하기 쉽습니다.
-   테스트가 편리함
    -   단위 테스트 시 Mock 객체를 생성자의 인자로 직접 넘길 수 있습니다.
-   런타임 오류 방지
    -   필수 의존성이 누락될 경우 컴파일 시점 또는 애플리케이션 구동 시점에 오류를 빠르게 발견할 수 있습니다.

```
@RequiredArgsConstructor
@Service
public class Service {
	private final Repository repository;
   	
    ...
}
```

롬복 애노테이션 중 \`@RequiredArgsConstructor\`을 사용하면

final 키워드가 붙은 필드에 대해 자동으로 생성자를 만들어주므로

코드에서 생성자를 생략해도 됩니다.

### 2\. Setter 주입 (Setter Injection)

의존성을 세터 메서드(또는 \`@Autowired가 붙은 일반 메서드)를 통해 주입 받습니다.

```
@Service
public class NotificationService {

    private EmailClient emailClient;

	// Setter로 주입
    // @Autowired 생략 가능
    public void setEmailClient(EmailClient emailClient) {
        this.emailClient = emailClient;
    }

    public void sendNotification(String message) {
		    if (emailClient == null) return;
        emailClient.send(message);
    }
}
```

-   객체 생성 이후에 세터를 통해 의존 객체를 변경할 수 있으므로 설정 정보를 동적으로 바꿔야할 때 유용합니다.
-   클래스가 생성 시점에 완전히 초기화되지 않을 수 있으므로 불변성이 떨어집니다.
-   필수 의존성에 사용하여 제대로 초기화하지 않으면 \`NullPointerException\`이 발생할 수 있습니다.

### 3\. 필드 주입 (Field Injection)

필드에 \`@Autowired\`를 붙여주면 자동으로 의존 관계가 주입됩니다.

```
@Service
public class Service {
	@Autowired
    private Repository repository;
    
    ...
}
```

-   코드가 간결해집니다.
-   final 키워드를 사용할 수 없어 불변성이 보장되지 않습니다.
    -   주입된 객체가 런타임 중에 변경될 수 있고, 예측 불가능한 동작을 할 수도 있습니다.
-   단위 테스트가 어렵습니다.  
    -   의존성을 직접 Mock 객체로 넣기 힘듭니다.
-   간편하게 의존 관계 주입이 가능하지만 참조 관계를 눈으로 확인하기 어렵고, 순환 참조를 막을 수 없습니다.
    -   생성자 주입은 의존성을 파라미터로 받기 때문에 어떤 객체가 필요한지 한 눈에 보입니다.
    -   생성자 주입은 순환 참조가 있으면 애플리케이션 시작 시 바로 예외가 발생하지만, 필드 주입은 프록시 초기화 시점까지 지연되어 런타임 도중에 문제가 드러날 수 있습니다.

단점이 많아 지양하는 방식입니다.

### 어떤 방법이 좋을까?

생성자 주입 방식이 일반적으로 [Spring 공식 문서](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)에서 가장 권장되는 방식입니다.

**불변성을 보장**하고,

**필수 의존성이 항상 초기화**되어 null일 걱정이 없기 때문입니다.

---

불변성이 중요한 이유는 의존 객체가 한 번 주입되면 바뀌지 않아 예측 가능한 동작만 하고, 
멀티스레드 환경에서 객체 상태 변경 위험이 줄어드니 스레드 안전성이 확보되고,
버그 추적도 쉽기 때문입니다.

그렇다고 항상 생성자 주입 방식만 사용하라는 건 아니고
세터 주입은 의존성 객체를 변경할 수 있다는 특징이 있으므로
필수 종속 관계에는 생성자 주입 방식을 사용하고, 선택적 종속 관계에는 세터 메서드를 혼합 사용할 수 있습니다.
(실무에서는 별로 없는 일이라고는 합니다.)

다음 코드는 생성자 주입과 세터 주입을 혼합한 예시입니다.

```
@Service
public class NotificationService {
    private final UserRepository userRepository;	// 필수적으로 필요한 관계
    private EmailClient emailClient;  // 선택적으로 필요한 관계

	// 생성자 주입
    public NotificationService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
	
    // 세터 주입
    @Autowired(required = false)
    public void setEmailClient(EmailClient emailClient) {
        this.emailClient = emailClient;
    }

    public void sendUserNotification(Long userId, String message) {
        if (emailClient == null) {
            // 이메일 클라이언트가 없는 경우, 로그만 남긴다 등..
            System.out.println("No EmailClient found. Skipping email...");
            return;
        }
        User user = userRepository.findById(userId);
        emailClient.sendToUser(user, message);
    }
}
```

---

## 🔑 요약

-   DI와 IoC
    -   IoC: 객체의 생명 주기를 개발자가 아닌 컨테이너가 관리
    -   DI: IoC를 구현하는 방법 중 하나로, 객체가 직접 의존 객체를 생성하지 않고 외부에서 주입 받아 사용하는 방식
-   DI의 장점
    -   객체 간 결합도 감소
    -   의존 객체 불변성 보장 가능 → 안정적이고 예측 가능한 동작
    -   테스트 용이성 → Mock 객체를 쉽게 주입 가능
-   DI 방법
    -   생성자 주입: 일반적으로 권장됨 / 불변성 보장, 테스트 용이, 순환 참조 또는 의존성 초기화 누락 쉽게 감지
    -   세터 주입: 런타임 중 의존 객체를 변경하고 싶을 때 사용
    -   필드 주입: 코드가 간결하나 지양

---

## 👩‍🏫 예상 면접 질문

| IoC와 DI를 설명해주세요. | IoC는 객체 생명주기 등 제어의 흐름이 개발자에서 프레임워크의 IoC 컨테이너로 넘어간 것을 말합니다.   이를 통해 개발자는 로직에 집중하고, 객체 간 결합도를 낮출 수 있습니다.      DI는 IoC를 구현하는 법 중 하나로 의존 객체를 직접 생성하지 않고 외부, Spring IoC 컨테이너에서 주입받는 방식입니다. |
| --- | --- |
| 의존성 주입의 장점이 무엇인가요? | 객체 간 결합도를 낮춰 의존 객체 변경 시 수정을 최소화할 수 있고   불변성을 보장하여 예측 가능한 동작만 하게 하고,   단위 테스트에서 Mock 객체를 쉽게 주입하여 테스트가 용이해집니다. |
| Spring에서 의존성 주입은 어떤 시점에 이루어지나요? | 애플리케이션 구동 시점에 IoC 컨테이너가 초기화되고, 빈 생성을 수행하며 의존 객체를 주입합니다. |
| 의존성 주입 방법에 대해 설명해주세요. | 생성자 주입은 생성자를 통해 의존성을 주입하여 불변성을 보장하고, 필수 의존성에 적합합니다.      세터 주입은 세터 메서드를 통해 주입하여 런타임 도중 의존 객체를 변경할 수 있는 특징이 있습니다.      필드 주입은 필드에 직접 주입하는 방식으로 간단하지만 테스트와 유지보수에 불리합니다. |
| 왜 생성자 주입 방식이 권장될까요? | 의존 객체가 객체 생성 시점에만 1회 주입되어 불변성이 보장되고,    필수 의존성이 누락되었을 때 금방 발견할 수 있기 때문입니다. |
| 불변성이 왜 중요한가요? DI와 어떤 관련이 있나요? | 의존 객체가 한 번만 주입되어 변경되지 않으면   예측 가능한 동작을 보장할 수 있고,   멀티스레드 환경에서 동시성 문제를 줄일 수 있습니다.   또한 상태 변화가 없으므로 디버깅과 유지보수가 쉬워집니다. |

---

참고자료

[Spring DI(Dependency Injection) - 의존 관계 주입 핵심 정리](https://backendcode.tistory.com/249)

[\[Spring\] 의존성 주입(Dependency Injection, DI)이란? 및 Spring이 의존성 주입을 지원하는 이유](https://mangkyu.tistory.com/150)

[Dependency Injection - Spring 공식 문서, 3가지 방식 참고](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)

[Dependency Injection in Spring Boot: The Wizard Behind the Curtain - 의존성 주입 시점 참고](https://dev.to/janisyed18/dependency-injection-in-spring-boot-the-wizard-behind-the-curtain-49n8?utm_source=chatgpt.com)
