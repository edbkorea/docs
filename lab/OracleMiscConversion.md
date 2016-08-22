# Lab: 기타 Oracle 변환 특이사항

## PL/SQL안에서의 COMMIT, ROLLBACK

PAS에서는 PL/SQL 내부에서 commit, rollback시 동작 방식에 약간의 차이점이 있다.


### 예제

```
create table x(x int primary key);
insert into x values(100);
commit;
```
```
edb=# select * from x;
 x
---
 100
(1 row)
```

```plsql
BEGIN
    DELETE DEPT WHERE DEPTNO >= 50;
    COMMIT;

    BEGIN
        INSERT INTO dept VALUES (50, 'FINANCE', 'DALLAS');
        INSERT INTO dept VALUES (60, 'MARKETING', 'CHICAGO');
        ROLLBACK;
        INSERT INTO dept VALUES (60, 'FINANCE', 'DALLAS');
        COMMIT;
        INSERT INTO dept VALUES (70, 'MARKETING', 'CHICAGO');
        INSERT INTO dept VALUES (80, 'HUMAN RESOURCES', 'CHICAGO');
    EXCEPTION
        WHEN OTHERS THEN
            DBMS_OUTPUT.PUT_LINE('SQLERRM: ' || SQLERRM);
            DBMS_OUTPUT.PUT_LINE('SQLCODE: ' || SQLCODE);
    END;
END;
```

### 실행 결과

#### Oracle
```
  1  BEGIN
  2	 DELETE X;
  3	 COMMIT;
  4
  5	 BEGIN
  6	     INSERT INTO X VALUES (1);
  7	     INSERT INTO X VALUES (2);
  8	     ROLLBACK;
  9	     INSERT INTO X VALUES (3);
 10	     COMMIT;
 11	     INSERT INTO X VALUES (4);
 12	     INSERT INTO X VALUES (3);
 13	 EXCEPTION
 14	     WHEN OTHERS THEN
 15		 DBMS_OUTPUT.PUT_LINE('SQLERRM: ' || SQLERRM);
 16		 DBMS_OUTPUT.PUT_LINE('SQLCODE: ' || SQLCODE);
 17	 END;
 18* END;
SQL> /
SQLERRM: ORA-00001: unique constraint (SYS.SYS_C0011851) violated
SQLCODE: -1

PL/SQL procedure successfully completed.

SQL> select * from x;
	 X
----------
	 3
	 4

SQL> rollback;
Rollback complete.

SQL> select * from x;
	 X
----------
	 3
```
Error가 발생하여도 commit 되지 않은 transaction은 그대로 남아 있다.

#### PAS

* Default

  ```
  edb=# BEGIN
  edb$#     DELETE X;
  edb$#     COMMIT;
  edb$#
  edb$#     BEGIN
  edb$#         INSERT INTO X VALUES (1);
  edb$#         INSERT INTO X VALUES (2);
  edb$#         ROLLBACK;
  edb$#         INSERT INTO X VALUES (3);
  edb$#         COMMIT;
  edb$#         INSERT INTO X VALUES (4);
  edb$#         INSERT INTO X VALUES (3);
  edb$#     EXCEPTION
  edb$#         WHEN OTHERS THEN
  edb$#             DBMS_OUTPUT.PUT_LINE('SQLERRM: ' || SQLERRM);
  edb$#             DBMS_OUTPUT.PUT_LINE('SQLCODE: ' || SQLCODE);
  edb$#     END;
  edb$# END;
  SQLERRM: duplicate key value violates unique constraint "x_pkey"
  SQLCODE: -1

  EDB-SPL Procedure successfully completed
  edb=# select * from x;
   x
  ---
   3
  (1 row)

  ```

  psql은 기본적으로 autocommit이므로 PL/SQL 블럭이 종료됨과 동식에 transaction이 COMMIT/ROLLBACK된다. 위의 경우 error가 발생하였으므로 transaction이 abort되었고 마지막으로 COMMIT된 변경 까지만 유지되어 있다.

