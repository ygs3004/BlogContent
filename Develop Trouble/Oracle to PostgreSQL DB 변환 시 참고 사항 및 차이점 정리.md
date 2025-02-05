최근 프로젝트에서 Oracle 프로시저 기반의 프로젝트를 Postgresql 로 변환하는 작업을 진행하고 있다. Aws Schema Conversion Tool를 이용하여 1차 변환 후 문제가 없는지 확인하며 추가 변환작업을 진행하는 형태로 진행하였다. 변환 작업을 하면서 많이 느꼈던 부분은 PostgreSQL의 경우 "데이터 타입"에 많은 신경을 써야한다는 것이었다. 

변환 작업을 하면서 변환 시 체크해야 하는 상황들을 일부 정리해본다.

##### 1. CURSOR 이용시 RECORD 타입 선언 필요

```sql
FOR V_RECORD IN SOME_CURSOR
LOOP
	~~~ V_RECROD 를 이용
END LOOP;
```

위와 같은 커서 문을 작성한다고 하면 
Oracle은 V_RECORD 라는 변수를 바로 참조하여 사용할 수 있지만 
PostgreSQL의 경우 V_RECORD라는 변수를 DECLARE 문에 RECORD 타입으로 미리 선언해주어야 했다.

##### 2. FROM DUAL
테이블의 컬럼을 바라보지 않는 Select 문 사용 시 
Oracle 은 FROM DUAL 문법을 뒤에 붙여주어야 하지만 
PostgreSQL 의 경우 필요하지 않았다.

##### 3. 파이프라인을 이용한 문자열 합치기

```SQL
# Oracle Case
SELECT 'a' || NULL || 'b' FROM DUAL; -- 결과: 'ab'

# Postgresql Case
SELECT 'a' || NULL || 'b'; -- 결과: NULL
```

파이프(||)를 이용한 문자열 합치기를 사용할 경우 Oracle 의 경우 NULL 값을 공백으로 처리하여 합쳐진다. 이는 Oracle 의 경우 공백을 NULL 로 판정하기 때문에 발생하는 결과인 것으로 예상된다.
조회 관련 쿼리에서 '%' 를 문자열의 앞뒤로 붙여 LIKE 검색을 할 경우 파이프(||)를 이용할 시 Oracle과 PostgreSQL의 결과가 완전히 달라지므로 주의해야 한다.  변환 작업 시 CONCAT_WS를 이용하였다.

##### 4. 현재 날짜 및 시간
Oracle 의 경우 SYSDATE, 
PostgreSQL 의 경우 NOW()를 이용하여 현재 시간을 불러올 수 있다.

##### 5. 날짜 연산
Oracle 의 경우 '+', '-' 연산을 이용하여 바로 날짜 계산이 가능하지만, 
Postgresql 의 아래와 같은 문법으로 타입을 명시해야한다.

```SQL
# Oracle Case
SELECT SYSDATE + 1 FROM DUAL;

# Postgresql Case
SELECT NOW() + '1 days'::INTERVAL; -- INTERVAL 타입으로 연산해야한다.

```

##### 6. 데이터 수 제한 검색 문법 및 ORDER BY 주의점
Oracle의 경우 ROWNUM 을 WHERE 절에서 비교하여  SELECT 문의 조회 건수를 제한할 수 있으며
PostgreSQL의 경우 LIMIT 문법을 이용하여 제한하여 조회할 수 있다.

이 때 ORDER BY 절이 포함된 SELECT 문이라면 변환 시 주의해야 한다. 
Oracle의 ROWNUM은 ORDER BY 로 정렬되기 전에 주어지고 WHERE 절에서 제한한다면,
PosgreSQL의 LIMIT의 경우 ORDER BY 된 상태에서 LIMIT 조회가 적용되기 때문이다.
Oracle에서 PostgreSQL로 변환 시에는 인라인뷰에서 ORDER BY  하여 SELECT 문을 작성한 이후 LIMIT 를 걸어야 했다.
```SQL
# Oracle Case
SELECT *
  FROM A_TABLE A
 WHERE ROWNUM <= 10;
 ORDER BY A.A_COLUMN

# PostgreSQL
SELECT *
  FROM (SELECT *
          FROM A_TABLE 
         ORDER BY A_COLUMN) A
 LIMIT 10;

```

