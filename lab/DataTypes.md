# Lab: Useful data types

## `bytea`

`bytea` 타입은 굉장히 유용하며 다양한 용도로 활용 되지만 실제 데이터와 화면에 표기되는 방식이 상이하기 때문에 해깔리기 쉽다. 다양한 방식으로 조회 해 보고 특징을 잘 숙지하도록 하자.

### 기본 표기
```
edb=# select 'abcde', 'abcde'::bytea;
 ?column? |    bytea     
----------+--------------
 abcde    | \x6162636465
```

위 예제에서 'abcde'가 '\x6162636465'로 표현 되었다. 이는 각 문자의 ASCII 코드가 16진수 표기로 a:61, b:62, c:63 등이기 때문이다.

```
edb=# select ascii('a'), ascii('b'), ascii('c');
 ascii | ascii | ascii
-------+-------+-------
    97 |    98 |    99
(1 row)

edb=# select to_hex(97), to_hex(98), to_hex(99);
 to_hex | to_hex | to_hex
--------+--------+--------
 61     | 62     | 63
(1 row)
```

즉 `\x`로 시작하는 문자열은 나머지 뒷부분이 HEX 값이라는 의미이다.
문제는 아래와 같이 해깔리기 쉽다는 점이다.

```
edb=# select '\x6162636465', 'abcde'::bytea;
   ?column?   |    bytea     
--------------+--------------
 \x6162636465 | \x6162636465
(1 row)
```

결과만 봐서는 같아 보이지만 사실은 전혀 다른 데이터이다. 전자는 말그대로 `\x6162636465`라는 값을 가지는 12 bytes의 문자열이고, 후자는 'abcde'의 ASCII 값을 가지는 5 bytes짜리의 `bytea` 데이터 이다. 이 점을 항상 주의하여야 한다.

### 식별법

다양한 방법이 있겠지만 길이를 확인해 보는 방법이 가장 보편적이다.

```
edb=# select length('\x6162636465'), length('abcde'::bytea);       
 length | length
--------+--------
     12 |      5
(1 row)
```

더 확실한 방법은 `pg_typeof` 함수를 사용하는 것이다.

```
edb=# select pg_typeof('\x6162636465'), pg_typeof('abcde'::bytea);
 pg_typeof | pg_typeof
-----------+-----------
 unknown   | bytea
(1 row)
```

PostgreSQL에서 문자열 상수는 기본적으로 `unknown` type 이다. PostgreSQL의 고유한 type system에서는 이 문자열 상수가 각 type들의 input function에 공급되어 실제 type으로 변환되기 때문이다.
명시적으로 형을 지정하거나 테이블에서 조회한 값이라면 당연히 type이 명시되어 있다.

```
edb=# select pg_typeof('\x6162636465'::varchar), pg_typeof('abcde'::bytea);
     pg_typeof     | pg_typeof
-------------------+-----------
 character varying | bytea
(1 row)

edb=# select ename, pg_typeof(ename) from emp;
 ename  |     pg_typeof     
--------+-------------------
 ALLEN  | character varying
 WARD   | character varying
 MARTIN | character varying
 BLAKE  | character varying
 TURNER | character varying
 CLARK  | character varying
 KING   | character varying
 MILLER | character varying
 SMITH  | character varying
 JONES  | character varying
 SCOTT  | character varying
 ADAMS  | character varying
 FORD   | character varying
 JAMES  | character varying
(14 rows)
```

### Encoding and decoding

* Encoding

  `bytea` 데이터를 다양한 형태로 encoding 할 수 있다.

  ```
  edb=# select encode('abcde'::bytea, 'hex');   
     encode   
  ------------
   6162636465
  (1 row)

  edb=# select encode('abcde'::bytea, 'base64');
    encode  
  ----------
   YWJjZGU=
  (1 row)

  edb=# select encode('abcde'::bytea, 'escape');
   encode
  --------
   abcde
  (1 row)

  edb=# select encode('안녕하세요 world'::bytea, 'hex');   
                     encode                   
  --------------------------------------------
   ec9588eb8595ed9598ec84b8ec9a9420776f726c64
  (1 row)

  edb=# select encode('안녕하세요 world'::bytea, 'base64');
              encode            
  ------------------------------
   7JWI64WV7ZWY7IS47JqUIHdvcmxk
  (1 row)

  edb=# select encode('안녕하세요 world'::bytea, 'escape');
                                 encode                               
  --------------------------------------------------------------------
   \354\225\210\353\205\225\355\225\230\354\204\270\354\232\224 world
  (1 row)
  ```