* `edb_stmt_level_tx = on`

  ```
  edb=# \set AUTOCOMMIT off
  edb=# SET edb_stmt_level_tx TO on;
  SET
  edb=# BEGIN
  edb$#     DELETE X;
  edb$#     COMMIT;
  edb$#
  edb$#     BEGIN
  edb$#         INSERT INTO X VALUES (1);
  edb$#         INSERT INTO X VALUES (2);
  edb$#         ROLLBACK;
  edb$#         INSERT INTO X VALUES (3);
  edb$#         COMMIT;
  edb$#         INSERT INTO X VALUES (4);
  edb$#         INSERT INTO X VALUES (3);
  edb$#     EXCEPTION
  edb$#         WHEN OTHERS THEN
  edb$#             DBMS_OUTPUT.PUT_LINE('SQLERRM: ' || SQLERRM);
  edb$#             DBMS_OUTPUT.PUT_LINE('SQLCODE: ' || SQLCODE);
  edb$#     END;
  edb$# END;
  SQLERRM: duplicate key value violates unique constraint "x_pkey"
  SQLCODE: -1

  EDB-SPL Procedure successfully completed
  edb=# select * from x;
   x
  ---
   3
   4
  (2 rows)

  edb=# rollback;
  ROLLBACK
  edb=# select * from x;
   x
  ---
   3
  (1 row)
  ```

  `edb_stmt_level_tx = on`이 적용되면 오라클과 유사한 형태로 동작하게 된다. 하지만 `edb_stmt_level_tx`는 성능에 영향을 미치기 때문에 반드시 필요한 경우에만 사용하여야 한다.
  PL/SQL에서 에러가 발생한다면 commit되지 않은 변경을 ROLLBACK하는 것이 당연하기 때문에 대부분의 경우 기본 설정으로 충분하다.

## JDBC CallableStatement

JDBC에서 stored procedure를 호출하는 방식은 여러가지가 있지만 `CallableStatement`를 이용할 경우 차이점이 있다.

### 호출 형식

원래 JDBC의 표준 형식은 대괄호를 이용하는 방식이다.

```java
CallableStatement stmt = conn.prepareCall("{call empInsert(?,?,?,?,?,?)}");
```

오라클의 경우 아래와 같은 방식도 지원하지만 PAS에서는 대괄호를 이용한 방식만 지원하기 때문에 표준 방식을 이용하여야 한다.

```java
CallableStatement stmt = conn.prepareCall("BEGIN; empInsert(?,?,?,?,?,?); END;");
```

### Out 변수 사용

EDB사의 JDBC 드라이버(`edb-jdbcXX.jar`)를 사용하는 경우 아래와 같이 표준 방식으로 사용 가능하다. 옛날에 만들어진 코드 중 `java.sql.Types`가 아니라 `oracle.jdbc.OracleTypes`를 이용한 코드가 있다면 `java.sql.Types`를 이용하도록 변경 해 주어야 한다.

```java
public void CallSample4(Connection con)
{
  try
  {
    Console c = System.console();
    String commandText = "{call emp_query(?,?,?,?,?,?)}";
    CallableStatement stmt = con.prepareCall(commandText);
    stmt.setInt(1, new Integer(c.readLine("Department No:")));
    stmt.setInt(2, new Integer(c.readLine("Employee No:")));
    stmt.setString(3, new String(c.readLine("Employee Name:")));
    stmt.registerOutParameter(2, Types.INTEGER);
    stmt.registerOutParameter(3, Types.VARCHAR);
    stmt.registerOutParameter(4, Types.VARCHAR);
    stmt.registerOutParameter(5, Types.TIMESTAMP);
    stmt.registerOutParameter(6, Types.NUMERIC);
    stmt.execute();
    System.out.println("Employee No: " + stmt.getInt(2));
    System.out.println("Employee Name: " + stmt.getString(3));
    System.out.println("Job : " + stmt.getString(4));
    System.out.println("Hiredate : " + stmt.getTimestamp(5));
    System.out.println("Salary : " + stmt.getBigDecimal(6));
  }
  catch(Exception err)
  {
    System.out.println("An error has occurred.");
    System.out.println("See full details below.");
    err.printStackTrace();
  }
}
```

### Out 변수에 `REFCURSOR` 사용

Out 변수에 `REFCURSOR` type을 사용하여 여러건의 데이터를 반환할 수 있다. 주의할 점은 아래와 같다.

1. 반드시 no autocommit 상태여야 한다.
2. OUT parameter의 type은 `Type.REF`를 사용한다.

* 예제 코드
  ```SQL
  CREATE OR REPLACE
  PROCEDURE get_objects (p_owner    IN  all_objects.owner%TYPE,
                        p_recordset OUT REFCURSOR) AS
  BEGIN
    OPEN p_recordset FOR
      SELECT object_type, object_name
      FROM   all_objects
      WHERE  owner = p_owner
      ORDER BY object_name;
  END get_objects;
  /
  ```
  
  ```java
  conn.setAutoCommit(false);
  CallableStatement stmt = conn.prepareCall("{CALL get_objects(?, ?)}");

  stmt.setString(1, "ENTERPRISEDB");
  stmt.registerOutParameter(2, Types.REF);

  stmt.execute();

  ResultSet rs;

  rs = (java.sql.ResultSet)stmt.getObject(2);
  while (rs.next()) {
    System.out.println(rs.getString(1) + ":" + rs.getString(2));
  }

  rs.close();
  stmt.close();
  ```

