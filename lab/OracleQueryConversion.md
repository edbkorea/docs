# Lab: 오라클 쿼리 변환시 특이 사항 

## `FIRST`/`LAST` 쿼리 변환
### 설명

`AGGR(A) KEEP (DENSE_RANK FIRST/LAST ORDER BY B) ... GROUP BY C` 의 형태로 사용되는 `FIRST`/`LAST` 함수는 다음과 같은 분석을 수행한다.

1. C로 groupping 한 다음
2. B의 최대 또는 최소값을 가지는 record들에 대하여
3. `AGGR(A)`를 계산한다.

아래의 예제를 참고하자.

```
  1  select department_id, salary, commission_pct
  2    from hr.employees
  3*  order by department_id, commission_pct, salary
SQL> /

DEPARTMENT_ID     SALARY COMMISSION_PCT
------------- ---------- --------------
           10       4400
           20       6000
           20      13000
           30       2500
           30       2600
           30       2800
           30       2900
           30       3100
           30      11000
           40       6500
           50       2100
           50       2200
           50       2200
           50       2400
           50       2400
           50       2500
           50       2500
           50       2500
           50       2500
           50       2500
           50       2600
           50       2600
           50       2600
           50       2700
           50       2700
           50       2800
           50       2800
           50       2800
           50       2900
           50       2900
           50       3000
           50       3000
           50       3100
           50       3100
           50       3100
           50       3200
           50       3200

DEPARTMENT_ID     SALARY COMMISSION_PCT
------------- ---------- --------------
           50       3200
           50       3200
           50       3300
           50       3300
           50       3400
           50       3500
           50       3600
           50       3600
           50       3800
           50       3900
           50       4000
           50       4100
           50       4200
           50       5800
           50       6500
           50       7900
           50       8000
           50       8200
           60       4200
           60       4800
           60       4800
           60       6000
           60       9000
           70      10000
           80       6100             .1
           80       6200             .1
           80       6200             .1
           80       6400             .1
           80       6800             .1
           80       7200             .1
           80       7000            .15
           80       7300            .15
           80       7400            .15
           80       9500            .15
           80       7500             .2
           80       8000             .2
           80       8400             .2

DEPARTMENT_ID     SALARY COMMISSION_PCT
------------- ---------- --------------
           80       8600             .2
           80       9600             .2
           80      10000             .2
           80      10500             .2
           80       7000            .25
           80       8800            .25
           80       9000            .25
           80       9500            .25
           80      10500            .25
           80      11500            .25
           80       7500             .3
           80       8000             .3
           80      10000             .3
           80      11000             .3
           80      11000             .3
           80      12000             .3
           80      13500             .3
           80       9000            .35
           80       9500            .35
           80      10000            .35
           80      14000             .4
           90      17000
           90      17000
           90      24000
          100       6900
          100       7700
          100       7800
          100       8200
          100       9000
          100      12008
          110       8300
          110      12008
                    7000            .15

107 rows selected.
```

```
SELECT department_id,
MIN(salary) KEEP (DENSE_RANK FIRST ORDER BY commission_pct) "Worst",
MAX(salary) KEEP (DENSE_RANK LAST ORDER BY commission_pct) "Best"
   FROM employees
   GROUP BY department_id;

DEPARTMENT_ID      Worst       Best
------------- ---------- ----------
           10       4400       4400
           20       6000      13000
           30       2500      11000
           40       6500       6500
           50       2100       8200
           60       4200       9000
           70      10000      10000
           80       6100      14000
           90      17000      24000
          100       6900      12000
          110       8300      12000
                    7000       7000
```

```
SELECT department_id,
MAX(salary) KEEP (DENSE_RANK FIRST ORDER BY commission_pct) "Worst",
MIN(salary) KEEP (DENSE_RANK LAST ORDER BY commission_pct) "Best"
   FROM employees
   GROUP BY department_id;

DEPARTMENT_ID      Worst       Best
------------- ---------- ----------
           10       4400       4400
           20      13000       6000
           30      11000       2500
           40       6500       6500
           50       8200       2100
           60       9000       4200
           70      10000      10000
           80       7200      14000
           90      24000      17000
          100      12000       6900
          110      12000       8300
                    7000       7000

12 rows selected.
```

