# Lab. Native Function

## 참고자료

> http://www.postgresql.org/docs/current/static/xfunc-c.html

## `add` Function

### 구현

가장 단순한 형태의 함수를 C로 구현해 보자.

* 작업 디랙토리 생성

  ```
  [enterprisedb@ppaslab ~]$ mkdir -p ~/work/ext/add_ab
  [enterprisedb@ppaslab ~]$ cd ~/work/ext/add_ab
  ```

* `add_ab.c`
  ```CPP
  #include "postgres.h"
  #include "fmgr.h"

  PG_MODULE_MAGIC;

  PG_FUNCTION_INFO_V1(add_ab);

  Datum
  add_ab(PG_FUNCTION_ARGS)
  {
      int32   arg_a = PG_GETARG_INT32(0);
      int32   arg_b = PG_GETARG_INT32(1);

      PG_RETURN_INT32(arg_a + arg_b);
  }
  ```
  위에서 사용된 `add_ab.c`의 내용은 version 1 형태의 함수 호출의 기본 구조를 보여준다. 이전 버전인 version 0는 7.0시절에나 사용되던 것이기 때문에 사실상 모든 함수는 이 규약을 따른다고 보면 된다.

* `Makefile`
  ```MakeFile
  MODULES = add_ab

  PG_CONFIG = pg_config
  PGXS := $(shell $(PG_CONFIG) --pgxs)
  include $(PGXS)
  ```
  `Makefile` 역시 이 형태가 기본 구조이다. PGXS라는 표준 빌드 시스템을 사용하기 때문에 이렇게만 해 주면 나머지 귀찮은 작업들을 모두 자동으로 처리해 준다.

### Build

이제 `make` & `make install`만 하면 자동으로 컴파일 및 설치가 완료된다. 의존성이 있는 header와 library를 자동으로 연결해 주는것을 볼 수 있다.
```
[enterprisedb@ppaslab add_ab]$ make
gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Wendif-labels -Wmissing-format-attribute -Wformat-security -fno-strict-aliasing -fwrapv -g -DLINUX_OOM_ADJ=0 -O2 -DMAP_HUGETLB=0x40000 -fpic -I. -I./ -I/opt/PostgresPlus/9.5AS/include/server -I/opt/PostgresPlus/9.5AS/include/internal -D_GNU_SOURCE -I/opt/local/20150616/181474c6-228d-4d58-b663-17981352ebce/include/libxml2 -I/opt/local/20150616/181474c6-228d-4d58-b663-17981352ebce/include -I/opt/local/20150616/181474c6-228d-4d58-b663-17981352ebce/include   -I/opt/local/Current/include/libxml2 -I/opt/local/Current/include  -c -o add_ab.o add_ab.c
gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Wendif-labels -Wmissing-format-attribute -Wformat-security -fno-strict-aliasing -fwrapv -g -DLINUX_OOM_ADJ=0 -O2 -DMAP_HUGETLB=0x40000 -fpic -L/opt/PostgresPlus/9.5AS/lib -L/opt/local/20150616/181474c6-228d-4d58-b663-17981352ebce/lib -L/opt/local/20150616/181474c6-228d-4d58-b663-17981352ebce/lib  -L/opt/local/Current/lib -L/opt/local/20150616/181474c6-228d-4d58-b663-17981352ebce/lib  -Wl,--as-needed -Wl,-rpath,'/opt/PostgresPlus/9.5AS/lib',--enable-new-dtags  -shared -o add_ab.so add_ab.o
[enterprisedb@ppaslab add_ab]$ make install
/bin/mkdir -p '/opt/PostgresPlus/9.5AS/lib'
/usr/bin/install -c -m 755  add_ab.so '/opt/PostgresPlus/9.5AS/lib/'
```

### Create function and test

이제 아래와 같이 shared library 경로와 함수 이름을 이용해서 function을 생성 할 수 있다.
```sql
CREATE OR REPLACE FUNCTION add(int, int) RETURNS INT
 AS '/opt/PostgresPlus/9.5AS/lib/add_ab.so', 'add_ab'
LANGUAGE C STRICT;
```

```
edb=# select add(1, 20);
 add
-----
  21
(1 row)

edb=# \df+ add
                                                                  List of functions
    Schema    | Name | Result data type | Argument data types |  Type  | Security | Volatility |    Owner     | Language | Source code | Description
--------------+------+------------------+---------------------+--------+----------+------------+--------------+----------+-------------+-------------
 enterprisedb | add  | integer          | integer, integer    | normal | invoker  | volatile   | enterprisedb | c        | add_ab      |
(1 row)

```

### 생성 스크립트 자동화

PGXS의 기능을 이용해 적절한 `CREATE FUNCTION` 구문을 자동으로 생성해 보자.

* `add_ab.sql.in`
  ```
  CREATE OR REPLACE FUNCTION add(int, int) RETURNS INT
   AS 'MODULE_PATHNAME', 'add_ab'
  LANGUAGE C STRICT;
  ```

* `Makefile` 수정
  ```Makefile
  MODULES = add_func
  DATA_built = add_ab.sql

  PG_CONFIG = pg_config
  PGXS := $(shell $(PG_CONFIG) --pgxs)
  include $(PGXS)
  ```

* Build
  ```
  [enterprisedb@ppaslab add_ab]$ make
  sed 's,MODULE_PATHNAME,$libdir/add_ab,g;s,$PG_VERSION,9.5.0.5,g' add_ab.sql.in >add_ab.sql
  [enterprisedb@ppaslab add_ab]$ make install
  /bin/mkdir -p '/opt/PostgresPlus/9.5AS/share/contrib'
  /bin/mkdir -p '/opt/PostgresPlus/9.5AS/lib'
  /usr/bin/install -c -m 644  add_ab.sql '/opt/PostgresPlus/9.5AS/share/contrib/'
  /usr/bin/install -c -m 755  add_ab.so '/opt/PostgresPlus/9.5AS/lib/'
  ```

* 생성된 파일 확인
  ```
  [enterprisedb@ppaslab add_ab]$ ls /opt/PostgresPlus/9.5AS/share/contrib/add_ab.sql
  /opt/PostgresPlus/9.5AS/share/contrib/add_ab.sql
  ```

  ```sql
  CREATE OR REPLACE FUNCTION add(int, int) RETURNS INT
   AS '$libdir/add_ab', 'add_ab'
  LANGUAGE C STRICT;
  ```

* 실행
  ```
  edb=# \i /opt/PostgresPlus/9.5AS/share/contrib/add_ab.sql
  CREATE FUNCTION
  ```
