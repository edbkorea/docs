# Lab: Custom Type

## 참고자료

* Extending SQL: http://www.postgresql.org/docs/current/static/extend.html
* User defined types: http://www.postgresql.org/docs/current/static/xtypes.html
* User defined aggregates: http://www.postgresql.org/docs/current/static/xaggr.html
* User defined operaters: http://www.postgresql.org/docs/current/static/xoper.html

## 디랙토리 생성

```
[enterprisedb@ppaslab ~]$ mkdir ~/work/ext/complex
[enterprisedb@ppaslab ~]$ cd ~/work/ext/complex
```

## Complex number
`complex.c`, `complex.sql.in`, `Makefile`은 complex number를 custom type으로 구현하기 위한 예제 이다. 아래 내용을 분석하여 구성 요소들을 이해 하고 직접 컴파일 하여 테스트 해 보자.

### 구현

* `complex.c`
  ```CPP
  /*
   * src/tutorial/complex.c
   *
   ******************************************************************************
    This file contains routines that can be bound to a Postgres backend and
    called by the backend in the process of processing queries.  The calling
    format for these routines is dictated by Postgres architecture.
  ******************************************************************************/

  #include "postgres.h"

  #include "fmgr.h"
  #include "libpq/pqformat.h"		/* needed for send/recv functions */

  PG_MODULE_MAGIC;

  typedef struct Complex
  {
    double		x;
    double		y;
  }	Complex;

  /*****************************************************************************
   * Input/Output functions
   *****************************************************************************/

  PG_FUNCTION_INFO_V1(complex_in);

  Datum
  complex_in(PG_FUNCTION_ARGS)
  {
    char	   *str = PG_GETARG_CSTRING(0);
    double		x,
          y;
    Complex    *result;

    if (sscanf(str, " ( %lf , %lf )", &x, &y) != 2)
      ereport(ERROR,
          (errcode(ERRCODE_INVALID_TEXT_REPRESENTATION),
           errmsg("invalid input syntax for complex: \"%s\"",
              str)));

    result = (Complex *) palloc(sizeof(Complex));
    result->x = x;
    result->y = y;
    PG_RETURN_POINTER(result);
  }

  PG_FUNCTION_INFO_V1(complex_out);

  Datum
  complex_out(PG_FUNCTION_ARGS)
  {
    Complex    *complex = (Complex *) PG_GETARG_POINTER(0);
    char	   *result;

    result = psprintf("(%g,%g)", complex->x, complex->y);
    PG_RETURN_CSTRING(result);
  }

  /*****************************************************************************
   * Binary Input/Output functions
   *
   * These are optional.
   *****************************************************************************/

  PG_FUNCTION_INFO_V1(complex_recv);

  Datum
  complex_recv(PG_FUNCTION_ARGS)
  {
    StringInfo	buf = (StringInfo) PG_GETARG_POINTER(0);
    Complex    *result;

    result = (Complex *) palloc(sizeof(Complex));
    result->x = pq_getmsgfloat8(buf);
    result->y = pq_getmsgfloat8(buf);
    PG_RETURN_POINTER(result);
  }

  PG_FUNCTION_INFO_V1(complex_send);

  Datum
  complex_send(PG_FUNCTION_ARGS)
  {
    Complex    *complex = (Complex *) PG_GETARG_POINTER(0);
    StringInfoData buf;

    pq_begintypsend(&buf);
    pq_sendfloat8(&buf, complex->x);
    pq_sendfloat8(&buf, complex->y);
    PG_RETURN_BYTEA_P(pq_endtypsend(&buf));
  }

  /*****************************************************************************
   * New Operators
   *
   * A practical Complex datatype would provide much more than this, of course.
   *****************************************************************************/

  PG_FUNCTION_INFO_V1(complex_add);

  Datum
  complex_add(PG_FUNCTION_ARGS)
  {
    Complex    *a = (Complex *) PG_GETARG_POINTER(0);
    Complex    *b = (Complex *) PG_GETARG_POINTER(1);
    Complex    *result;

    result = (Complex *) palloc(sizeof(Complex));
    result->x = a->x + b->x;
    result->y = a->y + b->y;
    PG_RETURN_POINTER(result);
  }

  /*****************************************************************************
   * Operator class for defining B-tree index
   *
   * It's essential that the comparison operators and support function for a
   * B-tree index opclass always agree on the relative ordering of any two
   * data values.  Experience has shown that it's depressingly easy to write
   * unintentionally inconsistent functions.  One way to reduce the odds of
   * making a mistake is to make all the functions simple wrappers around
   * an internal three-way-comparison function, as we do here.
   *****************************************************************************/

  #define Mag(c)	((c)->x*(c)->x + (c)->y*(c)->y)

  static int
  complex_abs_cmp_internal(Complex * a, Complex * b)
  {
    double		amag = Mag(a),
          bmag = Mag(b);

    if (amag < bmag)
      return -1;
    if (amag > bmag)
      return 1;
    return 0;
  }

  PG_FUNCTION_INFO_V1(complex_abs_lt);

  Datum
  complex_abs_lt(PG_FUNCTION_ARGS)
  {
    Complex    *a = (Complex *) PG_GETARG_POINTER(0);
    Complex    *b = (Complex *) PG_GETARG_POINTER(1);

    PG_RETURN_BOOL(complex_abs_cmp_internal(a, b) < 0);
  }

  PG_FUNCTION_INFO_V1(complex_abs_le);

  Datum
  complex_abs_le(PG_FUNCTION_ARGS)
  {
    Complex    *a = (Complex *) PG_GETARG_POINTER(0);
    Complex    *b = (Complex *) PG_GETARG_POINTER(1);

    PG_RETURN_BOOL(complex_abs_cmp_internal(a, b) <= 0);
  }

  PG_FUNCTION_INFO_V1(complex_abs_eq);

  Datum
  complex_abs_eq(PG_FUNCTION_ARGS)
  {
    Complex    *a = (Complex *) PG_GETARG_POINTER(0);
    Complex    *b = (Complex *) PG_GETARG_POINTER(1);

    PG_RETURN_BOOL(complex_abs_cmp_internal(a, b) == 0);
  }

  PG_FUNCTION_INFO_V1(complex_abs_ge);

  Datum
  complex_abs_ge(PG_FUNCTION_ARGS)
  {
    Complex    *a = (Complex *) PG_GETARG_POINTER(0);
    Complex    *b = (Complex *) PG_GETARG_POINTER(1);

    PG_RETURN_BOOL(complex_abs_cmp_internal(a, b) >= 0);
  }

  PG_FUNCTION_INFO_V1(complex_abs_gt);

  Datum
  complex_abs_gt(PG_FUNCTION_ARGS)
  {
    Complex    *a = (Complex *) PG_GETARG_POINTER(0);
    Complex    *b = (Complex *) PG_GETARG_POINTER(1);

    PG_RETURN_BOOL(complex_abs_cmp_internal(a, b) > 0);
  }

  PG_FUNCTION_INFO_V1(complex_abs_cmp);

  Datum
  complex_abs_cmp(PG_FUNCTION_ARGS)
  {
    Complex    *a = (Complex *) PG_GETARG_POINTER(0);
    Complex    *b = (Complex *) PG_GETARG_POINTER(1);

    PG_RETURN_INT32(complex_abs_cmp_internal(a, b));
  }
  ```

