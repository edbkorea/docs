# Lab: Procedural Languages

## SQL Function (Non procedural)

`IF`, `LOOP`등의 logic 처리가 필요없는 경우 plpgsql 보다 SQL function이 편리하다.

* 간단한 예제
  ```SQL
  CREATE OR REPLACE FUNCTION tf1 (deptno integer, increase_rate numeric) RETURNS numeric AS $$
      UPDATE emp
          SET sal = sal * tf1.increase_rate
          WHERE deptno = tf1.deptno;
      SELECT avg(sal) FROM emp WHERE deptno = tf1.deptno;
  $$ LANGUAGE SQL;
  ```

  ```
  edb=# select tf1(20, 1.2);
  Updating employee 7369
  ..Old salary: 1658.88
  ..New salary: 1990.66
  ..Raise     : 331.78
  Updating employee 7566
  ..Old salary: 6168.96
  ..New salary: 7402.75
  ..Raise     : 1233.79
  Updating employee 7788
  ..Old salary: 6220.80
  ..New salary: 7464.96
  ..Raise     : 1244.16
  Updating employee 7876
  ..Old salary: 2280.96
  ..New salary: 2737.15
  ..Raise     : 456.19
  Updating employee 7902
  ..Old salary: 6220.80
  ..New salary: 7464.96
  ..Raise     : 1244.16
  User enterprisedb updated employee(s) on 2016-02-14
            tf1          
  -----------------------
   5412.0960000000000000
  (1 row)
  ```

* 여러 건의 데이터를 return 하는 함수
여러건의 데이터를 반환하려면 `SETOF` 구문을 이용하면 된다.

  ```sql
  CREATE OR REPLACE FUNCTION distinctdept()
    RETURNS SETOF numeric AS $$
      SELECT distinct deptno from emp;
  $$ LANGUAGE SQL;
  ```

  ```
  edb=# select * from distinctdept();
   distinctdept 
  --------------
             30
             20
             10
  (3 rows)
  ```

* 여러건의 tuple을 return 하는 함수
  여러건의 multiple column tuple을 반환하는 함수는 어떻게 할 수 있을까? 결과 컬럼들을 `OUT` 변수로 지정하고 `SETOF record`를 사용하면 된다..

  ```
  CREATE OR REPLACE FUNCTION deptavgsal(OUT deptno numeric, OUT avgsal numeric(10,2))
    RETURNS SETOF record AS $$
      SELECT deptno, avg(sal) as avgsal FROM emp group by deptno;
  $$ LANGUAGE SQL;
  ```

  ```
  edb=# select * from deptavgsal();
   deptno |        avgsal
  --------+-----------------------
       30 | 1566.6666666666666667
       20 | 5412.0960000000000000
       10 |    35000.000000000000
  (3 rows)
  ```

* 오늘 날짜를 문자열로 가져오는 함수
  빈번하게 사용되는 기능을 함수로 만들어 보자.

  ```
  CREATE OR REPLACE FUNCTION today (format varchar DEFAULT 'YYYYMMDD') RETURNS varchar AS $$
    select to_char(now(), today.format);
  $$ LANGUAGE SQL;
  ```

  ```
  edb=# select today();
    today   
  ----------
   20160214
  (1 row)

  edb=# select today('YYYY/MM/DD');
     today    
  ------------
   2016/02/14
  (1 row)
  ```

## PL/pgsql

### Autonomous Transaction

#### 문제 상황

* 테이블 생성
  ```sql
  create table job_log
  (
  logdate timestamp,
  src varchar(100),
  logmessage text
  );

  create index job_log_idx1 on job_log(logdate);
  ```

* 로깅 함수 생성
  ```sql
  create or replace function log_message(
    message text,
    src varchar(100) DEFAULT 'MAIN'
  ) RETURNS boolean AS $$
  BEGIN
    RAISE NOTICE 'LOGGING MESSAGE: %', message;
    insert into job_log values (now(), src, message);
  RETURN FOUND;
  END;
  $$ LANGUAGE plpgsql;
  ```

* 동작 확인
  ```
  edb=# select log_message('test');
  NOTICE:  LOGGING MESSAGE: test
   log_message 
  -------------
   t
  (1 row)

  edb=# select * from job_log;
            logdate          | src  | logmessage 
  ---------------------------+------+------------
   14-FEB-16 11:45:54.820129 | MAIN | test
  (1 row)
  ```

  정상적으로 동작 하는것 처럼 보인다. 하지만 아래처럼 실제 상황에서는 동작하지 않음을 알 수 있다.

  ```
  create or replace function update_salary (
  empno numeric,
  newsal numeric
  ) RETURNS boolean AS $$
  BEGIN
    if newsal <= 0 then
      PERFORM log_message('Invalid Salary', 'UPDATE_SALARY');
      RAISE EXCEPTION 'Invalid Salary';
    end if;
    update emp set sal = update_salary.newsal where emp.empno = update_salary.empno;
    RETURN FOUND;
  END;
  $$ LANGUAGE plpgsql;
  ```

  아래와 같이 `newsal`에 0을 입력하면 의도한 에러가 발생한다. 그리고 `job_log` 함수가 호출되어 로그가 입력되었다는 메시지를 볼 수 있다.

  ```
  edb=# select update_salary(7900, 0);
  NOTICE:  LOGGING MESSAGE: Invalid Salary
  CONTEXT:  SQL statement "SELECT log_message('Invalid Salary', 'UPDATE_SALARY')"
  PL/pgSQL function update_salary(numeric,numeric) line 4 at PERFORM
  ERROR:  Invalid Salary
  ```

  하지만 아래와 같이 실제 테이블에는 데이터가 없음을 볼 수 있다. 원인이 무엇인가?

#### Workaround

아쉽게도 PostgreSQL/PPAS는 아직까지 autonomous transaction을 지원하지 않는다. 이 경우 아래와 같은 workaround를 통해 처리가 가능하다.

* `log_message` 함수 수정
autonomous transaction은 기존 transaction과 별개의 새로운 transaction을 이용하겠다는 것이기 때문에 사실상 별도의 session을 이용하는 것과 동일하다. 따라서 dblink를 이용하여 쿼리를 수행시키면 원하는 결과를 얻을 수 있다.
  ```
  edb=# CREATE EXTENSION dblink;
  CREATE EXTENSION
  ```

  ```sql
  create or replace function log_message(
    message text,
    src varchar(100) DEFAULT 'MAIN'
  ) RETURNS boolean AS $$
  BEGIN
    RAISE NOTICE 'LOGGING MESSAGE: %', message;
    perform dblink_connect('pragma', 'dbname=edb');
    perform dblink_exec('pragma', format('insert into job_log values (now(), %L, %L);', src, message));
    perform dblink_exec('pragma', 'commit;');
    perform dblink_disconnect('pragma');
  RETURN FOUND;
  END;
  $$ LANGUAGE plpgsql;
  ```

* 동작 확인
  이제 정상적으로 동작함을 확인 할 수 있다.

  ```
  edb=# select update_salary(7900, 0);
  NOTICE:  LOGGING MESSAGE: Invalid Salary
  CONTEXT:  SQL statement "SELECT log_message('Invalid Salary', 'UPDATE_SALARY')"
  PL/pgSQL function update_salary(numeric,numeric) line 4 at PERFORM
  ERROR:  Invalid Salary
  edb=# select * from job_log;
            logdate          |      src      |   logmessage   
  ---------------------------+---------------+----------------
   14-FEB-16 12:26:54.421744 | MAIN          | test
   14-FEB-16 12:27:09.864991 | UPDATE_SALARY | Invalid Salary
  (2 rows)
  ```