```
SELECT last_name, department_id, salary,
   MIN(salary) KEEP (DENSE_RANK FIRST ORDER BY commission_pct)
      OVER (PARTITION BY department_id) "Worst",
   MAX(salary) KEEP (DENSE_RANK LAST ORDER BY commission_pct)
      OVER (PARTITION BY department_id) "Best"
    FROM employees
    ORDER BY department_id, salary;

LAST_NAME       DEPARTMENT_ID     SALARY      Worst       Best
--------------- ------------- ---------- ---------- ----------
Whalen                     10       4400       4400       4400
Fay                        20       6000       6000      13000
Hartstein                  20      13000       6000      13000
Colmenares                 30       2500       2500      11000
Himuro                     30       2600       2500      11000
Tobias                     30       2800       2500      11000
Baida                      30       2900       2500      11000
Khoo                       30       3100       2500      11000
Raphaely                   30      11000       2500      11000
Mavris                     40       6500       6500       6500
Olson                      50       2100       2100       8200
Philtanker                 50       2200       2100       8200
Markle                     50       2200       2100       8200
Gee                        50       2400       2100       8200
Landry                     50       2400       2100       8200
Patel                      50       2500       2100       8200
Vargas                     50       2500       2100       8200
Marlow                     50       2500       2100       8200
Perkins                    50       2500       2100       8200
Sullivan                   50       2500       2100       8200
OConnell                   50       2600       2100       8200
Grant                      50       2600       2100       8200
Matos                      50       2600       2100       8200
Seo                        50       2700       2100       8200
Mikkilineni                50       2700       2100       8200
Geoni                      50       2800       2100       8200
Jones                      50       2800       2100       8200
Atkinson                   50       2800       2100       8200
Gates                      50       2900       2100       8200
Rogers                     50       2900       2100       8200
Cabrio                     50       3000       2100       8200
Feeney                     50       3000       2100       8200
Davies                     50       3100       2100       8200
Walsh                      50       3100       2100       8200
Fleaur                     50       3100       2100       8200
Taylor                     50       3200       2100       8200
Stiles                     50       3200       2100       8200

LAST_NAME       DEPARTMENT_ID     SALARY      Worst       Best
--------------- ------------- ---------- ---------- ----------
McCain                     50       3200       2100       8200
Nayer                      50       3200       2100       8200
Mallin                     50       3300       2100       8200
Bissot                     50       3300       2100       8200
Dellinger                  50       3400       2100       8200
Rajs                       50       3500       2100       8200
Dilly                      50       3600       2100       8200
Ladwig                     50       3600       2100       8200
Chung                      50       3800       2100       8200
Everett                    50       3900       2100       8200
Bell                       50       4000       2100       8200
Bull                       50       4100       2100       8200
Sarchand                   50       4200       2100       8200
Mourgos                    50       5800       2100       8200
Vollman                    50       6500       2100       8200
Kaufling                   50       7900       2100       8200
Weiss                      50       8000       2100       8200
Fripp                      50       8200       2100       8200
Lorentz                    60       4200       4200       9000
Austin                     60       4800       4200       9000
Pataballa                  60       4800       4200       9000
Ernst                      60       6000       4200       9000
Hunold                     60       9000       4200       9000
Baer                       70      10000      10000      10000
Kumar                      80       6100       6100      14000
Johnson                    80       6200       6100      14000
Banda                      80       6200       6100      14000
Ande                       80       6400       6100      14000
Lee                        80       6800       6100      14000
Sewall                     80       7000       6100      14000
Tuvault                    80       7000       6100      14000
Marvins                    80       7200       6100      14000
Bates                      80       7300       6100      14000
Smith                      80       7400       6100      14000
Cambrault                  80       7500       6100      14000
Doran                      80       7500       6100      14000
Smith                      80       8000       6100      14000

LAST_NAME       DEPARTMENT_ID     SALARY      Worst       Best
--------------- ------------- ---------- ---------- ----------
Olsen                      80       8000       6100      14000
Livingston                 80       8400       6100      14000
Taylor                     80       8600       6100      14000
Hutton                     80       8800       6100      14000
McEwen                     80       9000       6100      14000
Hall                       80       9000       6100      14000
Greene                     80       9500       6100      14000
Bernstein                  80       9500       6100      14000
Sully                      80       9500       6100      14000
Fox                        80       9600       6100      14000
Bloom                      80      10000       6100      14000
Tucker                     80      10000       6100      14000
King                       80      10000       6100      14000
Zlotkey                    80      10500       6100      14000
Vishney                    80      10500       6100      14000
Cambrault                  80      11000       6100      14000
Abel                       80      11000       6100      14000
Ozer                       80      11500       6100      14000
Errazuriz                  80      12000       6100      14000
Partners                   80      13500       6100      14000
Russell                    80      14000       6100      14000
De Haan                    90      17000      17000      24000
Kochhar                    90      17000      17000      24000
King                       90      24000      17000      24000
Popp                      100       6900       6900      12000
Sciarra                   100       7700       6900      12000
Urman                     100       7800       6900      12000
Chen                      100       8200       6900      12000
Faviet                    100       9000       6900      12000
Greenberg                 100      12000       6900      12000
Gietz                     110       8300       8300      12000
Higgins                   110      12000       8300      12000
Grant                               7000       7000       7000

107 rows selected.
```

### 전환 예시

아래 예시는 이런 류의 분석이 필요할 경우의 변환 예제이다.
하지만 이 함수가 꼭 필요해서 사용되기 보다는 불필요하게 사용되는 경우가 많다. 반드시 이런 류의 분석이 필요한 경우가 아니라면 좀더 직관적인 방식으로 변환하는 것이 더 편할 수 있다.

#### Oracle
```sql
SELECT
     transactionid,
     MAX(COMCreditCode) KEEP (DENSE_RANK FIRST ORDER BY written DESC) comcreditcode
 FROM transactionupdatequeue
 WHERE accountId = 1
 GROUP BY transactionid;
```
#### PAS
```sql
SELECT transactionid, max(COMCreditCode) as COMCreditCode
FROM (
	SELECT transactionid, FIRST_VALUE(written) over w as max_written, written, COMCreditCode
	FROM transactionupdatequeue
    WHERE accountId = 1
	window w as (partition by transactionid order by written desc)
)
WHERE written = max_written or (max_written is null and written is null)
GROUP BY transactionid;
```

## 계층 쿼리

### `CONNECT BY PRIOR`

PAS는 `CONNECT BY PRIOR` 구문을 이용한 계층 쿼리를 지원 하지만 `CONNECT BY`에 해당하는 조건을 2개 이상 사용할 수 없다. 이 경우 예전 방식처럼 `WITH RECURSIVE` 구문을 이용하여야 한다.

`RECURSIVE` 구문의 기본 동작 과정은

1. Non recursive 구문을 먼저 처리하여 working set을 만든다(start with)
2. Working set에 recursive 구문을 적용하여 temp set을 만든다.
3. Temp set을 이용하여 working set을 update한다.
4. Temp set이 아무런 데이터를 리턴하지 않을때 까지 반복한다.