* `complex.sql.in`
  ```sql
  DROP TABLE IF EXISTS test_complex;
  DROP TYPE IF EXISTS complex CASCADE;

  CREATE OR REPLACE FUNCTION complex_in(cstring)
     RETURNS complex
     AS 'MODULE_PATHNAME'
     LANGUAGE C IMMUTABLE STRICT;

  -- the output function 'complex_out' takes the internal representation and
  -- converts it into the textual representation.

  CREATE OR REPLACE FUNCTION complex_out(complex)
     RETURNS cstring
     AS 'MODULE_PATHNAME'
     LANGUAGE C IMMUTABLE STRICT;

  -- the binary input function 'complex_recv' takes a StringInfo buffer
  -- and turns its contents into the internal representation.

  CREATE OR REPLACE FUNCTION complex_recv(internal)
     RETURNS complex
     AS 'MODULE_PATHNAME'
     LANGUAGE C IMMUTABLE STRICT;

  -- the binary output function 'complex_send' takes the internal representation
  -- and converts it into a (hopefully) platform-independent bytea string.

  CREATE OR REPLACE FUNCTION complex_send(complex)
     RETURNS bytea
     AS 'MODULE_PATHNAME'
     LANGUAGE C IMMUTABLE STRICT;

  -- now, we can create the type. The internallength specifies the size of the
  -- memory block required to hold the type (we need two 8-byte doubles).

  CREATE TYPE complex (
     internallength = 16,
     input = complex_in,
     output = complex_out,
     receive = complex_recv,
     send = complex_send,
     alignment = double
  );

  -----------------------------
  -- Creating an operator for the new type:
  --      Let's define an add operator for complex types. Since POSTGRES
  --      supports function overloading, we'll use + as the add operator.
  --      (Operator names can be reused with different numbers and types of
  --      arguments.)
  -----------------------------

  -- first, define a function complex_add (also in complex.c)
  CREATE OR REPLACE FUNCTION complex_add(complex, complex)
     RETURNS complex
     AS 'MODULE_PATHNAME'
     LANGUAGE C IMMUTABLE STRICT;

  -- we can now define the operator. We show a binary operator here but you
  -- can also define unary operators by omitting either of leftarg or rightarg.
  CREATE OPERATOR + (
     leftarg = complex,
     rightarg = complex,
     procedure = complex_add,
     commutator = +
  );

  CREATE AGGREGATE complex_sum (
     sfunc = complex_add,
     basetype = complex,
     stype = complex,
     initcond = '(0,0)'
  );

  -----------------------------
  -- Interfacing New Types with Indexes:
  --      We cannot define a secondary index (eg. a B-tree) over the new type
  --      yet. We need to create all the required operators and support
  --      functions, then we can make the operator class.
  -----------------------------

  -- first, define the required operators
  CREATE OR REPLACE FUNCTION complex_abs_lt(complex, complex) RETURNS bool
     AS 'MODULE_PATHNAME' LANGUAGE C IMMUTABLE STRICT;
  CREATE OR REPLACE FUNCTION complex_abs_le(complex, complex) RETURNS bool
     AS 'MODULE_PATHNAME' LANGUAGE C IMMUTABLE STRICT;
  CREATE OR REPLACE FUNCTION complex_abs_eq(complex, complex) RETURNS bool
     AS 'MODULE_PATHNAME' LANGUAGE C IMMUTABLE STRICT;
  CREATE OR REPLACE FUNCTION complex_abs_ge(complex, complex) RETURNS bool
     AS 'MODULE_PATHNAME' LANGUAGE C IMMUTABLE STRICT;
  CREATE OR REPLACE FUNCTION complex_abs_gt(complex, complex) RETURNS bool
     AS 'MODULE_PATHNAME' LANGUAGE C IMMUTABLE STRICT;

  CREATE OPERATOR < (
     leftarg = complex, rightarg = complex, procedure = complex_abs_lt,
     commutator = > , negator = >= ,
     restrict = scalarltsel, join = scalarltjoinsel
  );
  CREATE OPERATOR <= (
     leftarg = complex, rightarg = complex, procedure = complex_abs_le,
     commutator = >= , negator = > ,
     restrict = scalarltsel, join = scalarltjoinsel
  );
  CREATE OPERATOR = (
     leftarg = complex, rightarg = complex, procedure = complex_abs_eq,
     commutator = = ,
     -- leave out negator since we didn't create <> operator
     -- negator = <> ,
     restrict = eqsel, join = eqjoinsel
  );
  CREATE OPERATOR >= (
     leftarg = complex, rightarg = complex, procedure = complex_abs_ge,
     commutator = <= , negator = < ,
     restrict = scalargtsel, join = scalargtjoinsel
  );
  CREATE OPERATOR > (
     leftarg = complex, rightarg = complex, procedure = complex_abs_gt,
     commutator = < , negator = <= ,
     restrict = scalargtsel, join = scalargtjoinsel
  );

  -- create the support function too
  CREATE OR REPLACE FUNCTION complex_abs_cmp(complex, complex) RETURNS int4
     AS 'MODULE_PATHNAME' LANGUAGE C IMMUTABLE STRICT;

  -- now we can make the operator class
  CREATE OPERATOR CLASS complex_abs_ops
      DEFAULT FOR TYPE complex USING btree AS
          OPERATOR        1       < ,
          OPERATOR        2       <= ,
          OPERATOR        3       = ,
          OPERATOR        4       >= ,
          OPERATOR        5       > ,
          FUNCTION        1       complex_abs_cmp(complex, complex);
  ```

