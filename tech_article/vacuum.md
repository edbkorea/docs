# Vacuum
## Vacuum 이란
### MVCC (Multi Version Concurrency Control)
MVCC는 여러 사용자가 조회와 동시에 변경이 가능하도록 하는 기능이므로 이를 구현하기 위해서는 DB내 어딘가에 데이터의 여러 버전이 저장되어 있어야 한다. 어떤 방식으로 버전들을 저장하고 조회할 것인지는 다양한 방식이 존제하고 각각의 장단점이 있다.

#### PostgreSQL의 MVCC 구현
PostgreSQL은 변경이 발생하면 기존 데이터는 그대로 두고 새로운 데이터를 하나 더 생성하는 전략을 사용한다. 새로운 데이터 역시 기존 데이터와 마찬가지로 __테이블 내부에__ 생성 되고 물리적으로 보면 테이블에 record가 2배로 늘어난 것과 동일하다.

아래 예를 보자

![alt MVCC concept](./images/MVCC_concept1.png "MVCC concept")

1. XID(transaction ID) 100에 첫번째 record가 insert
2. 101에 두번째 데이터가 insert 되었다.
3. XID 106이 첫번째 데이터의 col1을 20으로 update하였다.


만약 XID 100번이 아직 남아있다면 101번 데이터는 그 이후에 만들어 졌기 때문에 `(10, 'ABC')`인 데이터 1건만 보일 것이다. 또한

* XID 101번 transaction에게는 `(10, 'ABC'), (20, 'CDE')` 두건의 데이터가 보이게 된다.
* XID 106 이전의 transaction은 `(10, 'ABC'), (20, 'CDE')`를 보게 된다.
* 그 이후의 transaction은 `(20, 'CDE'), (20, 'ABC')`를 보게 된다.

이런 방식으로 MVCC를 구현하는 것이 PostgreSQL의 전략이다.

#### Vacuum
이 방식은 UNDO와 같이 별도의 공간을 사용하지 않고 테이블을 같이 사용한다는 특징이 있다. 이로 인해 발생하는 여러가지 장단점이 있는데 가장 특징적인 점으로는 vacuum을 들 수 있다.

위와 같은 방식에서는 update와 delete에 의해 더이상은 필요없어진 데이터가 점점 늘어나게 됨으로, 이 데이터를 정리해 주는 작업이 vacuum이다.

아래의 테스트 케이스를 보자

```
edb=# create table test (col1 integer, col2 varchar(100));
CREATE TABLE
edb=# alter table test set (autovacuum_enabled = false);
ALTER TABLE
edb=# insert into test values (10, 'ABC');
INSERT 0 1
edb=# insert into test values (20, 'CDE');
INSERT 0 1
edb=# update test set col1 = 30 where col1 = 10;
UPDATE 1
edb=# update test set col2 = 'DEF' where col1 = 20;
UPDATE 1
edb=# select * from test;
 col1 | col2
------+------
   30 | ABC
   20 | DEF
(2 rows)

```

2건의 데이터가 들어있고 각각 1번씩 update가 되었다. 테스트를 위해 autovacuum은 disable 해 두었다. 이제 이 테이블에 vacuum을 해 보면 아래와 같다.

```
edb=# vacuum verbose test;
INFO:  vacuuming "public.test"
INFO:  "test": found 2 removable, 2 nonremovable row versions in 1 out of 1 pages
DETAIL:  0 dead row versions cannot be removed yet.
There were 0 unused item pointers.
Skipped 0 pages due to buffer pins.
0 pages are entirely empty.
CPU 0.00s/0.00u sec elapsed 0.00 sec.
VACUUM
```

2번째 줄을 보면 2건의 `removable`이 정리되고 2건의 `nonremovable`이 남았다는 것을 볼 수 있다.

vacuum은 removable tuple에 표시만 한다. 그 공간이 정리가 되어 없어지지는 않고 나중에 빈공간이 필요할 경우 이 공간을 사용하도록 해 준다. 빈 공간을 모두 제거하여 완전히 정리하고 싶다면 `vacuum` 대신 `vacuum full` 명령을 사용할 수 있다.

```
edb=# select pg_relation_filepath('test');
 pg_relation_filepath
----------------------
 base/14845/16387
(1 row)

edb=# vacuum full verbose test;
INFO:  vacuuming "public.test"
INFO:  "test": found 0 removable, 2 nonremovable row versions in 1 pages
DETAIL:  0 dead row versions cannot be removed yet.
CPU 0.00s/0.00u sec elapsed 0.00 sec.
VACUUM
edb=# select pg_relation_filepath('test');
 pg_relation_filepath
----------------------
 base/14845/16390
(1 row)

```

이 경우 table에 lock이 걸리고 모든 live tuple만 모아서 새로운 테이블을 만들고 기존 테이블을 drop한다. `pg_relation_filepath` 함수로 확인해 보면 테이블이 저장된 파일의 이름이 변경된 것을 볼 수 있다. Full vacuum은 성능에 미치는 영향이 크기 때문에 운영 환경에서는 조심해서 사용해야 한다.

#### Vacuum Freeze & Transaction ID Wrap-around
또 다른 아키택처적인 특직으로 `vacuum freeze`가 있다. 앞서 MVCC 구현이 각 tuple별 xmin, xmax 값을 이용 한다는 점을 설명했다. 이 두 값은 각각 32bit integer로 되어 있기 때문에 최대 20억 정도의 값을 가질 수 있는데 이 값들은 공간적인 오버헤드이기 때문에 크기를 제한한 것이다.

그런데 20억이라는 값은 충분한 값일까? 문제는 그렇지 않다는 점이다. 실제 운영 환경에서 20억번의 transaction은 생각보다 금방 발생하며 부하가 높은 시스템의 경우 몇달 안에 20억번의 transaction이 발생될 수 있다.

XID가 20억을 넘어가게 되면 숫자가 초기화 되고 다시 2 부터 시작하게 되는데, 테이블에 아주 오래된 tuple들이 남아있다면 MVCC 메커니즘에 문제가 생긴다.

XID가 20억을 넘어서 다시 2가 되었다고 가정하고 위의 예를 다시 생각해 보자. 아주 오래전이 만들어진 데이터들이 갑자기 미래의 데이터로 보이기 시작한다. 예제의 데이터 모두 (min 100, 101, 106) 현재 XID인 2보다 크기 때문에 현재 transaction에게 보이면 안되는 데이터가 되어 버린다. 사용자 입장에서는 데이터가 갑자기 사라져 버리는 것이다.

이런 일이 벌어지면 안되기 때문에 MVCC 버전 처리가 더이상 필요 없는 데이터, 즉 오래된 데이터들의 xmin 값을 특수한 XID 값으로 정리해 주는 작업이 필요하다. 이미 20억번의 transaction이 발생하고 한바퀴를 돌아온 시점에서 예제의 데이터 들은 뭉뚱그려서 아주 오래전 부터 있었던 데이터라는 것 만 알면 되며, 지금 시점의 transaction들에게는 그냥 무조건 보여주면 되는 데이터가 된다는 의미이다. 이를 처리해 주는 작업이 바로 `vacuum freeze`이다.

`vacuum freeze`가 실행 되면 이런 데이터들의 xmin을 일괄적으로 2(`FrozenTransactionId`)로 변경하게 된다. 2는 DB에 존제 가능한 가장 작은 XID 값이기 때문에 이렇게 frozen된 데이터들은 더이상 위와 같은 문제를 격지 않아도 되기 때문이다.
