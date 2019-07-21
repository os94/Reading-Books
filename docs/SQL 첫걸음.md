# SQL 첫걸음

본 교재는 MySQL을 기준으로 설명

빠르게 훑으며, 각 개념들의 존재를 인지하고, 필요할때 찾아서 사용할수있도록 학습

[SQL 첫걸음](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788968482311&orderClick=LAH&Kc=)

# Contents

---

- SQL명령
    - DML
    - DDL
    - DCL
- DBMS
    - NoSQL (Key-Value Store)
    - RDBMS (use SQL)
        - Oracle
        - MS SQL Server
        - PostgreSQL
        - MySQL
        - SQLite
- SQL 제품마다 방언이 있다. 기능에 차이가 있을 수 있으며, 가급적 표준 SQL을 사용
- Desc : 테이블 구조
- SELECT 열 FROM 테이블명 WHERE 조건식
    - 연산자 : =, <>, >, ≥, <, ≤
    - IS NULL
- AND/OR/NOT
    - AND가 OR보다 우선순위가 높다
- 열 LIKE 패턴 : 패턴매칭, 부분검색
    - 메타문자 : `%`, `_`
    - 문자열상수 '의 이스케이프
- ~ ORDER BY
    - 반환값만 정렬하지, 테이블엔 영향을 주지 않는다
- ~ LIMIT 행수 (OFFSET 시작행)
    - MySQL, PostgreSQL만 지원
    - Oracle은 ROWNUM, SQL Server는 TOP 이용
- 산술 연산 : + - * / % MOD
- (AS) 별명
- 수치 연산 : ROUND, SIN, COS, SQRT, LOG, SUM 등...
- 문자열 연산
    - 결합 : +, ||, CONCAT (제품마다 다름)
    - SUBSTRING
    - TRIM
    - CHARACTER_LENGTH
- 날짜 연산
    - CURRENT_TIMESTAMP, CURRENT_DATE
    - INTERVAL
- CASE문
    - NULL값 변환 : COALESCE
- INSERT INTO 테이블명 VALUES ~
- DELETE FROM 테이블명 WHERE 조건식
- UPDATE 테이블명 SET 열1=값1, 열2=값2, ... WHERE 조건식
- 물리삭제(실제로 DELETE를 이용해 삭제) vs 논리삭제(flag를 이용해 삭제처럼 설정)
- 집계함수
    - COUNT, SUM, AVG, MIN, MAX
    - DISTINCT
- GROUP BY : 그룹화
    - HAVING
- ( SELECT 명령 ) : 서브쿼리
    - WHERE구에서 주로 사용
    - MySQL은 데이터 추가, 갱신시 동일한 테이블을 서브쿼리에서 사용할 수 없도록 되어있어, 에러없이 실행시키려면 인라인 뷰로 임시 테이블을 만들어야 함
    - 스칼라 값(단일 값) : SELECT명령이 하나의 값만 반환하는 경우
- 상관 서브쿼리
    - EXISTS, NOT EXISTS, IN
- CREATE, DROP, ALTER 등 테이블과 관련된 명령들 ..
    - 제약 조건들
- 인덱스
    - CREATE/DROP INDEX ~
    - EXPLAIN
- 뷰
    - CREATE/DROP VIEW
- 집합 연산
    - UNION
    - INTERSECT, EXCEPT(MINUS)는 MySQL 지원하지 않음
- 테이블 결합 (join)
    - 내부결합 : INNER JOIN
    - 외부결합 : LEFT JOIN, RIGHT JOIN (MySQL 불가)
- 정규화
    - 테이블을 개선해 나가는 것. 총 5단계가 존재함
    - `하나의 데이터는 한 곳에 있어야 한다` 규칙을 근거
- 트랜잭션
    1. START TRANSACTION
    2. COMMIT / ROLLBACK
- 내부 처리 순서

    WHERE -> GROUP BY -> SELECT -> ORDER BY