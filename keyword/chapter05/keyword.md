# 개요 및 목차?

이번 키워드들은 다음 역할로 연결된다!!

| 키워드 | 코드/개념상 사용 방식 |
|---|---|
| record vs static class | Request DTO / Response DTO를 어떤 방식으로 만들지 결정 |
| Builder Pattern | DTO나 테스트 객체를 읽기 좋게 생성 |
| Generic | `ApiResponse<T>`처럼 응답 껍데기는 공통, 내부 타입은 API마다 다르게 처리 |
| Optional | Repository 조회 결과가 없을 수 있음을 타입으로 표현 |
| @RestControllerAdvice | 예외를 잡아 통일된 JSON 실패 응답으로 변환 |

# 1. Builder Pattern

## 한 줄 정의

**Builder Pattern은 객체 생성에 필요한 값을 이름 기반으로 단계적으로 설정한 뒤, 마지막에 `build()`로 객체를 완성하는 생성 패턴이다.**

---

## 코드/개념적으로 어떻게 사용되는가

Builder Pattern은 주로 **복잡한 객체를 생성하는 코드**에서 사용된다. Spring API 코드에서는 특히 Response DTO, 테스트 객체, 일부 Entity 생성에서 자주 보인다.

```text
Entity
→ Converter
→ Response DTO 생성
→ Builder Pattern 사용 가능
```

예를 들어 Service에서 조회한 `Member` 엔티티를 그대로 클라이언트에게 반환하지 않고, `MemberResponse` DTO로 바꿔서 반환한다고 하자. 이때 필드가 여러 개면 생성자보다 Builder가 더 읽기 좋다.

```java
public class MemberConverter {

    public static MemberResDTO.MyPageResponse toMyPageResponse(Member member) {
        return MemberResDTO.MyPageResponse.builder()
                .name(member.getName())
                .profileUrl(member.getProfileUrl())
                .email(member.getEmail())
                .phoneNumber(member.getPhoneNumber())
                .point(member.getPoint())
                .build();
    }
}
```

이 코드는 단순히 객체를 생성하는 코드가 아니라, **Entity의 어떤 값이 Response DTO의 어떤 필드로 이동하는지 보여주는 매핑 코드**가 된다!

---

## 왜 필요한가

생성자 방식은 필드가 적을 때는 간단하다.

```java
new MyPageResponse("kim", "kim@example.com", 1000);
```

하지만 필드가 많아지면 문제가 생긴다.

```java
new MyPageResponse(
        "kim",
        "https://profile.com/a.png",
        "kim@example.com",
        null,
        1000
);
```

이 코드는 다음 문제가 있다.

- 각 인자가 어떤 필드에 들어가는지 한눈에 보기 어렵다.
- 같은 타입의 필드가 여러 개 있으면 순서를 바꿔도 컴파일 에러가 나지 않을 수 있다.
- 선택 필드가 많아지면 생성자 오버로딩이 늘어난다.
- 코드 리뷰에서 값 매핑 오류를 찾기 어렵다.

Builder를 사용하면 필드명이 코드에 드러난다.

```java
MyPageResponse response = MyPageResponse.builder()
        .name("kim")
        .profileUrl("https://profile.com/a.png")
        .email("kim@example.com")
        .phoneNumber(null)
        .point(1000)
        .build();
```

---

## 생성자, Setter와의 차이

| 방식 | 특징 | 적합한 상황 |
|---|---|---|
| 생성자 | 짧고 단순하지만 인자 순서에 의존 | 필드가 적고 필수값이 명확한 객체 |
| Setter | 생성 후 값을 바꿀 수 있음 | 변경 가능한 객체, 프레임워크 바인딩 |
| Builder | 필드명을 드러내며 객체 생성 | 필드가 많거나 선택값이 많은 DTO, 테스트 객체 |

Setter 방식은 객체가 불완전한 상태로 존재할 수 있다.

```java
MyPageResponse response = new MyPageResponse();
response.setName("kim");
// 아직 email, point가 없는 중간 상태
```

Builder 방식은 보통 `build()`가 호출되는 시점에 완성된 객체를 만든다.

---

## Lombok `@Builder`

Spring Boot 프로젝트에서는 직접 Builder 클래스를 구현하기보다 Lombok의 `@Builder`를 자주 사용한다.

