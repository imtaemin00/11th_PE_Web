- 1. 요구사항 → SQL로 번역하기

찾아보기: 화면 요구사항을 SELECT 컬럼, FROM 테이블, JOIN 관계, WHERE 조건, ORDER BY/LIMIT으로 나누는 방법을 정리해 보세요.

    - FROM - 어떤 테이블에서 데이터를 가져올지
    - SELECT - 화면에 무엇을, 어떤 열(컬럼)의 정보를 보여줄지

  ```jsx
  SELECT book_id, title, description FROM book
  ```

-book 이라는 테이블에서 book_id, title, description 이라는 열(컬럼)의 정보를 가져오라는 뜻이다

    - JOIN - 기준 테이블에 없는 정보가 필요할 때, ERD의 PK,FK 관계를  따라 다른 테이블을 붙인다
    - WHERE - 조건에 맞는 행만 남긴다
    - ORDER BY - 어떤 순서로 보여줄지
    - LIMIT - 최대 몇 개까지 보여 줄지

예를 들어, “문학 카테고리에서 대여 가능한 책을 최신순으로 10권 보여준다” 라는 요구사항에 대해 SQL문을 작성한다면

  ```jsx
  SELECT b.book_id, b.title, c.name AS category_name
  FROM book b
  JOIN category c ON b.category_id = c.category_id 
  WHERE c.name = '문학' AND b.is_available = TRUE 
  ORDER BY b.book_id DESC LIMIT 10;
  ```

보여줄 것 →  책 제목, 카테고리 이름 → SELECT

기준 테이블 - 책 목록 → FROM

카테고리 이름은 category 테이블에 존재 → JOIN

문학, 대여 가능 → WHERE

최신순, 10권 - ORDER BY, LIMIT

- 2. DDL과 DML

찾아보기: CREATE TABLE이 테이블 구조를, INSERT가 행 데이터를 담당하는 이유와 ALTER TABLE과의 차이를 살펴보세요.

DDL - 테이블의 틀을 다루는 명령
ex) CREATE, ALTER, DROP

DML - 틀 안의 데이터를 다루는 명령
ex) INSERT, UPDATE, DELETE, SELECT

    - CREATE가 담당하는 테이블 구조는 한 번 정하면 거의 안 바뀌지만, INSERT가 담당하는 데이터는 빈번하게 추가, 변경이 필요하다
    - DDL은 실행하는 즉시 확정(자동 커밋)되어 되돌릴 수 없다. 반면 DML은 트랜잭션 안에서 실행하면 ROLLBACK으로 되돌릴 수 있다. 그래서 DDL은 더 신중하게 다뤄야 한다.

ALTER TABLE - CREATE와 다르게 이미 있는 테이블의 구조를 변경하는 명령. 컬럼을 추가, 수정할 수 있다

  ```jsx
  ALTER TABLE book ADD COLUMN created_at DATETIME;
  ```

ALTER TABLE을 사용하면 위처럼 created_at 컬럼을 추가할 수 있지만 같은 변경을 DROP 후 CREATE 하게 되면 데이터가 전부 사라진다.

따라서 ALTER은 데이터를 지우지 않고 구조만 바꿀 수 있다는 점에서 필요하다

- 3. PK·FK와 JOIN 조건

찾아보기: PK·FK가 무엇을 보장하는지, ON 절에서 관계가 잘못 연결되면 왜 중복 행이 생기는지 확인해 보세요.

PK - 중복될 수 없고 NULL일 수 없다

FK - 참조하는 대상이 실제로 존재한다는 것을 보장한다 (참조 무결성)

  ```jsx
  INSERT INTO rental (user_id, book_id, rented_at, due_at)
  VALUES (1, 99, NOW(), NOW());
  -- Error 1452: foreign key constraint fails
  ```

위 코드는 book 테이블에 99번 책이 없기 때문에, 존재하지 않는 책을 빌리는 대여 기록은 FK에 의해 막힌다

#### ON 절이 잘못되면 중복 행이 생기는 이유

JOIN은 왼쪽 테이블의 행과 오른쪽 테이블의 행을 모든 조합으로 짝지은 뒤, ON 조건에 맞는 짝만 남기는 방식으로 동작

  ```jsx
  SELECT b.title, c.name
  FROM book b JOIN category c ON 1 = 1;
  ```

예를 들어 이런식으로 코드를  작성하면  1=1은 항상 참이라 아무것도 걸러지지 않으므로

| title | name |
    | --- | --- |
| 달빛 도서관 | 문학 |
| 달빛 도서관 | 과학 |
| 겨울의 편지 | 문학 |
| 겨울의 편지 | 과학 |
| 우주를 읽는 법 | 문학 |
| 우주를 읽는 법 | 과학 |

