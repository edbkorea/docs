# Lab: JSON & JSONB

## HStore
```
edb=# create extension hstore;
CREATE EXTENSION
```

```
CREATE TABLE hstore_data (data HSTORE);
INSERT INTO hstore_data (data) VALUES ('
  "cost"=>"500", 
  "product"=>"iphone", 
  "provider"=>"apple"');
```

```
edb=# select * from hstore_data; 
                          data
---------------------------------------------------------
 "cost"=>"500", "product"=>"iphone", "provider"=>"apple"
(1 row)

edb=# select data->'product' from hstore_data;
 ?column? 
----------
 iphone
(1 row)
```

## JSON

### Input

```
-- Simple scalar/primitive value
-- Primitive values can be numbers, quoted strings, true, false, or null
SELECT '5'::json;

-- Array of zero or more elements (elements need not be of same type)
SELECT '[1, 2, "foo", null]'::json;

-- Object containing pairs of keys and values
-- Note that object keys must always be quoted strings
SELECT '{"bar": "baz", "balance": 7.77, "active": false}'::json;

-- Arrays and objects can be nested arbitrarily
SELECT '{"foo": [true, "bar"], "tags": {"a": 1, "b": null}}'::json;
```

### Sample Data

* Data 1
  ```
  CREATE TABLE json_data (data JSON);

  INSERT INTO json_data (data)  VALUES 
    (' { 	"name": "Apple Phone",  
      "type": "phone", 
      "brand": "ACME", 
      "price": 200,  
      "available": true,  
      "warranty_years": 1 		
    } ');

  INSERT INTO json_data (data)  VALUES 
    ('{"full name": "John Joseph Carl Salinger",
    "names": 
      [
      {"type": "firstname", "value": "John"},
      {"type": "middlename", "value": "Joseph"},
      {"type": "middlename", "value": "Carl"},
      {"type": "lastname", "value": "Salinger"}	
      ]
    }');
  ```