```java
@Getter
@Builder
public class MyPageResponse {
    private String name;
    private String profileUrl;
    private String email;
    private String phoneNumber;
    private Integer point;
}
```

Lombok의 `@Builder`는 builder API를 자동 생성해주는 기능이다. 실무에서는 DTO, 테스트 fixture, 설정성 객체에 자주 사용된다.

---

## 실무 선택 기준

Builder를 무조건 쓰는 것이 좋은 것은 아니다.

### Builder가 잘 맞는 경우

- Response DTO 필드가 많다.
- 선택 필드가 많다.
- Entity → DTO 변환 코드에서 필드 매핑을 명확히 보여주고 싶다.
- 테스트 데이터를 만들 때 일부 필드만 바꿔가며 객체를 생성한다.
- 생성자 인자 순서 실수를 줄이고 싶다.

### Builder가 과한 경우

- 필드가 1~2개뿐이다.
- 단순 요청 DTO이고 record로 충분히 표현 가능하다.
- 객체 생성 방식이 너무 단순해서 Builder가 오히려 코드를 길게 만든다.

---

최근 Spring Boot 프로젝트에서는 DTO를 만들 때 크게 두 흐름이 같이 쓰인다.

```text
단순 Request DTO
→ record 사용

필드가 많거나 Builder가 필요한 Response DTO
→ static class + Lombok @Builder 사용
```

또한 테스트 코드에서는 Builder가 매우 자주 사용된다. 테스트에서 중요한 것은 “이 테스트가 어떤 상태의 객체를 만들고 있는가”가를 증명하는 것이라고 생각한다.

```java
Member member = Member.builder()
        .id(1L)
        .name("test-user")
        .email("test@example.com")
        .point(1000)
        .build();
```

단, JPA Entity에 Builder를 사용할 때는 주의가 필요하다. 연관관계, 기본값, 컬렉션 초기화가 깨지면 런타임 버그가 생길 수 있다. 컬렉션 필드는 `new ArrayList<>()`로 초기화하거나 Lombok 사용 시 `@Builder.Default`를 고려해야 한다.

```java
@Builder.Default
private List<Review> reviews = new ArrayList<>();
```

---

## 흔한 오해

### 오해 1. Builder를 쓰면 자동으로 불변 객체가 된다

아니다. Builder는 객체 생성 방식일 뿐이다. Setter가 열려 있으면 생성 후에도 값은 바뀔 수 있다.

```java
@Getter
@Setter
@Builder
public class MemberResponse {
    private String name;
}
```

이 객체는 Builder로 만들 수 있지만 불변 객체는 아니다.

### 오해 2. DTO에는 무조건 Builder가 필요하다

아니다. 단순 Request DTO는 record가 더 간결할 수 있다.

```java
public record LoginRequest(
        String email,
        String password
) {
}
```

### 오해 3. Builder는 비즈니스 로직을 담는 곳이다

아니다. Builder는 객체 생성 책임을 가진다. 검증을 약간 넣을 수는 있지만, 복잡한 비즈니스 규칙은 Service나 도메인 객체 쪽에서 다루는 것이 좋다.

---

## 간단 정리!?

```text
Request DTO가 단순하다
→ record 우선 고려

Response DTO의 필드가 많다
→ static class + @Builder 고려

Entity 생성이다
→ Builder 사용 가능하지만 연관관계, 기본값, JPA 요구사항 확인

테스트 객체다
→ Builder 적극 활용 가능
```

---

# 2. record vs static class

## 한 줄 정의

**record와 static class는 Spring API에서 Request DTO와 Response DTO를 정의하는 대표적인 두 방식이다.**

---

## 코드/개념적으로 어떻게 사용되는가

Controller는 클라이언트의 JSON 요청을 Java 객체로 받고, 처리 결과도 Java 객체로 만들어 JSON으로 반환한다. 이때 DTO를 `record`로 만들지, `static class`로 만들지 선택한다.

```text
Request Body(JSON)
→ Request DTO

Response DTO
→ JSON Response
```

이 DTO를 만들 때 `record`를 쓸 수도 있고, DTO 묶음 클래스 안에 `static class`를 만들 수도 있다.

---

## record

