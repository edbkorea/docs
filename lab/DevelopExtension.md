# Lab: Extension

## jumin extension package

앞서 만든 jumin type을 extension으로 packaging 해 보자.

* 준비
앞서 컴파일하면서 생성된 파일들을 정리해 주자.
  ```
  make clean
  ```

### 구현

* `jumin.control`
  ```ini
  default_version = '1.0'
  module_pathname = '$libdir/jumin'
  comment = 'custom type to store South Korean citizen identifier number (jumin number)'
  ```

* `Makefile`
  ```Makefile
  MODULES = jumin
  EXTENSION = jumin
  DATA_built = jumin--1.0.sql

  PG_CONFIG = pg_config
  PGXS := $(shell $(PG_CONFIG) --pgxs)
  include $(PGXS)
  ```

* `jumin--1.0.sql.in`
    Extension에서는 이 이름을 기준으로 버전을 식별한다. 즉 1.0버전을 설치할 경우 `jumin-1.0.sql`이 사용 되고, 1.0이 설치 되어 있는 상태에서 1.1을 설치할 겨우 `jumin--1.0--1.1.sql`이 실행 된다. 지금의 경우에는 1.0 버전이기 때문에 `jumin.sql.in`을 `jumin--1.0.sql.in`로 변경하여 준다.

### Compile

```
[enterprisedb@ppaslab jumin]$ ls
jumin--1.0.sql.in  jumin.c  jumin.control  Makefile
[enterprisedb@ppaslab jumin]$ make
sed 's,MODULE_PATHNAME,$libdir/jumin--1.0,g;s,$PG_VERSION,9.5.0.5,g' jumin--1.0.sql.in >jumin--1.0.sql
gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Wendif-labels -Wmissing-format-attribute -Wformat-security -fno-strict-aliasing -fwrapv -g -DLINUX_OOM_ADJ=0 -O2 -DMAP_HUGETLB=0x40000 -fpic -I. -I./ -I/opt/PostgresPlus/9.5AS/include/server -I/opt/PostgresPlus/9.5AS/include/internal -D_GNU_SOURCE -I/opt/local/20150616/181474c6-228d-4d58-b663-17981352ebce/include/libxml2 -I/opt/local/20150616/181474c6-228d-4d58-b663-17981352ebce/include -I/opt/local/20150616/181474c6-228d-4d58-b663-17981352ebce/include   -I/opt/local/Current/include/libxml2 -I/opt/local/Current/include  -c -o jumin.o jumin.c
gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Wendif-labels -Wmissing-format-attribute -Wformat-security -fno-strict-aliasing -fwrapv -g -DLINUX_OOM_ADJ=0 -O2 -DMAP_HUGETLB=0x40000 -fpic -L/opt/PostgresPlus/9.5AS/lib -L/opt/local/20150616/181474c6-228d-4d58-b663-17981352ebce/lib -L/opt/local/20150616/181474c6-228d-4d58-b663-17981352ebce/lib  -L/opt/local/Current/lib -L/opt/local/20150616/181474c6-228d-4d58-b663-17981352ebce/lib  -Wl,--as-needed -Wl,-rpath,'/opt/PostgresPlus/9.5AS/lib',--enable-new-dtags  -shared -o jumin.so jumin.o
[enterprisedb@ppaslab jumin]$ make install
/bin/mkdir -p '/opt/PostgresPlus/9.5AS/share/extension'
/bin/mkdir -p '/opt/PostgresPlus/9.5AS/share/extension'
/bin/mkdir -p '/opt/PostgresPlus/9.5AS/lib'
/usr/bin/install -c -m 644 .//jumin.control '/opt/PostgresPlus/9.5AS/share/extension/'
/usr/bin/install -c -m 644  jumin--1.0.sql '/opt/PostgresPlus/9.5AS/share/extension/'
/usr/bin/install -c -m 755  jumin.so '/opt/PostgresPlus/9.5AS/lib/'
```

### Create extension

```
edb=# create extension jumin;
CREATE EXTENSION
edb=# create extension jumin;
CREATE EXTENSION
edb=# \dx jumin
                                        List of installed extensions
 Name  | Version |    Schema    |                                Description                                 
-------+---------+--------------+----------------------------------------------------------------------------
 jumin | 1.0     | enterprisedb | custom type to store South Korean citizen identifier number (jumin number)
(1 row)

edb=# \dx+ jumin
           Objects in extension "jumin"
                Object Description                
--------------------------------------------------
 function jumin_birthday(jumin)
 function jumin_cmp(jumin,jumin)
 function jumin_eq(jumin,jumin)
 function jumin_ge(jumin,jumin)
 function jumin_gender(jumin)
 function jumin_gt(jumin,jumin)
 function jumin_in(cstring)
 function jumin_le(jumin,jumin)
 function jumin_lt(jumin,jumin)
 function jumin_out(jumin)
 operator class jumin_ops for access method btree
 operator <=(jumin,jumin)
 operator <(jumin,jumin)
 operator =(jumin,jumin)
 operator >=(jumin,jumin)
 operator >(jumin,jumin)
 type jumin
(17 rows)
```

### Drop extension

```
edb=# drop extension jumin;
DROP EXTENSION
```