* `Makefile`
  ```Makefile
  MODULES = complex
  DATA_built = complex.sql

  PG_CONFIG = pg_config
  PGXS := $(shell $(PG_CONFIG) --pgxs)
  include $(PGXS)
  ```

### 테스트

* 설치

   ```
  [enterprisedb@ppaslab complex]$ make
  [enterprisedb@ppaslab complex]$ make install

  ```

  ```
  edb=# \i /opt/PostgresPlus/9.5AS/share/contrib/complex.sql
  psql.bin:/opt/PostgresPlus/9.5AS/share/contrib/complex.sql:1: NOTICE:  table "test_complex" does not exist, skipping
  DROP TABLE
  psql.bin:/opt/PostgresPlus/9.5AS/share/contrib/complex.sql:2: NOTICE:  drop cascades to 11 other objects
  DETAIL:  drop cascades to function complex_in(cstring)
  drop cascades to function complex_out(complex)
  drop cascades to function complex_recv(internal)
  drop cascades to function complex_send(complex)
  drop cascades to function complex_add(complex,complex)
  drop cascades to function complex_abs_lt(complex,complex)
  drop cascades to function complex_abs_le(complex,complex)
  drop cascades to function complex_abs_eq(complex,complex)
  drop cascades to function complex_abs_ge(complex,complex)
  drop cascades to function complex_abs_gt(complex,complex)
  drop cascades to function complex_abs_cmp(complex,complex)
  DROP TYPE
  psql.bin:/opt/PostgresPlus/9.5AS/share/contrib/complex.sql:7: NOTICE:  type "complex" is not yet defined
  DETAIL:  Creating a shell type definition.
  CREATE FUNCTION
  psql.bin:/opt/PostgresPlus/9.5AS/share/contrib/complex.sql:15: NOTICE:  argument type complex is only a shell
  CREATE FUNCTION
  psql.bin:/opt/PostgresPlus/9.5AS/share/contrib/complex.sql:23: NOTICE:  return type complex is only a shell
  CREATE FUNCTION
  psql.bin:/opt/PostgresPlus/9.5AS/share/contrib/complex.sql:31: NOTICE:  argument type complex is only a shell
  CREATE FUNCTION
  CREATE TYPE
  CREATE FUNCTION
  CREATE OPERATOR
  CREATE AGGREGATE
  CREATE FUNCTION
  CREATE FUNCTION
  CREATE FUNCTION
  CREATE FUNCTION
  CREATE FUNCTION
  CREATE OPERATOR
  CREATE OPERATOR
  CREATE OPERATOR
  CREATE OPERATOR
  CREATE OPERATOR
  CREATE FUNCTION
  CREATE OPERATOR CLASS
  ```

