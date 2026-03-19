## 미션 기록

### 이전 ERD 설계 수정

![](https://raw.githubusercontent.com/chazy-d/md-images/main/uploads/20260319224601134.png)

마이페이지 와이어프레임에는 닉네임, 이메일, 전화번호가 표시되므로, 이를 조회할 수 있도록 users 테이블에 nickname, email, phone_number 속성을 추가했다.

### 1. 리뷰 작성하는 쿼리
![](https://raw.githubusercontent.com/chazy-d/md-images/main/uploads/20260319223506847.png)

```sql
INSERT INTO reviews (
    user_mission_id,
    rating,
    content,
    created_at,
    updated_at
)
VALUES (
    :user_mission_id,
    :rating,
    :content,
    NOW(),
    NOW()
);
```
리뷰 작성 화면에서 사용자가 별점과 리뷰 내용을 입력한 뒤 저장 버튼을 눌렀을 때 실행되는 쿼리이다.
reviews 테이블에 user_mission_id, rating, content를 저장하여 어떤 미션 수행에 대한 리뷰인지 연결되도록 설계했다.
사진 첨부 기능은 제외했기 때문에, 리뷰 이미지 관련 테이블은 사용하지 않았다.

리뷰 등록 후 user_missions의 상태를 REVIEWED로 변경하는 로직도 함께 처리할 수 있을 것 같다.


### 2. 내가 진행중, 진행 완료한 미션 모아서 보는 쿼리(페이징 포함)

![](https://raw.githubusercontent.com/chazy-d/md-images/main/uploads/20260319223734745.png)

```sql
SELECT
    um.user_mission_id,
    um.status,
    um.challenged_at,
    um.completed_at,
    m.mission_id,
    m.mission_content,
    m.reward_point,
    s.store_id,
    s.name AS store_name
FROM user_missions um
         JOIN missions m
              ON m.mission_id = um.mission_id
         JOIN stores s
              ON s.store_id = m.store_id
WHERE um.user_id = :user_id
  AND (
    (:tab_type = 'IN_PROGRESS' AND um.status IN ('IN_PROGRESS', 'SUCCESS_REQUESTED'))
        OR
    (:tab_type = 'COMPLETED' AND um.status IN ('COMPLETED', 'REVIEWED'))
    )
ORDER BY COALESCE(um.completed_at, um.challenged_at) DESC
    LIMIT :page_size OFFSET :offset;
```

사용자가 미션 탭에서 현재 진행 중인 미션과 완료한 미션을 구분하여 조회할 수 있도록 작성한 쿼리이다.
user_missions를 중심으로 missions, stores를 조인하여 미션 내용, 보상 포인트, 가게 이름을 함께 조회한다.
:tab_type 값에 따라 진행 중(IN_PROGRESS, SUCCESS_REQUESTED) 또는 완료(COMPLETED, REVIEWED) 상태의 미션만 조회하도록 조건을 분리했다.
또한 LIMIT과 OFFSET을 사용하여 페이지 단위로 데이터를 불러올 수 있도록 페이징 처리를 적용했다.

### 3. 마이 페이지 화면 쿼리

![](https://raw.githubusercontent.com/chazy-d/md-images/main/uploads/20260319223746788.png)

```sql
SELECT
    u.user_id,
    u.nickname,
    u.email,
    u.phone_number,
    u.current_point,
    u.created_at,
    r.name AS region_name
FROM users u
         JOIN regions r
              ON r.region_id = u.region_id
WHERE u.user_id = :user_id;
```

마이페이지 화면에서 사용자 기본 정보를 조회하기 위한 쿼리이다.
users 테이블에서 닉네임, 이메일, 전화번호, 현재 포인트, 가입일을 조회하고, regions 테이블과 조인하여 사용자가 속한 지역명도 함께 가져오도록 했다.
이를 통해 마이페이지에서 사용자 계정 정보와 현재 상태를 한 번에 보여줄 수 있다.

### 4. 홈 화면 쿼리 (현재 선택된 지역에서 도전이 가능한 미션 목록, 페이징 포함)

![](https://raw.githubusercontent.com/chazy-d/md-images/main/uploads/20260319223759787.png)

```sql
// 도전 가능한 미션 목록
SELECT
    m.mission_id,
    m.mission_content,
    m.reward_point,
    s.store_id,
    s.name AS store_name
FROM missions m
JOIN stores s
    ON s.store_id = m.store_id
WHERE s.region_id = :selected_region_id
  AND m.is_active = TRUE
  AND NOT EXISTS (
        SELECT 1
        FROM user_missions um
        WHERE um.user_id = :user_id
          AND um.mission_id = m.mission_id
      )
ORDER BY m.created_at DESC
LIMIT :page_size OFFSET :offset;

---

// 홈 상단 진행도 쿼리

SELECT
    r.region_id,
    r.name AS region_name,
    r.mission_goal_count,
    COUNT(um.user_mission_id) AS completed_count
FROM regions r
         LEFT JOIN stores s
                   ON s.region_id = r.region_id
         LEFT JOIN missions m
                   ON m.store_id = s.store_id
         LEFT JOIN user_missions um
                   ON um.mission_id = m.mission_id
                       AND um.user_id = :user_id
                       AND um.status IN ('COMPLETED', 'REVIEWED')
WHERE r.region_id = :selected_region_id
GROUP BY r.region_id, r.name, r.mission_goal_count;
```

도전 가능한 미션 목록 쿼리와 홈 상단 진행도 쿼리를 나누었다.

#### 1. 도전 가능한 미션 목록 쿼리 설명

현재 선택한 지역(:selected_region_id)에 속한 가게들의 미션 중에서, 아직 사용자가 도전하지 않은 미션만 조회하도록 구성했다.
이를 위해 NOT EXISTS를 사용하여 user_missions에 이미 존재하는 미션은 제외했다.
또한 LIMIT과 OFFSET을 사용하여 한 번에 일정 개수의 미션만 불러오도록 페이징 처리하였다.

#### 2. 홈 상단 진행도 쿼리 설명

홈 화면 상단의 진행도(7/10)를 계산하기 위한 쿼리이다.
현재 선택된 지역에 속한 미션 중에서 사용자가 완료한 미션 수를 집계하여 completed_count로 조회하고, regions 테이블의 mission_goal_count와 함께 사용하여 진행도를 표시할 수 있도록 했다.
이를 통해 별도의 진행도 테이블 없이도 지역별 미션 완료 현황을 계산할 수 있다.