동작 원리는 `CONNECT BY`와 동일하며 recursive part가 언젠가는 아무 데이터도 반환하지 않아야 쿼리가 종료된다.
차이점으로 오라클의 방식은 depth-first 순서로 조회하며 PAS는 breadth-first 방식으로 조회 한다.

#### Oracle

```sql
SELECT value
FROM taxonomy
CONNECT BY PRIOR key = taxHier
START WITH key = 0;
```

```sql
SELECT employee_id, last_name, manager_id, LEVEL
FROM   employees
CONNECT BY PRIOR employee_id = manager_id;
```

#### PAS

```sql
WITH RECURSIVE cte AS (
   SELECT key, value, 1 AS level
   FROM   taxonomy
   WHERE  key = 0
 UNION  ALL
   SELECT t.key, t.value, c.level + 1
   FROM   cte      c
   JOIN   taxonomy t ON t.taxHier = c.key
   )
SELECT value
FROM   cte
ORDER  BY level;
```

```sql
WITH RECURSIVE cte AS (
   SELECT employee_id, last_name, manager_id, 1 AS level
   FROM   employees
 UNION  ALL
   SELECT e.employee_id, e.last_name, e.manager_id, c.level + 1
   FROM   cte c
   JOIN   employees e ON e.manager_id = c.employee_id
   )
SELECT *
FROM   cte;
```

### `SYS_CONNECT_BY_PATH`

`SYS_CONNECT_BY_PATH`를 지원하지 않기 때문에 역시 `WITH RECURSIVE`를 이용하여 처리해야 한다.

```sql
SELECT COMPANY_CODE
      ,COMPANY_NAME
      ,SYS_CONNECT_BY_PATH(COMPANY_CODE,'>') AS FULL_COMPANY_CODE
      ,SYS_CONNECT_BY_PATH(COMPANY_NAME,'>') AS FULL_COMPANY_NAME
  FROM SAMPLE_TABLE
 START WITH COMPANY_CODE = '000001'
CONNECT BY PRIOR COMPANY_CODE = UPPER_COMPANY_CODE
```

```sql
WITH RECURSIVE X(COMPANY_CODE,COMPANY_NAME,ARRPATH,ARRPATH2) AS (
  SELECT COMPANY_CODE
        ,COMPANY_NAME
        ,COMPANY_CODE::TEXT
        ,COMPANY_NAME::TEXT
    FROM SAMPLE_TABLE
   WHERE COMPANY_CODE     = '000001'
 UNION ALL
  SELECT B.COMPANY_CODE
        ,B.COMPANY_NAME
        ,X.ARRPATH||'>'||B.COMPANY_CODE
        ,X.ARRPATH2||'>'||B.COMPANY_NAME
    FROM X, SAMPLE_TABLE B
   WHERE X.COMPANY_CODE = B.UPPER_COMPANY_CODE
)
SELECT COMPANY_CODE
      ,COMPANY_NAME
      ,ARRPATH AS FULL_COMPANY_CODE
      ,ARRPATH2 AS FULL_COMPANY_NAME
  FROM X ;
```

### `CONNECT_BY_ISLEAF`

`CONNECT_BY_ISLEAF`를 변환하기 위해서는 recursive 쿼리의 결과에 대해서 자식이 존재하는지 여부를 다시한번 check할 수 밖에 없다.

#### Oracle

```sql
SELECT ID
     , NEXTID
     , CONNECT_BY_ISLEAF AS ISLEAF
  FROM ISLEAFT
 START WITH NEXTID IS NULL
CONNECT BY PRIOR ID = NEXTID;
```

#### PAS

```sql
WITH RECURSIVE W(ID,NEXTID) AS
(
  SELECT ID,NEXTID
    FROM ISLEAFT
   WHERE NEXTID IS NULL
 UNION ALL
  SELECT B.ID,B.NEXTID
    FROM W, ISLEAFT B
   WHERE W.ID = B.NEXTID
)
SELECT ID
      ,NEXTID
      ,CASE WHEN EXISTS(SELECT 1 FROM ISLEAFT C WHERE W.ID = C.NEXTID) THEN 0 ELSE 1 END AS ISLEAF
  FROM W
 ORDER BY ID;
```

## 기타 구문 변환

### 임의 개수의 행 생성

Row 수를 원하는 만큼 생성할 경우 오라클에서는 Connect by Level 구문을 이용하지만, PPAS에서는 generate_series 함수로 더 편리하게 처리할 수 있다. `FROM` 절이나 Select list에서 모두 가능

#### Oracle

```sql
SQL> select level from dual connect by level <= 5;

LEVEL
----------
         1
         2
         3
         4
         5
```

#### PAS

```sql
test=# select generate_series(1,5) as level ;
 level
-------
     1
     2
     3
     4
     5
test=# select x as level from generate_series(1,5) x;
 level
-------
     1
     2
     3
     4
     5
```

### `MERGE INTO` 쿼리 변환
PAS에서는 `MERGE INTO` 구문을 지원하지 않는다. 9.5 버전에 추가된 `INSERT ON CONFLICT` 구문을 이용하거나 CTE를 이용하여 구현 할 수 있다.

#### Oracle
```sql
MERGE INTO myTable2 m
USING myTable d ON (m.pid = d.pid)
WHEN MATCHED THEN
	UPDATE SET m.sales = m.sales + d.sales , m. status = d.status
WHEN NOT MATCHED THEN
	INSERT VALUES ( d.pid ,d.sales ,'NEW' );
```

#### PAS
##### 9.4 이전 방식
```sql
WITH upsert AS 
(
  UPDATE myTable2 m SET sales = m.sales + d.sales , status = d.status
  FROM myTable d
  WHERE m.pid = d.pid RETURNING m.*
)
INSERT INTO myTable2
SELECT a.pid ,a.sales ,'NEW'
  FROM myTable a
 WHERE a.pid NOT IN ( SELECT b.pid FROM upsert b );
```