* Data 2

  아래 내용을 복사해서 `work/test_data.json`에 저장하자.
  참고로 이 데이터는 http://examples.citusdata.com/customer_reviews_nested_1998.json.gz 에서 일부 발최한 것으로 인터넷 접속이 가능하다면 전체 파일을 다운받아서 사용해도 된다. 파일 크기는 29MB이고 압축을 풀면 209MB의 60만건 가량의 리뷰 데이터로 여러 가지 테스트에 활용하기 좋다.
  ```
  { "customer_id": "AE22YDHSBFYIP", "review": { "date": "1970-12-30", "rating": 5, "votes": 10, "helpful_votes": 0 }, "product": { "id": "1551803542", "title": "Start and Run a Coffee Bar (Start & Run a)", "sales_rank": 11611, "group": "Book", "category": "Business & Investing", "subcategory": "General", "similar_ids": ["0471136174","0910627312","047112138X","0786883561","0201570483"] } }
  { "customer_id": "AE22YDHSBFYIP", "review": { "date": "1970-12-30", "rating": 5, "votes": 9, "helpful_votes": 0 }, "product": { "id": "1551802538", "title": "Start and Run a Profitable Coffee Bar", "sales_rank": 689262, "group": "Book", "category": "Business & Investing", "subcategory": "General", "similar_ids": ["0471136174","0910627312","047112138X","0786883561","0201570483"] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-06-19", "rating": 4, "votes": 19, "helpful_votes": 18 }, "product": { "id": "0898624932", "title": "The Power of Maps", "sales_rank": 407473, "group": "Book", "category": "Nonfiction", "subcategory": "Politics", "similar_ids": ["0226534219","0226534170","1931057001","0801870909","157230958X"] } }
  { "customer_id": "AH7OKBE1Z35YA", "review": { "date": "1995-06-23", "rating": 5, "votes": 4, "helpful_votes": 4 }, "product": { "id": "0521469112", "title": "Invention and Evolution", "sales_rank": 755661, "group": "Book", "category": "Science", "subcategory": "General", "similar_ids": ["1591391857"] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-07-14", "rating": 5, "votes": 0, "helpful_votes": 0 }, "product": { "id": "0679722955", "title": "The Names (Vintage Contemporaries (Paperback))", "sales_rank": 264928, "group": "Book", "category": "Literature & Fiction", "subcategory": "General", "similar_ids": ["0140152741","0679722947","0140156046","0679722939","0679722920"] } }
  { "customer_id": "A102UKC71I5DU8", "review": { "date": "1995-07-18", "rating": 4, "votes": 2, "helpful_votes": 2 }, "product": { "id": "0471114251", "title": "Bitter Winds ", "sales_rank": 154570, "group": "Book", "category": "Biographies & Memoirs", "subcategory": "General", "similar_ids": ["0812963741","081331769X","014010870X","0879611316","0060007761"] } }
  { "customer_id": "A1HPIDTM9SRBLP", "review": { "date": "1995-07-18", "rating": 5, "votes": 0, "helpful_votes": 0 }, "product": { "id": "0517887290", "title": "Fingerprints of the Gods ", "sales_rank": 13481, "group": "Book", "category": "Science", "subcategory": "Astronomy", "similar_ids": ["0517888521","0609804774","0671865412","1400049512","0517884542"] } }
  { "customer_id": "A1HPIDTM9SRBLP", "review": { "date": "1995-07-18", "rating": 5, "votes": 0, "helpful_votes": 0 }, "product": { "id": "1574531093", "title": "Fingerprints of the Gods (Alternative History)", "sales_rank": 410246, "group": "Book", "category": "Books on Tape", "subcategory": "Nonfiction", "similar_ids": ["0517888521","0609804774","0671865412","1400049512","0517884542"] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-07-18", "rating": 5, "votes": 1, "helpful_votes": 0 }, "product": { "id": "0962344788", "title": "Heavy Light", "sales_rank": 663630, "group": "Book", "category": "Arts & Photography", "subcategory": "Art", "similar_ids": [] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-07-18", "rating": 5, "votes": 1, "helpful_votes": 1 }, "product": { "id": "0195069056", "title": "Albion's Seed", "sales_rank": 4697, "group": "Book", "category": "Nonfiction", "subcategory": "Social Sciences", "similar_ids": ["0813917743","0195098315","0767916883","0195170342","0195162536"] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-07-19", "rating": 3, "votes": 1, "helpful_votes": 1 }, "product": { "id": "0375406999", "title": "Without Remorse", "sales_rank": 76345, "group": "Book", "category": "Books on CD", "subcategory": "Literature & Fiction", "similar_ids": ["0425147584","0425170349","0425158632","0425116840","0425109720"] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-07-19", "rating": 3, "votes": 1, "helpful_votes": 1 }, "product": { "id": "0399138250", "title": "Without Remorse", "sales_rank": 72220, "group": "Book", "category": "Mystery & Thrillers", "subcategory": "Thrillers", "similar_ids": ["0425147584","0425170349","0425158632","0425116840","0425109720"] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-07-19", "rating": 3, "votes": 1, "helpful_votes": 1 }, "product": { "id": "0425143325", "title": "Without Remorse", "sales_rank": 3868, "group": "Book", "category": "Mystery & Thrillers", "subcategory": "Thrillers", "similar_ids": ["0425147584","0425170349","0425158632","0425116840","0425109720"] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-07-19", "rating": 5, "votes": 0, "helpful_votes": 0 }, "product": { "id": "0312851405", "title": "The Great Hunt ", "sales_rank": 27411, "group": "Book", "category": "Science Fiction & Fantasy", "subcategory": "Fantasy", "similar_ids": ["0812513738","0812550307","0812513754","0812550285","0812550293"] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-07-19", "rating": 5, "votes": 0, "helpful_votes": 0 }, "product": { "id": "0812509714", "title": "The Great Hunt ", "sales_rank": 81136, "group": "Book", "category": "Science Fiction & Fantasy", "subcategory": "Fantasy", "similar_ids": ["0812513738","0812550307","0812513754","0812550285","0812550293"] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-07-19", "rating": 5, "votes": 0, "helpful_votes": 0 }, "product": { "id": "0812517725", "title": "The Great Hunt (The Wheel of Time, Book 2)", "sales_rank": 2317, "group": "Book", "category": "Science Fiction & Fantasy", "subcategory": "Fantasy", "similar_ids": ["0812513738","0812550307","0812513754","0812550285","0812550293"] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-07-19", "rating": 5, "votes": 0, "helpful_votes": 0 }, "product": { "id": "1578151333", "title": "The Great Hunt (Wheel of Time (Audio))", "sales_rank": 529797, "group": "Book", "category": "Books on Tape", "subcategory": "Literature & Fiction", "similar_ids": ["0812513738","0812550307","0812513754","0812550285","0812550293"] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-07-20", "rating": 5, "votes": 0, "helpful_votes": 0 }, "product": { "id": "0374373620", "title": "A Swiftly Tilting Planet", "sales_rank": 98444, "group": "Book", "category": "Teens", "subcategory": "Literature & Fiction", "similar_ids": ["0440498058","0440405483","0440208149","0440917190","0440901839"] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-07-20", "rating": 5, "votes": 0, "helpful_votes": 0 }, "product": { "id": "0440401585", "title": "A Swiftly Tilting Planet (Yearling Books)", "sales_rank": 22987, "group": "Book", "category": "Teens", "subcategory": "Literature & Fiction", "similar_ids": ["0440498058","0440405483","0440208149","0440917190","0440901839"] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-07-20", "rating": 5, "votes": 0, "helpful_votes": 0 }, "product": { "id": "0440901588", "title": "A Swiftly Tilting Planet", "sales_rank": 102542, "group": "Book", "category": "Teens", "subcategory": "Literature & Fiction", "similar_ids": ["0440498058","0440405483","0440208149","0440917190","0440901839"] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-07-20", "rating": 5, "votes": 0, "helpful_votes": 0 }, "product": { "id": "0807209163", "title": "A Swiftly Tilting Planet", "sales_rank": 532551, "group": "Book", "category": "Books on Tape", "subcategory": "Children's Books", "similar_ids": ["0440498058","0440405483","0440208149","0440917190","0440901839"] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-07-20", "rating": 5, "votes": 1, "helpful_votes": 1 }, "product": { "id": "0374373620", "title": "A Swiftly Tilting Planet", "sales_rank": 98444, "group": "Book", "category": "Teens", "subcategory": "Literature & Fiction", "similar_ids": ["0440498058","0440405483","0440208149","0440917190","0440901839"] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-07-20", "rating": 5, "votes": 1, "helpful_votes": 1 }, "product": { "id": "0440401585", "title": "A Swiftly Tilting Planet (Yearling Books)", "sales_rank": 22987, "group": "Book", "category": "Teens", "subcategory": "Literature & Fiction", "similar_ids": ["0440498058","0440405483","0440208149","0440917190","0440901839"] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-07-20", "rating": 5, "votes": 1, "helpful_votes": 1 }, "product": { "id": "0440901588", "title": "A Swiftly Tilting Planet", "sales_rank": 102542, "group": "Book", "category": "Teens", "subcategory": "Literature & Fiction", "similar_ids": ["0440498058","0440405483","0440208149","0440917190","0440901839"] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-07-20", "rating": 5, "votes": 1, "helpful_votes": 1 }, "product": { "id": "0807209163", "title": "A Swiftly Tilting Planet", "sales_rank": 532551, "group": "Book", "category": "Books on Tape", "subcategory": "Children's Books", "similar_ids": ["0440498058","0440405483","0440208149","0440917190","0440901839"] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-07-20", "rating": 5, "votes": 18, "helpful_votes": 17 }, "product": { "id": "0880113626", "title": "Coaching Volleyball Successfully", "sales_rank": 153469, "group": "Book", "category": "Sports", "subcategory": "Coaching", "similar_ids": ["0736039678","0736001360","1570281246","1570280843","0880117788"] } }
  { "customer_id": "ATVPDKIKX0DER", "review": { "date": "1995-07-20", "rating": 5, "votes": 2, "helpful_votes": 0 }, "product": { "id": "039330700X", "title": "Wonderful Life", "sales_rank": 68718, "group": "Book", "category": "Science", "subcategory": "Biological Sciences", "similar_ids": ["0393308189","0393308197","156098659X","1573922501","0393311031"] } }
  ```

  `copy` 명령으로 데이터를 로딩하자.

  ```
  create table json_data2 (data jsonb);

  \copy json_data2 from 'work/test_data.json';
  ```

