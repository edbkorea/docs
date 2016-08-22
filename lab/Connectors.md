# Lab: Connectors

## CopyManager in JDBC

```
[enterprisedb@ppaslab ~]$ mkdir -p ~/work/ext/copymanager
[enterprisedb@ppaslab ~]$ cd ~/work/ext/copymanager
```

### `InsertTest.java`

일반적인 prepared statement를 이용한 insert 테스트.

```Java
import java.sql.*;

public class InsertTest
{
  public static void main(String[] args) throws Exception {
    String url = "jdbc:edb://127.0.0.1:5444/edb";
    String user = "enterprisedb";
    String password = "ppas";
    String driver="com.edb.Driver";

    Class.forName(driver);
    Connection conn = DriverManager.getConnection(url, user, password);
    conn.setAutoCommit(false);

    PreparedStatement stmt = conn.prepareStatement("INSERT INTO TEST VALUES (?, ?, ?)");

    long start = System.nanoTime();

    int id = 0;
    for (int i = 0; i < 100; i++) {
      for (int j = 0; j < 10000; j++) {
        stmt.setInt(1, id++);
        stmt.setString(2, "ABCDE");
        stmt.setString(3, "ABCDEFGHIJKLMN");
        stmt.executeUpdate();
      }
      conn.commit();
    }

    long end = System.nanoTime();

    System.out.println((double)(end-start)/1000/1000/1000);

    conn.close();
  }
}
```

### `BatchInsertTest.java`

JDBC의 `executeBatch`를 이용한 테스트

```Java
import java.sql.*;

public class BatchInsertTest
{
  public static void main(String[] args) throws Exception {
    String url = "jdbc:edb://127.0.0.1:5444/edb";
    String user = "enterprisedb";
    String password = "ppas";
    String driver="com.edb.Driver";

    Class.forName(driver);
    Connection conn = DriverManager.getConnection(url, user, password);
    conn.setAutoCommit(false);

    PreparedStatement stmt = conn.prepareStatement("INSERT INTO TEST VALUES (?, ?, ?)");

    long start = System.nanoTime();

    int id = 0;
    for (int i = 0; i < 100; i++) {
      for (int j = 0; j < 10000; j++) {
        stmt.setInt(1, id++);
        stmt.setString(2, "ABCDE");
        stmt.setString(3, "ABCDEFGHIJKLMN");
        stmt.addBatch();
      }
      stmt.executeBatch();
      conn.commit();
    }

    long end = System.nanoTime();

    System.out.println((double)(end-start)/1000/1000/1000);

    conn.close();
  }
}
```

### `CopyTest.java`

PostgreSQL 및 EDB의 JDBC 드라이버가 제공하는 `CopyManager`를 이용한 테스트.

```Java
import java.sql.*;
import com.edb.core.*;
import com.edb.copy.*;

public class CopyTest
{
  public static void main(String[] args) throws Exception {
    String url = "jdbc:edb://127.0.0.1:5444/edb";
    String user = "enterprisedb";
    String password = "ppas";
    CopyIn cpIN=null;
    String driver="com.edb.Driver";

    Class.forName(driver);
    Connection conn = DriverManager.getConnection(url, user, password);

    CopyManager cm = new CopyManager((BaseConnection) conn);

    cpIN= cm.copyIn("COPY test(id, dat1, dat2) FROM STDIN WITH DELIMITER '|'");

    StringBuffer buf = new StringBuffer();

    long start = System.nanoTime();
    for (int i = 0; i < 1000000; i++) {
      byte[] data = buf.append(i).append("|ABCDE|ABCDEFGHIJKLMN\n").toString().getBytes();
      cpIN.writeToCopy(data, 0, data.length);
      buf.setLength(0);
    }
    cpIN.endCopy();

    long end = System.nanoTime();

    System.out.println((double)(end-start)/1000/1000/1000);

    conn.close();
  }
}
```

## 실행 결과

```SQL
drop table if exists test;
create table test (id numeric, dat1 char(10), dat2 text);
create index idx_test_1 on test(id);
create index idx_test_2 on test(dat1);
```

```
[enterprisedb@ppaslab copymanager]$ psql -c "truncate table test;"
TRUNCATE TABLE
[enterprisedb@ppaslab copymanager]$ javac InsertTest.java 
[enterprisedb@ppaslab copymanager]$ java -cp /opt/PostgresPlus/connectors/jdbc/edb-jdbc17.jar:. InsertTest
55.727020112000005
[enterprisedb@ppaslab copymanager]$ java -cp /opt/PostgresPlus/connectors/jdbc/edb-jdbc17.jar:. InsertTest
60.170132793
[enterprisedb@ppaslab copymanager]$ java -cp /opt/PostgresPlus/connectors/jdbc/edb-jdbc17.jar:. InsertTest
60.98178711
```

```
[enterprisedb@ppaslab copymanager]$ psql -c "truncate table test;"
TRUNCATE TABLE
[enterprisedb@ppaslab copymanager]$ javac BatchInsertTest.java 
[enterprisedb@ppaslab copymanager]$ java -cp /opt/PostgresPlus/connectors/jdbc/edb-jdbc17.jar:. BatchInsertTest
21.723017915
[enterprisedb@ppaslab copymanager]$ java -cp /opt/PostgresPlus/connectors/jdbc/edb-jdbc17.jar:. BatchInsertTest
23.577229869
[enterprisedb@ppaslab copymanager]$ java -cp /opt/PostgresPlus/connectors/jdbc/edb-jdbc17.jar:. BatchInsertTest
22.922289960999997
```

```
[enterprisedb@ppaslab copymanager]$ psql -c "truncate table test;"
TRUNCATE TABLE
[enterprisedb@ppaslab copymanager]$ javac CopyTest.java -cp /opt/PostgresPlus/connectors/jdbc/edb-jdbc17.jar 
[enterprisedb@ppaslab copymanager]$ java -cp /opt/PostgresPlus/connectors/jdbc/edb-jdbc17.jar:. CopyTest
6.766896774
[enterprisedb@ppaslab copymanager]$ java -cp /opt/PostgresPlus/connectors/jdbc/edb-jdbc17.jar:. CopyTest
7.662341825
[enterprisedb@ppaslab copymanager]$ java -cp /opt/PostgresPlus/connectors/jdbc/edb-jdbc17.jar:. CopyTest
8.713494516
```