##### 9.5 이후
```sql
INSERT INTO myTable2
SELECT a.pid ,a.sales ,'NEW'
  FROM myTable a
ON CONFLICT (pid)
DO UPDATE SET sales = sales + EXCLUDED.sales , status = EXCLUDED.status
```

#### 실습
아래 쿼리를 변환하여 보자.
```sql
create table dept2 (like dept);
insert into dept2 values (30, 'SALES', 'SEOUL'), (50, 'SE', 'JAPAN');

edb=# select * from dept;
 deptno |   dname    |   loc
--------+------------+----------
     10 | ACCOUNTING | NEW YORK
     20 | RESEARCH   | DALLAS
     30 | SALES      | CHICAGO
     40 | OPERATIONS | BOSTON
(4 rows)

edb=# select * from dept2;
 deptno | dname |  loc
--------+-------+-------
     30 | SALES | SEOUL
     50 | SE    | JAPAN
(2 rows)
```

```sql
MERGE INTO dept m
USING dept2 d ON (m.deptno = d.deptno)
WHEN MATCHED THEN
	UPDATE SET m.dname = d.dname, m.loc = d.loc
WHEN NOT MATCHED THEN
	INSERT VALUES (d.deptno, d.dname, d.loc);
```


### PAGING 쿼리 변환

PAS에서도 ROWNUM 문장을 그대로 사용할 수 있으나 native 연산자인 LIMIT 을 사용하는 것을 권고함

#### Oracle

```sql
SELECT * FROM EMP WHERE ROWNUM <=3;

SELECT * FROM (SELECT * FROM EMP ORDER BY SAL) WHERE ROWNUM <=3
```

#### PAS

```sql
SELECT * FROM EMP LIMIT 3;

SELECT * FROM EMP ORDER BY SAL LIMIT 3;
```

### 문자열 상수 사용

PAS에서는 문자열 상수를 `unknown` 형으로 가정한다.

```sql
edb=# select pg_typeof('hello');
 pg_typeof
-----------
 unknown
```

이는 PAS/PostgreSQL의 type system의 특성으로 이 문자열 상수가 어떤 type의 입력값으로도 사용될 수 있기 때문이다. 이로 인해 생기는 골치아픈 문제 중 한가지로 아래와 같은 경우가 있다.

```sql
edb=# select substring('abcdef', 1, 2);     -- 정상
 substring
-----------
 ab
(1 row)

edb=# select substring(t, 1, 2) from (select 'abcdef' as t) t;     -- 에러
ERROR:  failed to find conversion function from unknown to text

edb=# select substring(t, 1, 2) from (select 'abcdef'::text as t) t;     -- 정상
 substring
-----------
 ab
(1 row)
```

Sub-query의 결과가 `unknown` type일 경우 이 값을 이용한 함수 호출이 실패한다. 이 경우 위와 같이 type을 명시적으로 지정해 줘야 한다.

### `WM_CONCAT` / `LISTAGG`

`array_to_string` / `array_agg` 함수를 사용하여 변환 할 수 있다.

#### Oracle
```sql
SELECT DEPTNO
     , LISTAGG(ENAME,',') WITHIN GROUP (ORDER BY DEPTNO)
     , WM_CONCAT(ENAME)
  FROM EMP 
 GROUP BY DEPTNO; 
```

#### PAS

```sql
SELECT DEPTNO
     , ARRAY_TO_STRING(ARRAY_AGG(ENAME ORDER BY DEPTNO),',')
   FROM EMP 
GROUP BY DEPTNO; 
```

오라클의 `LISTAGG`는 `distinct`를 지원하지 않고 `WM_CONCAT`는 `order by`를 지원하지 않는 반면 PAS의 `array_agg`는 둘다 지원하기 때문에 더 간편하게 사용 가능하다.

```sql
edb=# select ARRAY_AGG(job) from emp;
                                                    array_agg
-----------------------------------------------------------------------------------------------------------------
 {CLERK,SALESMAN,SALESMAN,MANAGER,SALESMAN,MANAGER,MANAGER,ANALYST,PRESIDENT,SALESMAN,CLERK,CLERK,ANALYST,CLERK}
(1 row)

edb=# select ARRAY_AGG(distinct job) from emp;
                 array_agg
--------------------------------------------
 {ANALYST,CLERK,MANAGER,PRESIDENT,SALESMAN}
(1 row)

edb=# select ARRAY_AGG(job order by mgr) from emp;
                                                    array_agg
-----------------------------------------------------------------------------------------------------------------
 {ANALYST,ANALYST,SALESMAN,SALESMAN,SALESMAN,CLERK,SALESMAN,CLERK,CLERK,MANAGER,MANAGER,MANAGER,CLERK,PRESIDENT}
(1 row)
```

## 기타

### 숫자형 상수 처리

#### Oracle

Oracle에서는 숫자를 기본적으로 number 형으로 처리한다.

```
SQL> select '2147483647' + 0 from dual;

                          '2147483647'+0
----------------------------------------
                              2147483647

SQL> select '2147483647' + 1 from dual;

                          '2147483647'+1
----------------------------------------
                              2147483648

SQL> select '9223372036854775807' + 0 from dual;

                 '9223372036854775807'+0
----------------------------------------
                     9223372036854775807

SQL> select '9223372036854775807' + 1 from dual;

                 '9223372036854775807'+1
----------------------------------------
                     9223372036854775808
```

#### PAS