* 동작 테스트
  ```
  create table test_complex (a complex, b complex);

  insert into test_complex values ('(1, 2.5)', '(4.2,3.55');
  insert into test_complex values ('(33,51.4)', '(100.42,93.55)');
  insert into test_complex values ('(56,-22.5)', '(-43.2,-0.07)');
  insert into test_complex values ('(-91.9,33.6)', '(8.6,3)');
  ```
  ```
  edb=# select * from test_complex;
        a       |       b       
  --------------+---------------
   (1,2.5)      | (4.2,3.55)
   (33,51.4)    |
   (56,-22.5)   | (-43.2,-0.07)
   (-91.9,33.6) | (8.6,3)
  (4 rows)

  edb=# select a+b as abs_sum from test_complex;
       abs_sum     
  -----------------
   (5.2,6.05)
   (133.42,144.95)
   (12.8,-22.57)
   (-83.3,36.6)
  (4 rows)
  ```


## 주민번호 Type
마찬가지로 이번에는 주민등록 번호를 담기위한 `jumin` 타입을 만들어 보자. 제공하는 기능은 다음과 같다.

* 주민번호 뒷자리 암호화 및 마스킹
* Equality 비교
* 생년월일 순 정렬 및 비교
* 주민번호 뒷자리 중 첫 글자를 기준으로 1800, 1900, 2000년대 자동 처리 기능
* 생년월일 추출
* 성별 추출


## 디랙토리 생성

```
[enterprisedb@ppaslab ~]$ mkdir ~/work/ext/jumin
[enterprisedb@ppaslab ~]$ cd ~/work/ext/jumin
```

### 구현

* `Makefile`
  ```Makefile
  MODULES = jumin
  DATA_built = jumin.sql

  PG_CONFIG = pg_config
  PGXS := $(shell $(PG_CONFIG) --pgxs)
  include $(PGXS)
  ```