record는 Java에서 **데이터 전달용 객체를 간결하게 만들기 위한 문법**이다. Java 공식 문서에서는 record를 고정된 값 집합을 담는 얕은 불변 carrier로 설명한다.

```java
public record LoginRequest(
        String email,
        String password
) {
}
```

위 코드는 컴파일 시 다음 요소를 자동으로 가진다.

- 생성자
- 필드 접근자: `email()`, `password()`
- `equals()`
- `hashCode()`
- `toString()`

일반 class DTO였다면 직접 작성하거나 Lombok으로 생성했을 코드들이 record에서는 기본 제공된다.

---

## static class DTO

static class DTO는 보통 다음처럼 관련 DTO들을 하나의 외부 클래스 안에 묶는 방식이다.

```java
public class MemberReqDTO {

    public record MyPageRequest(
            Long id
    ) {
    }
}
```

또는 Builder가 필요한 응답 DTO를 이렇게 만들 수 있다.

```java
public class MemberResDTO {

    @Getter
    @Builder
    public static class MyPageResponse {
        private String name;
        private String profileUrl;
        private String email;
        private String phoneNumber;
        private Integer point;
    }
}
```

여기서 `static`을 붙이는 이유는 내부 DTO가 바깥 DTO 클래스의 인스턴스에 의존하지 않게 하기 위해서다.

```text
MemberResDTO.MyPageResponse
```

처럼 이름공간을 묶는 효과가 있다.

---

## 비교

| 기준             | record                   | static class             |
| -------------- | ------------------------ | ------------------------ |
| 코드 양           | 매우 적음                    | 상대적으로 많음                 |
| 불변성            | 기본적으로 강함                 | 설계에 따라 다름                |
| 접근자            | `name()` 형태              | 보통 `getName()` 형태        |
| Setter         | 없음                       | 만들 수 있음                  |
| Builder        | 직접 지원하지 않음, Lombok 조합 가능 | Lombok `@Builder`와 자주 사용 |
| 요청 DTO         | 매우 잘 맞음                  | 가능                       |
| 응답 DTO         | 단순 응답에 적합                | 복잡한 응답에 적합               |
| 프레임워크/라이브러리 호환 | 대부분 좋아졌지만 일부 도구에서 확인 필요  | 가장 전통적이고 유연함             |

---

## 선택 기준

### record가 적합한 경우

- 요청 DTO가 단순하다.
- 필드가 생성 후 바뀔 필요가 없다.
- 보일러플레이트 코드를 줄이고 싶다.
- DTO가 순수하게 데이터 전달 역할만 한다.

예시:

```java
public class MissionReqDTO {

    public record MissionSuccessRequest(
            Long userMissionId
    ) {
    }
}
```

### static class가 적합한 경우

- 응답 DTO 필드가 많다.
- Builder로 생성하고 싶다.
- Swagger, Validation, Jackson, Lombok 어노테이션을 다양하게 붙여야 한다.
- 중첩 DTO, 정적 팩토리 메서드, 커스텀 생성 로직이 필요하다.

예시:

```java
public class MissionResDTO {

    @Getter
    @Builder
    public static class MissionSuccessResult {
        private Long missionId;
        private String status;
        private String message;
    }
}
```

---

## 최신/현업 관점

최근 Java 17 이상을 사용하는 Spring Boot 프로젝트에서는 record 사용이 자연스러워졌다. Spring Boot 3 계열은 Java 17 이상을 기반으로 사용되는 경우가 많기 때문에, 단순 DTO에 record를 쓰는 팀이 늘었다.

다만 모든 DTO를 record로 통일하는 것이 항상 정답은 아니다. 실무에서는 아래처럼 혼합하는 경우가 많다.

```text
Request DTO
→ record 선호

단순 Response DTO
→ record 가능

복잡한 Response DTO
→ static class + @Builder 선호

JPA Entity
→ record 사용하지 않음
```

JPA Entity는 프록시, 기본 생성자, 변경 감지, 지연 로딩 등 ORM 메커니즘과 관련이 있으므로 일반적으로 record로 만들지 않는다.

---

## 흔한 오해

### 오해 1. record는 DTO의 완전한 상위호환이다

