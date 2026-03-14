# spring-tutorial-22rd
CEOS 백엔드 23기 스프링 튜토리얼

---

## 1. Spring이 지원하는 핵심 기술


### **IoC(Inversion of Control, 제어의 역전)**

Java 언어만을 사용해 개발한다면 `new`키워드로 객체를 만들고 의존되는 객체도 직접 연결해야 했다. Spring에서는 객체 생성과 연결의 주도권을 개발자 코드에서 Spring Container로 넘긴다. 즉, 객체를 외부에서 관리하게 되고, 실제로 필요한 순간 외부에서 제공해주는 객체를 받아오게끔 한다. 이러한 개념(디자인 패턴)이 바로 제어의 역전이다.

> Spring Container란? 
스프링에서 객체를 관리하는 주체로서 스프링 컨테이너가 관리하는 객체를 Bean이라고 부른다.
> 

```java
public class A {
	b = new B(); // 클래스A에서 new키워드로 클래스B의 객체 직접 생성
}

public class A {
	private B b; // 외부에서 받아온 클래스B의 객체를 변수 b에 할당
}
```

### **DI(Dependency Injection, 의존성 주입)**

IoC(제어의 역전)라는 디자인 패턴을 구현하기 위해 사용하는 방법이 DI(의존성 주입)이다. 만약 클래스A에서 클래스B의 객체가 필요하다면 A는 B를 의존한다고 말할 수 있고, DI를 통해 A가 B를 직접 생성하지 않고 외부(스프링 컨테이너)에서 넣어준다.(=주입시킨다.)

**의존성 주입의 방법**

의존성 주입 방법으로는 생성자 주입, 수정자 주입, 필드 주입 등이 존재한다. 스프링 공식 문서에서는 생성자 주입을 권장한다.

| **방식** | **특징** | **비고** |
| --- | --- | --- |
| **생성자 주입** | 생성자를 통해 의존성을 전달받음 | **가장 권장됨** (불변성 확보, 테스트 용이) |
| **수정자(Setter) 주입** | Setter 메서드를 통해 전달받음 | 주입받는 객체가 변경될 가능성이 있을 때 사용 |
| **필드 주입** | 변수에 `@Autowired`를 직접 붙임 | 코드가 간결하나 외부에서 변경이 어려워 권장되지 않음 |

### **AOP(Aspect Oriented Programming, 관점 지향 프로그래밍)**

프로그래밍 패러다임 중 하나로, 관점을 기준으로 묶어 개발하는 방식을 의미한다. 

애플리케이션의 핵심 비즈니스 로직과 관련 없는 부가적인 기능들을 모듈화하여 코드의 중복을 줄이고 유지보수성을 향상시키는 데에 주로 활용된다. 이를 통해 개발자는 반복 작업을 줄이고 핵심 기능 로직에만 집중할 수 있도록 한다. 쉽게 이해하면 AOP는 공통된 기능을 재사용하는 기법이라 말할 수 있다.

<img width="951" height="408" alt="Image" src="https://github.com/user-attachments/assets/432d1df5-cf0c-410d-837c-0becca07e6fe" />

첨부한 그림에서는 로그인, 검색, 게시판에서 중복되는 공통 코드 부분(Logging, Security, Transaction)을 별도의 영역으로 분리하여 소스코드의 중복을 줄이고, 필요할 때마다 가져다 쓸 수 있게끔 한다.

Spring은 **프록시 패턴**을 사용하여 AOP를 구현한다. 공통 기능(Aspect)을 직접 비즈니스 로직에 삽입하는 것이 아니라, 프록시를 앞에 세워 로직을 가로채도록 한다.

**프록시의 동작 과정**

1. 클라이언트가 특정 메서드를 호출하면, 실제 객체가 아닌 **프록시 객체**가 호출을 대신 받는다.

```java
public class Main {
    public static void main(String[] args) {
        HelloService target = new HelloService(); // 진짜
        HelloService proxy = new HelloServiceProxy(target); // 가짜(프록시)

        // 사용자는 평소처럼 호출하지만, 실제로는 프록시가 실행되어 시간이 측정됨
        proxy.sayHello();
    }
}
```

2. 프록시는 실제 로직 수행 전후에 **공통 기능(Advice)** (예: 트랜잭션 시작, 로그 출력)을 실행한다.

```java
public class HelloServiceProxy extends HelloService {
    private final HelloService target; // 진짜 객체

    public HelloServiceProxy(HelloService target) {
        this.target = target;
    }

    @Override
    public void sayHello() {
        System.out.println("[AOP] 시간 측정 시작"); // 공통 로직 (전처리)
        
        target.sayHello(); // 실제 핵심 로직 실행
        
        System.out.println("[AOP] 시간 측정 완료"); // 공통 로직 (후처리)
    }
}
```