PAS에서는 숫자를 크기에 따라 int, bigint, number로 처리한다.
* int: signed 32bit integer(-2147483648 ~ 2147483647)
* bigint: signed 64bit integer(-9223372036854775808 ~ 9223372036854775807)
* number/numeric: 가변길이 숫자

```
edb=# select pg_typeof(2147483647), pg_typeof(2147483648), pg_typeof(9223372036854775808);
 pg_typeof | pg_typeof | pg_typeof
-----------+-----------+-----------
 integer   | bigint    | numeric
(1 row)
```

따라서 문자열 상수와 숫자형 상수의 연산시 아래 내용을 주의하여야 한다.

```
edb=# select '2147483647' + 0 from dual;
  ?column?
------------
 2147483647
(1 row)

edb=# select '2147483647' + 1 from dual;
ERROR:  integer out of range
edb=# select '2147483647' + 1::bigint from dual;
  ?column?
------------
 2147483648
(1 row)
```

```
edb=# select '9223372036854775807' + 0 from dual;
ERROR:  value "9223372036854775807" is out of range for type integer
LINE 1: select '9223372036854775807' + 0 from dual;
               ^
edb=# select '9223372036854775807' + 0::bigint from dual;
      ?column?
---------------------
 9223372036854775807
(1 row)

edb=# select '9223372036854775807' + 1::bigint from dual;
ERROR:  bigint out of range
edb=# select '9223372036854775807' + 1::numeric from dual;
      ?column?
---------------------
 9223372036854775808
(1 row)
```

### 문자열 형 처리

문자열 형 처리는 Oracle과 크게 다르지 않으나 `CHAR`의 경우 오라클과 다르게 뒤에 자동으로 공백 문자가 붙지 않는다.

```
create table test2 (id number, val1 varchar(10), val2 char(10), val3 varchar2(10));
insert into test2 values (1, 'abc', 'abc', 'abc');
insert into test2 values (2, 'abc    ', 'abc    ', 'abc    ');
insert into test2 values (3, '    abc', '    abc', '    abc');
insert into test2 values (4, '  abc  ', '  abc  ', '  abc  ');

```

#### Oracle

```
SQL> select * from test2;
        ID VAL1       VAL2       VAL3
---------- ---------- ---------- ----------
         1 abc        abc        abc
         2 abc        abc        abc
         3     abc        abc        abc
         4   abc        abc        abc

SQL> select id, length(val1), length(val2), length(val3) from test2;
        ID LENGTH(VAL1) LENGTH(VAL2) LENGTH(VAL3)
---------- ------------ ------------ ------------
         1            3           10            3
         2            7           10            7
         3            7           10            7
         4            7           10            7
```

#### PAS

```
edb=# select * from test2;
 id |  val1   |    val2    |  val3
----+---------+------------+---------
  1 | abc     | abc        | abc
  2 | abc     | abc        | abc
  3 |     abc |     abc    |     abc
  4 |   abc   |   abc      |   abc

edb=# select id, length(val1), length(val2), length(val3) from test2;
 id | length | length | length
----+--------+--------+--------
  1 |      3 |      3 |      3
  2 |      7 |      3 |      7
  3 |      7 |      7 |      7
  4 |      7 |      5 |      7
```

### Date type

#### `SYSDATE` & `now()`, `current_timestamp`, `localtimestamp`

  * sysdate는 오라클 호환성 기능으로 제공되어 사용이 가능
  * now() 함수의 경우, 트랜잭션으로 묶이게되면 트랜잭션 동안 트랜잭션이 시작된 시점으로 시간이 고정됨

  ```sql
  select sysdate, now, current_timestamp, localtimestamp;
  select pg_sleep(1);
  select sysdate, now, current_timestamp, localtimestamp;
  ```
  Auto commit 상태
  ```
  -[ RECORD 1 ]-----+--------------------------------
  sysdate           | 16-FEB-16 14:44:09
  now               | 16-FEB-16 14:44:09.28651 +09:00
  current_timestamp | 16-FEB-16 14:44:09.28651 +09:00
  localtimestamp    | 16-FEB-16 14:44:09.286643

  -[ RECORD 1 ]
  pg_sleep |

  -[ RECORD 1 ]-----+---------------------------------
  sysdate           | 16-FEB-16 14:44:10
  now               | 16-FEB-16 14:44:10.287492 +09:00
  current_timestamp | 16-FEB-16 14:44:10.287492 +09:00
  localtimestamp    | 16-FEB-16 14:44:10.287646
  ```
	
  명시적으로 TX를 시작한 경우. `now()`는 TX의 시작시간에 고정됨
  ```
  BEGIN;

  -[ RECORD 1 ]-----+---------------------------------
  sysdate           | 16-FEB-16 14:44:10
  now               | 16-FEB-16 14:44:10.287806 +09:00
  current_timestamp | 16-FEB-16 14:44:10.287899 +09:00
  localtimestamp    | 16-FEB-16 14:44:10.288149

  -[ RECORD 1 ]
  pg_sleep |

  -[ RECORD 1 ]-----+---------------------------------
  sysdate           | 16-FEB-16 14:44:11
  now               | 16-FEB-16 14:44:10.287806 +09:00
  current_timestamp | 16-FEB-16 14:44:11.290238 +09:00
  localtimestamp    | 16-FEB-16 14:44:11.290355

  COMMIT;
  ```

#### Interval truncation

PAS에서는 날짜간의 연산 결과가 interval type인데 interval type은 `trunc()`함수를 사용할 수 없다.

##### Oracle

```sql
SQL> select (sysdate + 1 + 1/24) - sysdate from dual;

(SYSDATE+1+1/24)-SYSDATE
------------------------
	      1.04166667

SQL> select trunc((sysdate + 1 + 1/24) - sysdate) from dual;

TRUNC((SYSDATE+1+1/24)-SYSDATE)
-------------------------------
			      1
```