아니다. record는 간결하지만 모든 상황에 적합하지는 않다. 복잡한 생성 제어, 일부 프레임워크 요구사항, Builder 중심 생성이 필요한 경우 class DTO가 더 편할 수 있다.

### 오해 2. static class DTO는 구식이다

아니다. static class DTO는 여전히 실무에서 많이 쓰인다. 특히 응답 DTO가 복잡하거나 Lombok Builder를 적극적으로 사용할 때 유용하다.

### 오해 3. record의 접근자는 getter다

개념적으로는 getter처럼 값을 읽지만 메서드 이름은 `getName()`이 아니라 `name()`이다.

```java
request.email();   // record 접근자
```

---

## 간단정리!?

```text
단순 요청 DTO
→ record

응답 필드가 적고 단순함
→ record 가능

응답 필드가 많고 builder가 필요함
→ static class + @Builder

JPA Entity
→ 일반 class
```

---

# 3. Generic

## 한 줄 정의

**Generic은 클래스, 인터페이스, 메서드에서 사용할 타입을 미리 고정하지 않고, 사용하는 시점에 지정하게 하는 Java 문법이다.**

---

## 코드/개념적으로 어떻게 사용되는가

Generic은 **공통 구조는 유지하면서 내부 타입만 바꾸고 싶을 때** 사용된다. Spring API 코드에서는 특히 `ApiResponse<T>`, `JpaRepository<Entity, ID>`, `Optional<T>`, `List<T>` 같은 타입에서 자주 보인다.

API 응답의 공통 껍데기는 같다.

```json
{
  "isSuccess": true,
  "code": "COMMON200",
  "message": "성공입니다.",
  "result": {}
}
```

하지만 `result` 안에 들어가는 데이터는 API마다 다르다.

```text
회원 조회 API
→ result = MemberResponse

미션 성공 요청 API
→ result = MissionSuccessResult

삭제 API
→ result = null 또는 Void
```

이 문제를 해결하기 위해 `ApiResponse<T>`를 사용한다.

```java
public class ApiResponse<T> {

    private Boolean isSuccess;
    private String code;
    private String message;
    private T result;
}
```

여기서 `T`는 “나중에 정해질 타입”이다.

---

## 왜 필요한가

Generic 없이 `Object`를 쓰면 모든 타입을 담을 수는 있다.

```java
public class ApiResponse {
    private Object result;
}
```

하지만 이 방식은 타입 안정성이 약하다. 값을 꺼낼 때 어떤 타입인지 개발자가 직접 알고 있어야 한다.

Generic을 쓰면 컴파일 시점에 타입을 더 명확히 표현할 수 있다.

```java
ApiResponse<MemberResponse> response;
ApiResponse<MissionResponse> response;
ApiResponse<Void> response;
```

이 코드는 다음 의미를 드러낸다.

```text
이 ApiResponse의 result에는 MemberResponse가 들어간다.
이 ApiResponse의 result에는 MissionResponse가 들어간다.
이 ApiResponse는 result가 없다.
```

---

## 코드에서는 어떻게 보이는가

### 공통 응답 객체

```java
@Getter
@AllArgsConstructor
public class ApiResponse<T> {

    private Boolean isSuccess;
    private String code;
    private String message;
    private T result;

    public static <T> ApiResponse<T> onSuccess(
            BaseSuccessCode code,
            T result
    ) {
        return new ApiResponse<>(
                true,
                code.getCode(),
                code.getMessage(),
                result
        );
    }

    public static <T> ApiResponse<T> onFailure(
            BaseErrorCode code,
            T result
    ) {
        return new ApiResponse<>(
                false,
                code.getCode(),
                code.getMessage(),
                result
        );
    }
}
```

### Controller 사용 예시

```java
@GetMapping("/api/users/me")
public ApiResponse<MemberResDTO.MyPageResponse> getMyPage() {
    MemberResDTO.MyPageResponse result = memberService.getMyPage();
    return ApiResponse.onSuccess(MemberSuccessCode.MEMBER_FOUND, result);
}
```

### result가 없는 경우

```java
@DeleteMapping("/api/reviews/{reviewId}")
public ApiResponse<Void> deleteReview(@PathVariable Long reviewId) {
    reviewService.deleteReview(reviewId);
    return ApiResponse.onSuccess(ReviewSuccessCode.REVIEW_DELETED, null);
}
```

