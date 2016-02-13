# pgpool 접속 설정

pgpool의 가장 큰 문제점은 설정 값들의 이름이 실제 의미와 다른경우가 많다는 점 입니다. 따라서 설정 값만 보고 의미를 유추하지 마시고 반드시 문서를 찾아보시기 바랍니다.


## num_init_children : 최대 접속 가능 client 수

이 설정의 의미는 이 숫자만큼 pgpool 프로세스를 미리 띄워 두겠다는 의미 입니다. 설정값 이름은 init 이지만 더 늘어날 수가 없기 때문에 이 값이 이 pgpool 데몬을 통해 접속 할 수 있는 최대 client 수가 됩니다.

즉 **__최대 접속 가능 client 수__**의 의미로 이해 하시면 됩니다.

## child process

pgpool에서 말하는 children, child, server process 등의 용어들은 이 `num_init_children` 값에 의해 미리 만들어지게 되는 아래 프로세스들을 의미 합니다.

```
enterpr+ 14651 14648  0 16:51 pts/1    00:00:00 pgpool: wait for connection request
enterpr+ 14940 14648  0 16:57 pts/1    00:00:00 pgpool: wait for connection request
```

`num_init_children = 2`일 경우 위와 같이 2개의 child가 생성 되게 되며 각각이 client의 접속을 처리하게 됩니다. 따라서 최대 2개의 client만이 동시에 접속할 수 있게 되고 3번째 이후에는 대기로 빠지게 됩니다. 이 child 들은 pgpool 기동시에 `num_init_children` 만큼 만들어지고 이후에 늘어나지도 줄어들지도 않습니다.

따라서 ***반드시*** 이 값을 ++최대 동시 접속 수 만큼 늘려++ 잡아 주셔야 합니다.

아래 두 parameter들은 child의 주기적인 재기동에 관한 파라메터입니다.

* `child_max_connections` : 흔히 1개의 child가 동시에 몇개의 접속을 처리할 것인가로 오해 하시는 경우가 있는데, 몇개의 client를 처리한 뒤 child가 재기동 될 것인지를 의미합니다. **어떤 경우에도 1개의 child는 동시에 1개의 client만 처리 합니다.**

* `child_life_time`: client 접속이 몇초간 없을 경우 child를 재기동 할 것인지를 의미합니다.

위 두가지 설정은 혹시 모를 child의 memory leak 등을 방지하고자 주기적으로 child를 죽였다가 다시 띄우도록 하는 설정입니다. 따라서 이 두가지 설정은 **동시에 몇개의 client가 접속 할 수 있는지에는 아무런 영향을 미치지 않습니다.**

* listen_backlog_multiplier : `num_init_children`을 넘어서는 동시 접속이 들어 오게 되면 기존 접속이 끝나기를 대기를 하게 되는데, 이 값은 그 때 대기가 가능한 client의 수를 의미 합니다. 즉,
 * `num_init_children`을 넘어가면 대기하게 되고
 * `num_init_children * listen_backlog_multiplier`를 넘어가면 대기로 빠지지도 않고 에러가 발생하게 됩니다.

대부분의 경우 DB 접속이 안되서 대기로 빠지게 되면 이미 장애 상황이기 때문에, 이 값을 늘려서 더 많이 대기 하게 하는것은 아무 의미가 없습니다.

한줄로 요약하면, **__어떤 경우에도 1개의 child는 동시에 1개의 client만 처리하고 다른 설정들은 모두 부수적이다.__**

## connection_cache

이 설정 역시 매우 많은 분들이 오해를 하시는데, 대부분의 경우 필요가 없는 기능 입니다. AP의 connection pooling과 `connection_cache`를 동시에 사용할 경우 문제가 발생 할 수 있으니 다음 내용을 참고하시기 바랍니다.

* `connection_cache`: Child layer의 connection pool을 사용할 것인지 설정.
* `max_pool` : 각 child 가 보유하게 될 connection cache의 크기

위에서 설명한 child는 client와 실제 DB 사이의 통신을 중계하는 역활을 하게 됩니다. 이때 DB와의 접속을 backend connection이라고 부르는데, client가 빈번하게 접속/해제를 하게 되면 backend connection역시 빈번하게 생성/해제를 반복하게 되며 많은 부하가 발생 될 수 있습니다. 이를 방지하기 위해서 backend connection을 caching하기 위한 설정이 `connection_cache`입니다.

이 설정을 사용 하시면 각 child가 `max_pool` 만큼의 backend connection을 cache하게 됩니다. 그런데 위에서 강조한 바와 같이 1개의 client는 동시에 1개의 client만 처리하기 때문에 **__동시에는 각 child의 cache 중 최대 1개만 사용이 되게 됩니다__**.

응용단에서 connection pooling을 하지 않아서 client 접속/해제가 빈번하게 일어나는 경우에는 문제가 되지 않고 오히려 도움이 되는 기능이지만 반대의 경우 문제가 될 수 있습니다.

응용단에서 connection pooling을 사용하는 경우

* Client와 child간의 접속은 끊어지지 않고 오랜기간 지속되며, 재사용 된다.
* Child와 backend DB와의 접속 역시 client가 접속하는 순간 생성 되어 끊어지지 않고 유지된다.
* 이 기간 동안 child의 connection cache에 들어있는 backend connection들은 사용 되지 못하고 계속 inactive 상태로 남아있게 된다.

응용에서 사용하는 DB와, DB 계정이 단 1개 인 경우 connection cache에 다른 backend connection이 들어갈 일이 없기 때문에 낭비되는 일이 발생하지 않습니다. 하지만 이런 경우가 아닌 경우 문제가 될 수 있고, 이 기능으로 인해 얻는 점이 거의 없기 때문에 **응용 layer에 connection pool이 있다면** 아래와 같은 설정을 권장 합니다.

## AP layer의 connection pool이 있는 경우 권고사항

* `num_init_children`: 최대 가능한 동시 접속 수 이상으로 설정
* `connection_cache = off`
* `max_pool = 1`
* 각 backend DB의 `max_connections`: `num_init_children` 이상. 모든 접속이 pgpool을 통해서만 들어온다면 `num_init_children` 이상의 client 접속은 발생하지 않아야 정상 입니다. 하지만 vacuum이나 기타 관리자 접속등에 필요하기 때문에 충분한 여유를 확보해 주시기 바랍니다.