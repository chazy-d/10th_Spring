# 1. RESTful API란?

## 1. HTTP 란?

RESTful API를 이해하려면 먼저 HTTP가 무엇인지, 그리고 웹에서 리소스를 어떻게 다루는지를 알아야한다.

HTTP는 웹에서 자원을 가져오고 주고받기 위한 프로토콜이며, 클라이언트와 서버가 요청과 응답을 주고받는 구조를 가진다.

![](https://raw.githubusercontent.com/chazy-d/md-images/main/uploads/20260325215220379.png)

Rest는 이러한 HTTP 통신 규약 위에서 리소스를 일관된 방식으로 다루기 위한 아키텍쳐 스타일이다. 여기서 리소스는 사용자, 게시글, 주문처럼 시스템이 관리하는 대상이다. 즉 REST는 함수를 호출하듯 API를 설계하는 방식보다는 대상을 중심으로 URI를 만들고 HTTP 메서드로 그 대상에 대한 행위를 표현하는 방식에 가깝다!

## 2. REST 원칙

### 1. 클라이언트-서버 분리

클라이언트와 서버의 역할을 분리한다.

클라이언트는 요청과 UI에 집중하고, 서버는 데이터와 비즈니스 로직 처리에 집중한다.

### 2. 무상태(Stateless)

각 요청은 독립적이어야 하며, 서버가 이전 요청의 맥락을 자동으로 기억하는 구조에 의존하지 않는 방향을 지향한다.

이건 앞에서 설명한 무상태와 직접 연결된다.

### 3. 리소스 중심

시스템의 대상을 리소스로 보고, URI로 식별한다.

예를 들면 사용자, 게시글, 주문 같은 것들이 리소스가 된다.

### 4. HTTP 메서드의 의미 활용

리소스에 대해 무엇을 할지는 GET, POST, PUT, DELETE, PATCH 같은 HTTP 메서드로 표현한다.

### 5. 일관된 인터페이스(Uniform Interface)

API를 사용하는 쪽이 예측 가능하도록, 자원 식별 방식과 요청 방식이 일관되어야 한다.

## 3. 여러가지 예시

예를 들어 사용자를 다루는 API가 있다면 다음처럼 설계 가능하다.

- `GET /users` : 사용자 목록 조회
- `GET /users/1` : 1번 사용자 조회
- `POST /users` : 사용자 생성
- `PUT /users/1` : 1번 사용자 전체 수정
- `DELETE /users/1` : 1번 사용자 삭제

이런 구조가 RESTful 하다고 불리는 이유는, URL이 행위보다 리소스를 중심으로 구성되고, 실제 동작의 의미는 HTTP 메서드가 담당하기 때문이다. 즉 `/getUser`, `/createUser`처럼 행위를 URL에 직접 넣기보다는 `/users`, `/users/{id}`처럼 자원을 나타내고, `GET`, `POST`, `DELETE` 등의 메서드로 의도를 표현한다. HTTP 메서드는 요청의 목적과 기대되는 의미를 나타내도록 정의되어 있다.

다만 RESTful API는 "공식 정답이 있어 딱 떨어지는 하나의 문법"아라기 보다는 위의 REST 원칙을 어느 정도 충실히 반영해 설계된 API를 가리킨다고 생각하면 좋을 것 같다. 어차피 사람마다 같은 기능이여도 설계하는 형태가 다르며 실무에서 완전한 REST 제약을 모두 만족하지 않더라도, 중요한 **자원 중심 URL**과 **HTTP 메서드 의미**를 잘 살린 API를 관용적으로 REST API또는 RESTful API라고 부르는 경우가 많다고 생각한다.

예를들어

```sql
// 1. 행위(action)이 URL에 들어간 경우
POST /orders/1/cancel

// 2. 엄밀히 보면 덜 REST하지만 많이 사용되는 형태
POST /login
POST auth/logout
POST /signup

// 3. RPC 스타일 -> 진짜 REST 하지 않은 경우
GET /getUser?id=1
POST /createUser
```

1번은 완전히 자원 중심이라고 하기에는 행위가 URL에 섞인 형태이다.

2번은 매우 흔하게 사용되는데 형태인데 다시 생각해보면 login, logout과 같은 것들은 리소스라기 보다는 행위에 가깝다.

이와 같이 완벽한 REST는 아니더라도 REST스러움을 보이며 사람들도 크게 거슬려 하지 않는 경우 관용적으로 REST API의 예시라고 할 수 있다.

3번을 본다면 지금까지 본 API들과는 다르게 확연히 URL이 자원보다 행위를 드러내는 특성을 가지는데 이는 HTTP 메서드가 의미 전달의 역할보다 단순 함수 호출 수단처럼 쓰였기 때문이다.

[https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overvie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview)w

# 2. 멱등성이란?

## 1. 멱등성(Idempotency)이란?

멱등성은 같은 요청을 여러번 반복해서 보내더라도, 서버의 최조 상태가 한번 보냈을 때와 같게 유지되는 성질을 말한다. 이 개념은 HTTP 메서드의 성질을 이해할 때 중요하다.

예를들어 `DELETE /users/1`요청을 생각해본다면, 첫번째 요청에서 1번 사용자가 삭제되었다면, 같은 요청을 다시 보내더라도 결과적으로 1번 사용자가 없다라는 최종 상태는 변하지 않는다. 이런 경우 해당 요청을 멱등적이라 한다.

`PUT`도 비슷하게 같은 전체 데이터를 여러번 덮어쓴다고 해도 최종 상태가 동일하다면 멱등적이다.

하지만 `POST /orders`처럼 호출할 때마다 새로운 주문이 추가된다면 같은 요청을  여러번 보낼때 각각의 주문이 달라질 수 있으므로 일반적으로 멱등적이지 않다.  이와 같이 몇몇 안전한 요청 메서드에서 유효함을 알 수 있다.

## 2. 요점

중요한 것은 **응답**이 완전히 똑같아야 한다는 뜻이 아니라, **서버의 최종 상태**가 같아야 한다는 점이다.

예를들어 첫 번째 DELETE는 성공하여 `200 OK` or `204 No content`를 반환할 것이다.두 번쨰 DELETE는 이미 대상이 없으므로 [`404 Not Found`](https://www.youtube.com/watch?v=zhHB4dZTChw)를 반환할 수도 있다. 이때 응답 코드는 달라질 수 있지만 최종 상태는 둘 다 "해당 리소스가 없음"으로 같게 되기 때문에 멱등성의 개념에 만족한다. 이점 때문에 멱등성은 **반복 요청에 안전한가**를 이해하는데 중요하다.!

## 3. 왜 배움???

사실 이런 멱등성이라는 특성이 어디에 쓰이게 되는지 잘 와닿지 않는다. 단순히 HTTP 메서드의 성질을 외우기 위한 개념이 아니다.

실제 네트워크 환경을 생각해보면 요청이 한 번만 정확히 전달되고 한 번만 정확히 처리된다고 보장하기 어렵다. 요청을 보냈는데 응답이 늦게 오거나, 도중에 연결이 끊기거나, 타임아웃이 발생하거나, 중간에 실패가 발생할 수 있다. 이때 클라이언트나 프록시, 라이브러리, 네트워크 계층은 요청을 다시 보내야하나? 라는 판단을 하게 된다.

**이 지점에서 멱등성이 중요하다.** 어떤 요청이 멱등적이라면 같은 요청을 여러 번 보내더라도 서버의 최종 상태가 같게 유지되므로 재시도에 아주 훨씬 안전하다. 반대로 멱등적이지 않은 요청은 재시도했을 때 의도하지 않은 중복 생성, 중복 결제, 중복 주문 등의 문제가 생기기 아주 쉽다.

멱등적인 요청은 여러 번 실행되어도 서버의 최종 상태가 같게 유지되므로 자동 재시도나 오류 복구에 유리하다. 반면 멱등적이지 않은 요청은 중복 생성이나 중복 처리 같은 문제를 일으킬 수 있으므로, 클라이언트는 원래 요청이 적용되지 않았음을 확실히 알 수 없거나 실제 의미상 멱등적이라는 근거가 없는 한 자동 재시도해서는 안 된다. **따라서 멱등성은 HTTP 메서드 선택의 형식적 규칙이 아니라, 네트워크 실패 상황에서 API를 안전하고 예측 가능하게 만들기 위한 핵심 설계 원리라고 볼 수 있다.**

따라서 위의 `Post`메서드. 예를들어 생성 요청은 HTTP 의미상으로는 보통 비멱등적이지만 주문-결제-예약처럼 중복 실행 비용이 큰 도메인에서는 애플리케이션 차원에서 묶어 멱등성을 별도로 보장해야하는 경우가 많다.!

## 4. 멱등성 키 (IdemPotency Key)

![](https://raw.githubusercontent.com/chazy-d/md-images/main/uploads/20260325224539896.png)

멱등성 키는 같은 요청의 재시도인지, 아니면 새로운 요청인지 구분하기 위해 클라이언트가 HTTP 요청 헤더 `Idempotency-Key`에 담아서 함께 보내는 식별자이다. 이는 위의 POST나 PATCH 같은 태생이 비멱등 메서드인 애들을 네트워크등의 장애에 더 강하게 만들기 위한 수단으로 사용된다.

예를들어 결제 생성처럼 중복되면 안되는 요청에서 많이 사용된다.

```sql
POST /payments
Idempotency-Key: 8f4c2d1e-7b8a-4f49-a2c1-91f3c7d2b111
Content-Type: application/json

{
  "orderId": 123,
  "amount": 15000
}
```

위의 요청을 2번 보내면 결제건이 두개 생길 수 있다. 그러넫 클라이언트가 `Idempotency-Key`를 붙여서 재시도한다면 서버는 **이미 처리한 요청**으로 간주하여 새 결제건을 만들지 않고 이전 처리 결과를 돌려주거나 중복 처리를 막는다. 따라서 실무에서 post나 patch도 비멱동 요청으로 재시도 가능하게 만드는 도구라고 생각하면 좋다.

이때 멱등성 키는 UUID 같은 고유 값을 권장한다. 또한 **같은 키를 다른 요청 내용에 재사용하면 안 된다**. Stripe는 같은 키를 다른 엔드포인트나 다른 파라미터 조합에 재사용하면 `idempotency_error`가 날 수 있다고 설명한다.

## 5. 멱등성 보장 방식은 문제별로 다르다

### 1) 요청 중복 실행 방지: Idempotency-Key

- 같은 요청의 재시도 식별
- POST/결제/주문/예약에 적합

### 2) 동시 수정 충돌 방지: Optimistic Concurrency Control

- ETag / If-Match / version
- lost update 방지
- 수정 API에 적합

### 3) 최종 상태 보호: DB Constraint / Transaction

- UNIQUE, FK, transaction
- 무결성 보장
- 마지막 안전장치

멱등성 키와 낙관적 동시성 제어는 비슷해 보이지만 해결하는 문제가 다르다. 멱등성 키는 같은 요청의 중복 실행을 막기 위한 장치이고, 낙관적 동시성 제어는 서로 다른 요청이 동일 자원을 동시에 수정할 때 발생하는 충돌을 방지하기 위한 장치이다. HTTP에서는 `ETag`와 `If-Match` 같은 조건부 요청을 이용해 낙관적 동시성 제어를 구현할 수 있다.

출처:

https://developer.mozilla.org/ko/docs/Glossary/Idempotent

https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods

https://httpwg.org/specs/rfc9110.html#idempotent.methods

https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Idempotency-Key
# HTTP 메서드 종류

HTTP 메서드는 **클라이언트가 특정 리소스에 대해 어떤 동작을 원하고 있는지 나타내는 표준화된 요청 방식**이다. MDN은 HTTP request methods를 “요청의 목적과 성공 시 기대되는 의미를 나타내는 것”이라고 설명한다.

- 주요 메소드
    - GET :리소스 조회
    - POST: 요청 데이터 처리, 주로 등록에 사용
    - PUT : 리소스를 대체(덮어쓰기), 해당 리소스가 없으면 생성
    - PATCH : 리소스 부분 변경 (PUT이 전체 변경, PATCH는 일부 변경)
    - DELETE : 리소스 삭제
- 기타 메소드
    - HEAD : GET과 동일하지만 메시지 부분(body 부분)을 제외하고, 상태 줄과 헤더만 반환
    - OPTIONS : 대상 리소스에 대한 통신 가능 옵션(메서드)을 설명(주로 CORS에서 사용)
    - CONNECT : 대상 자원으로 식별되는 서버에 대한 터널을 설정
    - TRACE : 대상 리소스에 대한 경로를 따라 메시지 루프백 테스트를 수행

![](https://raw.githubusercontent.com/chazy-d/md-images/main/uploads/20260326055432498.png)

## GET

`GET`은 리소스를 **조회**할 때 쓴다. 서버 상태를 바꾸지 않는 read-only 성격이며, MDN은 GET을 safe하고 idempotent한 메서드로 분류한다. ([MDN 웹 문서](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods?utm_source=chatgpt.com))

```sql
GET /users/1 HTTP/1.1
Host: api.example.com
Accept: application/json
```

의미는 “1번 사용자 정보를 달라”이다.

응답 예시는 이런 느낌이다.

```sql
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 1,
  "name": "Taehun",
  "email": "taehun@example.com"
}
```

---

## POST 예시

`POST`는 보통 **새 리소스 생성**이나 **서버 측 처리 위임**에 쓴다. MDN은 POST를 표준 메서드 목록에서 resource-specific processing 용도로 설명하고, 일반적으로 멱등적이지 않은 경우가 많다. ([MDN 웹 문서](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods?utm_source=chatgpt.com))

```sql
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "name": "Taehun",
  "email": "taehun@example.com"
}
```

의미는 “새 사용자를 생성해 달라”이다.

성공 시 보통:

```sql
HTTP/1.1 201 Created
Location: /users/101
Content-Type: application/json

{
  "id": 101,
  "name": "Taehun",
  "email": "taehun@example.com"
}
```

처럼 새 리소스의 위치를 알려줄 수 있다. 같은 요청을 두 번 보내면 사용자가 두 명 생길 수 있으므로 보통 비멱등적이다.

---

## PUT 예시

`PUT`은 보통 대상 리소스를 **전체 대체**하는 의미로 쓴다. MDN은 PUT을 “target resource의 현재 representation 전체를 요청 본문으로 대체”하는 메서드로 설명한다. ([MDN 웹 문서](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods?utm_source=chatgpt.com))

```sql
PUT /users/1 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "id": 1,
  "name": "Taehun",
  "email": "taehun@example.com"
}
```

의미는 “1번 사용자 리소스를 이 내용으로 통째로 바꿔라”이다.

같은 내용을 여러 번 보내도 최종 상태는 같으므로 보통 멱등적으로 이해한다.

---

## PATCH 예시

`PATCH`는 리소스의 **일부만 수정**할 때 쓴다. MDN도 PATCH를 partial modification에 쓰는 메서드로 설명한다. ([MDN 웹 문서](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods?utm_source=chatgpt.com))

```sql
PATCH /users/1 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "name": "Taehun"
}
```

의미는 “1번 사용자의 이름만 바꿔라”이다.

다만 PATCH는 어떤 형식의 patch document를 쓰는지, 같은 patch를 반복 적용했을 때 결과가 동일한지에 따라 멱등적일 수도 있고 아닐 수도 있어서 PUT처럼 단정적으로 말하기 어렵다.

---

## DELETE 예시

`DELETE`는 대상 리소스를 **삭제**할 때 쓴다. MDN은 DELETE를 specified resource를 삭제하는 메서드로 설명한다. ([MDN 웹 문서](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods?utm_source=chatgpt.com))

```sql
DELETE /users/1 HTTP/1.1
Host: api.example.com
```

의미는 “1번 사용자를 삭제해 달라”이다.

처음 한 번 삭제하고 다시 같은 요청을 보내더라도 최종 상태는 여전히 “1번 사용자가 없다”이므로 보통 멱등적으로 본다. 다만 두 번째 요청의 응답 코드가 `404`일 수도 있다는 점은 별개다.

---

## HEAD 예시

`HEAD`는 GET과 비슷하지만 **응답 본문 없이 헤더만 확인**할 때 쓴다. MDN은 HEAD를 GET으로 받았을 헤더 메타데이터만 요청하는 메서드라고 설명한다. ([MDN 웹 문서](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/HEAD?utm_source=chatgpt.com))

```sql
HEAD /files/report.pdf HTTP/1.1
Host: api.example.com
```

이 요청은 “파일 자체를 다운로드하지 말고, 크기나 타입 같은 헤더 정보만 달라”는 의미다.

예를 들어 큰 파일을 내려받기 전에 `Content-Length`를 보고 용량을 확인할 수 있다.

---

## OPTIONS 예시

`OPTIONS`는 특정 리소스나 서버가 **어떤 통신 옵션과 메서드를 지원하는지 확인**할 때 쓴다. MDN은 allowed HTTP methods를 테스트하거나 CORS preflight 요청에서 쓸 수 있다고 설명한다. ([MDN 웹 문서](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/OPTIONS?utm_source=chatgpt.com))

```sql
OPTIONS /users HTTP/1.1
Host: api.example.com
Origin: <https://frontend.example.com>
Access-Control-Request-Method: POST
```

이건 브라우저가 CORS preflight로 “`POST /users` 요청을 보내도 되나요?”라고 미리 확인하는 전형적인 예시다.

응답에는 보통 `Allow`나 `Access-Control-Allow-Methods` 같은 헤더가 들어간다.

---

## CONNECT 예시

`CONNECT`는 **프록시와 목적지 사이 터널 생성** 용도다. 일반적인 REST 엔드포인트 설계 예시라기보다 네트워크/프록시 문맥에 가깝다. ([MDN 웹 문서](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/CONNECT?utm_source=chatgpt.com))

```sql
CONNECT api.example.com:443 HTTP/1.1
Host: api.example.com:443
```

이건 “프록시야, `api.example.com:443`까지 터널을 열어 줘”라는 요청이다. HTTPS 프록시 환경과 연결된다.

---

## TRACE 예시

`TRACE`는 **요청 메시지 반사 테스트** 용도다. 주로 진단용이며, 운영 서비스에서는 보안상 막아두는 경우가 많다. ([MDN 웹 문서](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/TRACE?utm_source=chatgpt.com))

```sql
TRACE /debug HTTP/1.1
Host: api.example.com
Max-Forwards: 5
```

이 요청은 “이 요청이 경로를 따라 어떻게 전달되는지 확인해 보자”는 뜻에 가깝다. 응답은 보통 요청을 반사한 `message/http` 형태가 된다.

출처:
https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods 및 하위 문서들