---

## 실무에서 중요한 점

### 1. API 응답 구조를 안정적으로 유지한다

`ApiResponse<T>`를 사용하면 모든 API가 같은 구조를 유지한다.

```text
isSuccess
code
message
result
```

이 구조는 프론트엔드와 협업할 때 중요하다. 프론트엔드는 성공/실패 여부, 메시지, 실제 데이터를 예측 가능한 위치에서 읽을 수 있다.

### 2. 공통 구조와 개별 데이터를 분리한다

Generic은 다음 분리를 가능하게 한다.

```text
공통 응답 정책
→ ApiResponse<T>

API별 실제 데이터
→ T
```

즉, `ApiResponse`는 모든 API가 공유하고, `T`만 API마다 달라진다.

### 3. Swagger 문서화와도 관련 있다

응답 타입을 `ApiResponse<?>`로만 쓰면 Swagger에서 구체적인 result 타입이 흐릿하게 보일 수 있다. 가능하면 Controller 반환 타입에 구체적인 DTO 타입을 적는 편이 문서화에 좋다.

```java
public ApiResponse<MemberResDTO.MyPageResponse> getMyPage()
```

이렇게 쓰면 API 문서를 읽는 사람이 result 구조를 이해하기 쉽다.

---

## 흔한 오해

### 오해 1. `T`는 특별한 문법 이름이다

아니다. `T`는 관례적으로 Type을 의미할 뿐이다. 이름은 바꿀 수 있다.

```java
public class ApiResponse<Result> {
    private Result result;
}
```

다만 Java 관례상 `T`, `E`, `K`, `V` 등을 많이 쓴다.

### 오해 2. Generic은 런타임에도 타입 정보를 완벽히 유지한다

Java Generic은 타입 소거가 있다. 컴파일 시점에는 타입 안정성을 주지만, 런타임에는 일부 타입 정보가 지워진다. 지금 단계에서는 깊게 몰라도 되지만, 나중에 JSON 역직렬화나 리플렉션을 다룰 때 다시 중요해진다.

### 오해 3. `ApiResponse<?>`가 항상 가장 좋다

`?`는 “정확한 타입을 모른다”는 의미에 가깝다. Controller 응답 타입을 구체적으로 적을 수 있다면 구체 타입을 쓰는 편이 좋다.

---

## 간단 정리!?

```text
공통 응답 객체
→ ApiResponse<T>

조회 API
→ ApiResponse<ResponseDTO>

삭제/상태 변경 API 중 result가 필요 없음
→ ApiResponse<Void>

타입을 모르는 공통 처리
→ ApiResponse<?> 가능
```

---

# 4. Optional

## 한 줄 정의

**Optional은 값이 있을 수도 있고 없을 수도 있는 상황을 명시적으로 표현하는 Java 컨테이너 객체이다.**

---

## 코드/개념적으로 어떻게 사용되는가

Optional은 **값이 없을 수 있는 반환값**을 표현할 때 사용된다. Spring/JPA 코드에서는 Repository에서 데이터를 조회할 때 자주 등장한다.

```java
Optional<Member> findById(Long id);
```

DB에서 `id`로 회원을 조회했는데, 해당 회원이 없을 수도 있다. 이 가능성을 `null` 대신 `Optional<Member>`로 표현한다.

```text
memberRepository.findById(id)
→ Optional<Member>
```

---

## 왜 필요한가

Repository 조회 결과가 없을 때 `null`을 반환하면 사용하는 쪽에서 null 체크를 빼먹을 수 있다.

```java
Member member = memberRepository.findById(id); // null 가능
member.getName(); // NullPointerException 가능
```

Optional을 사용하면 “없을 수 있음”이 타입에 드러난다.

```java
Optional<Member> member = memberRepository.findById(id);
```

이제 개발자는 값을 바로 꺼내기 전에 “없을 때 어떻게 할 것인가”를 결정해야 한다.

---

## 코드에서는 어떻게 보이는가

Spring Service 코드에서는 `orElseThrow()`와 가장 자주 연결된다.

```java
Member member = memberRepository.findById(memberId)
        .orElseThrow(() -> new MemberException(MemberErrorCode.MEMBER_NOT_FOUND));
```