## psql에서 bind 변수 포함 쿼리 실행 및 실행 계획 확인

오라클과 다르게 PAS 에서는 바인드 변수기호로 $ 를 사용하는데 이런 쿼리를 실행하거나 실행 계획을 확인하려면 `prepare` 구문을 이용하여야 한다.

```
scottdb=> explain select * from emp where empno=$1;
ERROR:  there is no parameter $1
LINE 1: explain select * from emp where empno=$1;
```

```
scottdb=> prepare stmt(int) as select * from emp where empno=$1;
PREPARE
scottdb=> explain execute stmt(7902);
                     QUERY PLAN
----------------------------------------------------
 Seq Scan on emp  (cost=0.00..1.18 rows=1 width=37)
   Filter: (empno = 7902)
(2 rows)
```

## Application Name

개발 시 application_name을 설정하면 문제 발생 시 운영자가 원인 파악이 쉬워진다.

### 설정 및 확인 방법

* SQL에서 설정
  ```sql
  set application_name = '프로그램 이름';
  ```

* JDBC의 `conn.setClientInfo()` 메소드 이용
  ```java
  Connection.setClientInfo("ApplicationName", "XXX PROGRAM");
  ```

* 확인 방법
  ```
  edb=# select application_name from pg_stat_activity;
   application_name
  ------------------
   psql.bin
  (1 row)
  ```

## JDBC Fetch Size

JDBC의 default fetch size가 Oracle의 경우 10으로 설정되어 있지만 PAS/PostgreSQL의 경우 0으로 설정 되어 있다. 따라서 쿼리 결과 건 수가 많은 경우 부분범위 처리를 위해서는 적당히 fetch size를 조절해 줄 필요가 있다. 또는 `limit`절을 추가하여 결과 건을 제한해 주어야 한다.

```java
// make sure autocommit is off
conn.setAutoCommit(false);
Statement st = conn.createStatement();

// Turn use of the cursor on.
st.setFetchSize(15);
```

* Partial fetch가 동작하기 위해서는 반드시 아래 조건이 충족되어 야 한다.
 * no autocommit 모드여야 한다.
   autocommit 상태에서는 쿼리가 완료된 이후 모든 cursor가 닫히기 때문에 추가적인 fetch가 불가능 하다.
 * `Statement` 또는 `PreparedStatement`가 `ResultSet.TYPE_FORWARD_ONLY`로 생성되어야 한다.
   이는 기본값이기 때문에 특별히 변경하지 않았다면 신경쓰지 않아도 된다.
* 위의 조건이 충족되지 않으면 자동으로 full fetch가 일어난다.

### `defaultRowFetchSize`

접속 URL에 아래와 같이 `defaultRowFetchSize`를 설정하면 기본 fetch size를 변경할 수 있다.

```
jdbc:edb://192.168.56.101:5444/edb?defaultRowFetchSize=100
```

하지만 실제 이 기능이 동작 하려면 위의 조건이 모두 충족되어야 한다. 즉 no autocommit로 변경을 해야만 100건 단위 fetch가 동작을 하게 되면 그렇지 않을 경우 전체 결과가 한번에 fetch 된다.

```java
import java.sql.*;

public class EDBTest
{
  public static void doTest(Connection conn, boolean autocommit) throws Exception {
    conn.setAutoCommit(autocommit);

    long sum = 0;
    long ts = System.currentTimeMillis();

    Statement stmt = conn.createStatement();
    ResultSet rs = stmt.executeQuery("select generate_series(1, 10000000)::integer as a");

    System.out.println("Fetch Size: " + stmt.getFetchSize());
    while (rs.next()) {
      sum = rs.getInt(1);
      if (sum > 100) break;
    }
    System.out.println("" + (System.currentTimeMillis() - ts) + " ms");

    rs.close();
    stmt.close();
    if (!autocommit) {
      conn.commit();
    }
  }

  public static void main(String[] args) throws Exception {
    Class.forName("com.edb.Driver");

    String edburl = "jdbc:edb://192.168.56.101:5444/edb?defaultRowFetchSize=100";
    String user = "enterprisedb";
    String password = "ppas";

    Connection conn = DriverManager.getConnection(edburl, user, password);

    // Test 1: autocommit
    System.out.println("## Autocommit ##");
    doTest(conn, true);
    // Test 2: no autocommit
    System.out.println("## No autocommit ##");
    doTest(conn, false);

    conn.close();
  }
}
```

```
$ java -cp edb-jdbc17.jar:. EDBTest
## Autocommit ##
Fetch Size: 100
6820 ms
## No autocommit ##
Fetch Size: 100
4 ms
```

## JDBC Null 처리 이슈