* Decoding

  `decode` 함수로 앞의 인코딩 과정을 반대로 수행 할 수 있다.

  ```
  edb=# select decode('ec9588eb8595ed9598ec84b8ec9a9420776f726c64', 'hex');
                      decode                    
  ----------------------------------------------
   \xec9588eb8595ed9598ec84b8ec9a9420776f726c64
  (1 row)

  edb=# select decode('7JWI64WV7ZWY7IS47JqUIHdvcmxk', 'base64');
                      decode                    
  ----------------------------------------------
   \xec9588eb8595ed9598ec84b8ec9a9420776f726c64
  (1 row)

  edb=# select decode('\354\225\210\353\205\225\355\225\230\354\204\270\354\232\224 world', 'escape');
                      decode                    
  ----------------------------------------------
   \xec9588eb8595ed9598ec84b8ec9a9420776f726c64
  (1 row)
  ```

  단, decoding의 결과는 `bytea`일 뿐 문자열이 아니다. PostgreSQL 입장에서는 이 `bytea`가 표현하는 데이터가 그림인지 문자열인지 아니면 다른 바이너리 데이터인지 알 방법이 없다. 또한 문자열이라면 인코딩이 UTF-8인지 EUC-KR인지 아무런 정보가 없다. 따라서 decoding의 결과로 얻어진 `bytea`를 문자열로 다시 변환하려면 추가적인 변환이 필요하다.
  이 데이터를 문자열로 바꾸려면 이 데이터가 UTF-8 인코딩의 문자열이라는 정보를 아래와 같이 알려 주면 된다.

  ```
  edb=# select convert_from(decode('ec9588eb8595ed9598ec84b8ec9a9420776f726c64', 'hex'), 'utf-8');
     convert_from   
  ------------------
   안녕하세요 world
  (1 row)
  ```

* Character set 변환

	UTF-8 -> EUC-KR 변환
  ```
  edb=# select convert('안녕하세요 world', 'utf8', 'euc-kr');
                convert               
  ------------------------------------
   \xbec8b3e7c7cfbcbcbfe420776f726c64
  (1 row)
  ```
  EUC-KR -> UTF-8 변환
  ```
  edb=# select convert(decode('bec8b3e7c7cfbcbcbfe420776f726c64', 'hex'), 'euc-kr', 'utf8');
                     convert                    
  ----------------------------------------------
   \xec9588eb8595ed9598ec84b8ec9a9420776f726c64
  (1 row)
  ```

  `bytea` -> `varchar`

  ```
  edb=# select convert_from(decode('bec8b3e7c7cfbcbcbfe420776f726c64', 'hex'), 'euc-kr');
     convert_from   
  ------------------
   안녕하세요 world
  (1 row)

  edb=# select convert_from(decode('ec9588eb8595ed9598ec84b8ec9a9420776f726c64', 'hex'), 'utf8');
     convert_from   
  ------------------
   안녕하세요 world
  (1 row)
  ```

## Network address types

앞서 강의 자료에서 설명한 내용을 실습해 보고 특성을 파악해 보자.

```
edb=# select cidr '192.168.56';       
      cidr       
-----------------
 192.168.56.0/24

edb=# select inet '192.168.56.200/24';  # invalid subnet
       inet        
-------------------
 192.168.56.200/24

edb=# select cidr '192.168.56.200/24';
ERROR:  invalid cidr value: "192.168.56.200/24"
LINE 1: select cidr '192.168.56.200/24';
                    ^
DETAIL:  Value has bits set to right of mask.
```

```
edb=# select cidr '192.168.56.192/26' >> inet '192.168.56.200';
 ?column?
----------
 t
(1 row)

edb=# select cidr '192.168.56.192/26' >> inet '192.168.56.100';
 ?column?
----------
 f
(1 row)
```

```
edb=# select macaddr '08002b:010203', macaddr '08-00-2b-01-02-03';
      macaddr      |      macaddr      
-------------------+-------------------
 08:00:2b:01:02:03 | 08:00:2b:01:02:03
(1 row)
edb=# select trunc(macaddr '08002b:010203');                      
       trunc       
-------------------
 08:00:2b:00:00:00
```

## UUID

앞서 강의 자료에서 설명한 내용을 실습해 보고 특성을 파악해 보자.

```
edb=# select uuid 'A0EEBC99-9C0B-4EF8-BB6D-6BB9BD380A11';
                 uuid                 
--------------------------------------
 a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11

edb=# create extension pgcrypto;
CREATE EXTENSION

edb=# select gen_random_uuid();
           gen_random_uuid            
--------------------------------------
 58deed14-e3b8-4799-b394-6e182eb66626
```

`UUID` type을 사용할 경우와 `varchar`를 사용한 경우의 용량 차이를 확인해 보자.

```
drop table if exists uuid_test1;
create table uuid_test1
  as
select generate_series(1, 10000) as id, gen_random_uuid() dat, gen_random_uuid() dat2;

drop table if exists uuid_test2;
create table uuid_test2
  as
select id, dat::varchar, dat2::varchar from uuid_test1;
```

```
edb=# \dt+ uuid*                                                                                                           
                            List of relations
    Schema    |    Name    | Type  |    Owner     |  Size   | Description
--------------+------------+-------+--------------+---------+-------------
 enterprisedb | uuid_test1 | table | enterprisedb | 672 kB  |
 enterprisedb | uuid_test2 | table | enterprisedb | 1080 kB |
(2 rows)
```

```
edb=# select pg_column_size(gen_random_uuid());
 pg_column_size
----------------
             16
(1 row)

edb=# select pg_column_size(gen_random_uuid()::varchar);
 pg_column_size
----------------
             40
(1 row)
```
