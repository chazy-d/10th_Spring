# 0. 피어 리뷰

![](https://raw.githubusercontent.com/chazy-d/md-images/main/uploads/20260326124544132.png)

멱등성에서 각 메서드 종류를 나누어서 설명하고, DELETE에서 응답은 달라도 최종 상태는 같다는 점을 통해 이해를 하는데 많은 도움이 되었다.

# 1. 홈 화면 조회

## URL: GET /api/home

## 설명
현재 선택한 지역의 홈 화면 정보를 조회한다. 상단의 현재 포인트/ 지역별 미션 진행도와 하단의 도전 가능한 미션 목록을 커서 페이징을 통해 함께 반환한다.

## 설계 포인트

### 1. 상단 정보와 목록이 필요하다
홈 화면은 단순히 미션 목록만 보여주는 화면이 아니라, 현재 포인트/지역별 진행도/목표 달성 보상/ 도전 가능한 미션 목록이 필요한 화면이다.
따라서 사용자 와 지역 상태를 따로 모두 받아주는 형태로 설계했다.

### 2. 포인트의 종류?

개별 미션 완료시 받는 미션 보상 포인트와 지역 목표 미션 수를 모두 달성하면 받는 보상 포인트가 따로 존재한다.
둘다 최종적으로는 사용자 포인트에 합산되는 포인트이겠지만 지급 기준이 다르기에 구분해서 설계했다.

### 페이징 방법

홈 화면의 미션 목록은 모바일에서 아래로 스크롤하며 이어서 보는 형태이기에 페이지 번호 기반보다 커서 페이징이 더 자연스럽다고 판단했다.
스크롤시 데이터가 중간에 추가되더라도 안정적으로 다음 목록을 가져올 수 있다.

### Request

**Request Header**

| key | 설명 | value 타입 | 필수 | 예시 |
| --- | --- | --- | --- | --- |
| Content-Type | 요청 바디 형식 | string | O | application/json |
| Authorization | 엑서스 토큰 | string | O | Bearer {token} |

**Query parameter**

| key | 설명 | value 타입 | 필수 | 옵션 | 예시 |
| --- | --- | --- | --- | --- | --- |
| regionId | 현재 선택한 지역 | number | O | 존재하는 지역 ID | 1 |
| cursor | 다음 조회 시작 기준값 | number | X | 첫 조회시 생략 가능함 | 20 |
| size | 한번에 조회할 미션 개수 | number | O | 1 이상 | 5 |

+++

cursor에 missionId에 대한 값이라는 것에 대한 설명이 추가되는 것이 더 이해하는데 쉬울 것 같음!

**Path Variable**

| key | 설명 | value 타입 | 필수 | 예시 |
| --- | --- | --- | --- | --- |
| X |  |  |  |  |
|  |  |  |  |  |

Request Body

| key | 설명 | value 타입 | 필수 | 옵션 | Nullable | 예시 |
| --- | --- | --- | --- | --- | --- | --- |
| X |  |  |  |  |  |  |
|  |  |  |  |  |  |  |

**Example**

```json
// GET /api/home?regionId=1&cursor=20&size=10
```

### Response

| key |  |  | 설명 | value 타입 | 비고 | Nullable | 옵션 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| status |  |  | 요청 결과 상태 | string | success/ error | X |  |
| message |  |  | 결과 메시지 | string |  | X |  |
| data |  |  | 데이터 | object |  | X |  |
|  | data.userPoint |  | 현재 사용자 포인트 | number |  | X |  |
|  | data.regionId |  | 현재 지역 ID | number |  | X |  |
|  | data.regionName |  | 현재 지역명 | string |  | X |  |
|  | data.completedMissionCount |  | 현재 지역에서 완료한 미션 수  | number |  | X |  |
|  | data.goalMissionCount |  | 현재 지역 목표 미션 수 | number |  | X |  |
|  | data.rewardPoint |  | 목표 달성시 지급 포인트 | numer | 10개 달성시 추가 지급 포인트? | X |  |
|  | data.missions |  | 도전 가능한 미션 목록 |  |  |  |  |
|  |  | data.missions[].missionId | 미션Id | number |  | X |  |
|  |  | data.missions[].storeId | 가게ID | number |  | X |  |
|  |  | data.missions[].storeName | 가게명 | string |  | X |  |
|  |  | data.missions[].missionContent | 미션 내용 | string |  | X |  |
|  |  | data.missions[].missionPoint | 미션 보상 포인트 | number |  | X |  |
|  |  | data.missions[].daysRemaining | 남은 기한.? | number |  | X |  |
|  | data.size |  | 요청한 크기 | number |  | X |  |
|  | data.nextCursor |  | 다음 조회 커서 | number | 마지막이면 NULL | O |  |
|  | data.hasNext |  | 다음페이지 존재 여부 | boolean |  | X |  |

**Example**

```json
{
  "status": "success",
  "message": "홈 화면 조회에 성공했습니다.",
  "data": {
    "userPoint": 999999,
    "regionId": 1,
    "regionName": "오금동",
    "completedMissionCount": 7,
    "goalMissionCount": 10,
    "rewardPoint": 1000,
    "missions": [
      {
        "missionId": 101,
        "storeId": 1,
        "storeName": "춘리마라탕",
        "missionContent": "10,000원 이상의 식사 시",
        "missionPoint": 500,
        "daysRemaining": 7
      },
      {
        "missionId": 102,
        "storeId": 2,
        "storeName": "짜장전설",
        "missionContent": "12,000원 이상의 식사 시",
        "missionPoint": 600,
        "daysRemaining": 3
      }
    ],
    "size": 10,
    "nextCursor": 12,
    "hasNext": true
  },
}
```

### Status

| status | response content |
| --- | --- |
| 200 |  |
| 400 |  |

# 2. 미션 목록 조회

## URL: GET /api/missions

## 설명

사용자의 미션 수행 목록을 조회한다. 상태 값에 따라 진행중 미션 또는 진행 완료 미션 목록을 반환한다.
모바일 리스트 UI에 맞춰 커서 페이징을 사용한다.

## 설계 포인트

### 1. user_mission을 중심으로 조회

미션 목록은 단순 미션 정보를 조회하는 것이 아니라, 사용자가 실제로 어떤 미션에 도전했고, 현재 어떤 상태인지를 보여주는 기능이다.
따라서 missions가 아니라 user_missions를 중심으로 missions와 stores를 조인하는 구조로 설계했다.



### Request

**Request Header**

| key | 설명 | value 타입 | 필수 | 예시 |
| --- | --- | --- | --- | --- |
| Content-Type | 요청 바디 형식 | string | O | application/json |
| Authorization | 엑서스 토큰 | string | O | Bearer {token} |

**Query parameter**

| key | 설명 | value 타입 | 필수 | 옵션 | 예시 |
| --- | --- | --- | --- | --- | --- |
| regionId | 조회할 지역 ID | number | X | 지역 필터 | 1 |
| status | 조회 상태 | string | O | IN_PROGRESS, COMPLETED | IN_PROGRESS |
| cursor | 다음 조회 시작 기준값 | number | X | 첫 조회 시 생략 가능 | 120 |
| size | 한 번에 조회할 개수 | number | O | 1 이상 | 10 |

### Path Variable

**Path Variable**

| key | 설명 | value 타입 | 필수 | 예시 |
| --- | --- | --- | --- | --- |
| x |  |  |  |  |
|  |  |  |  |  |

Request Body

| key | 설명 | value 타입 | 필수 | 옵션 | Nullable | 예시 |
| --- | --- | --- | --- | --- | --- | --- |
| x |  |  |  |  |  |  |
|  |  |  |  |  |  |  |

**Example**

```json
// GET /api/missions?regionId=1&status=IN_PROGRESS&cursor=120&size=10
```

### Response

| key |  |  | 설명 | value 타입 | 비고 | Nullable | 옵션 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| status |  |  | 요청 결과 상태 | string | success / error | X |  |
| message |  |  | 결과 메시지 | string |  | X |  |
| data |  |  | 미션 목록 데이터 | object |  | X |  |
|  | data.status |  | 요청한 상태 구분 | string |  | X |  |
|  | data.missions |  | 미션 목록 | object[] |  | X |  |
|  |  | data.missions[].userMissionId | 사용자 미션 ID | number |  | X |  |
|  |  | data.missions[].missionId | 미션 ID | number |  | X |  |
|  |  | data.missions[].storeId | 가게 ID | number |  | X |  |
|  |  | data.missions[].storeName | 가게명 | string |  | X |  |
|  |  | data.missions[].missionContent | 미션 내용 | string |  | X |  |
|  |  | data.missions[].missionPoint | 미션 보상 포인트 | number |  | X |  |
|  |  | data.missions[].status | 사용자 미션 상태 | string |  | X |  |
|  |  | data.missions[].challengedAt | 도전 시각 | string |  | O |  |
|  |  | data.missions[].completedAt | 완료 시각 | string |  | O |  |
|  |  | data.missions[].daysRemaining | 남은 일수 | number | missions.deadline_at 필요 | O |  |
|  | data.size |  | 요청한 size | number |  | X |  |
|  | data.nextCursor |  | 다음 조회 커서 | number | 마지막이면 null | O |  |
|  | data.hasNext |  | 다음 페이지 존재 여부 | boolean |  | X |  |

**Example**

```json
{
  "status": "success",
  "message": "미션 목록 조회에 성공했습니다.",
  "data": {
    "status": "IN_PROGRESS",
    "missions": [
      {
        "userMissionId": 44,
        "missionId": 101,
        "storeId": 1,
        "storeName": "춘리마라탕",
        "missionContent": "10,000원 이상의 식사 시",
        "missionPoint": 500,
        "status": "IN_PROGRESS",
        "challengedAt": "2026-03-25T12:12:12",
        "completedAt": null,
        "daysRemaining": 7
      }
    ],
    "size": 10,
    "nextCursor": null,
    "hasNext": false
  },
}
```

### Status

| status | response content |
| --- | --- |
| 200 |  |
| 400 |  |

# 3. 미션 성공 누르기

## URL: PATH /api/missions/{userMissionId}/success

+++ 

URL에서 missions라고 하기보다는 사용자의 특정 미션에 접근하는 느낌이 바로 날 수 있도록 user-missions등으로 변경하는것이 좋아보임.


## 설명: 사용자가 특정 미션 수행 기록에 대해 성공 요청을 보낸다.

## 설계 포인트

### 1. 상태를 바꾸는 동작.

미션 성공 누르기는 미션 성공 **요청**이다. 이미 존재하는 유저 미션의 상태를 진행중에서 변경하는 동작이므로 PATCH 메서드를 선택했다.

### 2. 성공 요청과 실제 완료를 분리

기획에 따라 미션 성공 누르기가 곧바로 완료를 의미할 수도 있고, 중간에 검증 단계가 필요한 성공 요청 상태일 수도 있다.
확인할 방법이 없으므로 현재 설계에서는 확장 가능성을 고려해 SUCCESS_REQUESTED 상태를 두어 요청했다.

### Request

**Request Header**

| key | 설명 | value 타입 | 필수 | 예시 |
| --- | --- | --- | --- | --- |
| Content-Type | 요청 바디 형식 | string | O | application/json |
| Authorization | 엑서스 토큰 | string | O | Bearer {token} |

**Query parameter**

| key | 설명 | value 타입 | 필수 | 옵션 | 예시 |
| --- | --- | --- | --- | --- | --- |
| x |  |  |  |  |  |
|  |  |  |  |  |  |

**Path Variable**

| key | 설명 | value 타입 | 필수 | 예시 |
| --- | --- | --- | --- | --- |
| userMissionId | 성공 요청할 사용자미션 ID | number | O | 10 |
|  |  |  |  |  |

Request Body

| key | 설명 | value 타입 | 필수 | 옵션 | Nullable | 예시 |
| --- | --- | --- | --- | --- | --- | --- |
| x |  |  |  |  |  |  |
|  |  |  |  |  |  |  |

**Example**

```json
// PATCH /api/missions/10/success

```

### Response

| key |  | 설명 | value 타입 | 비고 | Nullable | 옵션 |
| --- | --- | --- | --- | --- | --- | --- |
| status |  | 요청 결과 상태 | string | success / error | X |  |
| message |  | 결과 메시지 | string |  | X |  |
| data |  | 성공 요청 결과 | object |  | X |  |
|  | data.userMissionId | 사용자 미션 ID | number |  | X |  |
|  | data.status | 변경된 상태 | string | SUCCESS_REQUESTED | X |  |
|  | data.successRequestedAt | 성공 요청 시각 | string |  | X |  |

**Example**

**Example**

```json
{
  "status": "success",
  "message": "미션 성공 요청이 완료되었습니다.",
  "data": {
    "userMissionId": 44,
    "status": "SUCCESS_REQUESTED",
    "successRequestedAt": "2026-03-25T12:12:12"
  }
}
```

### Status

| status | response content |
| --- | --- |
| 200 |  |
| 400 |  |

# 4. 가게 리뷰 작성

## URL: /api/stores/{storeId}/reviews

## 설명

사용자가 특정 가게에 대한 리뷰를 작성한다. 리뷰는 가게페이지에서 하는게 일반적이라고 생각되지만, 마이페이지에서도 가능하다는 조건이 있으므로, 가게 자원 기준으로 설계한다.

## 설계 포인트

### 1. 리뷰는 가게 리뷰다?

리뷰 작성 기능은 기획만 보자면 마이페이지나 완료 목록 페이지나 가게 페이지등 여러곳에서 진입이 가능할 것 같다.
이때 실제로 생성되는 데이터는 특정 가게에 달리는 리뷰이다. 따라서 화면 진입 위치와 무관하게 API는 store 기준으로 설계하는 것이 자연스럽다고 생각한다.

위의 맥락에서 미션에서 왜 "마이페이지" 리뷰 작성에 초점을 두었는지는 잘 모르겠지만 어떤 곳에서 리뷰 작성을 하던 같은 리뷰 생성 동작이기에 하나의 API를 재사용하는 방식이 더 일관적이다.
좀더 나아가면 어디서 리뷰 작성을 누르던 결국 상점 페이지의 리뷰작성 페이지에 리다이렉트 되게 만들게 되지 않을까 생각한다.

### Request

**Request Header**

| key | 설명 | value 타입 | 필수 | 예시 |
| --- | --- | --- | --- | --- |
| Content-Type | 요청 바디 형식 | string | O | application/json |
| Authorization | 엑서스 토큰 | string | O | Bearer {token} |

**Query parameter**

| key | 설명 | value 타입 | 필수 | 옵션 | 예시 |
| --- | --- | --- | --- | --- | --- |
| x |  |  |  |  |  |
|  |  |  |  |  |  |

**Path Variable**

| key | 설명 | value 타입 | 필수 | 예시 |
| --- | --- | --- | --- | --- |
| storeId | 리뷰를 작성할 가게 ID | number | O | 1 |
|  |  |  |  |  |

Request Body

| key | 설명 | value 타입 | 필수 | 옵션 | Nullable | 예시 |
| --- | --- | --- | --- | --- | --- | --- |
| reviewContent | 리뷰 내용 | string | O | 최대 500자 등 | X | 항상 맛있습니다! |
| starRating | 별점 | number | O | 1~5 | X | 5 |
| images | 리뷰 이미지 URL 목록 | string[] | X | 이미지 첨부 시 사용 | O | ["imageurl1", "imageurl2"] |

**Example**

```json
// POST /api/stores/1/reviews

{
  "reviewContent": "벌레 나왔어요. 😡",
  "starRating": 1,
  "images": [
    "imageurl1",
    "imageurl2"
  ]
}
```

### Response

| key |  | 설명 | value 타입 | 비고 | Nullable | 옵션 |
| --- | --- | --- | --- | --- | --- | --- |
| status |  | 요청 결과 상태 | string | success / error | X |  |
| message |  | 결과 메시지 | string |  | X |  |
| data |  | 생성된 리뷰 정보 | object |  | X |  |
|  | data.reviewId | 리뷰 ID | number |  | X |  |
|  | data.storeId | 가게 ID | number |  | X |  |
|  | data.reviewContent | 리뷰 내용 | string |  | X |  |
|  | data.starRating | 별점 | number |  | X |  |
|  | data.images | 리뷰 이미지 URL 목록 | string[] | 이미지 없으면 빈 배열 가능 | O |  |
|  | data.createdAt | 생성 시각 | string | ISO-8601 | X |  |

**Example**

```json
{
  "status": "success",
  "message": "리뷰 작성이 완료되었습니다.",
  "data": {
    "reviewId": 1,
    "storeId": 1,
    "reviewContent": "벌레 나왔어요. 😡",
    "starRating": 1,
    "images": [
      "imageurl1",
      "imageurl2"
    ],
    "createdAt": "2026-03-24T12:12:12"
  }
}
```

### Status

| status | response content |
| --- | --- |
| 200 |  |
| 400 |  |

# 5. 회원가입

## URL: Post /api/auth/signup

## 설명

기본 회원가입 정보를 받아 사용자를 생성한다.

## 설계 포인트

### 1. 회원가입시에 유저 생성, 관계 테이블 저장까지 포함하는 동작이 실행된다.

약관 동의 정보와 선호 음식 카테고리 정보까지 함께 저장해야함을 인지해야한다.

### 2. 약관 동의와 선호 카테고리를 배열 형태

약관과 선호 음식은 각각 여러개가 존재하고 선택가능하기에 요청 형태에서도 배열 형태로 받는 구조가 자연스럽다.
N:M관계 느낌으로?

### Request

**Request Header**

| key | 설명 | value 타입 | 필수 | 예시 |
| --- | --- | --- | --- | --- |
| Content-type | 요청 바디 형식 | string | O | applicatin/json  |
|  |  |  |  |  |

**Query parameter**

| key | 설명 | value 타입 | 필수 | 옵션 | 예시 |
| --- | --- | --- | --- | --- | --- |
| x |  |  |  |  |  |
|  |  |  |  |  |  |

**Path Variable**

| key | 설명 | value 타입 | 필수 | 예시 |
| --- | --- | --- | --- | --- |
| x |  |  |  |  |
|  |  |  |  |  |

Request Body

| key |  | 설명 | value 타입 | 필수 | 옵션 | Nullable | 예시 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| nickname |  | 닉네임 | string | O | 최대 20자 | X | 그린 |
| name |  | 이름 | string | O | 최대 20자 | X | 차그린 |
| gender |  | 성별 | string | O | MALE, FEMALE, NONE | X | MALE |
| birthDate |  | 생년월일 | string | O | YYYY-MM-DD | X | 2001-07-10 |
| regionId |  | 지역 ID | number | O | 존재하는 지역 ID | X | 1 |
| addressDetail |  | 상세 주소 | string | O | 최대 100자 | X | 서울 송파구 어쩌고 저쩌고 |
| email |  | 이메일 | string | O | 유효한 이메일 형식 | X | my0710my@gmail.com |
| phoneNumber |  | 전화번호 | string | O | 숫자/하이픈 정책 선택 | X | 01010101010 |
| terms |  | 약관 동의 목록 | object[] | O | 최소 1개 이상 | X |  |
|  | terms[].termId | 약관 ID | number | O |  | X | 1 |
|  | terms[].isAgree | 동의 여부 | boolean | O |  | X | true |
| favoriteFoodCategoryIds |  | 선호 음식 카테고리 ID 목록 | number[] | O | 최소 1개 이상 | X | [1, 4] |

**Example**

```json
// POST /api/auth/signup

{
  "nickname": "그린",
  "name": "차그린",
  "gender": "MALE",
  "birthDate": "2001-07-10",
  "regionId": 1,
  "addressDetail": "서울 송파구 오금동 ",
  "email": "my0710my@gmail.com",
  "phoneNumber": "01010101010",
  "terms": [
    { "termId": 1, "isAgree": true },
    { "termId": 2, "isAgree": true },
    { "termId": 3, "isAgree": true },
    { "termId": 4, "isAgree": true }
  ],
  "favoriteFoodCategoryIds": [1, 2, 3, 4, 5, 6, 7]
}
```

### Response

| key |  | 설명 | value 타입 | 비고 | Nullable | 옵션 |
| --- | --- | --- | --- | --- | --- | --- |
| status |  | 요청 결과 상태 | string | success / error | X |  |
| message |  | 결과 메시지 | string |  | X |  |
| data |  | 생성된 사용자 정보 | object |  | X |  |
|  | data.userId | 생성된 사용자 ID | number |  | X |  |
|  | data.nickname | 닉네임 | string |  | X |  |
|  | data.regionId | 지역 ID | number |  | X |  |
|  | data.email | 이메일 | string |  | X |  |
|  | data.phoneNumber | 전화번호 | string |  | X |  |
|  | data.createdAt | 생성 시각 | string |  | X |  |

**Example**

```json
{
  "status": "success",
  "message": "성공적으로 회원가입이 완료되었습니다.",
  "data": {
    "userId": 5,
    "nickname": "그린티",
    "regionId": 1,
    "email": "my0710my@gmail.com",
    "phoneNumber": "01010101010",
    "createdAt": "2026-03-26T16:42:53.234Z"
  }
}
```

### Status

| status | response content |
| --- | --- |
| 200 |  |
| 400 |  |