##### PAS

```sql
edb=# select (sysdate + 1 + 1/24) - sysdate from dual;
    ?column?
----------------
 1 day 01:00:00
(1 row)

edb=# select trunc((sysdate + 1 + 1/24) - sysdate) from dual;
ERROR:  function trunc(interval) does not exist
LINE 1: select trunc((sysdate + 1 + 1/24) - sysdate) from dual;
               ^
HINT:  No function matches the given name and argument types. You might need to add explicit type casts.
```

이 경우 `date_part()` 함수를 이용할 수 있다.

```sql
edb=# select date_part('days', (sysdate + 1 + 1/24) - sysdate) from dual;
 date_part
-----------
         1
(1 row)
```

첫번째 parameter에 `microseconds`, `milliseconds`, `second`, `minute`, `hour`, `day`, `week`, `month`, `quarter`, `year`, `decade`, `century`, `millennium`등을 지정하여 원하는 값을 얻을 수 있다.

### `NULL` 처리

#### `''` 처리

* Null값이 포함된 연산은 결과가 `NULL`
* EDB PAS에서는 오라클과 동일하게 `'TEXT'||NULL` = `'TEXT'`
* 오라클에서는 `''` 값을 NULL로 인식하나 PAS에서는 값 (공백)으로 인식
* 오라클은 Date 형 타입에 입력 시 `''`을 입력하면 `NULL`로 입력되지만, PAS에서는 `''` 사용 불가. 명시적으로 `NULL` 로 입력해야함
* 예시
  ```SQL
  INSERT INTO test VALUES ('', NULL);
  ```

  | SQL                                         | ORACLE | EDB PAS |
  |---------------------------------------------|--------|---------|
  | `select count(*) from test where a = ''`    |    0   |    1    |
  | `select count(*) from test where a is null` |    1   |    0    |
  | `select count(*) from test where b = ''`    |    0   |    0    |
  | `select count(*) from test where b is null` |    1   |    1    |

#### `NULL`과 Index

* EDB PAS에서는 null도 index에 포함 된다.

  ```
  edb=# create table null_idx as select generate_series(1,1000) x;
  SELECT 1000
  edb=#
  edb=# create unique index idx_null_idx on null_idx (x);
  CREATE INDEX
  edb=#
  edb=# insert into null_idx values (null);
  INSERT 0 1
  edb=# insert into null_idx values (null);
  INSERT 0 1
  edb=#
  edb=# explain analyze select * from null_idx where x=1;
                                                           QUERY PLAN
  ----------------------------------------------------------------------------------------------------------------------------
   Index Only Scan using idx_null_idx on null_idx  (cost=0.28..8.29 rows=1 width=4) (actual time=0.049..0.050 rows=1 loops=1)
     Index Cond: (x = 1)
     Heap Fetches: 1
   Planning time: 0.092 ms
   Execution time: 0.087 ms
  (5 rows)

  edb=#
  edb=# explain analyze select * from null_idx where x is null;
                                                           QUERY PLAN
  ----------------------------------------------------------------------------------------------------------------------------
   Index Only Scan using idx_null_idx on null_idx  (cost=0.28..4.29 rows=1 width=4) (actual time=0.021..0.024 rows=2 loops=1)
     Index Cond: (x IS NULL)
     Heap Fetches: 2
   Planning time: 0.070 ms
   Execution time: 0.048 ms
  (5 rows)

  edb=#
  ```

  모든 index type에 해당되는 것은 아니며 각 index type 별로 null 값에 대한 indexing을 지원하는 지 여부는 아래와 같이 확인할 수 있다.
  
  ```
  edb=# select amname, amsearchnulls from pg_am;
   amname | amsearchnulls
  --------+---------------
   btree  | t
   hash   | f
   gist   | t
   gin    | f
   spgist | t
   brin   | t
  (6 rows)

  edb=#
  ```

  - amname : Name of the access method
  - amsearchnulls :  "Does the access method support IS NULL/NOT NULL searches?"
  - btree 타입의 인덱스에서는 인덱스 스캔을 통해서 null 검색이 가능

### 예약어

Ansi SQL 키워드 및 PAS 전용 키워드는 Alias명이나 변수 명으로 직접 사용 불가.

* 예시
  ```
  edb=# select a vacuum, b table from test;
  ERROR:  syntax error at or near "vacuum"
  LINE 1: select a vacuum, b table from test;
                   ^
  edb=# select a as vacuum, b as table, a "vacuum", b "table" from test;
   vacuum | table | vacuum | table
  --------+-------+--------+-------
        1 | test1 |      1 | test1
        2 | test2 |      2 | test2
  (2 rows)
  ```

* 예약어 목록
	http://www.postgresql.org/docs/9.5/static/sql-keywords-appendix.html

### 식별자 대소문자 구별

EDB PAS 역시 오라클과 마찬가지로 테이블, 컬럼 명 등의 식별자에 대해 대소문자를 가리지 않는다.

```
edb=# create table test (a integer, B integer);
CREATE TABLE
edb=# select A, b from Test;
 a | b
---+---
(0 rows)
```

하지만 오라클과 반대로 내부적으로 모든 식별자를 소문자로 저장하기 때문에 `""`를 이용하여 대소문자를 구별하도록 처리한 경우 아래와 같이 차이가 발생한다. 일반적인 경우 문제가 되지 않으나 외부 툴을 이용하여 DDL을 생성하는 경우 식별자가 `""`로 감싸여져 있는 경우가 있으므로 주의가 필요하다.

#### 예제