### Query

* Child 찾기

  ```
  edb=# select data->'name' from json_data ;
     ?column?    
  ---------------
   "Apple Phone"

  (2 rows)

  edb=# select pg_typeof(data->'name') from json_data ;
   pg_typeof 
  -----------
   json
   json
  (2 rows)

  edb=# select data->>'name' from json_data ;
    ?column?   
  -------------
   Apple Phone

  (2 rows)

  edb=# select pg_typeof(data->>'name') from json_data ;
   pg_typeof 
  -----------
   text
   text
  (2 rows)
  ```

	`->`와 `->>` 두가지 operator로 자식 node를 찾을 수 있다. `->`의 결과는 여전히 `json` 타입이기 때문에 계속해서 다른 operator를 적용할 수 있지만 `->>`의 결과물은 `text`인 점이 다르다. `->>`는 최종 결과물의 값을 받아올때 사용된다.

  ```
  edb=# select data->'names' from json_data ; 
                      ?column?                    
  ------------------------------------------------

   [                                             +
       {"type": "firstname", "value": "John"},   +
       {"type": "middlename", "value": "Joseph"},+
       {"type": "middlename", "value": "Carl"},  +
       {"type": "lastname", "value": "Salinger"} +
       ]
  (2 rows)

  edb=# select data->'names'->1 from json_data ;
                   ?column?                  
  -------------------------------------------

   {"type": "middlename", "value": "Joseph"}
  (2 rows)

  edb=# select data->'names'->1->>'type' from json_data ;
    ?column?  
  ------------

   middlename
  (2 rows)

  edb=# select data->>'names'->1 from json_data ;
  ERROR:  operator does not exist: text -> integer
  LINE 1: select data->>'names'->1 from json_data ;
                               ^
  HINT:  No operator matches the given name and argument type(s). You might need to add explicit type casts.
  ```

  `#>`과 `#>>`을 사용하면 좀더 편리하게 경로 탐색을 할 수 있다.
  ```
  edb=# select data #>'{names,1,type}' from json_data ; 
     ?column?   
  --------------

   "middlename"
  (2 rows)

  edb=# select data #>>'{names,1,type}' from json_data ;    
    ?column?  
  ------------

   middlename
  (2 rows)
  ```

  추가적인 operator 들에 대한 정보를 아래 메뉴얼에서 확인해 보자.
  >http://www.postgresql.org/docs/current/static/functions-json.html

