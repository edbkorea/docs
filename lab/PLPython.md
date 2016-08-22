# Lab: PL/Python

## How to install

* Language pack 설치
`edb_languagepack_95.bin` 파일을 이용해 language pack을 설치한다.

* 환경변수 설정
  `LD_LIBRARY_PATH` 설정을 `pgplus_env.sh` 파일에 추가하고 적용한다.

  ```
  export LD_LIBRARY_PATH=/opt/EnterpriseDB/LanguagePack/9.5/Python-3.3/lib:$LD_LIBRARY_PATH
  ```

  ```
  [enterprisedb@ppaslab ~]$ . pgplus_env.sh
  ```

* DB 재기동
  변경된 환경변수가 반영되도록 db를 재기동 한다.

  ```
  [enterprisedb@ppaslab ~]$ pg_ctl restart
  waiting for server to shut down.... done
  server stopped
  server starting
  ```

* 설치

  ```
  edb=# CREATE LANGUAGE plpython3u;
  CREATE LANGUAGE
  ```

## Simple function

```
CREATE FUNCTION gethostbyname(hostname text)
  RETURNS inet
AS $$
  import socket
  return socket.gethostbyname(hostname)
$$ LANGUAGE plpython3u SECURITY DEFINER;
```

```
edb=# SELECT gethostbyname('www.postgresql.org');
 gethostbyname
---------------
 87.238.57.232
(1 row)
```

## Table functions
```
CREATE FUNCTION even_numbers_from_list(up_to int)
  RETURNS SETOF int
AS $$
    return range(0,up_to,2)
$$ LANGUAGE plpython3u;
```

```
CREATE OR REPLACE FUNCTION even_numbers_from_generator(up_to int)
  RETURNS TABLE (even int, odd int)
AS $$
    return ((i,i+1) for i in range(0,up_to,2))
$$ LANGUAGE plpython3u;
```

```
CREATE OR REPLACE FUNCTION even_numbers_with_yield(up_to int,
                                     OUT even int, OUT odd int)
  RETURNS SETOF RECORD
AS $$
    for i in range(0,up_to,2):
        yield i, i+1
$$ LANGUAGE plpython3u;
```

### Query from PL/Python
```
CREATE OR REPLACE FUNCTION get_table_oid(table_name text)
  RETURNS OID
AS $$
	stmt = plpy.prepare("select oid from pg_class where relname = $1", ["text"])
	res = plpy.execute(stmt, [table_name])
	return res[0]["oid"]
$$ LANGUAGE plpython3u;
```

```
edb=# select get_table_oid('test');
 get_table_oid
---------------
         16408
(1 row)
```

### Debugging
```
CREATE OR REPLACE FUNCTION fact(x int) RETURNS int
AS $$
    global x
    f = 1
    while (x > 0):
        f = f * x
        x = x - 1
        plpy.notice('f:%d, x:%d' % (f, x))
    return f
$$ LANGUAGE plpython3u;
```

```
edb=# select * from fact(3);
NOTICE:  f:3, x:2
CONTEXT:  PL/Python function "fact"
NOTICE:  f:6, x:1
CONTEXT:  PL/Python function "fact"
NOTICE:  f:6, x:0
CONTEXT:  PL/Python function "fact"
 fact
------
    6
(1 row)

edb=# SET client_min_messages TO WARNING;
SET
edb=# select * from fact(3);
 fact
------
    6
(1 row)
```