이렇게 중복 행이 생기게 된다

- 4. WHERE와 NULL

찾아보기: WHERE 조건에서 NULL을 = 과 비교할 수 없는 이유와 IS NULL을 사용하는 이유를 알아보세요.

NULL - 0이나 빈 문자열이 아니라 "값을 모른다 / 아직 없다"는 뜻

SQL은 NULL과의 비교 결과를 참(TRUE)도 거짓(FALSE)도 아닌 UNKNOWN(알 수 없음)으로 처리한다

  ```jsx
  SELECT * FROM rental WHERE returned_at = NULL;   
  SELECT * FROM rental WHERE returned_at IS NULL;
  ```

따라서 첫 번째 쿼리는 오류 없이 0행이 나오게 되고, 오류가 나지 않기 때문에 “대여 중인 책이 없다”고 잘못 판단하게 된다

IS NULL -  값이 NULL인지 아닌지를 확인하는 전용 문법
UNKNOWN이 아니라 TRUE 또는 FALSE를 돌려준다

그래서 "아직 반납하지 않은 책"처럼 NULL 여부가 조건인 경우에는 반드시 IS NULL을 사용해야 한다

- 5. ORDER BY와 일관된 정렬

찾아보기: ORDER BY가 없을 때 목록 순서가 보장되지 않는 이유와 동일 값일 때의 보조 정렬 기준을 찾아보세요.

#### ORDER BY가 없으면 순서가 보장되지 않는 이유

    - SQL의 테이블은 이론적으로 순서 없는 행들의 집합이다

#### 그럼 지금은 왜 늘 같은 순서로 나올까?

    - MySQL은 내부적으로 데이터를 PK 순서대로 저장함
      그래서 SELECT * FROM book; 을 하면 대부분 book_id 1, 2, 3 순으로 나오지만 이건 저장 방식 때문에 우연히 그렇게 나오는 것이다

#### 보조 정렬 기준

  ```jsx
  SELECT book_id, title, is_available
  FROM book
  ORDER BY is_available DESC;
  ```

이 코드를 실행하면

| book_id | title | is_available |
    | --- | --- | --- |
| 1 | 달빛 도서관 | 1 |
| 3 | 우주를 읽는 법 | 1 |
| 2 | 겨울의 편지 | 0 |

위와 같은 결과가 나오는데  2번이 맨 뒤로 가는건 확실하지만, 1번과 3번은 ORDER BY가 이 둘의 순서를 정해 주지 않았기 때문에, 1 → 3으로 나올지 3 → 1로 나올지는 보장되지 않는다

이 경우에는 보조 정렬 기준을 붙이면 해결 된다

  ```jsx
  SELECT book_id, title, is_available
  FROM book
  ORDER BY is_available DESC, book_id DESC;
  ```

book_id 라는 보조 기준을 붙이면 is_available이 동일 값 일때도 book_id로 순서가 정해지므로 몇 번을 실행해도 이 순서로만 나온다

*보조 기준으로 PK를 쓰는 이유는 PK는 절대 중복되지 않아서 더 이상 동점이 생길 수 없기 때문

- 6. LIMIT / OFFSET과 페이지네이션

찾아보기: LIMIT/OFFSET이 페이지 번호 방식과 어떻게 연결되는지, 데이터가 많아질 때 어떤 한계가 있는지 살펴보세요.

    - LIMIT: 가져올 최대 개수
      LIMIT은 "정확히 N개"가 아니라 "최대 N개"라서, 조건에 맞는 데이터가 적으면 그만큼만 나온다. 미션 1에서 LIMIT 10인데 1행만 나온 이유다
    - OFFSET: 앞에서부터 건너뛸 개수

**페이지 번호 방식과의 연결**

페이지 번호를 OFFSET으로 바꿔서 조회한다.

  ```jsx
  SELECT book_id, title FROM book
  ORDER BY book_id DESC
  LIMIT 10 OFFSET 10;  
  ```

페이지를 나눌 때는 ORDER BY가 반드시 필요하다. 순서가 정해지지 않으면 페이지마다 같은 데이터가 중복되거나 빠질 수 있다.

**데이터가 많아질 때의 한계**

    1. 뒤쪽 페이지일수록 느려진다
       OFFSET은 건너뛸 행을 실제로 전부 읽고 버린다. LIMIT 10 OFFSET 100000 이면 10만 10개를 읽고 10개만 돌려준다.
    2. 중간에 데이터가 바뀌면 중복·누락이 생긴다
       1페이지를 보는 사이 새 글이 추가되면 전체가 한 칸씩 밀려서, 2페이지에서 이미 본 글이 다시 나온다. 삭제되면 반대로 한 개를 못 보고 지나간다.