3. 이후 프록시가 실제 객체(Target)의 메서드를 호출한다.

```java
public class HelloService {
    public void sayHello() {
        System.out.println("안녕하세요! 핵심 로직 실행 중...");
    }
}
```

### **PSA(Portable Service Abstraction, 이식 가능한 서비스 추상화)**

PSA는 환경의 변화와 관계없이 일관된 방식의 추상화된 인터페이스를 제공하는 원칙이다. 스프링에서 제공하는 다양한 기술들을 추상화해 개발자가 쉽게 사용할 수 있게끔 한다.

예시: Spring Transaction (`@Transactional`)

개발자는 DB 기술이 무엇이든(JDBC, JPA, Hibernate 등) 상관없이 `@Transactional`어노테이션으로 트랙잭션을 관리할 수 있다. 내부적으로는 각 DB 기술에 맞는 구현체가 동작하지만, 개발자는 이를 알 필요 없이 추상화된 인터페이스만으로 적용이 가능하다.

---

## 2. Spring Bean

Spring Bean은 Spring Container가 생성하고 관리하는 자바 객체(POJO)를 의미한다.

```java
@Component
public class MemberService {
}
// 또는
@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        return new MemberService();
    }
}
```

일반 `new` 키워드를 통해 생성한 객체만으로 Bean이 되는 것이 아니며, 위와 같이 등록되면 Spring Bean이다.

Bean이 되면 Spring은 빈의 생성, 의존성 주입, 초기화, 후처리, 소멸을 관리한다.

### **Bean Lifecycle**

Bean Lifecycle은 빈이 생성되어 사용되고 사라질 때까지의 생명 주기다.

<img width="1474" height="753" alt="Image" src="https://github.com/user-attachments/assets/5349e5b1-7dbb-420c-ad94-40a7b79ee322" />

Bean의 라이프사이클은 다음과 같다.

1. 스프링 컨테이너 생성

애플리케이션 실행 시 컨테이너를 생성한다. 이 컨테이너는 어떤 빈을 만들고 어떻게 관리할지에 대한 정보를 가진다.

2. 스프링 빈 생성

컨테이너는 빈 정의 정보를 바탕으로 객체를 생성한다. 객체가 메모리에 올라가는 단계.

3. 의존관계 주입

생성된 빈에 필요한 다른 빈들을 연결한다. 앞서 정리한 생성자 주입, 수정자 주입, 필드 주입이 이 단계와 관련된다.

4. 초기화 콜백

빈이 실제로 사용되기 전에 초기화 작업을 수행한다. 예를 들어, DB 연결 확인, 외부 API client 준비, 설정값 검증 같은 작업이 수행된다. 보통 `@PostConstruct`, `afterPropertiesSet()`, `initMethod` 등을 사용할 수 있다.

5. 사용

초기화가 완료된 빈은 애플리케이션에서 실제로 사용된다. 서비스 로직 수행, 컨트롤러 요청 처리, 리포지토리 조회 등 대부분의 비즈니스 동작이 이 단계에서 이루어진다.

6. 소멸전 콜백

컨테이너가 종료되기 직전에 빈이 정리 작업을 수행한다. 예를 들어 연결 해제, 리소스 반납, 스레드 종료 등이 여기에 해당한다. 보통 `@PreDestroy`, `destroy()`, `destroyMethod` 등을 사용한다.

7. 스프링 종료

모든 정리 작업이 끝나면 스프링 컨테이너가 종료된다.

### Bean Scope

Bean Scope는 빈이 몇 개 생성되고 어느 범위에서 살아있는가를 의미한다.

<img width="842" height="458" alt="Image" src="https://github.com/user-attachments/assets/abf5fa4b-4e6c-411e-8fff-bbbd2c49a616" />

**Singleton**

기본 설정이며, Spring Container당 빈 1개만 생성한다. 대부분의 Service, Repository, Controller는 싱글톤으로 사용된다.

**Prototype**

요청할 때마다 새 객체를 만든다. 주의할 점은 prototype 빈은 생성과 주입까지만 컨테이너가 관리하고, 이후 소멸 관리까지는 기본적으로 책임지지 않는다는 것이다.

**Web scope**

웹 환경에서만 사용 가능하다.

- request: HTTP 요청마다 1개
- session: 세션마다 1개
- application: ServletContext 범위
- websocket: 웹소켓 세션 범위