이 코드는 다음 흐름이다.

```text
findById(memberId)
→ Optional<Member> 반환
→ 값이 있으면 Member 꺼냄
→ 값이 없으면 MemberException 발생
→ @RestControllerAdvice가 실패 응답으로 변환
```

---

## 자주 쓰는 메서드

```java
Optional.empty();
Optional.of(value);
Optional.ofNullable(value);
```

| 메서드 | 의미 |
|---|---|
| `Optional.empty()` | 빈 Optional 생성 |
| `Optional.of(value)` | null이 아닌 값을 담음. null이면 예외 |
| `Optional.ofNullable(value)` | null이면 빈 Optional 생성 |

값을 처리할 때는 다음 메서드를 자주 쓴다.

| 메서드 | 의미 |
|---|---|
| `orElse(defaultValue)` | 값이 없으면 기본값 반환 |
| `orElseGet(supplier)` | 값이 없으면 supplier 실행 |
| `orElseThrow(exceptionSupplier)` | 값이 없으면 예외 발생 |
| `ifPresent(consumer)` | 값이 있으면 실행 |
| `map(function)` | 값이 있으면 변환 |
| `filter(predicate)` | 조건에 맞으면 유지, 아니면 empty |

---

## 실무에서 중요한 점

### 1. Repository 조회 실패를 예외 흐름으로 연결한다

Spring API에서는 “없는 회원을 조회했다”는 상황을 다음처럼 처리하는 경우가 많다.

```text
조회 결과 없음
→ Optional.empty()
→ orElseThrow()
→ Custom Exception
→ @RestControllerAdvice
→ ApiResponse 실패 응답
```

이 흐름을 이해하면 Optional과 전역 예외 처리의 연결이 보인다.

### 2. Optional은 주로 반환 타입에 사용한다

Java 공식 문서에서도 Optional은 “결과 없음”을 표현할 명확한 필요가 있는 메서드 반환 타입으로 주로 의도되었다고 설명한다. 따라서 필드나 파라미터에 무분별하게 사용하는 것은 피하는 편이 좋다.

좋은 예:

```java
Optional<Member> findByEmail(String email);
```

애매한 예:

```java
private Optional<String> nickname;
```

필드에서는 보통 그냥 nullable 필드로 두고, API 응답에서는 명시적으로 null을 허용할지 정책을 정한다.

### 3. `get()`을 바로 쓰지 않는다

```java
Member member = optionalMember.get();
```

값이 없으면 `NoSuchElementException`이 발생한다. Optional을 쓰면서도 null 처리와 비슷한 위험을 다시 만드는 셈이다.

더 나은 방식:

```java
Member member = optionalMember
        .orElseThrow(() -> new MemberException(MemberErrorCode.MEMBER_NOT_FOUND));
```

---

## `orElse`와 `orElseGet` 차이

둘 다 값이 없을 때 대체값을 제공하지만, 평가 시점이 다르다.

```java
String result = optional.orElse(createDefaultValue());
```

`orElse()`는 Optional에 값이 있어도 `createDefaultValue()`가 먼저 실행될 수 있다.

```java
String result = optional.orElseGet(() -> createDefaultValue());
```

`orElseGet()`은 값이 없을 때만 람다가 실행된다.

비용이 있는 기본값 생성이라면 `orElseGet()`이 더 적합하다.

---

## 흔한 오해

### 오해 1. Optional을 쓰면 NullPointerException이 완전히 사라진다

아니다. Optional 변수를 null로 만들 수도 있고, `get()`을 잘못 쓰면 다른 예외가 발생한다.

```java
Optional<Member> member = null; // 이런 코드는 피해야 함
```

### 오해 2. 모든 nullable 값을 Optional로 감싸야 한다

아니다. Optional은 주로 반환 타입에서 “없을 수 있음”을 표현할 때 유용하다. DTO 필드, Entity 필드, 메서드 파라미터에 남용하면 오히려 코드와 직렬화가 복잡해질 수 있다.

### 오해 3. Optional은 예외 처리를 대체한다

아니다. Optional은 값이 없을 가능성을 표현한다. 그 상황을 기본값으로 처리할지, 예외로 처리할지는 별도의 설계 결정이다.

---

## 갇단 정리!?