JDBC에서 null 값을 입력하는 방법은 `setNull()`을 이용하는 방법과 `setXXX()`을 이용할때 `null`값을 입력하는 방법 2가지가 있다. 하지만 `setInt()`와 같이 primitive type의 경우 null을 입력할 수 없으므로 반드시 `setNull()`을 이용하여야 한다. 이 때 java의 null은 data type이 없지만 SQL의 null은 data type이 있기 때문에 몇 가지 문제가 발생한다.

#### 테스트 테이블
```
create table test (name varchar2(100), val1 number, val2 varchar2(10), val3 date);
```

#### 테스트 코드
```java
import java.sql.*;

public class NullTest
{
  private static void test1(Connection conn)
    throws SQLException
  {
    try {
      PreparedStatement stmt = conn.prepareStatement("INSERT INTO TEST VALUES ('TEST1', ?, ?, ?)");

      stmt.setNull(1, Types.VARCHAR);
      stmt.setNull(2, Types.VARCHAR);
      stmt.setNull(3, Types.VARCHAR);

      stmt.executeUpdate();
      stmt.close();

      conn.commit();
    } catch (SQLException e) {
      System.err.println(e);
      conn.rollback();
    }
  }

  private static void test2(Connection conn)
    throws SQLException
  {
    try {
      PreparedStatement stmt = conn.prepareStatement("INSERT INTO TEST VALUES ('TEST2', ?, ?, ?)");

      stmt.setNull(1, Types.NULL);
      stmt.setNull(2, Types.NULL);
      stmt.setNull(3, Types.NULL);

      stmt.executeUpdate();
      stmt.close();

      conn.commit();
    } catch (SQLException e) {
      System.err.println(e);
      conn.rollback();
    }
  }

  private static void test3(Connection conn)
    throws SQLException
  {
    try {
      PreparedStatement stmt = conn.prepareStatement("INSERT INTO TEST VALUES ('TEST3', ?, ?, ?)");

      stmt.setString(1, null);
      stmt.setString(2, null);
      stmt.setString(3, null);

      stmt.executeUpdate();
      stmt.close();

      conn.commit();
    } catch (SQLException e) {
      System.err.println(e);
      conn.rollback();
    }
  }

  private static void test4(Connection conn)
    throws SQLException
  {
    try {
      PreparedStatement stmt = conn.prepareStatement("INSERT INTO TEST VALUES ('TEST4', ?::number, ?::varchar2, ?::date)");

      stmt.setNull(1, Types.VARCHAR);
      stmt.setNull(2, Types.VARCHAR);
      stmt.setNull(3, Types.VARCHAR);

      stmt.executeUpdate();
      stmt.close();

      conn.commit();
    } catch (SQLException e) {
      System.err.println(e);
      conn.rollback();
    }
  }

  public static void main(String[] args) throws Exception {
    Class.forName("com.edb.Driver");
    Connection conn;

    try {
      conn = DriverManager.getConnection("jdbc:edb://192.168.56.105:5444/edb", "enterprisedb", "ppas");
      conn.setAutoCommit(false);

      test1(conn);
      test2(conn);
      test3(conn);
      test4(conn);

      conn.close();
    } catch (SQLException e) {
      System.err.println(e);
    }

    try {
      conn = DriverManager.getConnection("jdbc:oracle:thin:@192.168.56.181:1521:orcl", "dbox", "dbox");
      conn.setAutoCommit(false);

      test1(conn);
      test2(conn);
      test3(conn);

      conn.close();
    } catch (SQLException e) {
      System.err.println(e);
    }
  }
}
```

#### 실행
```
$ javac NullTest.java
$ java -cp .:ojdbc6.jar:edb-jdbc17.jar NullTest
com.edb.util.PSQLException: ERROR: column "val3" is of type date but expression is of type character varying
  힌트 : You will need to rewrite or cast the expression.
  위치 : 43
com.edb.util.PSQLException: ERROR: column "val3" is of type date but expression is of type character varying
  힌트 : You will need to rewrite or cast the expression.
  위치 : 43
```

#### 결과

* Oracle
  ```
  SQL> select * from test;

  NAME			   VAL1 VAL2	   VAL3
  -------------------- ---------- ---------- ---------
  TEST1
  TEST2
  TEST3
  ```

* PAS
  ```
  edb=# select * from test;
   name  | val1 | val2 | val3
  -------+------+------+------
   TEST2 |      |      |
   TEST4 |      |      |
  (2 rows)
  ```
  위의 결과와 같이 PAS는 data type을 까다롭게 check하기 때문에 에러가 발생하게 된다. 따라서 반드시 `setNull()`에 정확한 data type을 지정하여 주거나 쿼리에서 명시적인 casting이 필요하다.