### Table 변환

`row_to_json()`함수로 테이블의 row를 `json` type으로 변환할 수 있다.
```
edb=# select row_to_json(emp) from emp;
                                                              row_to_json
---------------------------------------------------------------------------------------------------------------------------------------
 {"empno":7499,"ename":"ALLEN","job":"SALESMAN","mgr":7698,"hiredate":"1981-02-20T00:00:00","sal":1600.00,"comm":300.00,"deptno":30}
 {"empno":7521,"ename":"WARD","job":"SALESMAN","mgr":7698,"hiredate":"1981-02-22T00:00:00","sal":1250.00,"comm":500.00,"deptno":30}
 {"empno":7654,"ename":"MARTIN","job":"SALESMAN","mgr":7698,"hiredate":"1981-09-28T00:00:00","sal":1250.00,"comm":1400.00,"deptno":30}
 {"empno":7698,"ename":"BLAKE","job":"MANAGER","mgr":7839,"hiredate":"1981-05-01T00:00:00","sal":2850.00,"comm":null,"deptno":30}
 {"empno":7844,"ename":"TURNER","job":"SALESMAN","mgr":7698,"hiredate":"1981-09-08T00:00:00","sal":1500.00,"comm":0.00,"deptno":30}
 {"empno":7782,"ename":"CLARK","job":"MANAGER","mgr":7839,"hiredate":"1981-06-09T00:00:00","sal":29400.00,"comm":null,"deptno":10}
 {"empno":7839,"ename":"KING","job":"PRESIDENT","mgr":null,"hiredate":"1981-11-17T00:00:00","sal":60000.00,"comm":null,"deptno":10}
 {"empno":7934,"ename":"MILLER","job":"CLERK","mgr":7782,"hiredate":"1982-01-23T00:00:00","sal":15600.00,"comm":null,"deptno":10}
 {"empno":7369,"ename":"SMITH","job":"CLERK","mgr":7902,"hiredate":"1980-12-17T00:00:00","sal":1990.66,"comm":null,"deptno":20}
 {"empno":7566,"ename":"JONES","job":"MANAGER","mgr":7839,"hiredate":"1981-04-02T00:00:00","sal":7402.75,"comm":null,"deptno":20}
 {"empno":7788,"ename":"SCOTT","job":"ANALYST","mgr":7566,"hiredate":"1987-04-19T00:00:00","sal":7464.96,"comm":null,"deptno":20}
 {"empno":7876,"ename":"ADAMS","job":"CLERK","mgr":7788,"hiredate":"1987-05-23T00:00:00","sal":2737.15,"comm":null,"deptno":20}
 {"empno":7902,"ename":"FORD","job":"ANALYST","mgr":7566,"hiredate":"1981-12-03T00:00:00","sal":7464.96,"comm":null,"deptno":20}
 {"empno":7900,"ename":"JAMES","job":"CLERK","mgr":7698,"hiredate":"1981-12-03T00:00:00","sal":100.00,"comm":null,"deptno":30}
(14 rows)
```