* `jumin.c`
  ```CPP
  /******************************************************************************
    Jason Kim (jason.kim@enterprisedb.com)
  ******************************************************************************/
  #include <unistd.h>

  #include "postgres.h"
  #include "libpq/md5.h"

  #include "fmgr.h"

  #include "utils/builtins.h"

  PG_MODULE_MAGIC;

  /*****************************************************************************
   * Type definition
   *****************************************************************************/
  #define MD5_DIGEST_LENGTH  16

  typedef struct Jumin
  {
    char data[7];
    char md5hash[MD5_DIGEST_LENGTH];
  } Jumin;

  /*****************************************************************************
   * Utility function definitions
   *****************************************************************************/
  static uint8 decode_era(char);
  static void md5(const char*, size_t, char*);
  static int jumin_cmp_eq_internal(Jumin *, Jumin *);
  static uint8 decode_era(char);
  static int jumin_cmp_internal(Jumin *, Jumin *);

  /*****************************************************************************
   * Input/Output functions
   *****************************************************************************/

  PG_FUNCTION_INFO_V1(jumin_in);

  Datum
  jumin_in(PG_FUNCTION_ARGS)
  {
    char *str = PG_GETARG_CSTRING(0);
    char part1[6];
    char part2[7];
    Jumin    *result;

    if (strlen(str) != 14 ||
        sscanf(str, "%6c-%7c", (char*)&part1, (char*)&part2) != 2)
      ereport(ERROR,
          (errcode(ERRCODE_INVALID_TEXT_REPRESENTATION),
           errmsg("invalid input syntax for jumin: \"%s\"",
              str)));

    result = (Jumin *) palloc0(sizeof(Jumin));
      strncpy(result->data, part1, 6);
      strncpy(result->data+6, part2, 1);
    md5((const char*)part2, 7, result->md5hash);

    PG_RETURN_POINTER(result);
  }

  PG_FUNCTION_INFO_V1(jumin_out);

  Datum
  jumin_out(PG_FUNCTION_ARGS)
  {
    Jumin    *jumin = (Jumin *) PG_GETARG_POINTER(0);
    char result[14+1];
      char *r;

    strncpy(result, jumin->data, 6);
    result[6] = '-';
    strncpy(result+7, jumin->data+6, 1);

    r = psprintf("%.6s-%.1s******", jumin->data, jumin->data);

    PG_RETURN_CSTRING(r);
  }

  PG_FUNCTION_INFO_V1(jumin_gender);

  Datum
  jumin_gender(PG_FUNCTION_ARGS)
  {
    Jumin    *jumin = (Jumin *) PG_GETARG_POINTER(0);
    char result = 'U';

    if (jumin->data[6] == '9'
      || jumin->data[6] == '1'
      || jumin->data[6] == '3'
      || jumin->data[6] == '5'
      || jumin->data[6] == '7') result = 'M';
    else if (jumin->data[6] == '0'
      || jumin->data[6] == '2'
      || jumin->data[6] == '4'
      || jumin->data[6] == '6'
      || jumin->data[6] == '8') result = 'F';

    PG_RETURN_TEXT_P(cstring_to_text_with_len(&result, 1));
  }

  PG_FUNCTION_INFO_V1(jumin_birthday);

  Datum
  jumin_birthday(PG_FUNCTION_ARGS)
  {
    Jumin    *jumin = (Jumin *) PG_GETARG_POINTER(0);
      char result[8+1];
    int era;

    era = decode_era(jumin->data[6]);

  //    ereport(NOTICE,
  //            (errcode(ERRCODE_SUCCESSFUL_COMPLETION),
  //             errmsg("Era %.7s, %d", jumin->data, era)));

    sprintf(result, "%u%.6s", era, jumin->data);

    PG_RETURN_TEXT_P(cstring_to_text(result));
  }

  /*****************************************************************************
   * Operator class for defining B-tree index
   *
   * It's essential that the comparison operators and support function for a
   * B-tree index opclass always agree on the relative ordering of any two
   * data values.  Experience has shown that it's depressingly easy to write
   * unintentionally inconsistent functions.  One way to reduce the odds of
   * making a mistake is to make all the functions simple wrappers around
   * an internal three-way-comparison function, as we do here.
   *****************************************************************************/

  PG_FUNCTION_INFO_V1(jumin_lt);

  Datum
  jumin_lt(PG_FUNCTION_ARGS)
  {
    Jumin    *a = (Jumin *) PG_GETARG_POINTER(0);
    Jumin    *b = (Jumin *) PG_GETARG_POINTER(1);

    PG_RETURN_BOOL(jumin_cmp_internal(a, b) < 0);
  }

  PG_FUNCTION_INFO_V1(jumin_le);

  Datum
  jumin_le(PG_FUNCTION_ARGS)
  {
    Jumin    *a = (Jumin *) PG_GETARG_POINTER(0);
    Jumin    *b = (Jumin *) PG_GETARG_POINTER(1);

    PG_RETURN_BOOL(jumin_cmp_internal(a, b) <= 0);
  }

  PG_FUNCTION_INFO_V1(jumin_eq);

  Datum
  jumin_eq(PG_FUNCTION_ARGS)
  {
    Jumin    *a = (Jumin *) PG_GETARG_POINTER(0);
    Jumin    *b = (Jumin *) PG_GETARG_POINTER(1);

    PG_RETURN_BOOL(jumin_cmp_eq_internal(a, b) == 0);
  }

  PG_FUNCTION_INFO_V1(jumin_ge);

  Datum
  jumin_ge(PG_FUNCTION_ARGS)
  {
    Jumin    *a = (Jumin *) PG_GETARG_POINTER(0);
    Jumin    *b = (Jumin *) PG_GETARG_POINTER(1);

    PG_RETURN_BOOL(jumin_cmp_internal(a, b) >= 0);
  }

  PG_FUNCTION_INFO_V1(jumin_gt);

  Datum
  jumin_gt(PG_FUNCTION_ARGS)
  {
    Jumin    *a = (Jumin *) PG_GETARG_POINTER(0);
    Jumin    *b = (Jumin *) PG_GETARG_POINTER(1);

    PG_RETURN_BOOL(jumin_cmp_internal(a, b) > 0);
  }

  PG_FUNCTION_INFO_V1(jumin_cmp);

  Datum
  jumin_cmp(PG_FUNCTION_ARGS)
  {
    Jumin    *a = (Jumin *) PG_GETARG_POINTER(0);
    Jumin    *b = (Jumin *) PG_GETARG_POINTER(1);

    PG_RETURN_INT32(jumin_cmp_internal(a, b));
  }

  /*****************************************************************************
   * Utility functions
   *****************************************************************************/
  static uint8 decode_era(char eracode)
  {
    uint8 era = 0;
    switch(eracode) {
    case '9':
    case '0':
      era = 18;
      break;
    case '1':
    case '2':
      era = 19;
      break;
    case '3':
    case '4':
      era = 20;
      break;
    case '5':
    case '6':
      era = 19;
      break;
    case '7':
    case '8':
      era = 20;
      break;
    }

    return era;
  }

  static int jumin_cmp_internal(Jumin * a, Jumin * b)
  {
    int era_a = 0, era_b = 0;
    int t = 0;

    era_a = decode_era(a->data[6]);
    era_b = decode_era(b->data[6]);

    if (era_a < era_b)
      return -1;

    if (era_a > era_b)
      return 1;

    // if era_a = era_b
    t = strncmp(a->data, b->data, 6);
    if (t == 0) {
      t = strncmp((const char *)a->md5hash, (const char *)b->md5hash, MD5_DIGEST_LENGTH);
    }
    return t;
  }

  // more efficient version for eq test only
  static int jumin_cmp_eq_internal(Jumin * a, Jumin * b)
  {
    int t = 0;

    t = strncmp(a->data, b->data, 7);
    if (t == 0) {
      t = strncmp((const char *)a->md5hash, (const char *)b->md5hash, MD5_DIGEST_LENGTH);
    }
    return t;
  }

  static void  md5(const char* value, size_t len, char* result)
  {
    if (pg_md5_binary(value, len, result) == false) {
      ereport(ERROR,
          (errcode(ERRCODE_OUT_OF_MEMORY),
           errmsg("out of memory")));
    }
  }
  ```