##### 7. Alias
Oracle 의 경우 인라인뷰에는 Alias 가 필수가 아니지만
PostgreSQL의 경우 인라인뷰에 Alias 가 필수다.

Oracle의 경우 MERGE, UPDATE, INSERT 문 등에서도 테이블에 Alias를 붙이는 것에 제한사항이 없다.
PostgeSQL의 경우 변경되는 기준이 되는 컬럼에는 Alias를 붙이면 안된다.
```SQL
# Oracle Case
UPDATE A_TABE A
   SET A.A_COLUMN = 'NEW_A';Z

# PostgreSQL
# A.ACOLUMN 이런식으로 쓰면 안된다. 우항에서 값을 참조할때는 Alias 를 사용 가능하다.
UPDATE A_TABE A
   SET A_COLUMN = 'NEW_A';
```

##### 8. 재귀 쿼리
Oracle 의 경우 CONNECT BY LEVEL 문법이 존재하여 재귀 쿼리를 짧게 작성할 수 있다.
PostgreSQL 의 경우 WITH RECURSIVE 문을 이용하여 재귀 쿼리를 작성 해야한다. Oracle에 비하여 쿼리 길이가 길어지며 가독성이 떨어진다고 느꼈다. 쿼리 작성 난이도도 상승한다.

##### 9. SELECT 'STRICT' INTO
Oracle 에서 변수에 값을 담기 위하여 SELECT INTO 문법 사용 시 조회 되는 값이 없을 경우 NO DATA EXCEPTION 이 발생한다.
PostgreSQL의 경우 SELECT INTO 문법 사용 시 조회 되는 값이 없어도 EXCEPTION이 발생하지 않는다. 

Oracle -> PostgreSQL 변환 시 Oracle과 같은 형태로 Exception 발생을 위해선 SELECT INTO STRICT를 사용하여야 한다.


##### 10. Mybatis 관련
Spring Mybatis 에서 Function 을 호출하여 CURSOR 결과를 리턴 받을 시 jdbcType 설정
Oracle의 경우 "CURSOR" 타입으로 설정한 후 CALL 값을 등호로 받는다.
PostgreSQL의 경우 "OTHER" 타입으로 설정한 후 Function의 추가 파라미터로 넣은 후 CALL 한다.

##### 11. 프로시져 실행 시 CASE WHEN 문의 오류 검사
변환 작업 시 CASE WHEN 문을 이용하여 잘못된 결과일 경우 1/0 값을 결과로 하여 강제로 오류를 발생 시키는 로직이 있었다. 

Oracle 의 경우 실제 CASE WHEN 문에서 조건에 해당할 경우 DIVISION BY ZERO 오류가 발생하여 원하는 대로 컨트롤 할 수 있지만.
PostgreSQL의 경우 CASE WHEN 조건에 걸리지 않더라도 바로 DIVISION BY ZERO가 발생하였다. 아마도 PostgreSQL 에선 CASE WHEN 문의 결과 값을 미리 체크하는 형태로 동작하는 것으로 생각되었다. 하지만 1/0의 값을 CASE WHEN 문에 바로 작성하지 않고 별도의 Function 에서 리턴하는 형태로 작성할 경우 오류가 발생하지 않고 조건에 걸리는 경우에만 발생하였다. 따라서 기존 Oracle 프로그램과 같은 형태로 변환하기 위하여 DIVISION BY ZERO 를 리턴할 수 있는 별도의 Function으로 처리하기로 하였다.

##### 12. ROW 위치를 지정하는 논리적 주소 값
Oracle의 경우 Row 마다 위치를 지정하는 물리적 주소 값 ROWID 가 존재한다. UPDATE 시에도 변하지 않는다.
PostgreSQL의 경우 Row 마다 위치를 지정하는 물리적 주소 값 CTID 가 존재한다. UPDATE 시에는 값이 변한다.

Oracle 에서 UPDATE 로직이 있는 ROWID 와 관련된 값을 PosgreSQL로 변환하기 위해선 별도의 Unique 값을 채번해서 사용하여야 한다. 만약 UPDATE 가 없는 경우라면 CTID 값으로 변환하여 적용하여도 문제는 없을 것으로 판단된다.