# Installation Hands-on Lab

## 설치 및 DB 기동

1. `EPAS9.5_WIN_BIN.zip` 파일 `C:\`에 압축 해제
2. `C:\PostgresPlus\9.5AS\bin` 시스템 path에 등록
3. `CMD.EXE` 실행
4. `C:\PostgresPlus\9.5AS\pgplus_env.bat` 실행

  ```
  C:\>C:\PostgresPlus\9.5AS\pgplus_env.bat
  ```
  
  이렇게 하면 적절한 환경변수들을 자동으로 적용해 준다.

5. `pg_ctl` 도움말 확인

  ```
  C:\>pg_ctl.exe --help
  pg_ctl 프로그램은 PostgreSQL 서버를 초기화, 시작, 중지, 제어하는 도구입니다.

  사용법:
    pg_ctl init[db]               [-D 데이터디렉터리] [-s] [-o "옵션"]
    pg_ctl start   [-w] [-t 초] [-D 데이터디렉터리] [-s] [-l 로그파일] [-o "옵션"]
    pg_ctl stop    [-W] [-t 초] [-D 데이터디렉터리] [-s] [-m 중지모드]
    pg_ctl restart [-w] [-t 초] [-D 데이터디렉터리] [-s] [-m 중지모드]
                   [-o "옵션"]
    pg_ctl reload  [-D 데이터디렉터리] [-s]
    pg_ctl status  [-D 데이터디렉터리]
    pg_ctl promote [-D 데이터디렉터리] [-s]
    pg_ctl kill    시그널이름 PID
    pg_ctl register   [-N 서비스이름] [-U 사용자이름] [-P 암호] [-D 데이터디렉터리]
                      [-S 시작형태] [-w] [-t 초] [-o "옵션"]
    pg_ctl unregister [-N 서비스이름]

  일반 옵션들:
    -D, --pgdata=데이터디렉터리  데이터베이스 자료가 저장되어있는 디렉터리
    -e SOURCE              서비스가 실행 중일때 쌓을 로그를 위한 이벤트 소스
    -s, --silent           일반적인 메시지는 보이지 않고, 오류만 보여줌
    -t, --timeout=초      -w 옵션 사용 시 대기 시간(초)
    -V, --version          버전 정보를 보여주고 마침
    -w                     작업이 끝날 때까지 기다림
    -W                     작업이 끝날 때까지 기다리지 않음
    -?, --help             이 도움말을 보여주고 마침
  (기본 설정은 중지 할 때는 기다리고, 시작이나 재시작할 때는 안 기다림.)
  -D 옵션을 사용하지 않으면, PGDATA 환경변수값을 사용함.

  start, restart 때 사용할 수 있는 옵션들:
    -c, --core-files       이 플랫폼에서는 사용할 수 없음
    -l, --log=로그파일     서버 로그를 이 로그파일에 기록함
    -o 옵션들              PostgreSQL 서버프로그램인 postgres나 initdb
                           명령에서 사용할 명령행 옵션들
    -p PATH-TO-POSTGRES    보통은 필요치 않음

  stop, restart 때 사용 할 수 있는 옵션들:
    -m, --mode=모드        모드는 "smart", "fast", "immediate" 중 하나

  중지방법 설명:
    smart       모든 클라이언트의 연결이 끊기게 되면 중지 됨
    fast        클라이언트의 연결을 강제로 끊고 정상적으로 중지 됨
    immediate   그냥 무조건 중지함; 다시 시작할 때 복구 작업을 할 수도 있음

  사용할 수 있는 중지용(for kill) 시그널 이름:
    ABRT HUP INT QUIT TERM USR1 USR2

  서비스 등록/제거용 옵션들:
    -N SERVICENAME  서비스 목록에 등록될 PostgreSQL 서비스 이름
    -P PASSWORD     이 서비스를 실행할 사용자의 암호
    -U USERNAME     이 서비스를 실행할 사용자 이름
    -S 시작형태     서비스로 등록된 PostgreSQL 서버 시작 방법

  시작형태 설명:
    auto       시스템이 시작되면 자동으로 서비스가 시작됨 (초기값)
    demand     수동 시작

  Report bugs to <support@enterprisedb.com>.
  ```

4. DB 기동

  ```
  C:\>pg_ctl.exe start -D C:\PostgresPlus\9.5AS\data
  서버를 시작합니다

  C:\Users\dialogbox>2016-06-08 19:21:56 KST LOG:  redirecting log output to logging collector process
  2016-06-08 19:21:56 KST HINT:  Future log output will appear in directory "pg_log".
  ```
  
  앞서 실행한 `pgplus_env.bat`에서 환경변수를 설정 해 주기 때문에 `-D C:\PostgresPlus\9.5AS\data`를 생략해도 동작 한다.

5. `psql`을 실행하여 DB에 접속 해 본다.

  ```
  C:\>psql
  enterprisedb 사용자의 암호:
  psql (9.5.3.8)
  도움말을 보려면 "help"를 입력하십시오.

  edb=# select version();
                               version
  -----------------------------------------------------------------
   EnterpriseDB 9.5.3.8, compiled by Visual C++ build 1800, 64-bit
  (1개 행)
  
  edb=#
  ```

## 서비스 등록 방법

서비스 등록시 관리자 권한으로 `CMD.EXE`실행

```
C:\>C:\PostgresPlus\9.5AS\pgplus_env.bat

C:\>pg_ctl.exe register -N epas95 -D C:\PostgresPlus\9.5AS\data

C:\>net start epas95
epas95 서비스를 시작합니다..
epas95 서비스가 잘 시작되었습니다.
```