* `jumin.sql.in`
  ```sql
  DROP TYPE IF EXISTS jumin CASCADE;

  CREATE OR REPLACE FUNCTION jumin_in(cstring)
     RETURNS jumin
     AS 'jumin'
     LANGUAGE C IMMUTABLE STRICT;

  -- the output function 'complex_out' takes the internal representation and
  -- converts it into the textual representation.

  CREATE OR REPLACE FUNCTION jumin_out(jumin)
     RETURNS cstring
     AS 'jumin'
     LANGUAGE C IMMUTABLE STRICT;

  -- now, we can create the type. The internallength specifies the size of the
  -- memory block required to hold the type (we need 23 bytes).

  CREATE TYPE jumin (
     internallength = 23,
     input = jumin_in,
     output = jumin_out,
     alignment = double
  );

  -----------------------------
  -- Interfacing New Types with Indexes:
  --    We cannot define a secondary index (eg. a B-tree) over the new type
  --    yet. We need to create all the required operators and support
  --      functions, then we can make the operator class.
  -----------------------------
  CREATE OR REPLACE FUNCTION jumin_gender(jumin) RETURNS text
     AS 'jumin' LANGUAGE C IMMUTABLE STRICT;
  CREATE OR REPLACE FUNCTION jumin_birthday(jumin) RETURNS text
     AS 'jumin' LANGUAGE C IMMUTABLE STRICT;

  -- first, define the required operators
  CREATE OR REPLACE FUNCTION jumin_lt(jumin, jumin) RETURNS bool
     AS 'jumin' LANGUAGE C IMMUTABLE STRICT;
  CREATE OR REPLACE FUNCTION jumin_le(jumin, jumin) RETURNS bool
     AS 'jumin' LANGUAGE C IMMUTABLE STRICT;
  CREATE OR REPLACE FUNCTION jumin_eq(jumin, jumin) RETURNS bool
     AS 'jumin' LANGUAGE C IMMUTABLE STRICT;
  CREATE OR REPLACE FUNCTION jumin_ge(jumin, jumin) RETURNS bool
     AS 'jumin' LANGUAGE C IMMUTABLE STRICT;
  CREATE OR REPLACE FUNCTION jumin_gt(jumin, jumin) RETURNS bool
     AS 'jumin' LANGUAGE C IMMUTABLE STRICT;

  CREATE OPERATOR < (
     leftarg = jumin, rightarg = jumin, procedure = jumin_lt,
     commutator = > , negator = >= ,
     restrict = scalarltsel, join = scalarltjoinsel
  );
  CREATE OPERATOR <= (
     leftarg = jumin, rightarg = jumin, procedure = jumin_le,
     commutator = >= , negator = > ,
     restrict = scalarltsel, join = scalarltjoinsel
  );
  CREATE OPERATOR = (
     leftarg = jumin, rightarg = jumin, procedure = jumin_eq,
     commutator = = ,
     -- leave out negator since we didn't create <> operator
     -- negator = <> ,
     restrict = eqsel, join = eqjoinsel
  );
  CREATE OPERATOR >= (
     leftarg = jumin, rightarg = jumin, procedure = jumin_ge,
     commutator = <= , negator = < ,
     restrict = scalargtsel, join = scalargtjoinsel
  );
  CREATE OPERATOR > (
     leftarg = jumin, rightarg = jumin, procedure = jumin_gt,
     commutator = < , negator = <= ,
     restrict = scalargtsel, join = scalargtjoinsel
  );

  -- create the support function too
  CREATE OR REPLACE FUNCTION jumin_cmp(jumin, jumin) RETURNS int4
     AS 'jumin' LANGUAGE C IMMUTABLE STRICT;

  -- now we can make the operator class
  CREATE OPERATOR CLASS jumin_ops
      DEFAULT FOR TYPE jumin USING btree AS
          OPERATOR        1       < ,
          OPERATOR        2       <= ,
          OPERATOR        3       = ,
          OPERATOR        4       >= ,
          OPERATOR        5       > ,
          FUNCTION        1       jumin_cmp(jumin, jumin);
  ```