### Annotation(어노테이션)

사전적 의미로는 주석이란 뜻이며, 코드에 부가 정보를 붙이는 메타데이터 기능을 수행한다. 어노테이션이 수행하는 기능은 아래와 같다.

- 컴파일러에게 코드 작성 문법 에러를 체크하도록 정보를 제공
- 소프트웨어 개발 툴이 빌드나 배치시 코드를 자동으로 생성할 수 있도록 정보를 제공
- 실행시(런타임시) 특정 기능을 실행하도록 정보를 제공

어노테이션은 로직에 직접 영향을 주지는 않지만, 해당 코드를 사용하는 도구(프레임워크, 라이브러리 등)가 이를 해석하여 특별한 처리를 할 수 있게 한다.

**Java에서의 어노테이션 구현 원리: 리플렉션(Reflection)**

리플렉션은 실행 중인 자바 프로그램이 자기 자신의 구조(클래스, 메서드, 필드 등)를 조사하고 수정할 수 있게 해주는 기술이다.

[어노테이션 구현 예시]

```java
// 1. 어노테이션 정의
@Target(ElementType.METHOD) // 메서드에 붙이겠다고 선언
@Retention(RetentionPolicy.RUNTIME) // 실행 시까지 정보를 유지하겠다고 선언
public @interface MyAnnotation {
    String value() default "기본값";
}

// 2. 어노테이션 사용
public class MyClass {
    @MyAnnotation(value = "테스트")
    public void myMethod() { ... }
}
```

`@Target`

어디에 붙일 수 있는가: 클래스, 메서드, 필드, 파라미터 등

`@Retention`

언제까지 유지되는가

- `SOURCE`: 컴파일 후 버려짐
- `CLASS`: class 파일에는 남지만 런타임 리플렉션 보장은 없음
- `RUNTIME`: 런타임까지 유지, 리플렉션 가능

어노테이션은 `@interface` 키워드를 사용하여 선언하며, 프로그램이 실행 중일 때(Runtime) 자바의 Reflection API를 사용하여 특정 클래스나 메서드에 붙은 어노테이션 정보를 읽어온다. 정보를 읽어온 후 그 결과에 따라 객체를 빈으로 등록하거나, 프록시를 생성하는 등의 로직을 수행한다.

**어노테이션을 통한 빈(Bean) 등록 과정**

스프링에서 @Component와 같은 어노테이션이 붙은 클래스가 빈으로 등록되기까지는 다음과 같은 과정이 일어난다.

1. Bean Deifinition 생성

스프링은 어노테이션이 붙은 클래스를 발견하면, 해당 클래스의 정보를 바탕으로 빈의 설계도인 BeanDefinition 객체를 생성한다.

2. BeanDefinitionRegistry 등록

생성된 설계도들을 BeanDefinitionRegistry라는 저장소에 등록한다.

3. 빈 생성 및 의존관계 주입

등록된 설계도를 보고 스프링 컨테이너가 실제 자바 객체를 생성하고, 필요한 의존성을 주입한다.   


**`@ComponentScan` 어노테이션**

ComponentScan 어노테이션은 스프링이 어디서부터 컴포넌트를 찾을지 결정하며 AppConfig.class와 같은 설정 정보 없이 `@Component`가 붙은 모든 클래스를 자동으로 스프링 빈을 등록하는 기능을 제공한다.

**컴포넌트 스캔의 과정**

1. 스캔 범위 결정

별도의 설정을 하지 않으면 `@Component` 이 붙은 설정 클래스가 위치한 패키지와 그 하위 패키지가 스캔의 범위

2. 후보 클래스 탐색

`ConfigurationClassParser`가 프로젝트의 모든 클래스 파일을 훑으며, 내부적으로는 ASM이라는 바이트코드 분석 라이브러리를 통해 클래스를 실제로 로드하지 않고 파일 구조만 보고 어노테이션 여부를 판단

3. 컴포넌트 필터링

`@Component` 뿐만 아니라, 내부적으로 `@Component`를 포함하고 있는 특수 어노테이션들을 모두 찾아낸다.

- `@Component`
- `@Controller` & `@RestController`
- `@Service`
- `@Repository`
- `@Configuration`

4. 빈 이름 결정 및 등록

찾아낸 클래스들을 빈으로 등록할 때 기본적으로는 클래스명의 첫 글자를 소문자로 바꾼 이름을 빈 이름으로 사용한다. 예를 들어, HelloService는 helloService로 등록된다. 만약 중복된 이름이 있다면 예외가 발생한다.

