# Lab: MVCC

## Transaction Isolation

### Read Commited
1. 준비

  ```sql
  drop table test;

  create table test (val timestamp);

  insert into test values (now());
  insert into test values (now());
  insert into test values (now());
  insert into test values (now());
  insert into test values (now());
  ```

2. 2개의 psql 세션을 열고 `READ COMMITTED`로 transaction을 시작 한다.

  ```sql
  -- 1번 세션
  BEGIN WORK;

  -- 2번 세션
  BEGIN WORK;
  ```

3. 두 세션 모두에서 테이블의 데이터를 확인 한다.
  * Session 1

    ```
    edb=# select * from test;
                val            
    ---------------------------
     09-FEB-16 13:47:49.72599
     09-FEB-16 13:47:49.727748
     09-FEB-16 13:47:49.729081
     09-FEB-16 13:47:49.730509
     09-FEB-16 13:47:51.831905
    (5 rows)
    ```

  * Session 2

    ```
    edb=# select * from test;
                val            
    ---------------------------
     09-FEB-16 13:47:49.72599
     09-FEB-16 13:47:49.727748
     09-FEB-16 13:47:49.729081
     09-FEB-16 13:47:49.730509
     09-FEB-16 13:47:51.831905
    (5 rows)
    ```

4. 1번 세션에서 데이터를 변경한 뒤 데이터를 확인 한다.
  * Session 1

    ```
    edb=# update test set val = now();
    UPDATE 5
    edb=# select * from test;         
                val            
    ---------------------------
     09-FEB-16 13:48:01.462116
     09-FEB-16 13:48:01.462116
     09-FEB-16 13:48:01.462116
     09-FEB-16 13:48:01.462116
     09-FEB-16 13:48:01.462116
    (5 rows)
    ```

  * Session 2

    ```
    edb=# select * from test;
                val            
    ---------------------------
     09-FEB-16 13:47:49.72599
     09-FEB-16 13:47:49.727748
     09-FEB-16 13:47:49.729081
     09-FEB-16 13:47:49.730509
     09-FEB-16 13:47:51.831905
    (5 rows)
    ```

5. 1번 세션에서 commit을 한 다음 다시 데이터를 확인 한다.
  * Session 1

    ```
    edb=# commit;
    COMMIT
    edb=# select * from test;
                val            
    ---------------------------
     09-FEB-16 13:48:01.462116
     09-FEB-16 13:48:01.462116
     09-FEB-16 13:48:01.462116
     09-FEB-16 13:48:01.462116
     09-FEB-16 13:48:01.462116
    (5 rows)
    ```

  * Session 2

    ```
    edb=# select * from test;
                val            
    ---------------------------
     09-FEB-16 13:48:01.462116
     09-FEB-16 13:48:01.462116
     09-FEB-16 13:48:01.462116
     09-FEB-16 13:48:01.462116
     09-FEB-16 13:48:01.462116
    (5 rows)
    ```

6. 결과에 대해 분석해 보자.

### Repeatable Read
1. 준비

  ```sql
  drop table test;

  create table test (val timestamp);

  insert into test values (now());
  insert into test values (now());
  insert into test values (now());
  insert into test values (now());
  insert into test values (now());
  ```

2. 2개의 psql 세션을 열고 1번 세션은 `READ COMMITED`로 1번 세션은 `REPEATABLE READ`로 transaction을 시작 한다.

  ```sql
  -- 1번 세션
  BEGIN WORK;

  -- 2번 세션
  BEGIN WORK ISOLATION LEVEL REPEATABLE READ;
  ```

3. 위의 테스트를 반복하면서 차이점을 확인 한다.

4. 결과에 대해 분석해 보자.

### Read Commited VS Repeatable Read
1. 준비

  ```sql
  drop table test;

  create table test (val integer);

  insert into test values (1);
  insert into test values (2);
  ```

2. 2개의 psql 세션을 열고 transaction을 시작 한다.

  ```sql
  -- 1번 세션
  BEGIN WORK;

  -- 2번 세션
  BEGIN WORK;
  ```