### 테스트

* 설치
  ```
  [enterprisedb@ppaslab jumin]$ make
  sed 's,MODULE_PATHNAME,$libdir/jumin,g;s,$PG_VERSION,9.5.0.5,g' jumin.sql.in >jumin.sql
  gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Wendif-labels -Wmissing-format-attribute -Wformat-security -fno-strict-aliasing -fwrapv -g -DLINUX_OOM_ADJ=0 -O2 -DMAP_HUGETLB=0x40000 -fpic -I. -I./ -I/opt/PostgresPlus/9.5AS/include/server -I/opt/PostgresPlus/9.5AS/include/internal -D_GNU_SOURCE -I/opt/local/20150616/181474c6-228d-4d58-b663-17981352ebce/include/libxml2 -I/opt/local/20150616/181474c6-228d-4d58-b663-17981352ebce/include -I/opt/local/20150616/181474c6-228d-4d58-b663-17981352ebce/include   -I/opt/local/Current/include/libxml2 -I/opt/local/Current/include  -c -o jumin.o jumin.c
  gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Wendif-labels -Wmissing-format-attribute -Wformat-security -fno-strict-aliasing -fwrapv -g -DLINUX_OOM_ADJ=0 -O2 -DMAP_HUGETLB=0x40000 -fpic -L/opt/PostgresPlus/9.5AS/lib -L/opt/local/20150616/181474c6-228d-4d58-b663-17981352ebce/lib -L/opt/local/20150616/181474c6-228d-4d58-b663-17981352ebce/lib  -L/opt/local/Current/lib -L/opt/local/20150616/181474c6-228d-4d58-b663-17981352ebce/lib  -Wl,--as-needed -Wl,-rpath,'/opt/PostgresPlus/9.5AS/lib',--enable-new-dtags  -shared -o jumin.so jumin.o
  [enterprisedb@ppaslab jumin]$ make install
  /bin/mkdir -p '/opt/PostgresPlus/9.5AS/share/contrib'
  /bin/mkdir -p '/opt/PostgresPlus/9.5AS/lib'
  /usr/bin/install -c -m 644  jumin.sql '/opt/PostgresPlus/9.5AS/share/contrib/'
  /usr/bin/install -c -m 755  jumin.so '/opt/PostgresPlus/9.5AS/lib/'
  ```

  ```
  edb=# \i /opt/PostgresPlus/9.5AS/share/contrib/jumin.sql
  psql.bin:jumin.sql:1: NOTICE:  type "jumin" does not exist, skipping
  DROP TYPE
  psql.bin:jumin.sql:6: NOTICE:  type "jumin" is not yet defined
  DETAIL:  Creating a shell type definition.
  CREATE FUNCTION
  psql.bin:jumin.sql:14: NOTICE:  argument type jumin is only a shell
  CREATE FUNCTION
  CREATE TYPE
  CREATE FUNCTION
  CREATE FUNCTION
  CREATE FUNCTION
  CREATE FUNCTION
  CREATE FUNCTION
  CREATE FUNCTION
  CREATE FUNCTION
  CREATE OPERATOR
  CREATE OPERATOR
  CREATE OPERATOR
  CREATE OPERATOR
  CREATE OPERATOR
  CREATE FUNCTION
  CREATE OPERATOR CLASS
  ```