## `JSONB`

`JSONB`는 `JSON`과 매우 유사하며 내부에 저장되는 방식이 다르다. 앞에서 만든 데이터를 변환해 보자.

```
edb=# select data from json_data; 
                      data
------------------------------------------------
  {  "name": "Apple Phone",                    +
     "type": "phone",                          +
     "brand": "ACME",                          +
     "price": 200,                             +
     "available": true,                        +
     "warranty_years": 1                       +
   } 
 {"full name": "John Joseph Carl Salinger",    +
   "names":                                    +
     [                                         +
     {"type": "firstname", "value": "John"},   +
     {"type": "middlename", "value": "Joseph"},+
     {"type": "middlename", "value": "Carl"},  +
     {"type": "lastname", "value": "Salinger"} +
     ]                                         +
   }
(2 rows)


edb=# select data::jsonb from json_data;
                                                                                                             data
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 {"name": "Apple Phone", "type": "phone", "brand": "ACME", "price": 200, "available": true, "warranty_years": 1}
 {"names": [{"type": "firstname", "value": "John"}, {"type": "middlename", "value": "Joseph"}, {"type": "middlename", "value": "Carl"}, {"type": "lastname", "value": "Salinger"}], "full name": "John Joseph Carl Salinger"}
(2 rows)
```

위에서 보는 바와 같이 `JSONB`는 데이터를 파싱한 다음 바이너리 형태로 저장하고 나중에 화면에 출력할 일이 있으면 다시 encoding하는 절차를 거친다. 따라서 INPUT/OUTPUT의 관점에서는 추가적인 overhead가 발생한다. 반면 저장된 데이터를 처리하는 경우에는 `JSON`에 비해 훨씬 휴율적이다. 따라서 용도에 맞춰서 사용하는 것이 좋다.

### Indexing

`JSON`과 `JSONB`의 가장 큰 차이점은 index를 생성 할 수 있는지 일 것이다. 아래와 같은 expression index는 `JSON`에도 생성할 수 있다. 이 operator의 결과가 `text` type의 일반적인 값이기 때문이다.

```
edb=# create index on json_data2 ((data #>> '{product,category}'));
CREATE INDEX
```

하지만 product의 항목중 category 이외에 다른 동적으로 변화하는 항목들을 조회해야할 경우 이 방식은 한계가 있다. `JSONB`의 경우 아래와 같이 값 자체에 GIN index를 생성할 수 있다.

```
CREATE INDEX on json_data2 USING GIN (data);
```

이제 아래와 같은 조회 쿼리가 index scan을 할 수 있다. 아쉽게도 지금의 테스트 데이터는 너무 작기 때문에 옵티마이저가 index를 사용하지는 않는다.

```
select data #>> '{product,title}'
from json_data2
where data @> '{"product": {"category": "Science"}}';

         ?column?          
---------------------------
 Invention and Evolution
 Fingerprints of the Gods 
 Wonderful Life
(3 rows)
```

위의 index는 아래의 4가지 operator를 모두 지원한다.

* JSON @> JSON is a subset
* JSON ? TEXT contains a value
* JSON ?& TEXT[] contains all the values
* JSON ?| TEXT[] contains at least one value

하지만 index의 용량이 다소 크다는 문제점이 있다. 대부분의 경우 `@>`만으로도 충분하기 때문에 이럴 경우에는 아래와 같이 index를 생성하면 훨씬 효율적이다. 크기도 훨씬 작고 `@>` operator를 처리하는데에는 성능도 빠르다.

```
CREATE INDEX on json_data2 USING GIN (data jsonb_path_ops);
```