```text
Repository findById, findByEmail
→ Optional<T> 반환

조회 결과가 반드시 필요함
→ orElseThrow(CustomException)

조회 결과가 없어도 기본값 가능
→ orElse 또는 orElseGet

Optional.get()
→ 가급적 사용하지 않음
```

---

# 5. @RestControllerAdvice

## 한 줄 정의

**`@RestControllerAdvice`는 여러 Controller에서 발생한 예외를 한 곳에서 잡아 JSON 응답으로 처리하게 해주는 전역 예외 처리용 어노테이션이다.**

---

## 코드/개념적으로 어떻게 사용되는가

Spring REST API에서 중요한 문제 중 하나는 **정상 응답과 실패 응답의 형식을 일관되게 유지하는 것**이다.

정상 응답은 Controller에서 `ApiResponse.onSuccess(...)`로 감싼다.

```java
return ApiResponse.onSuccess(MemberSuccessCode.MEMBER_FOUND, result);
```

하지만 예외가 발생하면 Spring 기본 에러 응답이 나갈 수 있다. 그러면 우리가 정한 응답 구조가 깨진다.

`@RestControllerAdvice`는 이 문제를 해결한다.

```text
Exception 발생
→ @RestControllerAdvice가 감지
→ @ExceptionHandler 메서드 실행
→ ApiResponse.onFailure(...) 반환
```

---

## 왜 필요한가

Controller마다 try-catch를 작성하면 코드가 지저분해진다.

```java
@GetMapping("/api/users/{id}")
public ApiResponse<?> getUser(@PathVariable Long id) {
    try {
        ...
    } catch (MemberException e) {
        ...
    }
}
```

이 방식은 다음 문제가 있다.

- Controller마다 예외 처리 코드가 반복된다.
- API마다 실패 응답 형식이 달라질 수 있다.
- 비즈니스 로직보다 에러 처리 코드가 더 눈에 띈다.
- 새로운 예외가 추가될 때 수정 지점이 많아진다.

전역 예외 처리를 사용하면 Controller는 정상 흐름에 집중하고, 실패 흐름은 Advice에서 관리할 수 있다.

---

## 코드에서는 어떻게 보이는가

```java
@RestControllerAdvice
public class GeneralExceptionAdvice {

    @ExceptionHandler(ProjectException.class)
    public ResponseEntity<ApiResponse<Void>> handleProjectException(
            ProjectException e
    ) {
        BaseErrorCode errorCode = e.getErrorCode();

        return ResponseEntity
                .status(errorCode.getStatus())
                .body(ApiResponse.onFailure(errorCode, null));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiResponse<String>> handleUnknownException(
            Exception e
    ) {
        BaseErrorCode errorCode = GeneralErrorCode.INTERNAL_SERVER_ERROR;

        return ResponseEntity
                .status(errorCode.getStatus())
                .body(ApiResponse.onFailure(errorCode, e.getMessage()));
    }
}
```

핵심은 두 가지다.

```text
@ExceptionHandler(ProjectException.class)
→ ProjectException과 그 하위 예외를 처리

ResponseEntity.status(...)
→ HTTP 상태 코드 지정

ApiResponse.onFailure(...)
→ 우리가 정한 JSON 실패 응답 구조로 변환
```

---

## 예외 처리 흐름

```text
Service
→ throw new MemberException(MemberErrorCode.MEMBER_NOT_FOUND)

MemberException
→ ProjectException 상속

@RestControllerAdvice
→ ProjectException 처리 메서드 실행

ApiResponse.onFailure(...)
→ JSON 실패 응답 반환
```

예시 응답:

```json
{
  "isSuccess": false,
  "code": "MEMBER404_1",
  "message": "회원을 찾을 수 없습니다.",
  "result": null
}
```

---

## `@ControllerAdvice`와 차이

Spring 공식 문서 기준으로 `@RestControllerAdvice`는 `@ControllerAdvice`와 `@ResponseBody`가 합쳐진 조합이다. 즉, `@ExceptionHandler` 메서드의 반환값이 View 이름으로 해석되는 것이 아니라 응답 본문으로 직렬화된다.

| 어노테이션 | 주 사용처 |
|---|---|
| `@ControllerAdvice` | MVC View 기반 공통 처리 |
| `@RestControllerAdvice` | REST API JSON 응답 기반 공통 처리 |