* 동작 확인

  ```sql
  DROP TABLE test_jumin;
  -- eg. we can use it in a table

  CREATE TABLE test_jumin (
    a	jumin,
    b	char(14)
  );

  CREATE INDEX test_jumin_ind ON test_jumin
     USING btree(a);

  -- data for user-defined types are just strings in the proper textual
  -- representation.

  INSERT INTO test_jumin VALUES ('951010-9723134','951010-9723134');
  INSERT INTO test_jumin VALUES ('750107-1253452','750107-1253452');
  INSERT INTO test_jumin VALUES ('750107-1253452','750107-1253452');
  INSERT INTO test_jumin VALUES ('900208-2575294','900208-2575294');
  INSERT INTO test_jumin VALUES ('830207-2187272','830207-2187272');
  INSERT INTO test_jumin VALUES ('760907-1756006','760907-1756006');
  INSERT INTO test_jumin VALUES ('761209-2436370','761209-2436370');
  INSERT INTO test_jumin VALUES ('731218-1151958','731218-1151958');
  INSERT INTO test_jumin VALUES ('880205-1163291','880205-1163291');
  INSERT INTO test_jumin VALUES ('951227-2438973','951227-2438973');
  INSERT INTO test_jumin VALUES ('870520-2508631','870520-2508631');
  INSERT INTO test_jumin VALUES ('960806-2405962','960806-2405962');
  INSERT INTO test_jumin VALUES ('960806-2205963','960806-2205963');
  INSERT INTO test_jumin VALUES ('971101-0723134','971101-0723134');
  INSERT INTO test_jumin VALUES ('951010-9723134','951010-9723134');
  INSERT INTO test_jumin VALUES ('951010-9723134','951010-9723134');

  SELECT * FROM test_jumin;

  SELECT * from test_jumin where a = '960806-2405962';
  SELECT * from test_jumin where a < '960806-2405962';
  SELECT * from test_jumin where a > '960806-2405962';
  SELECT * from test_jumin order by a;
  SELECT * from test_jumin where b like '960806-2%';
  SELECT * from test_jumin where a = '960806-2405962';
  SELECT a, jumin_birthday(a), jumin_gender(a) from test_jumin order by a;
  ```