* 테이블 생성

	```
  create table "test" ("A" integer, "B" integer);
  insert into "test" values (1, 1);
  ```

* Oracle

  오라클은 기본적으로 식별자를 대문자로 처리한다. 따라서 `test`를 조회하는 경우 `"TEST"`를 찾으려고 하지만 `"test"`만 존제하기 때문에 에러가 발생한다.

  ```
  SQL> select * from test;
  select * from test
                *
  ERROR at line 1:
  ORA-00942: table or view does not exist

  SQL> select * from "test";

     A	    B
  ---------- ----------
     1	    1

  SQL> select * from "TEST";
  select * from "TEST"
                *
  ERROR at line 1:
  ORA-00942: table or view does not exist

  SQL> select a, b from "test";

     A	    B
  ---------- ----------
     1	    1

  SQL> select "A", "B" from "test";

     A	    B
  ---------- ----------
     1	    1

  SQL> select "a", "b" from "test";
  select "a", "b" from "test"
              *
  ERROR at line 1:
  ORA-00904: "b": invalid identifier
  ```

* PAS
	PAS는 기본적으로 소문자로 처리하며 `test`를 조회하는 경우 `"test"`를 찾으려고 한다.

  ```
  edb=# select * from test;
   A | B
  ---+---
   1 | 1
  (1 row)

  edb=# select * from "test";
   A | B
  ---+---
   1 | 1
  (1 row)

  edb=# select * from "TEST";
  ERROR:  relation "TEST" does not exist
  LINE 1: select * from "TEST";
                        ^
  edb=# select a, b from "test";
  ERROR:  column "a" does not exist
  LINE 1: select a, b from "test";
                 ^
  edb=# select "A", "B" from "test";
   A | B
  ---+---
   1 | 1
  (1 row)

  edb=# select "a", "b" from "test";
  ERROR:  column "a" does not exist
  LINE 1: select "a", "b" from "test";
                 ^
  ```

## PostgreSQL 고유 기능
### Domain Data Type

**예제**: TEST1 과 TEST2 테이블의 컬럼은 아래 CHECK 제약 조건에 의해 VARCHAR 형이지만 2자리 숫자 타입의 값만 받아들여야함!

```
edb=# CREATE TABLE TEST1 (COL1 varchar(2)  CHECK (COL1 ~'[[:digit:]]{2}'));  -- 정규식 (regular expression)
CREATE TABLE
edb=#
edb=# CREATE TABLE TEST2 (COL1 varchar(2)  CHECK (COL1 ~'[[:digit:]]{2}'));
CREATE TABLE
edb=#
edb=# INSERT INTO TEST1 VALUES('01');
INSERT 0 1
edb=#
edb=# INSERT INTO TEST2 VALUES('12');
INSERT 0 1
edb=#
edb=# INSERT INTO TEST1 VALUES('9'); -- check 조건에 의해 에러발생
ERROR:  new row for relation "test1" violates check constraint "test1_col1_check"
DETAIL:  Failing row contains (9).
edb=#
edb=# SELECT * FROM TEST1;
col1
------
01
(1 row)

edb=# SELECT * FROM TEST2;
col1
------
12
(1 row)

edb=#
```

위 CHECK 조건을 만족하는 컬럼값이 많은 테이블에서 정의되어야 한다면, 매번 CHECK 조건을 적어주기 매우 번거로움. 이때 DOMAIN data type을 활용

```
edb=# CREATE DOMAIN DIGITV AS VARCHAR(2) CHECK( VALUE ~'[[:digit:]]{2}');
CREATE DOMAIN
edb=#
edb=# DROP TABLE TEST1;
DROP TABLE
edb=# DROP TABLE TEST2;
DROP TABLE
edb=#
edb=# CREATE TABLE TEST1(COL1 DIGITV);
CREATE TABLE
edb=# CREATE TABLE TEST2(COL1 DIGITV);
CREATE TABLE
edb=#
edb=# INSERT INTO TEST1 VALUES('02');
INSERT 0 1
edb=# INSERT INTO TEST2 VALUES('13');
INSERT 0 1
edb=#
edb=# \d test1
    Table "public.test1"
 Column |  Type  | Modifiers
--------+--------+-----------
 col1   | digitv |
edb=#
edb=# \d test2
    Table "public.test2"
 Column |  Type  | Modifiers
--------+--------+-----------
 col1   | digitv |
edb=#
```

### UPDATE Returning

```
edb=# create table emp2 as select * from emp;
edb=#
edb=# select * from emp2;
 empno | ename  |    job    | mgr  |      hiredate      |   sal   |  comm   | deptno
-------+--------+-----------+------+--------------------+---------+---------+--------
  7369 | SMITH  | CLERK     | 7902 | 17-DEC-80 00:00:00 |  800.00 |         |     20
  7499 | ALLEN  | SALESMAN  | 7698 | 20-FEB-81 00:00:00 | 1600.00 |  300.00 |     30
  7521 | WARD   | SALESMAN  | 7698 | 22-FEB-81 00:00:00 | 1250.00 |  500.00 |     30
  7566 | JONES  | MANAGER   | 7839 | 02-APR-81 00:00:00 | 2975.00 |         |     20
  7654 | MARTIN | SALESMAN  | 7698 | 28-SEP-81 00:00:00 | 1250.00 | 1400.00 |     30
  7698 | BLAKE  | MANAGER   | 7839 | 01-MAY-81 00:00:00 | 2850.00 |         |     30
  7782 | CLARK  | MANAGER   | 7839 | 09-JUN-81 00:00:00 | 2450.00 |         |     10
  7788 | SCOTT  | ANALYST   | 7566 | 19-APR-87 00:00:00 | 3000.00 |         |     20
  7839 | KING   | PRESIDENT |      | 17-NOV-81 00:00:00 | 5000.00 |         |     10
  7844 | TURNER | SALESMAN  | 7698 | 08-SEP-81 00:00:00 | 1500.00 |    0.00 |     30
  7876 | ADAMS  | CLERK     | 7788 | 23-MAY-87 00:00:00 | 1100.00 |         |     20
  7900 | JAMES  | CLERK     | 7698 | 03-DEC-81 00:00:00 |  950.00 |         |     30
  7902 | FORD   | ANALYST   | 7566 | 03-DEC-81 00:00:00 | 3000.00 |         |     20
  7934 | MILLER | CLERK     | 7782 | 23-JAN-82 00:00:00 | 1300.00 |         |     10
(14 rows)

edb=#
edb=# update emp2 set sal=sal+100 where sal < 1000 returning *;
 empno | ename |  job  | mgr  |      hiredate      |   sal   | comm | deptno
-------+-------+-------+------+--------------------+---------+------+--------
  7369 | SMITH | CLERK | 7902 | 17-DEC-80 00:00:00 |  900.00 |      |     20
  7900 | JAMES | CLERK | 7698 | 03-DEC-81 00:00:00 | 1050.00 |      |     30
(2 rows)

UPDATE 2
edb=#
```

