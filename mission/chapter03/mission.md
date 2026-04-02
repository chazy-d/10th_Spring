# 피어 리뷰

![](https://img.boostad.site/2026/04/d8fde05adf3c9ed7ecf6e782c96ae6e1.png)

생성자 주입의 장점을 빠르게 불변셩, 필수 의존성 강제, 테스트 용이성 부분에서 설명을 해주셔서 왜 생성자 주입이 실무에서 많이 쓰이는지 빠르게 리마인드하기 좋았다!

또한 생성자 주입이 순환 참조를 빠르게 드러내기에 실행후 찾는 것이 아닌 표면에 드러난 구조 문제를 파악하는데 동둠이 된다는 관점을 알게 되었다!.!

# 시각 자료를 통해 Spring MVC 요청 흐름의 해부도를 이해하기

[PDF 보기](![](https://img.boostad.site/2026/04/28b160e2b397a78452eb4db206db3873.pdf))

## 1. 클라이언트가 HTTP 요청을 보낸다

브라우저, 프론트엔드 앱, Postman 같은 클라이언트가 서버로 HTTP 요청을 보낸다.

예:

- `GET /members/1`
- `POST /orders`

이 시점에는 아직 Spring 관점이 아니라, 그냥 **HTTP 요청**일 뿐이다.

---

## 2. Tomcat 같은 서블릿 컨테이너가 요청을 받는다

Spring Boot 웹 애플리케이션이라면 보통 내장 Tomcat이 떠 있고, 이 Tomcat이 HTTP 요청을 먼저 받는다.

여기서 해야 하는 일은:

- 소켓 수준의 입력 받기
- HTTP 요청 라인과 헤더 해석
- `HttpServletRequest`, `HttpServletResponse` 같은 서블릿 객체 준비
- 어떤 서블릿으로 넘길지 결정할 준비

즉 이 단계는 Spring보다 아래 레벨이고, **웹 실행 환경** 쪽 책임이다.

### 코드에서 어떻게 나타나는가

보통 개발자는 이 단계를 직접 코딩하지 않는다.

Spring Boot가 실행될 때 내장 Tomcat이 뜨고, 이게 자동으로 처리한다.

### 이해 포인트

여기서 Spring은 아직 “중앙 웹 구조 관리자”이지, HTTP 요청을 처음 받는 가장 바깥 계층은 아니다.

---

## 3. Filter 체인을 통과한다

서블릿 컨테이너는 요청을 바로 DispatcherServlet으로 보내지 않고, 먼저 등록된 **Filter**들을 통과하게 할 수 있다.

Filter는 Spring MVC보다 바깥쪽, 즉 **서블릿 계층에서 동작하는 공통 전처리/후처리 장치**라고 이해하면 된다.

보통 여기서 하는 일:

- 인코딩 설정
- 로깅
- 보안 토큰 검사
- 공통 헤더 처리
- 요청/응답 공통 전처리

### 코드에서 어떻게 나타나는가

예를 들어 `OncePerRequestFilter`를 상속해서 필터를 만들 수 있다.

```java
public class LoggingFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        System.out.println("before filter");
        filterChain.doFilter(request, response);
        System.out.println("after filter");
    }
}
```

이 코드에서 핵심은 `filterChain.doFilter(...`이걸 호출해야 다음 필터나 다음 단계로 요청이 진행된다.

### 이해 포인트

Filter는 **DispatcherServlet보다 앞**에 있다. 즉 “Spring MVC 내부 기능”이라기보다, 더 바깥쪽 공통 처리 계층이다.

---

## 4. DispatcherServlet으로 진입한다

이제 요청은 Spring MVC의 핵심 진입점인 **DispatcherServlet**으로 들어간다.

DispatcherServlet은 Spring MVC의 **Front Controller**다.

즉 모든 요청을 먼저 한곳에서 받고, 그다음 적절한 컨트롤러로 분배한다.

이게 중요한 이유는, URL마다 서블릿을 따로 두는 대신 **중앙 집중식 진입점**을 만들 수 있기 때문이다.

### 코드에서 어떻게 나타나는가

개발자가 DispatcherServlet을 직접 호출하진 않는다.

Spring Boot와 Spring MVC 설정이 이걸 등록하고, 서블릿 컨테이너가 요청을 이 서블릿으로 전달한다.

### 이해 포인트

DispatcherServlet은 요청을 “직접 다 처리하는 비즈니스 객체”가 아니라, **요청 처리를 조정하고 연결하는 중앙 관리자**다.

---

## 5. HandlerMapping이 어떤 컨트롤러가 처리할지 찾는다

DispatcherServlet은 요청을 받으면

“이 URL과 HTTP 메서드를 누가 처리해야 하지?”를 판단해야 한다.

이 역할을 하는 것이 **HandlerMapping**이다.

예를 들어:

- `GET /members/1` → `MemberController#getMember()`
- `POST /orders` → `OrderController#createOrder()`

이렇게 매핑해준다.

### 코드에서 어떻게 나타나는가

개발자는 보통 `@GetMapping`, `@PostMapping`, `@RequestMapping` 등을 붙인다.

```java
@RestController
public class MemberController {
    @GetMapping("/members/{id}")
    public MemberResponse getMember(@PathVariable Long id) {
        ...
    }
}
```

이 어노테이션 정보가 런타임에 읽혀서, HandlerMapping이 “이 요청은 이 메서드가 처리해야 한다”고 찾게 된다.

### 이해 포인트

HandlerMapping은 **누가 처리할지 찾는 역할**이지, 직접 호출까지 담당하지는 않는다.

---

## 6. HandlerAdapter가 실제 호출 방식을 맞춘다

핸들러를 찾았다고 바로 실행하는 건 아니다. Spring은 핸들러 구현 방식이 여러 가지일 수 있기 때문에,

중간에서 “이 핸들러를 어떻게 실행할지”를 맞춰주는 계층이 필요하다. 그게 **HandlerAdapter**다.

### 역할

- 찾은 핸들러를 실제로 실행 가능한 형태로 맞춘다
- 컨트롤러 메서드 호출 전에 필요한 준비를 한다
- 파라미터 바인딩, 반환값 처리 흐름으로 이어진다

### 왜 필요한가

DispatcherServlet이 컨트롤러 종류마다 직접 호출 로직을 다 알고 있으면, 특정 구현 방식에 강하게 묶이게 된다.

그래서 HandlerAdapter를 둬서 **호출 방식을 추상화**한다.

### 이해 포인트

- HandlerMapping = 누가 처리할지 찾기
- HandlerAdapter = 어떻게 실행할지 맞추기

이 둘을 분리한 이유는 **확장성과 추상화** 때문이다.

---

## 7. 컨트롤러 메서드 호출 전에 파라미터를 준비한다

컨트롤러 메서드는 그냥 무작정 호출되지 않는다. 그 전에 스프링이 메서드 파라미터를 해석하고 필요한 값을 준비한다.

예를 들어:

- `@PathVariable`
- `@RequestParam`
- `@RequestBody`
- `HttpServletRequest`
- 인증 사용자 정보

같은 것들을 파라미터에 넣어줘야 하는데,,

이건 주로 **ArgumentResolver**, DataBinder, MessageConverter 같은 구성요소가 협력해서 한다.

### 코드에서 어떻게 나타나는가

예를 들어 이런 메서드가 있으면:

```java
@PostMapping("/orders")
public OrderResponse createOrder(@RequestBody CreateOrderRequest request) {
    ...
}
```

Spring은 요청 바디(JSON)를 읽고, `CreateOrderRequest` 객체로 변환해서 메서드에 넣어준다.

### 이해 포인트

개발자는 메서드 시그니처만 선언하지만, 실제로는 Spring이 호출 전에 상당한 준비 작업을 한다.

---

## 8. Controller가 실행된다

이제 실제로 컨트롤러 메서드가 실행된다.

컨트롤러는 보통:

- 요청 진입점 역할
- 입력 검증/해석
- 서비스 호출
- 응답 객체 반환

을 담당한다.

즉 컨트롤러는 웹 계층의 “입구”다.

### 코드에서 어떻게 나타나는가

```java
@RestController
public class OrderController {
    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @PostMapping("/orders")
    public OrderResponse createOrder(@RequestBody CreateOrderRequest request) {
        return orderService.createOrder(request);
    }
}
```

이 예시에서 컨트롤러는 비즈니스 로직을 전부 직접 구현하지 않고,

서비스 계층으로 넘기는 구조를 가진다.

### 이해 포인트

좋은 구조에서는 컨트롤러가 너무 많은 책임을 지지 않는다.

AI가 만든 코드에서 자주 보이는 문제는 컨트롤러가 서비스/도메인 책임까지 먹어버리는 것이다.

---

## 9. Service / Repository 등 비즈니스 계층이 실행된다

이제 애플리케이션 내부 비즈니스 로직으로 들어간다.

- Service: 비즈니스 규칙/유스케이스 조합
- Repository: 데이터 접근
- Domain/Object: 핵심 모델

이 단계는 Spring MVC 공통 파이프라인이라기보다, **애플리케이션 내부 구조**에 해당한다.

즉 모든 요청이 반드시 같은 내부 계층을 똑같이 거치는 건 아니지만, 보통 Spring 프로젝트에서는 이 구조를 많이 쓴다. 계층을 나눠 사용할 때의 표준 같은 너낌?

### 이해 포인트

여기서 DI, Bean, Container, 생성자 주입 같은 개념이 실제로 살아난다.

- Controller가 Service를 주입받고
- Service가 Repository를 주입받고
- 이 객체들을 Spring 컨테이너가 관리한다

즉 요청 흐름을 따라가다 보면, 상위 개념이 필요한 지점이 자연스럽게 나타난다.

---

## 10. Controller가 반환값을 돌려준다

비즈니스 로직이 끝나면 컨트롤러는 결과를 반환한다.

반환값은 여러 형태일 수 있다.

- 객체
- 문자열
- `ResponseEntity`
- 뷰 이름
- `ModelAndView`

여기서 중요한 건,

**반환값 자체가 곧바로 HTTP 응답이 되는 건 아니라는 점**이다.

그 반환값을 어떻게 HTTP 응답으로 바꿀지, 스프링이 다음 단계에서 처리한다.

---

## 11. 반환값을 어떤 방식으로 응답으로 바꿀지 결정한다

이 단계에서 스프링은 “이 반환값을 JSON으로 쓸까, 뷰를 렌더링할까, 다른 방식일까?”를 판단한다.

여기서 두 갈래로 나뉘는것이 중요하다.

---

## 12-A. REST / JSON 응답 경로: HttpMessageConverter

`@RestController`나 `@ResponseBody` 계열에서는 보통 반환 객체를 JSON으로 직렬화해서 응답 본문에 쓴다. 이때 핵심 역할을 하는 것이 **HttpMessageConverter**다.

### 코드에서 어떻게 나타나는가

```java
@RestController
public class MemberController {
    @GetMapping("/members/{id}")
    public MemberResponse getMember(@PathVariable Long id) {
        return new MemberResponse(...);
    }
}
```

이 경우 `MemberResponse` 객체가 바로 HTTP로 나가는 게 아니라, Spring이 적절한 MessageConverter를 골라서 JSON 문자열로 바꿔 응답 본문에 쓴다. 보통 Jackson 기반 컨버터가 많이 쓰인다.

### 요청에서도 쓰인다

이건 응답뿐 아니라 요청에서도 쓰인다.

- 요청 바디 JSON → Java 객체
- Java 객체 → 응답 바디 JSON

즉 양방향 변환 계층이라고 보면 된다.

---

## 12-B. SSR / View 경로: ViewResolver

만약 전통적인 MVC 방식으로 뷰 이름을 반환하면, Spring은 그 뷰 이름을 실제 템플릿/뷰 객체로 해석해서 렌더링한다.

이때 핵심 역할을 하는 것이 **ViewResolver**다. 예를 들어 컨트롤러가 `"member/detail"` 같은 뷰 이름을 반환하면, ViewResolver가 실제 템플릿 파일을 찾아서 HTML을 렌더링한다.

### 이해 포인트

즉 응답 처리도 단순하지 않고, 반환값의 의미를 해석하는 계층이 별도로 존재한다.

---

## 13. Interceptor가 컨트롤러 전후를 가로챌 수 있다

Interceptor는 Filter와 비슷해 보이지만, **Spring MVC 내부**에서 컨트롤러 전후를 제어한다는 점이 다르다.

보통 다음 시점에 개입할 수 있다.

- `preHandle()` : 컨트롤러 실행 전
- `postHandle()` : 컨트롤러 실행 후, 뷰 렌더링 전
- `afterCompletion()` : 요청 전체 완료 후

### 이해 포인트

- Filter: 더 바깥 서블릿 계층
- Interceptor: Spring MVC 내부 계층

이 차이는 꼭 구분해야 한다!!

---

## 14. 응답이 다시 Filter 체인을 지나간다

응답도 요청이 들어온 경로를 반대로 빠져나가며, 필터에서 후처리될 수 있다.

예:

- 응답 로깅
- 공통 응답 헤더 추가
- 보안 관련 마무리 작업

즉 Filter는 요청 전처리만이 아니라 응답 후처리에도 관여할 수 있다.

---

## 15. Tomcat이 최종 HTTP 응답을 클라이언트로 보낸다

마지막으로 서블릿 컨테이너가 준비된 응답을 실제 HTTP 메시지로 만들어 네트워크를 통해 클라이언트에게 보낸다.

이로써 요청-응답 한 사이클이 끝난다.

---

# 이 플로우에서 무엇을 중심으로 이해해야 할까?.?

## 1. Spring은 모든 걸 혼자 하지 않는다

가장 아래에는 아래의 것들이 존재한다.

- HTTP
- Tomcat
- Servlet Container

Spring은 그 위에서 **애플리케이션 구조와 요청 흐름을 조직**한다.

---

## 2. DispatcherServlet이 중앙 진입점이다

Spring MVC에서 가장 중요한 웹 구조 포인트는 **Front Controller = DispatcherServlet**이다.

모든 요청이 여기로 모이고, 여기서 분기된다.

---

## 3. Spring MVC는 “찾기”와 “실행”을 분리한다

- HandlerMapping = 누가 처리할지 찾기
- HandlerAdapter = 어떻게 실행할지 맞추기

이건 프레임워크 설계 품질을 보여주는 지점이기도 하다.

---

## 4. 애플리케이션 계층과 프레임워크 계층을 구분해야 한다

- Filter / DispatcherServlet / HandlerAdapter / MessageConverter

  → 프레임워크 구조

- Controller / Service / Repository

  → 애플리케이션 구조