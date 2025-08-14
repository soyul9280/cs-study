이 글에서는 Spring의 핵심 개념 중 IoC(Inversion of Control): 제어의 역전에 대해 알아봅니다.

---

## 📌 제어의 역전 - IoC란?

전통적인 프로그래밍 방식에서는 개발자가 언제 어느 객체를 생성하고 폐기할지, 어떤 흐름으로 실행할지를 모두 직접 결정했습니다.

제어의 역전은 **애플리케이션의 제어 흐름을** **개발자가 아닌 프레임워크가 관리**하도록 하는 원칙입니다.

즉, **객체의 생성과 생명주기, 의존성 관리를 개발자가 직접 하지 않고, 프레임워크에 위임하는 것**이죠.

애플리케이션 제어 권한이 개발자에서 프레임워크로 역전되어 **"제어의 역전"**이라고 합니다

이때 프레임워크의 어느 구성 요소가 이 역할을 할까요?

## 🪣 IoC 컨테이너

컨테이너란?

-   객체의 생성, 초기화, 소멸 등 생명 주기를 관리하며, 생성된 객체에 추가적인 기능을 제공하는 것

스프링에서는 자바 객체를 빈(Bean)이라 합니다.

즉, IoC 컨테이너는 내부에 존재하는 빈의 생명 주기를 관리(생성, 소멸 등)하며, 필요한 의존성을 관리해줍니다.

IoC 컨테이너에는 빈 팩토리(Bean Factory)와 애플리케이션 컨텍스트(Application Context)가 있습니다.

-   빈 팩토리
    -   컨테이너에서 객체(Bean)을 생성하고, DI를 처리하는 기본적인 기능만 제공
    -   Bean을 등록, 생성, 조회, 제거 등 관리
-   애플리케이션 컨텍스트
    -   빈 팩토리에서 확장된 컨테이너
    -   국제화, 이벤트 전달, AOP 지원 등 추가 기능이 있어 보통 애플리케이션 컨텍스트를 사용

## 📌 IoC의 장점

제어 흐름의 권한이 개발자가 아닌 프레임워크에 있다면 어떤 점이 좋을까요?

-   **✅ 객체 간의 결합도를 낮출 수 있습니다.**
-   **✅ → 개발자는 비즈니스 로직에 집중할 수 있습니다.**
-   **✅ **→**  유지보수가 쉬운 코드를 작성할 수 있습니다.**
-   **✅ **→**  테스트를 작성하기 쉽습니다.**  
    -   실제 구현 대신 테스트용 Mock 객체를 쉽게 주입할 수 있어서

### 👩‍🏫 왜 결합도를 낮출 수 있을까?

결합도란, 두 클래스가 서로 얼마나 강하게 연결되어 있는지 정도를 말합니다.

결합도가 높으면 한쪽이 바뀌면 다른 쪽도 같이 바꿔야 해서 유지보수가 어려워집니다.

**기본적인 제어 흐름**

```
public class Service {

	private Repository repository;

	public Service() {
		this.repository = new Repository();  // 의존 객체 직접 생성
	}

	public void performAction() {
		repository.save();
	}
}
```

이 코드에서는 Service 클래스가 Repository 객체를 직접 new()로 생성하고 있습니다.

만약 Repository 클래스가 내부 로직을 바꾸거나, 생성자 파라미터가 추가되면

Service 코드도 함께 수정해야 합니다.

또한 Service 클래스가 Repository를 어떻게 생성하는지까지 책임지기 때문에, 

두 클래스가 아주 강하게 결합한 형태입니다.

**IoC를 적용한 제어 흐름**

```
public class Service {

	private Repository repository;

	public Service(Repository repository) {
		this.repository = repository; // 외부에서 의존성 주입
	}

	public void performAction() {
		repository.save();
	}
}
```

Service 클래스는 Repository 객체를 직접 생성하지 않고, 외부에서 생성자를 통해 주입받고 있습니다.

이미 만들어진 Repository를 주입받는 것이기 때문에 Service는 Reposiory가 어떻게 만들어지는 알 필요가 없습니다.

Repository 내부 로직을 변경하거나 교체해도 Service 클래스의 코드를 바뀔 확률이 낮고,

따라서 두 클래스 간의 결합도가 낮습니다.

-   직접 생성하면 Service가 Repository의 구체적인 클래스를 알아야 하므로 강하게 결합합니다.
-   의존성 주입을 하면 Service는 단지 Repository라는 역할(인터페이스/클래스)을 받기만 할 뿐, 내부 구현은 신경쓰지 않습니다.

---

## 📌 IoC 구현: DI와 DL

두 가지 방법이 있습니다.

### 💉 DI(Dependency Injection): 의존성 주입

-   의존 관계를 외부에서 결정(주입)해주는 방식
-   각 클래스 간의 의존관계를 빈 설정을 바탕으로 컨테이너가 자동으로 연결해줍니다.
-   주요 방식 
    -   생성자 주입
    -   필드 주입
    -   Setter 주입

```
// 생성자 주입

public class Service {
    private Repository

    public Service(Repository repository) {	// 외부에서 주입
        this.repository = repository;
    }
}
```

### 🔎 DL(Dependency Lookup): 의존성 검색

-   의존관계를 필요한 시점에 직접 컨테이너에서 검색해서 찾아오는 방식
-   어떤 클래스의 객체를 이용할지와 객체를 생성하고 관리하는 것은 컨테이너에게 맡기지만  
    의존 객체를 가져올 때는 의존 객체가 필요한 객체에서 스스로 컨테이너에게 요청하는 방식을 사용합니다.

```
public class Service {
	AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
	private Repository repository;
    
    public Service() {
    	this.repository = ac.getBean(Repository.class);
    }
}
```

-   DI가 더 깔끔하고 유지보수에 유리한 편입니다.
-   그러나 DL은 동적으로 빈을 찾아야 하거나, 런타임에 어떤 객체를 사용할지 결정해야 하는 상황에서 유용할 수 있습니다.  
    자주 사용하지는 않습니다.

---

## 🔑 요약 및 예상 질문

| IoC란 무엇인가요? | 애플리케이션에서 객체 생명주기 등 제어의 흐름이 개발자가 아닌 프레임워크에 있는 것입니다. |
| --- | --- |
| IoC 컨테이너란 무엇인가요? | 빈의 생성과 생명 주기를 관리하는 스프링의 구성 요소입니다. |
| IoC의 장점은 무엇인가요? | 객체 간의 결합도를 낮춰 유지보수와 테스트 작성이 쉬워집니다.   개발자가 객체를 직접 관리하지 않아 로직에만 집중할 수 있습니다. |
| DI와 DL이 무엇인가요? | DI(의존성 주입)은 의존 객체를 생성자, 세터 등을 통해 외부에서 주입 받는 방식입니다.      DL(의존성 검색)은 필요한 의존 객체를 직접 컨테이너에서 조회하는 방식입니다. |

---

참고자료

[\[Spring\] IoC 컨테이너 (Inversion of Control) 란?](https://dev-coco.tistory.com/80)

[\[Spring\] Spring-Container, IoC, DI, Singleton 개념 정리](https://lucas-owner.tistory.com/39)

[\[Java\] Spring Framework 주요 특징 이해하기 : DI, IoC, POJO, AOP](https://adjh54.tistory.com/298#2\)%20%EC%A0%9C%EC%96%B4%EC%9D%98%20%EC%97%AD%EC%A0%84%20%3AIOC\(Inversion%20of%20Control\)-1-1)

[\[spring\] DI와 DL의 차이 ,IoC](https://toki0411.tistory.com/54)