오라클 경우에도 UPDATE .. RETURNING 문을 지원하나 PL/SQL 등에서만 사용해야함

### Distinct On

a 컬럼에 대해 distinct 한 값 로우만을 가져오는 동시에, b 컬럼까지 한번에 select 가능

```
edb=# create table dist_on (a varchar, b int);
CREATE TABLE
edb=#
edb=# insert into dist_on values ('lion', 1);
INSERT 0 1
edb=# insert into dist_on values ('lion', 2);
INSERT 0 1
edb=# insert into dist_on values ('tiger', 2);
INSERT 0 1
edb=# insert into dist_on values ('tiger', 1);
INSERT 0 1
edb=# insert into dist_on values ('rabbit', 1);
INSERT 0 1
edb=# insert into dist_on values ('rabbit', 2);
INSERT 0 1
edb=# insert into dist_on values ('rabbit', 1);
INSERT 0 1
edb=#
edb=# table dist_on ;
   a    | b
--------+---
 lion   | 1
 lion   | 2
 tiger  | 2
 tiger  | 1
 rabbit | 1
 rabbit | 2
 rabbit | 1
(7 rows)

edb=#
edb=# select distinct a, b from dist_on order by 1,2;
   a    | b
--------+---
 lion   | 1
 lion   | 2
 rabbit | 1
 rabbit | 2
 tiger  | 1
 tiger  | 2
(6 rows)

edb=#
edb=# select distinct on (a) a, b from dist_on order by 1,2;
   a    | b
--------+---
 lion   | 1
 rabbit | 1
 tiger  | 1
(3 rows)

edb=#
```

### Regular Expression

```
edb=# create table reg_exp (a varchar);
CREATE TABLE
edb=#
edb=# insert into reg_exp values ('가나다');
INSERT 0 1
edb=# insert into reg_exp values (100);
INSERT 0 1
edb=# insert into reg_exp values (2);
INSERT 0 1
edb=# insert into reg_exp values ('a한글123');
INSERT 0 1
edb=# insert into reg_exp values ('999test');
INSERT 0 1
edb=# insert into reg_exp values ('a89adf33');
INSERT 0 1
edb=#
edb=# table reg_exp ;
    a
----------
 가나다
 100
 2
 a한글123
 999test
 a89adf33
(6 rows)

edb=#
edb=# -- 로우중에서 숫자가 하나라도 들어가 있는 로우만 추출
edb=# select * from reg_exp where a ~ '[0-9]+';
    a
----------
 100
 2
 a한글123
 999test
 a89adf33
(5 rows)

edb=#
edb=# -- 숫자가 하나도 들어가 있지 않은 로우만 추출
edb=# select * from reg_exp where a !~ '[0-9]+';
   a
--------
 가나다
(1 row)

edb=#
edb=# -- 데이터중 숫자부분만 추출
edb=# select a, regexp_matches (a, '[0-9]+') from reg_exp;
    a     | regexp_matches
----------+----------------
 100      | {100}
 2        | {2}
 a한글123  | {123}
 999test  | {999}
 a89adf33 | {89}
(5 rows)

edb=#
edb=# -- regexp_matches는 배열로 값을 리턴. 배열중 밸류값만 추출
edb=# select a, (regexp_matches (a, '[0-9]+'))[1] from reg_exp;
    a     | regexp_matches
----------+----------------
 100      | 100
 2        | 2
 a한글123  | 123
 999test  | 999
 a89adf33 | 89
(5 rows)

edb=#
edb=# -- 데이터중 모든 숫자부분 추출
edb=# select a, regexp_matches (a, '[0-9]+', 'g') from reg_exp;
    a     | regexp_matches
----------+----------------
 100      | {100}
 2        | {2}
 a한글123  | {123}
 999test  | {999}
 a89adf33 | {89}
 a89adf33 | {33}
(6 rows)

edb=#
edb=# -- 배열에서 밸류만 추출
edb=# select a, (regexp_matches (a, '[0-9]+', 'g'))[1] from reg_exp;
    a     | regexp_matches
----------+----------------
 100      | 100
 2        | 2
 a한글123  | 123
 999test  | 999
 a89adf33 | 89
 a89adf33 | 33
(6 rows)

edb=#
edb=# -- 숫자만으로 이루어진 로우만 필터링
edb=# select a from reg_exp where a ~'^[0-9]+$';
  a
-----
 100
 2
(2 rows)

edb=#
```