3. 1번 세션에서 아래와 같이 값을 update한다.

  ```sql
  update test set val = val + 1;
  ```

4. 2번 세션에서 아래와 같이 값을 delete한다.

  ```sql
  delete test where val = 1;
  ```
  이 쿼리는 lock이 걸린다. 원인은 무엇인가?

5. 1번 세션에서 commit을 한다.

  ```sql
  commit;
  ```
  예상되는 결과는 무엇인가?

6. 2번 세션도 commit을 한다.

7. 이 테스트를 2번 세션을 `REPEATABLE READ`로 시작한 다음 반복한다.

8. 차이점에 대하여 고찰 해 본다.

### Serializable - 1

1. 준비

  ```sql
  drop table test;
  create table test
  (
    id int not null primary key,
    color text not null
  );
  insert into dots
    with x(id) as (select generate_series(1,10))
    select id, case when id % 2 = 1 then 'black'
      else 'white' end from x;
  ```

2. 2개의 psql 세션을 `SERIALIZABLE`로 시작한다.

  ```sql
  BEGIN WORK ISOLATION LEVEL SERIALIZABLE;
  ```

3. 서로 다른 세션에서 아래 쿼리를 각각 실행한다.

  ```sql
  -- 1번 세션
  update dots set color = 'black'
    where color = 'white';
  -- 2번 세션
  update dots set color = 'white'
    where color = 'black';
  ```

4. 각 세션에서 결과를 확인한다.

5. 한 세션씩 `commit`을 하면서 결과를 확인한다.

### Serializable - 2

1. 준비

  ```sql
  drop table mytab;
  CREATE TABLE mytab
  (
    class int NOT NULL,
    value int NOT NULL
  );
  INSERT INTO mytab VALUES
  (1, 10), (1, 20), (2, 100), (2, 200);
  ```

2. 2개의 세션에서 `SERIALIZABLE` TX를 시작한다.

  ```sql
  BEGIN WORK ISOLATION LEVEL SERIALIZABLE;
  ```

3. 각 세션에서 다른 데이터를 update한다.

  ```sql
  -- 1번 세션
	INSERT INTO mytab select 2, sum(value) from mytab where class = 1;

  -- 2번 세션
	INSERT INTO mytab select 1, sum(value) from mytab where class = 2;
  ```

4. 각 세션을 하나씩 commit하면서 결과를 확인한다.

  에러가 발생한 원인이 무엇인가?

5. TX를 `REPEATABLE READ`로 이 테스트를 반복한 다음 결과를 비교해 보자.

  ```sql
  BEGIN WORK ISOLATION LEVEL REPEATABLE READ;
  ```

## MVCC

### `xmin`

```
edb=# select xmin, xmax, cmin, cmax, * from mytab;
 xmin | xmax | cmin | cmax | class | value
------+------+------+------+-------+-------
 1964 |    0 |    0 |    0 |     1 |    10
 1964 |    0 |    0 |    0 |     1 |    20
 1964 |    0 |    0 |    0 |     2 |   100
 1964 |    0 |    0 |    0 |     2 |   200
 1965 |    0 |    0 |    0 |     2 |    30
 1967 |    0 |    0 |    0 |     2 |    30
 1969 |    0 |    0 |    0 |     2 |    30
(7 rows)

edb=# begin;
BEGIN
edb=# select txid_current();
 txid_current
--------------
         1970
(1 row)

edb=# insert into mytab values (3, 10);
INSERT 0 1
edb=# insert into mytab values (3, 11);           
INSERT 0 1
edb=# insert into mytab values (3, 12);
INSERT 0 1
edb=# insert into mytab values (3, 13);
INSERT 0 1
edb=# select xmin, xmax, cmin, cmax, * from mytab;
 xmin | xmax | cmin | cmax | class | value
------+------+------+------+-------+-------
 1964 |    0 |    0 |    0 |     1 |    10
 1964 |    0 |    0 |    0 |     1 |    20
 1964 |    0 |    0 |    0 |     2 |   100
 1964 |    0 |    0 |    0 |     2 |   200
 1965 |    0 |    0 |    0 |     2 |    30
 1967 |    0 |    0 |    0 |     2 |    30
 1969 |    0 |    0 |    0 |     2 |    30
 1970 |    0 |    0 |    0 |     3 |    10
 1970 |    0 |    1 |    1 |     3 |    11
 1970 |    0 |    2 |    2 |     3 |    12
 1970 |    0 |    3 |    3 |     3 |    13
(11 rows)

edb=# commit;
COMMIT
```