**추가 내용) 하나의 인터페이스를 상속받은 구현체가 여러 개인 상황**

이러한 상황에서 스프링은 어떤 빈을 주입해야 할지 몰라 `NoUniqueBeanDefinitionException`을 발생시킨다. 이를 해결하기 위한 방법으로는 우선순위 지정, 빈 이름 명시, 모든 빈 주입이 있다.

1. `@Primary` 사용 (우선순위 지정)

여러 후보 중 기본적으로 주입될 빈을 설정하는 방법이다. 가장 간편하며 프로젝트 전반에서 특정 구현체를 주로 사용할 때 적합하다.

```java
@Component
@Primary // PaymentService 주입 요청 시 이 빈을 우선적으로 선택함
public class KakaoPay implements PaymentService {}

@Component
public class NaverPay implements PaymentService {}
```

2. `@Qualifier` 사용 (빈 이름 명시)

주입받는 지점에서 빈의 이름을 직접 지정하는 방법이다. 특정 상황에서 특정 구현체가 필요할 때 사용하며, `@Primary`보다 우선순위가 높다.

```java
@Service
public class OrderService {
    private final PaymentService paymentService;

    // 빈 이름이 naverPay인 것을 찾아 주입함
    public OrderService(@Qualifier("naverPay") PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

3. 컬렉션(List, Map) 주입 (모든 빈 주입)

해당 인터페이스를 구현한 모든 빈을 한꺼번에 주입받는 방법이다. 동적으로 구현체를 선택해야 할 때 유용하다.

```java
@Service
public class PaymentFactory {
    // 빈 이름을 Key로, 객체를 Value로 모든 구현체를 담음
    private final Map<String, PaymentService> paymentMap;

    public PaymentFactory(Map<String, PaymentService> paymentMap) {
        this.paymentMap = paymentMap;
    }

    public void pay(String payType) {
        // 상황에 맞는 구현체를 맵에서 꺼내어 사용
        PaymentService service = paymentMap.get(payType);
        service.process();
    }
}
```

---

## 3. MVC 패턴과 Spring MVC

**MVC 패턴**은 소프트웨어를 역할별로 나누는 설계 아이디어다. MVC 패턴을 도입하면 UI 영역과 비즈니스 로직 영역이 구분되어 서로에게 영향을 주지 않으면서 개발과 유지보수가 가능해진다.

보통 다음처럼 나눈다.

- **Model**: 데이터와 비즈니스 로직

클라이언트의 요청 사항을 처리하기 위한 작업을 한다.

- **View**: 사용자에게 보여지는 화면

애플리케이션의 화면에 보이는 리소스를 제공하는 역할을 한다.

- **Controller**: 요청을 받아 흐름을 조정하는 역할

컨트롤러는 클라이언트의 요청을 직접적으로 전달받는 엔드포인트이며 Model과 View의 중간에서 역할을 수행한다. 

### Spring MVC

Spring MVC는 Spring Framework와 Servlet API를 기반으로 하는 웹 애플리케이션 프레임워크이다. 중심에는 `DispatcherServlet`이 있는 **Front Controller 패턴**이 있다. 즉, Spring MVC는 Controller, Model, View를 나누는 것과 더불어 HTTP 요청을 받아 적절한 핸들러에 연결하고, 결과를 뷰나 응답 본문으로 바꾸는 전체 실행 구조까지 제공한다.

### 서블릿(Servlet)이란?

서블릿(Servlet)이란 동적 웹 페이지를 만들 때 사용되는 자바 기반의 웹 애플리케이션 프로그래밍 기술이다. 서블릿은 웹 요청과 응답의 흐름을 간단한 메서드 호출만으로 체계적으로 다룰 수 있게 해준다. 서버에서 실행되다가 웹 브라우저에서 요청을 하면 해당 기능을 수행한 후 웹 브라우저에 결과를 전송한다.

여기서 웹 요청의 큰 흐름은 다음과 같다.

1. 사용자가 브라우저에서 URL을 요청한다.
2. HTTP 요청이 서버로 전달된다.
3. 톰캣 같은 서블릿 컨테이너가 이 요청을 받는다.
4. 요청 URL에 맞는 서블릿을 찾는다.
5. 해당 서블릿이 요청을 처리한다.
6. 응답을 만들어 브라우저에 돌려준다.

서블릿을 사용한 전통적인 코드는 아래와 같다.

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.getWriter().write("hello");
    }
}
```

> Spring MVC를 사용하면 개발자가 이런 식으로 요청마다 서블릿을 직접 만들기보다, **`DispatcherServlet`** 하나가 먼저 모든 요청을 받고 그 뒤를 분배한다는 점이 다르다.
> 