REST API 서버에서는 보통 `@RestControllerAdvice`가 더 자연스럽다.

---

## 적용 범위 제한

기본적으로 `@RestControllerAdvice`는 모든 Controller에 적용된다. 필요하면 특정 패키지나 특정 Controller에만 적용할 수 있다.

```java
@RestControllerAdvice(basePackages = "com.example.api")
public class ApiExceptionAdvice {
}
```

```java
@RestControllerAdvice(assignableTypes = MemberController.class)
public class MemberExceptionAdvice {
}
```

다만 작은 프로젝트나 초기 학습 단계에서는 전역 Advice 하나로 시작하는 것이 이해하기 쉽다.

---

## 실무에서 중요한 점

### 1. ErrorCode enum과 함께 사용한다

실무에서는 예외 메시지를 문자열로 흩뿌리지 않고, ErrorCode enum으로 관리하는 경우가 많다.

```java
public enum MemberErrorCode implements BaseErrorCode {
    MEMBER_NOT_FOUND(HttpStatus.NOT_FOUND, "MEMBER404_1", "회원을 찾을 수 없습니다.");
}
```

이렇게 하면 에러 코드, HTTP 상태, 메시지를 한 곳에서 관리할 수 있다.

### 2. 클라이언트에게 내부 예외 메시지를 그대로 노출하지 않는다

```java
e.getMessage()
```

를 그대로 반환하면 DB 오류, 내부 클래스명, 보안상 민감한 정보가 노출될 수 있다.

현업에서는 보통 사용자에게 보여줄 메시지와 서버 로그 메시지를 분리한다.

```text
Client response:
"일시적인 서버 오류가 발생했습니다."

Server log:
NullPointerException at MemberService.java:42
```

### 3. Validation 예외도 여기서 처리한다

나중에 `@Valid`, `@NotNull`, `@Size` 등을 쓰면 검증 실패 예외가 발생한다. 이런 예외도 `@RestControllerAdvice`에서 처리해 같은 응답 형식으로 맞춘다.

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<ApiResponse<?>> handleValidationException(
        MethodArgumentNotValidException e
) {
    ...
}
```

### 4. 관측 가능성과 연결된다

현업에서는 예외를 단순히 클라이언트 응답으로 바꾸는 것에서 끝내지 않는다. 로그, 모니터링, 알림과 연결한다.

```text
Exception 발생
→ Advice에서 응답 생성
→ 로그 기록
→ 필요 시 Sentry, CloudWatch, Datadog 등으로 전송
```

즉, `@RestControllerAdvice`는 API 응답 통일뿐 아니라 운영 환경에서 문제를 추적하는 시작점이 되기도 한다.

---

## 흔한 오해

### 오해 1. `@RestControllerAdvice`가 예외를 없애준다

아니다. 예외를 없애는 것이 아니라, 예외를 잡아서 클라이언트가 이해할 수 있는 응답으로 바꿔준다.

### 오해 2. 모든 예외를 `Exception.class` 하나로 처리하면 충분하다

학습 단계에서는 가능하지만 실무에서는 좋지 않다. 비즈니스 예외, 검증 예외, 인증/인가 예외, 알 수 없는 서버 예외를 구분해야 한다.

```text
Business Exception
Validation Exception
Authentication Exception
Authorization Exception
Unknown Exception
```

### 오해 3. HTTP Status와 내부 code는 같은 것이다

다르다.

```text
HTTP Status
→ HTTP 표준 수준의 결과

내부 code
→ 프로젝트 도메인 수준의 세부 결과
```

예를 들어 둘 다 404일 수 있지만, 내부 코드는 다르게 줄 수 있다.

```text
MEMBER404_1: 회원을 찾을 수 없음
STORE404_1: 가게를 찾을 수 없음
MISSION404_1: 미션을 찾을 수 없음
```

---

## 간단정리!?

```text
정상 응답
→ Controller에서 ApiResponse.onSuccess(...)

도메인 예외
→ Service에서 CustomException throw

실패 응답
→ @RestControllerAdvice에서 ApiResponse.onFailure(...)

알 수 없는 예외
→ 서버 로그에는 자세히, 클라이언트에는 일반 메시지
```