### `xmax`

1. 두개의 세션에서 TX를 시작하고 현재 txid를 확인한다.

  * 1번 세션

    ```
    edb=# BEGIN WORK ISOLATION LEVEL REPEATABLE READ;
    BEGIN
    edb=# select txid_current();
     txid_current
    --------------
             1973
    (1 row)
    ```

  * 2번 세션

    ```
    edb=# BEGIN WORK ISOLATION LEVEL REPEATABLE READ;
    BEGIN
    edb=# select txid_current();
     txid_current
    --------------
             1974
    (1 row)
    ```

2. 두 세션에서 hidden column들의 값을 확인한다.

  ```
  edb=# select xmin, xmax, cmin, cmax, * from mytab;
   xmin | xmax | cmin | cmax | class | value
  ------+------+------+------+-------+-------
   1964 |    0 |    0 |    0 |     1 |    10
   1964 |    0 |    0 |    0 |     1 |    20
   1964 |    0 |    0 |    0 |     2 |   100
   1964 |    0 |    0 |    0 |     2 |   200
   1965 |    0 |    0 |    0 |     2 |    30
   1967 |    0 |    0 |    0 |     2 |    30
   1969 |    0 |    0 |    0 |     2 |    30
   1970 |    0 |    0 |    0 |     3 |    10
   1970 |    0 |    1 |    1 |     3 |    11
   1970 |    0 |    2 |    2 |     3 |    12
   1970 |    0 |    3 |    3 |     3 |    13
  (11 rows)
  ```

3. 1번 세션에서 데이터를 삭제하고 2번 세션에서 값을 다시 확인 한다.

  * 1번 세션

    ```
    edb=# delete mytab where class = 3;
    DELETE 4
    edb=# select xmin, xmax, cmin, cmax, * from mytab;
     xmin | xmax | cmin | cmax | class | value
    ------+------+------+------+-------+-------
     1964 |    0 |    0 |    0 |     1 |    10
     1964 |    0 |    0 |    0 |     1 |    20
     1964 |    0 |    0 |    0 |     2 |   100
     1964 |    0 |    0 |    0 |     2 |   200
     1965 |    0 |    0 |    0 |     2 |    30
     1967 |    0 |    0 |    0 |     2 |    30
     1969 |    0 |    0 |    0 |     2 |    30
    (7 rows)
    ```

  * 2번 세션

    ```
    edb=# select xmin, xmax, cmin, cmax, * from mytab;
     xmin | xmax | cmin | cmax | class | value
    ------+------+------+------+-------+-------
     1964 |    0 |    0 |    0 |     1 |    10
     1964 |    0 |    0 |    0 |     1 |    20
     1964 |    0 |    0 |    0 |     2 |   100
     1964 |    0 |    0 |    0 |     2 |   200
     1965 |    0 |    0 |    0 |     2 |    30
     1967 |    0 |    0 |    0 |     2 |    30
     1969 |    0 |    0 |    0 |     2 |    30
     1970 | 1973 |    0 |    0 |     3 |    10
     1970 | 1973 |    0 |    0 |     3 |    11
     1970 | 1973 |    0 |    0 |     3 |    12
     1970 | 1973 |    0 |    0 |     3 |    13
     (11 rows)
    ```

4. 두 세션을 모두 commit 하고 결과를 다시 확인한다.