### WAS란?

기본적으로 **Web Server**는 HTML, CSS, JavaScript, 이미지 파일과 같은 **정적인 콘텐츠**를 클라이언트에게 응답하는 역할을 수행한다.

반면 **WAS(Web Application Server)** 는 웹 애플리케이션을 실행해 주는 서버 환경이다. WAS는 정적인 파일만 전달하는 서버가 아니라, 자바 코드 같은 애플리케이션 로직을 실행해서 동적인 응답을 만드는 역할까지 포함한다. 예를 들어 사용자의 요청에 따라 데이터베이스를 조회하거나, 비즈니스 로직을 수행한 뒤 그 결과를 HTML이나 JSON 형태로 반환하는 작업은 WAS가 담당한다.

WAS와 Web Server를 분리하는 이유는 아래와 같다.

- 기능을 분리하여 서버의 부하 방지
- 보안 강화
- 여러 대의 WAS 연결 (로드 밸런싱)
- 무중단 운영을 위한 배포 편의성

**Tomcat이란?**

대표적인 자바 기반의 WAS임과 동시에 **서블릿 컨테이너**이다. 클라이언트의 HTTP 요청을 받아 적절한 서블릿으로 전달하고, 서블릿이 처리한 결과를 다시 HTTP 응답 형태로 반환하는 역할을 수행한다.

여기서 스블릿 컨테이너란, Tomcat이 단순히 요청을 중계만 하는 것이 아니라 서블릿의 생성, 초기화, 실행, 종료까지 관리하는 실행 환경이라는 뜻이다. 따라서 개발자는 비즈니스 로직에 집중할 수 있고, Tomcat은 해당 로직이 웹 환경에서 실행될 수 있도록 지원한다.

정리하면 다음과 같다.

- **Web Server**: 정적 자원 제공에 강점
- **WAS**: 애플리케이션 로직 실행, 동적 응답 생성
- **Tomcat**: 자바 웹 애플리케이션에서 요청을 받아 서블릿을 실행하는 대표적인 WAS이자 서블릿 컨테이너

> Spring Boot 애플리케이션은 실행과 동시에 Tomcat이 떠서 HTTP 요청을 처리할 수 있다. 스프링 부트에서는 보통 내장 톰캣을 사용하기 때문이다. 별도의 외부 서버 설정 없이도 애플리케이션 실행만으로 웹 서버가 함께 구동된다.
> 

### DispatcherServlet이란?

`DispatcherServlet`은 **Spring MVC의 중심이 되는 서블릿**이다. 클라이언트의 요청을 가장 먼저 받아서, 어떤 컨트롤러가 처리해야 하는지 결정하고, 처리 결과를 최종 응답으로 만들어 주는 역할을 한다.

<img width="1280" height="598" alt="Image" src="https://github.com/user-attachments/assets/df3f566f-9cad-4090-ab14-445077f3189e" />

`DispatcherServlet`의 실제 요청 처리 핵심은 `doDispatch()` 메서드에 담겨 있다.

doDispatch() 메서드를 통한 DispatcherServlet 동작 흐름은 아래와 같다.

1. **요청 전처리**
    
    먼저 `checkMultipart(request)`를 통해 파일 업로드 요청인지 확인한다.
    
2. **핸들러 탐색**
    
    `getHandler(processedRequest)`를 호출하여 현재 요청을 처리할 컨트롤러를 찾는다. 만약 적절한 핸들러가 없으면 더 이상 진행하지 않는다.
    
3. **인터셉터 전처리**
    
    `applyPreHandle()`을 호출하여 로그인 체크, 권한 확인, 로깅 같은 공통 작업을 수행한다. 여기서 조건이 맞지 않으면 요청 처리가 중단될 수 있다.
    
4. **HandlerAdapter 조회 및 컨트롤러 실행**
    
    `getHandlerAdapter(...)`로 실행에 필요한 어댑터를 찾고, `ha.handle(...)`을 통해 실제 컨트롤러 메서드를 호출한다.
    
5. **후처리**
    
    컨트롤러 실행 후 `applyDefaultViewName(...)`, `applyPostHandle(...)` 등이 수행된다. 즉, 뷰 이름 보정이나 인터셉터 후처리가 이루어진다.
    
6. **결과 처리**
    
    마지막으로 `processDispatchResult(...)`가 호출되어 뷰 렌더링 또는 예외 처리가 이루어진다.
    
7. **자원 정리**
    
    요청 처리가 끝나면 multipart 요청 관련 자원 등을 정리한다.
