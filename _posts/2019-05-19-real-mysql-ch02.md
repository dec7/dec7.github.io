---
layout: post
title: "Real MySql Ch02"
description: "Real MySql Ch"
date: 2019-05-19
tags: [sql,anti,pattern]
comments: true
share: true
---

## 설치와 열정
### 소스로부터 설치
```
./configure \
    '--prefix=/usr/local/mysql'\
    '--localstatedir=/usr/local/mysql/data'\
    '--libexecdir=/usr/local/mysql/bin'\
    '--with-comment=Toto Mysql standard 64bit server'\
    '--with-server-suffix=-toto_standard'\
    '--enable-thread-safe-client'\
    '--enable-local-infile'\
    '--enable-assembler'\
    '--with-pic'\
    '--with-fast-mutexes'\
    '--with-client-ldflags=-static'\
    '--with-mysqld-ldflags=-static'\
    '--with-big-tables'\
    '--with-readline'\
    '--with-extra-charsets=complex'\
    '--with-plugins=partition,archive,blackhole,csv,federated,heap,myisam,myisammrg,innodb_plugin'\
    '--with-zlib-dir=bundled'\
    'CC=gcc' 'CXX=gcc' 'CFLAGS=-02' 'CXXFLAGS=-02'
```
- ```configure --help```
- ```--prefix```
    - mysql 서버를 설치할 디렉토리
- ```--localstatedir```
    - mysql 서버 데이터 파일, 정보성 파일의 기본 위치 
    - 로그, 복제를 위한 바이너리, 릴레이 로그의 실행 상태 
- ```--libexecdir```
    - 서버의 실행 프로그램 mysqld와 클라이언트 mysql 및 기타 유틸리티 프로그램이 설치되는 위치 
- ```--with-comment, --with-server-suffix```
    - mysql 서버에 로그인 후 처음 출력되는 기본적인 정보성 내용 중 mysql 서버 버전과 관련된 내용
    - 위의 버전 정보에 코멘트를 추가하는 옵션
- ```--with-extra-charsets```
    - mysql 서버에서 추가로 어떤 문자 집함을 더 포함할지 선택하는 옵션 
    - 필요한 문자 타입을 나열하면 됨
    - complex
        - 특별히 지정하지 않고 여러가지 문자 혼용할 경우
    - euckr ...
        - euckr과 지정된 문자집합 위주로 사용할 경우 
- ```--with-plugins```
    - MySql5.1 부터 플러그인 스토리지 엔진 모델을 채택하고 있음
    - 어떤 스토리지 엔진을 쓸수 있는지는 ```configure --help```로 확인 가능
    - 위 명시된 플러그인 엔진을 포함하길 권장
        - innodb는 플러그인 이전의 InnoDB스토리지 엔진, innodb_plugin은 플러그인 버전의 InnoDB 버전 
        - innodb_plugin을 선택하길 권장 

- 빌드
    - configure
        - 실행시, 현재 플랫폼에 맞게 사용할 수 있는 라이브러리나 API로 컴파일될 수 있게 Makefile이 만들어짐
        - 이때, 원하는 기능, 더 나은 라이브러리 혹은 시스템 기능을 사용할 수 있는지 점검 가능 
    - make
        - 소스코드를 컴파일하고 링크하는 역할 
    - make install
        - make 명령으로 만드렁진 이진 실행 프로그램을 confiure의 prefix 로 지정해준 디렉토리로 복사하는 작업 

- 주의사항
    - mysql 직접 컴파일 시 각 디렉토리에 대한 권한 설정 주의 
    - root가 아닌 별도 계정으로 기동하기 위해 디렉토리 권한 변경 필요 
    - mysql 계정으로 실행하기 위해 
```
$ groupadd dba
$ useradd -g dba mysql

$ chown -R root /usr/local/mysql
$ chown -R mysql /usr/local/mysql/data
$ chown -R mysql /usr/local/mysql/tmp
$ chown -R mysql /usr/local/mysql/logs
$ chown -R dba /usr/local/mysql
```
- mysql 서버가 실행시 권한/테이블 메타 정보를 관리하는 시스템 필요
- mysql 서버를 직접 컴파일해서 설치할 경우 이러한 테이블을 직접 생성해야함
    - MYSQL_HOME/bin/mysql_install_db 스크립트로 실행 가능 

### 서버 설정 
- mysql은 my.cnf 라는 하나의 설정파일만 사용 
    - 여러 경로에 있는 파일을 순차적으로 탐색하면서 처음 발견된 파일을 사용
    - 직접 컴파일할 경우 경로가 다를 수 있음 
    - ```mysqld --verbose --help```
    - ```mysql --help```
    - 한 장비에 여러 인스턴스가 올라가는 경우, 설정파일 경로에 따라 충돌할 수도 있음 
```
[client]
default-character-set = utf-8

[mysql]
socket      = /usr/local/mysql/tmp/mysql.sock
port        = 13304

[mysqldump]
socket      = /usr/local/mysql/tmp/mysql.sock
port        = 13305

[mysqld_safe]

[mysqld]
socket      = /usr/local/mysql/tmp/mysql.sock
port        = 13306

```
- client
    - mysqld, mysqld_safe 를 제외한 대부분 클라이언트 프로그램이 공유하는 영역 
- mysqld_safe
    - mysql이 비정상적으로 종료되었을 때  재시작하는 일만 하는 프로세스 
    - 잘 사용되지 않으나, mysql 서버 타임존 설정 시 꼭 사용 필요 


#### mysql 시스템 변수의 특징 
- ```show variables```
- 설정파일의 내용을 읽어 시스템 변수로 저장

#### 글로벌변수와 세션변수
- 동시 적용된 변수의 경우 Both
- 변수로 서버 인스턴스에서 전체적인 영향을 미치는 변수를 의미함 
    - 주로 mysql 서버 자체에 관한 설정 이 많음
    - query_cache_size, key_buffer_size (MyISam 키 캐시 크기), innodb_buffer_pool_size
- 세션범위의 시스템 변수는 mysql 클라이언트가 서버에 접속할 때 기본적으로 부여하는 옵션값 제어 
    - 기본값 외에 개별 커넥션 단위로 다른 값으로 변경 할 수 있음
    - auto commit 변수가 대표적 
    - 세션변수는 각 커넥션별로 설정값을 다르게 지정할 수 있으며, 연결된 커넥션 변수는 서버에서 강제로 변경 불가 

#### 동적변수와 정적변수 
- mysql 서버 시스템 변수는 서버가 기동중인 상태에서 변경 가능한지 여부에 따라 동적/정적 변수로 나뉨 
- set 명령을 통해 설정내용을 바꿀수 있으나, my.cnf파일이 바뀌는건 아니므로 현재 기동중인 인스턴스에만 유효 

#### my.cnf 설정 파일 
- mysql 을 직접 컴파일 / 패키지 행태로 설치시 mysql 설정 파일 작성 필요
- my.cnf 서버설정은 버전별 차이가 있으므로 my.cnf파일 변경 후 재시작 후 로그 확인이 반드시 필요 


#### mysqld 설정 그룹 
- /usr/local/mysql
    - mysql 홈 디렉토리 
- /usr/local/mysql/bin
    - 서버/클라이언트/유틸리티 저장
- /usr/local/mysql/data
    - 데이터 파일 저장 
- /usr/local/mysql/logs
    - 바이너리 로그와 릴레이 로그 를 포함한 모든 로그 디렉터리 
- /usr/local/mysql/tmp
    - 내부 임시 테이블, 소켓 파일이 저장 

```
[mysqld]
## MySql 서버 기본 옵션 
server-id       = 1

user            = mysql
port            = 13306
basedir         = /usr/local/mysql
datadir         = /usr/local/mysql/data
tmpdir          = /usr/local/mysql/tmp
socket          = /usr/local/mysql/tmp/mysql.sock

character-set-server    = utf8
collation-server        = utf8_general_ci
default-storage-engine  = InnoDB
skip-name-resolve
skip-external-locking


## MySql 스케줄러를 사용하면 아래 event-scheduler 옵션을 on으로 변경 
event-scheduler     = OFF
sysdate-is-now

back_log                = 100
max_connections         = 300
max_connect_errors      = 999999
thread_cache_size       = 50
table_open_cache        = 400
wait_timeout            = 28800

max_allowed_packet      = 32M
max_heap_table_size     = 32M
tmp_table_size          = 512K

sort_buffer_size        = 128K
join_buffer_size        = 128K
read_buffer_size        = 128K
read_rnd_buffer_size    = 128K

query_cache_size        = 32M
query_cache_limit       = 2M

group_concat_max_len    = 1024

## Master MySql 서버에서 '레코드 기반 복제' 를 사용할 때 READ-COMMITED 사용 가능
## 복제에 참여하지 않는 MySql 서버에서 READ-COMMITED 사용 가능
## 그외에는 반드시 REPEATABLE-READ 사용
transaction-isolation   = REPEATABLE-READ

## InnoDB 플러그인 옵션
innodb_use_sys_malloc       = 1
innodb_stats_on_metadata    = 1
innodb_stats_sample_pages   = 8
innodb_max_dirty_pages_pct  = 90
innodb_adaptive_hash_index  = 1
innodb_file_format          = barracuda
innodb_strict_mode          = 0
innodb_io_capacity          = 600
innodb_write_io_threads     = 4
innodb_read_io_threads      = 4
innodb_autoinc_lock_mode    = 1
innodb_adaptive_flushing    = 1
innodb_change_buffering     = inserts
innodb_old_blocks_time      = 500
ignore_builtin_innodb

## 아래 plugin-load 설정은 반드시 한 줄에 작성 
plugin-load=innodb=ha_innodb_plugin.so;innodb_trx=ha_innodb_plugin.so;innodb_locks=ha_innodb_plugin.so;innodb_lock_waits=ha_innodb_plugin.so;innodb_cmp=ha_innodb_plugin.so;innodb_cmp_reset=ha_innodb_plugin.so;innodb_cmpmem=ha_innodb_plugin.so;innodb_cmpmem_reset=ha_innodb_plugin.so

## InnoDB 기본옵션
## InnoDB 사용하지 않는다면 innodb_buffer_pool_size 를 최소화하거나 InnoDB 스토리지 엔진을 기동하지 않도록 설정 
innodb_buffer_pool_size         = 10G
innodb_additional_mem_pool_size = 16M
innodb_file_per_table           = 1
innodb_data_home_dir            = /usr/local/mysql/data
innodb_data_file_path           = ib_system:100M:autoextend
innodb_autoextend_increment     = 100
innodb_log_group_home_dir       = /usr/local/mysql/data
innodb_log_buffer_size          = 16M
innodb_log_file_size            = 1024M
innodb_log_files_in_group       = 2
innodb_support_xa               = OFF
innodb_thread_concurrency       = 0
innodb_lock_wait_timeout        = 60
innodb_flush_log_at_trx_commit  = 1
innodb_force_recovery           = 0
innodb_flush_method             = O_DIRECT
innodb_doublewrite              = 1
innodb_sync_spin_loops          = 20
innodb_table_locks              = 1
innodb_thread_sleep_delay       = 1000
innodb_max_purge_lag            = 0
innodb_commit_concurrency       = 0
innodb_concurrency_tickets      = 500

## MyISAM 옵션
## InnoDB를 사용하지 않고 MyISAM 만 사용할 경우 key_buffer_size를 4GB 까지 설정 
key_buffer_size             = 32M
bulk_insert_buffer_size     = 32M
myisam_sort_buffer_size     = 1M
myisam_max_sort_file_size   = 2G
myisam_repair_threads       = 1
myisam_recover
ft_min_word_len             = 3

## 로깅 옵션
pid_file            = /usr/local/mysql/logs/mysqld.pid
log_warnings        = 1
log-error           = /usr/local/mysql/logs/mysqld

## General 로그 사용시 아래 설정 유지, MySQL 서버 로그인한 후 "SET GLOBAL general_log = 1" 명령으로 활성화 
general_log         = 0
general_log_file    = /usr/local/mysql/logs/general_query.log

log_slow_admin_statements
slow-query-log      = 1
long_query_time     = 1
slow_query_log_file = /usr/local/mysql/logs/slow_query.log

## 복제 옵션
## 만약 현재 서버가 마스터 MySQL 인 경우 아래 주석 해제 
# log-bin           = /usr/local/mysql/logs/binary_log
# binlog_cache_size = 128K
# max_binlog_size   = 512M
# expire_logs_days  = 14
# log-bin-trust-function-creates = 1
# sync_binlog       = 1

## 현재 서버가 슬레이브 MySQL 인경우 아래 주석 해제 
# relay-log         = /usr/local/mysql/logs/relay_log
# relay_log_purge   = TRUE
# read_only

## 현재 서버가 슬레이브이면서 마스터 MySQL 인 경우 
## 현재 MySQL 서버로 복제되는 쿼리를 바이너리 로그에 저장하려면 아래 주석 해제 
# log-slave-updates
```

##### server-id
- mysql 이 내부적으로 식별하는 값 
- 1보다 큰 정수 값 설정, 값이 중요하지 않고 복제그룹 내에서 유일한 값이면 됨 

##### user
- MySQL 이 설치된 서버의 운영체제 계정 
- MySQL 서버는 입력된 계정으로 인스턴스 실행, 명시되지 않은경우 MySQL 서버를 실행한 운영체제 계정으로 인스턴스 실행 
- MySQL 서버가 원격으로 해킹당하는 피해를 줄이기 위해 운영체제 관리자 계정으로 실행하지 않는 것이 좋음

##### basedir
- mysql 서버의 홈 디렉토리 
- 절대 경로가 사용되지 않는 한 여기 설정 된 값이 기본 디렉터리가 됨

##### datadir
- MyISAM의 데이터 파일이 저장되는 디렉터리 
- InnoDB 이외의 별도 데이터 디렉터리를 지정하지 않는 스토리지 엔진 (CSV engine, Archive Storage engine ..) 은 이 디렉터리가 데이터 디렉터리가 됨
- InnoDB는 별도의 파리미터를 통해 데이터 파일의 경로를 지정함 

##### tmpdir
- MySQL 서버는 정렬이나 그룹핑과 같은 처리를 위해 내부적으로 임시 테이블 생성
- tmpdir은 내부적으로 임시 테이블 데이터 파일이 저장되는 위치
- 이 디렉터리에 생성되는 데이터 파일은 쿼리가 종료되면 자도응로 삭제됨 
- 여기의 임시 테이블은 사용자가 쿼리로 생성하는 테이블과 성격이 다름 

##### character-set-server, collation-server
- MySQL 서버의 기본 문자집합 설정
- 별도로 DB나 테이블, 컬럼에서 사용할 문자집합을 재정의 하지 않는 경우 계속 사용됨 
- 기본적으로 MySQL에서 문자열에서 대소문가 구분 안함 
    - 대소문자 구분 필요시 collation-server에서 utf8_bin 같은 콜레이션 설정 

##### default-storage-engine
- 기본적으로 사용할 스토리지 엔진 정의
- 사용자가 테이블 생성시 어떤 스토리지 엔진을 사용할지 정의하지 않는 경우 이 값을 사용함 
- 이 값으로 MySQL 서버가 내부적으로 생성하는 임시 테이블의 스토리지 엔진을 결정하지 못함 
    - 내부적으로 사용하는 임시 테이블은 MyISAM만 사용함 

##### skip-name-resolve
- 클라이언트가 MySQL 서버에 접속하면 MySQL 서버는 해당 클라이언트가 접속이 허용된 사용자인지 확인하기 위해 클라이언트의 ip를 이용해 역으로 DNS이름을 가져와야 하나 비용이 많은 작업 
- 이 옵션 지정시, Reverse lookup을 하지 않음 
- 접속 가능한 사용자 명시할때 도메인,호스트 네임을 사용할 수 없음 
- 성능을 이유로 Reverse Lookup은 비활성화 사용하는게 일반적 

##### event-scheduler
- MySQL5.1부터 크론같은 스케쥴러 사용 가능
- 이벤트 스케쥴러는 MySQL 내에서 실행되는 별도의 스레드를 필요로 함 
- 이벤트 스케쥴러를 사용할 일이 없는 경우 OFF로 설정 

##### sysdate-is-now
- MySQL에서 현재 시간을 가져오는 함수는 sysdate(), now() 있으나 동작 방식은 크 차이 있음 (p397)
- 이 설정은 사용자의 실수를 없애고자, sysdate()함수가 now() 함수와 동일하게 작동하도록 만드는 것
- sysdate()의 독특한 작동방식이 꼭 필요하지 않거나 모를경우 이 설정 꼭 필요 


##### back_log
- 수 많은 클라이언트가 한꺼번에 MySQL 서버로 접속 시도시 MySQL 서버의 인증을 거칠때 까지 기다리게 되는데, 이때 대기 큐에 담아 둘수 있는 커넥션 수 
- 여러 클라이언트 동시연결 숫자를 늘릴수도 있으나, 무한정 늘릴 수는 없음 

##### max_connections
- MySQL 서버가 최대한 허용할 수 있는 클라이언트의 연결 수를 제한하는 설정 
- 많은 사용자는 이 값을 몇 천으로 설정하지만, 그 정도 커넥션까지 필요한지 고민 필요 
    - 정상적인 경우라면 문제 없겠으나
    - 한 두개의 무거운 쿼리가 자원을 모두 써버리거나 일시적으로 많은 사용자가 쿼리를 실행할때 MySQL 서버의 응답은 자연히 느려짐, 그러면, 어플리케이션 서버는 사용자 요청을 처리하기 위해 더 많은 커넥션 생성하며 결국 작업불능, max_connections 수가 높을 수록 서버의 작업불능 확률이 높아짐 
- 웹서버의 경우 데이터를 관리하지 않으므로 재시도해도 문제 없으나, db의 경우 사용자의 데이터를 날릴 수도 있음 
- 최근 웹서버는 커넥션풀을 사용하지만, 터무니 없는 많은 커넥션을 보유하는 경우도 있음

##### thread_cache_size
- MySQL 서버에서 스레드와 커넥션은 거의 같은 의미로 사용되나, 사실 커넥션은 클라이언트와 서버와의 연결 그 자체를 의미하고, 스레드는 해당 커넥션으로 오는 작업 요청을 처리하는 주체
- 최초 클라이언트로부터 접속오청이 오면 MySQL 서버는 스레드를 준비해 그 커넥션에 작업요청을 처리할 스레드를 맵핑하는 형태
- 아무리 스레드가 경량이더라도 생성, CPU처리시간이 있으므로 스레드 재사용함, (스레드품)
- 최대 몇개까지 스레드를 스레드풀에 보관할지 결정 
- 어플리케이션 서버에성 커넥션 풀을 사용하지 않는다면 이 값을 크게 유지하는게 좋으나 MySQL 서버가 스레드를 캐시하려면 메모리가 필요하므로 이값을 크게 늘리기보다, 클라이언트 프로그램이 커넥션 풀을 사용할 수 있게 하는게 좋음 


##### wait_timeout
- MySQL 서버에 연결된 클라이언트가 wait_timeout에 지정된 시간동안 요청이 없는 경우 서버는 해당 커넥션을 강제로 종료
- 이 설정값의 단위는 초, 28800초 (8시간)가 기본값 
- MySQL 서버와 웹서버 사이에는 수 많은 네트워크를 거치므로 커넥션이 예상보다 짧은 시간 내에 종료되는 문제는 사실 MySQL서버 자체보다 각 서버의 운영체제에 설정된 idle-timeout 등의 문제인 경우가 많음 
- MySQL 서버 앞쪽에 L4 와 같은 로드밸런서 장비가 있는 경우 그 장비의 idle-timeout도 반드시 확인 필요 
- 네트워크 불안정, 예상외의 timeout시간의 경우 운영체제의 keep-alive 설정도 함께 확인 필요

- Keep-Alive
    - 네트워크를 통해 만들어진 TCP 커넥션을 계속 유효한 상태로 유지하는 것의 의미 
    - MySQL 서버의 커넥션도 TCP 기반으로 운영체제의 KeepAlive 관리 대상이 됨
    - KeepAlive Probe 과정을 통해 유효여부 확인
        - 몇번 이상 실패시 커넥션 종료등 판단
    - MySQL 서버가 L4와 함께 사용되는 경우 L4에 설정된 타임아웃 시간보다 MySQL 서버가 설치된 운영체제의 KeepAlive Probe 간격을 더 낮게 설정하는게 좋음 

##### max_allowed_packet
- 네트워크 문제나 MySQL 서버/클라의 버그로 인해 잘못된 패킷이 전달되는 경우를 막기 위해 최대 패킷 크기 이하일거라고 가정하고 처리
- 만약 이 패킷보다 클경우, 설정값 변경이 필요하며 TEXT나 Blob에서 발생할 수 있음 
- 클라이언트가 MySQL 서버로 보내는 쿼리 요청은 무조건 하나의 패킷만 사용할 수 있고, 결과는 여러개의 패킷으로 나눠서 받게 됨 

##### max_heap_table_size
- 메모리 스토리지 엔진을 사용한 메모리 테이블은 heap 테이블이라고 도 함 
- 만약 빠른 처리를 위해 메모리 테이블을 사용한다면 값을 그에 맞게 적절히 변경하는게 좋음 
    - 사용자가 지정한 테이블 생성 뿐만 아니라 MySQL이 사용자의 쿼리를 사용하기 위해 내부적으로 생성하는 임시테이블도 포함 

##### tmp_table_size
- 임시테이블의 최대 크기를 제어하는 설정 값
- MySQL에서 임시테이블은 저장매체의 종류에 따라 크게 디스크에 저장되는 임시테이블과 메모리에 저장되는 임시테이블로 구분 가능
- tmp_table_size는 메모리에 생성되는 임시 테이블만 제어
- 사용자가 생성한 table (```sql CREATE TABLE ... ENGINE=MEMORY```) 의 경우 임시테이블이 아니므로 이 설정과 무관

##### sort_buffer_size
- mysql에서 인덱스를 이용하거나 별도의 메모리나 디스크 공간에 결과를 저장해 정렬을 수행할 수 있음
    - 인덱스를 이용하는 경우 정렬된 상태로 저장된 인덱스를 순서대로 읽는 것을 의미, 실제 정렬 알고리즘 수행되지 않으므로 빠르게 처리됨 
    - 정렬을 목적으로 인덱스를 사용할 수 없는 경우 정렬 대상 데이터를 메모리/디스크버퍼에 저장해 정렬 알고리즘 (일반적 퀵 소트) 을 통해 데이터를 정렬
    - 데이터가 많은경우 오랜시간 소요됨 
- 일반적인 DBMS에서 가장 큰 부하를 일으키는 사용자 요청이 정렬(정렬, 그룹핑) 작업, sort_buffer_size는 인덱스를 사용할 수 없는 정렬에 메모리 공간을 얼마나 할당할지 결정하는 설정 값
- 이 값이 크다고 해서 절대 빨라지지 않음 
    - 적절한 값은 64K ~ 512K 
    - 2M 이상 설정시 더 느려질 수도 있음 
- 이 크기가 작아지면 디스크를 사용할 확률이 높아짐, 커질 수록 각 클라이언트 스레드가 사용하는 메모리 양이 커져서 메모리 낭비가 심해짐 
- DBMS 최대 병목지점이 디스크므로 메모리 낭비더라도 좀 크게 설정할 수 있으나 MySQL 서버는 정렬, 임시테이블 생성이 필요 이상 메모리를 할당해서 메모리 낭비가 발생할 때가 많으므로 권장값을 사용이 권장됨

##### join_buffer_size
- MySQL에서 조인이 발생할 때마다 사용되는 버퍼가 아님
- 적절한 조인 조건이 없어서 드리븐 테이블의 검색이 풀 테이블 스캔으로 유도되는 경우에 조인 버퍼가 사용됨 
- 일반적으로 이런 경우 "Using join buffer" 라는 내용이 실행 계획에 표시됨
- 실제 서비스용 쿼리에서 이러한 풀 테이블 스캔은 거의 발생하지 않으므로 정상적으로 튜닝된 시스템에서 이 버퍼가 거의 사용되지 않는다고 봄 
- 특별한 요건이 없는 경우 128K ~ 512K 정도 적용 

##### read_buffer_size
- MySQL 매뉴얼에는 풀 테이블 스캔이 발생하는 경우 사용하는 버퍼라고 표시돼 있지만 많은 스토리지 엔진에서 다른 용도로 사용하기도 함, 정체를 알기 어려움
- 128K 일떄 성능이 젤 좋았음.. (16K~32M)

##### read_rnd_buffer_size
- MySQL에서 인덱스를 사용해 정렬할 수 없는 경우 정렬 대상 크기에 따라 Single-pass / Two-Pass 알고리즘 중 하나를 사용
- 정렬 데이터가 큰 경우 two-pass 를 사용하며, 정렬기준칼럼 값만 가지고 정렬을 수행하며 정렬 완료 후 한번더 데이터를 읽어야 함 
- 정렬 순서대로 데이터를 읽을때 동일 데이터 페이지(블럭) 에 있는 것들을 모아서 읽으면 더 빠르게 데이터를 가져올 수 있고, 읽어야 할 데이터 레코드를 버퍼링할때 필요한게 read_rnd_buffer고 이 크기를 결정하는 값 
- 64K ~ 128K 수준이 좋음 

##### sort_buffer_size, join_buffer_size, read_buffer_size, read_rnd_buffer_size
- 세션 범위의 변수므로 MySQL서버는 커넥션(세션) 별로 설정된 크기의 메모리 공간을 각각 할당하게 됨 
- 평균 커넥션이 500개일 경우 4개 버퍼를 각각 2M 씩 할당 했다면, 커넥션당 최대 8M, 최악의 경우 4G 사용함
- 그러므로 세션 단위로 할당되는 메모리 버퍼는 절대 불필요하게 크게 설정하지 않는 것이 좋음 
- 특정 커넥션만 대량의 레코드를 배치 형태로 처리해야한다면, 해당 세션에만 이 버퍼 값을 변경하는 방법으로 해결할 수 있음 

##### query_cache_size, query_cache_limit
- 무조건 큰게 좋은게 아님, 메모리가 아주 많더라도 128M 이상 설정하지 않는게 좋음 
- 데이터가 절대 변경되지 않고, 읽기 전용으로만 사용될 경우 128M에서 조금씩 더 크게 늘리면서 성능을 확인한후 확장하는게 좋음 
- 메모리가 충분하지 않거나, 테이블의 데이터 변경이 빈번하다면 64M 이상으로 설정하지 않는게 좋음


##### group_concat_max_len
- MySQL에서 GROUP_CONCAT() 함수로 Group BY 된 레코드의 특정 값이나 표현식을 구분자로 연결해 가져오는 것 가능.
- 이때 설정값으로 지정된 크기 이상 연결하는 것은 불가능
- 하지만 GROUP_CONCAT() 동작 중 버퍼가 부족한 경우 경고가 발생
    - JDBC 드라이버 사용시 SQLException이 발생해 에러로 처리됨 

##### transaction-isolation
- 트랜젝션 격리수준을 결정하는 설정값, 기본값은 REPEATABLE-READ
- 사용 가능한 값
    - READ-UNCOMMITED, READ-COMMITED, RELEATABLE-READ, SERIALIZABLE

##### plugin-load
- MySQL 5.0 까지는 모든 스토리지 엔진이 MySQL 서버와 같이 컴파일 됐어야 했음. 결과적으로 MySQL 버전에 종속적일 수 밖에 없었음
- MySQL 5.1 부터 스토리지 엔진에 플러그인 개념이 도입되어 MySQL 서버 버전과 플러그인 엔진 버전을 따로 선택할 수 있게 됨 
- 이 옵션은 MySQL 서버가 기동되면서 어떠한 스토리지 엔진을 로드할지 결정하는 옵션, 모두 한줄에 작성되어야 함 
- ``` plugin-load=플러그인 이름1=플러그인라이브러리;플러그인 이름2=플러그인라이브러리;...```
- MySQL 5.5 버전부터 오라클 인수 후, InnoDB 가 디폴트 스토리지 엔진으로 채택됨 , 별도 옵션지정 안해도 됨 

##### ignore_buildtin_innodb (MySQL 5.1에서만 사용 가능)
- 빌트인 버전의 innodb 스토리지 엔진을 무시하는 것 

##### innodb_buffer_pool_size
- InnoDB 스토리지 엔진에서 가장 중요한 옵션
- 디스크의 데이터를 메모리에 캐싱함과 동시에 데이터의 변경을 버퍼링하는 역할 수행
- 일반적으로 innodb_buffer_pool_size 는 운영체제나 MySQL 클라이언트에 대한 서버 스레드가 사용할 메모리를 제외하고 남는 거의 모든 메모리 공간을 설정 
- 세션단위 메모리 최대 가용량을 예측할 수 있느나 현실성이 떨어지므로, 실제 사용하는 메모리는 쿼리의 특성에 따라 달리잠. 그래서 일반적으로 50%~80% 까지 수준에서 설정 
- 낮은 값으로 설정해 점차 늘려가는게 좋음 
- innodb_buffer_pool은 정적 변수라 MySQL 서버 재시작 필요 


##### innodb_additional_mem_pool_size
- MySQL 자체적으로 각 테이블 메타 정보를 메모리에 관리하나, InnoDB 스토리지 엔진도 자체적으로 각 테이블의 메타 정보나 통계 정보를 내부적으로 가지고 있음. 
- 이 옵션은 메타정보/통계정보가 저장되는 공간 크기를 결정하는 옵션
- 테이블 수가 1000개 미만시, 16M, 그 이상시 32M 면 충분


##### innodb_file_per_table
- innoDB 스토리지 엔진을 사용하는 테이블은 ```*.ibd``` 확장자로 생성
- 오라클과 같은 테이블 스페이스 개념을 사용 
    - 오라클처럼 모든 테이블을 하나의 테이블 스페이스에 모아서 생성도 가능
    - 테이블 단위로 테이블 스페이스를 할당하는 방법도 가능 
- 이 변수를 1로 설정시, 데이터 파일과 테이블 스페이스를 생성해 데이터를 저장 
    - 0으로 설정시, 하나의 테이블 스페이스 (시스템 테이블 스페이스) 에 모든 테이블의 데이터가 저장 
- 하나의 테이블 스페이스에 모든 테이블이 저장되는 경우 
    - 테이블이 삭제 (drop, truncate) 되어도 테이블 스페이스가 점유하던 공간을 운영체제로 반납하지 않음 
    - 테이블 스페이스가 할당된 경우 공간이 다시 반납되므로 일반적으로 설정을 1로 하여 사용 
- 운영시 이 값이 자주 변경되면 테이블마다 저장된 테이블 스페이스가 달라져서 가능하면 변경하지 않고 사용 권장 

##### innodb_data_home_dir
- innoDB 스토리지 엔진을 사용하는 테이블에 대한 데이터 파일이 저장할 위치를 설정 
- 일반적으로 MySQL의 기본 저장 디텍터리와 동일하게 설정 

##### innodb_data_file_path
- InnoDB의 데이터 개념
    - 시스템 데이터, 사용자 데이터로 나뉨 
    - 시스템 데이터
        - 사용자가 생성한 테이블에 대한 메타 정보, 트랜젝션을 위한 undo와 같이 InnoDB 스토리지 엔진이 임의적으로 만들어 낸것 
        - 시스템 데이터는 항상 시스템 테이블 스페이스에 저장
        - 시스템 테이블 스페이스는 innodb_data_file_path에 명시된 파일에 생성
    - 사용자 데이터
        - innodb_file_per_table 옵션에 따라 좀 다름
        - innodb_file_per_table 옵션이 1로 설정되면 이 값이 관계없이 테이블별로 별도의 파일을 사용
        - 0으로 설정시 모든 테이블의 데이터가 이 옵션에 설정된 파일에 저장 
- 포멧
    - 파일의 이름:초기파일 크기:자동확장 여부
    - ```innodb_data_file_path=ib_system:100M:authextend```
- 시스템 테이블 스페이스는 오랜 시간 처리되는 트랜젝션이 많은 경우 10GB까지 확정되는 경우도 있음 
    - 한번 확장된 InnoDB 시스템 테이블 스페이스는 다시 줄어들지 않음 

##### innodb_log_group_home_dir
- InnoDB 같이 트랜젝션을 지원하는 RDBMS는 ACID 보장과 동시에 성능향성을 목적으로 데이터 변경이력을 별도의 파일에 순차적 기록
    - 트랜잭션 로그, Redo 로그
    - 이 로그는 사람이 읽을 수 있는 로그가 아닌, MySQL이 갑자기 종료됐거나, 종료되지 않은 트랜잭션을 복구하기 위한 용도로 사용

##### innodb_log_buffer_size
- InnoDB 스토리지 엔진에서 데이터가 변경될 때 해당 변경사항을 바로 리두로그에 기록하면 디스크 입출력 요청이 빈번하여 비효율적
- 메모리에 일시적 로그를 버퍼링하며, 이때 사용할 버퍼의 크기를 설정 
- 세션단위가 아니므로 MySQL 서버에서 하나의 메모리 버퍼만 생성됨
- 일반적으로 16~32MB 

##### innodb_log_file_size, innodb_log_files_in_group
- InnoDB의 리두 로그파일은 1개 이상 파일로 구성되며 순환하면서 사용
- innodb_log_file_size는 파일 하나의 크기 
- innodb_log_files_in_group은 파일을 몇 개 사용할지 설정 값 
- InnoDB 스토리지 엔진은 물리적인 파일을 논리적으로 각 파일의 시작과 끝에 연결해 circular queue와 같이 사용
- 값이 너무 작으면 설정된 버버풀의 공간을 제대로 사용하지 못할 수 있음
    - 버퍼풀이 10G 이상시: 리두 파일 개수와 무관하게 전체공간이 2G~4G로 
    - 이하시: 리두 파일 전체 크기를 2G 이하로 설정 
- 데이터 변경 쿼리가 매우 빈번하게 호출시 전체 로그 크기를 4G로 늘려서 설정하는게 좋음
    - 5.1까지 최대 4G, 5.6부터는 4G이상 가능 

##### innodb_lock_wait_timeout (초 단위)
- 잠금 획득을 위해 최대 대기할 수 있는 시간 (초)
- 잠금 획득 실패시 "Lock wait timeout exceed" 오류 발생시키고, 쿼리 실패
    - 실패하더라도 트랜잭션 자체가 롤백되지 않음
    - 이 에러는 InnoDB 레코드 잠금끼리만 발생하며 테이블에 대한 쿼리가 테이블 잠금을 기다리는 경우엔 타임아웃이 적용되지 않음 
- 일반적으로 50초, (5.0 에서는 재시작 필요, 5.1이상에서는 SET으로 런타임 설정 가능)

##### innodb_flush_log_at_trx_commit
- InnoDB에서 트랜잭션이 커밋될 떄마다 리두 로그를 디스크에 플러시할지 결정하는 옵션 
    - fsync(), fdatasync()
- DBMS의 병목은 디스크고, 디스크 병목의 주 원인은 fsync(), fdatasync() 와 같은 시스템콜
    - 중요한 데이터의 경우 반드시 디스크 데이터 동기화가 필요하나, 많은 양의 데이터 처리시 손실돼도 무방하다면, 0으로 설정해 입출력 성능을 크게 향상시킬 수 있음 
    - DBMS가 직접 데이터 동기화 하지 않는 경우 운영체제에서 적절한 시점에 동기화 호출하며, 대략 4~5초 정도됨 

##### innodb_flush_method
- 어떤 운영체제에서든 디스크에 데이터를 쓰는 작업은 크게 아래 2단계를 가짐 
    - 운영체제의 버퍼로 기록
    - 버퍼의 내용을 디스크로 복사
- 이 두 단계 작업을 어떻게 조합하느냐에 따라 3가지 관점으로 쓰기 방식을 나눠 볼 수 있음 
    - 실행방식에 따라
        - Sync IO - 두 단계의 작업을 동시에 같이 실행하는 방식
        - Async IO - 두 단계의 작업을 각기 다른 시점에 실행
    - 데이터가 변경시 그와 동시에 파일의 변경일시 같은 메타 데이터도 함께 변경
        - fsync - 데이터, 메타데이터 함꼐 변경
        - fdatasync - 메타 정보 무시하고, 순수하게 사용자 데이터만 변경
    - 1단계 작업을 무시 
        - Direct IO - 운영체제 버퍼 기록 단계 생략하고, 사용자 데이터를 디스크로 쓰는 경우
        - innodb_flush_method 는 InnoDB 스토리지 엔진이 로그 파일과 로그 데이터 파일을 어떤 방식으로 디스크에 기록, 동기화 할지 결정
- 설정 값
    - 유닉스 계열 운영체제
        - O_DSYNC, O_DIRECT, fdatasync
        - 5.1.23 이하: 3값 모두 설정
        - 5.1.24 이상: O_DSYNC, O_DIRECT만 설정하며, 값이 없는 경우 fdatasync
    - 윈도우 계열     
        - async_unbuffered로 고정
- 값 특징
    - O_DSYNC
        - 사용자 데이터, 메티데이터를 Sync IO로 기록
    - O_DIRECT
        - 운영체제 버퍼기록을 무시한 O_DSYNC
    - fdatasync
        - O_DSYNC로 설정때와 동일하나, 파일의 메타데이터를 변경하지 않는 형태 
- InnoDB는 OS캐시보다 더 효율적인 InnoDB 버퍼풀이 있으므로, O_DIRECT 가 좋음
    - 하지만, RAID컨트롤러 (캐시 메모리가 장착된)가 없거나 데이터 스토리지로 SAN을 사용할때 O_DIRECT 사용하지 않는게 좋음 

##### innodb_old_blocks_time
- 5.0 의 InnoDB엔진에서 관리하는 버퍼풀의 모든 페이지는 하나의 리스트로 관리 
- 버퍼풀 리스트의 앞부터 5/8 영역까지 young 혹은 new 영역, 뒤쪽을 old 영역이라 함 
- 앞쪽은 MRU, 뒤쪽은 LRU 로 구성됨 
    - 이렇게 나눈 이유는 풀 테이블 스캔이나 인덱스 풀 스캔이 실행시 아주 빈번히 사용되는 중요한 페이지가 버퍼 풀에서 제거되지 않게 하기 위해서 
    - 먼저 LRU에 저장되고 포그라운드 스레드에 의해 페이지가 사용되면 MRU로 옮겨짐 
- 풀 테이블 스캔이나, 인덱스 풀 스캔과 같이 대량 데이터 페이지를 읽고 처리하는 쿼리에서 순간적으로 그 페이지 필요하고 그 이후에는 사용되지 않는게 일반적.
    - 이 경우 LRU등록 후 즉시 MRU로 이동 LRU리스트에서 MRU리스트로 옯겨지기전에 대기할 시간 (밀리초)동안 대기 

##### key_buffer_size
- InnoDB에서 가장 중요한 설정값이 innodb_buffer_pool_size라면, MyISAM에서는 key_buffer_size 
- InnoDB의 버퍼 인덱스와 모든 데이터 페이지에 대해 캐시와 버퍼 역할을 동시에 수행
- MyISAM의 키 버퍼는 주로 인덱스에 대해서만 캐시 
    - 그러므로 InnoDB 처럼 크게 설정하면 안됨, 전체 메모리의 30~50% 수준, 나머지는 운영체제 캐시를 위해 남겨둬야함 

##### general_log, general_log_file
- MySQL에는 DBMS 서버에서 실행되는 모든 쿼리를 로그 파일로 기록하는 기능
    - 쿼리로그 혹은 제너럴 로그 
    - 5.1 부터 동적변수로 됨 
- 서버가 많은 쿼리를 수행한다면 로그 기록을 위해 많은 자원을 소모할 수 있으므로 되도록 사용하지 않는게 좋음 

##### slow-query-log, long_query_time, slow_query_log_file
- 지정된 시간 이상 쿼리가 실행되는 경우 저장되는 로그 파일
- 글로벌 동적 변수

##### log_slow_admin_statements
- DDL 문장의 슬로우 쿼리 로그 기록 여부를 설정 

##### log-bin, max_binlog_size, expire_logs_days
- MySQL에서 복제를 구축하기 이ㅜ해 반드시 바이너리 로그 사용 
- 바이너리 로그는 마스터서버만 기록, 슬레이브는 마스터에서 기록된 바이너리로그를 가져와서 재실행하는 형태로 데이터 동기화 
- log-bin: 로그파일의 prefix
- max_binlog_size: 최대 로그파일 크기 제한
- expire_log_days: 만료 로그를 보관날짜 설정 

##### binlog_cache_size
- 바이너리 로그의 버퍼링 사이즈
- 대용량 칼럼이 많지 않은 경우 56 ~ 256KB 정도로 설정 

##### log-bin_trust-function_creators
- 바이너리 로그가 활성화된 MySQL에서 스토어드 함수를 생성하는 경우, "바이너리 로그로 인한 복제가 안전하지 않다"는 에러메시지를 보내고 함수 생성 실패할 수 있는데, 이를 무시하는 설정 

##### sync_binlog
- MySQL에서 성능문제가 발생하는 두 요소 (리두 로그 동기화, 바이너리 로그 동기화)
- 특히 바이너리 로그 동기화는 상당한 부하를 만들어 냄
- 읽기 부하: 인덱스, 데이터 파일 
- 쓰기 부하: 바이너리 로그, 리두 로그 
- 디스크 입출력 대역폭은 7:3 ~ 8:2
- MySQL 복제로 구성되더라도 데이터 변경은 항상 마스터만 이뤄지므로 마스터의 쓰기 부하를 분산할 수 있는 방법이 없음 
- 이 옵션을 1로 설정할 경우 트랜잭션이 커밋될때마다 바이너리 로그를 디스크에 플러시 함 
- 0으로 설정시 디스크에 기록은 하나 MySQL서버가 플러시하지 않기 떄문에 운영체제 버퍼만 기록하고 즉시 처리 완료 
- 이 값의 여부는 복제가 얼마나 중요한 역할을 하느냐에 따라 달라짐 

##### relay-log, relay_log_perge
- 마스터에서 바이너리 로그를 생성하면 슬레이브에서는 마스터의 바이너리 로그를 읽어 릴레이 로그라는 파일을 생성 
- 슬레이브 MySQL에서 ```CHANGE MASTER``` 명령 실행시 슬레이브는 기본 경로에 릴레이 로그를 생성 
- relay-log: 릴레이 로그의 파일 위치 
- relay_log_purge: 옵션을 TRUE, 1로 설정시 필요하지 않은 오래된 릴레이 로그를 자동 삭제 

##### log-slave-updates
- MySQL 서버 복제 구성에서 하나의 MySQL 서버가 슬레이브면서 동시에 마스터가 될 수 있음 
- 이때 다른 마스터로부터 바이너리 로그를 가져와 재실행되는 쿼리가 자기 자신의 바이너리 로그에 기록되게 할지 여부를 결정하는 옵션 

##### read-only
- 복제구성시 데이터 변경은 마스터가, 단순 조회는 슬레이브가 담당하게 됨
- 이때 슬레이브의 데이터가 마스터와 관계없이 변경되면 충돌될 수 있으므로 읽기전용으로 만드는게 일반적 
- 동적변수 

#### client 설정 그룹 
- mysql이나, mysqldump 등과 같은 MySQL 클라이언트 프로그램이 공통적으로 사용하는 옵션 
- 서버 접속을 위한 소켓 파일 경로나, 서버의 포트정보를 공통설정 가능

```sql
[client]
socket      = /usr/local/mysql/tmp/mysql.sock
port        = 13306
```

#### mysql 설정 그룹 
```sql
[mysql]
default-character-set   = utf8
no-auto-rehash
show-warnings
prompt  = \u@h:\d\_\R:\m:\\s>
pager="less -n -i -F -X -E"
```

##### no-auto-rehash
- MySQL 클라이언트도 리눅스 쉘 처럼 탭키로 자동완성 기능 이용 가능
- 위 기능을 위해 클라이언트는 서버 접속시 테이블이나 칼럼의 이름을 모두 읽어와서 분석 필요
- 테이블 많은 경우 시간이 많이 소요됨 

##### show-warnings
- SQL 명령 실행시 에러는 없지만 경고가 있는 경우 항상 ```show warnings;``` 명령을 통해 어떤 경고가 있었는지 확인 하는 작업 필요
- show-warings 활성화시 경고 발생시 경고 메시지 자동 출력 

##### prompt
- sql clinet의 프롬프르 포멧 

##### pager
- mysql 클라이언트로 접속시 전송된 결과를 쉽게 페이지 

#### 메모리 크기에 따른 mysql서버 설정 변경 
- 서버는 특히 메모리 설정이 중요 
- 스토리지 엔진별로 주요 메모리 공간이 공유되지 않으므로 스트리지 엔진에 맞게 메모리 사용을 제한하는게 중요 


### MySql 서버 시작/종료 

#### 시작과 종료
- rpm 패키지로 설치시 자동으로 /etc/init.d/mysql 스크립트 파일이 생성됨 
- 소스파일을 직접 컴파일해서 설치한 경우 MySql 시작 종료를 위한 시작 스크립트를 /etc/init.d/ 로 복사 필요 
    - 스크립트 파일은 MySQL 설치 디렉터리에서 share/mysql/mysql.server 라는 파일로 저장되어 있음 
```
$ cd ${MYSQL_HOME}
$ cp share/mysql/mysql.server /etc/init.d/mysql
$ chkconfig mysql on

$ service mysql start
$ service mysql stop
```

- mysql 프로세스
    - mysqld: 실제 서비스하는 MySql 데몬 프로세스
    - mysqld_safe: mysqld가 비정상 종료되었을떄 다시 기동하는 역할 (watcher, angel process)

```
# mysqld_safe를 사용한 실행 
$ /usr/local/mysql/bin/mysqld_safe  --defaults-file=/etc/my.cnf&
```

- innodb의 경우 트랜잭션이 정상적으로 커밋되어도 데이터 파일에 기록되지 않을 수 있음
    - 데이터파일엔 없고, 리두로그엔 있을 수 있음
    - 정상적인 상황이며 (용량이 클수록)
    - 만약 서버 종료시 innodb 엔진이 모든 커밋내용을 데이터 파일에 기록되길 원하면 아래 옵션 설정 필요
        - Clean shutdonw ``` SET GLOBAL innodb_fast_shutdown=0; ```
        - 클린 셧다운의 경우 서버 기동시 별도 트랜잭션 복구과정이 없으므로 빠르게 시작, 리두 파일 없어도 정상 기동 

#### 서버 연결 테스트 
```
$ mysql -uroot -p --host=localhost --socket=/tmp/mysql.sock
$ mysql -uroot -p --host=127.0.0.1 --port=3306
$ mysql -uroot -p
```
- 원격접속을 위헤서 두번째 방법 사용
- mysql 접속시 host별 차이
    - localhost: 항상 소켓 파일을 통해 데이터를 주고 받음 (Unix domain socket, IPC)
    - 127.0.0.1: loopback ip를 사용해 tcp/ip 통신
    - 명시하지 않을 경우, localhost가 되고 소켓파일을 사용함 (위치는 mysql설정파일에서 참고)
- 기본 db
    - information_schema
        - mysql서버에 존재하는 오브젝트의 정보(테이블, 칼럼, 스토어드 프로시져)를 담고 있는 메타정보 테이블이 저장된 db
        - 모두 MyISAM 스토리지엔진을 사용 
        - 디스크상에 존재하지 않음 
    - mysql
        - 접속 가능한 호스트, 계정정보, 각정 계정별 권한이 저장됨 
    - test
        - 불필요
        - 누구들 볼수 있으므로 운영에서는 삭제권장 

### MySql 복제 구축 
- 복제하고자하는 MySql 서버 준비 
    - 마스터는 host_master, 슬레이브는 host_slave 포트는 둘다 3306 이라고 가정 

#### 설정 준비
- 서로 복제그룹내 중복되지 않는 server-id 필요 
- 마스터 mysql 서버는 반드시 바이너리 로그가 활성화 되어 있어야 함 
- 위 사항이 적용된 상태에서 서버 시작 

##### master
```
[mysqld]
server-id   = 101
log-big     = binary_log
sync_binlog = 1
binlog_cache_size   = 5M
max_binlog_size     = 512M
expire_logs_days    = 14
log-bin-trust-function-creators = 1
```
- 동기화 방법을 지정하기 위해 sync_binlog 설정 사용시 파일크기, 캐시메모리 크기 지정 가능
- log-bin-trust-function-creators 는 마스터 MySql 에서 스토어드 함수나 트리거를 생성시 발생하는 경고메시지 제거 

##### slave
```
[mysqld]
server-id   = 102
relay-log   = relay_log
relay_log_purge = TRUE
read_only
```
- 중복되지 않는 server-id 설정
- show master status; 명령으로 로그가 정상적으로 기록되고 있는지 확인 


#### 복제 계정 준비 
- 슬레이브가 mysql 마스터로 부터 바이너리 로그를 가져오기 위한 계정 (복제용 계정)
    - replication slave 권한이 필요 

```sql
> create user 'repl_user'@'%' identified by 'slavepass';
> grant replication slave on *.* to 'repl_user'@'%';
```


#### 데이터 복사
- 마스터 데이터를 슬레이브 서버로 가져와서 적재
    - enterprize backup, mysqldump 를 사용해 복사 
    - 일반적으로 데이터가 크지 않을 경우 mysqldump를 사용 
    - 명령은 한줄로, ``` --single-transaction --master-data=2 ``` 옵션 필수 추가
        - single-transaction
            - 테이블이나 레코드 잠금을 걸지 않고 InnoDB 테이블 백업할 수 있게 함
        - master-data=2 
            - 바이너리 로그의 정보를 백업 파일에 같이 기록해줌 (필수)
            - 하지만 이 명령은 ```flush tables with read lock``` 명령을 사용하여 글로벌 리드락을 걸게함

```
$ mysqldump -uroot -p --opt --single-transaction --hex-blob --master-data=2 --routines --triggers --all-databases > master_data.sql

> source /tmp/master_data>sql
```


#### 복제 시작 
```
$ less /tmp/master_data.sql

> change master to master_log_file='binary_log.000007', master_log_pos=2741, master_host='host_master', master_port=3306, master_user='repl_user', master_password='slavepass'

> show slave status
```

### 권한 관리 
- mysql의 사용자 계정은 접근한 ip에 대해서도 확인
- 권한을 묶어서 관리하는 role의 개념이 없음 (보통 모든 권한을 부여할때가 많은데 잘못된 것임)

#### 사용자의 식별 
- 계정과 사용자의 접속지점(호스트명, 도메인, ip)도 계정의 일부가 됨 
- mysql에서 계정을 언급할땐 아이디와 호스트가 같이 명시되어야 함 
- ``` 'svc_id'@'127.0.0.1' ```
    - 위 계정은 mysql 서버가 기동중인 로컬호스트에서 svc_id 라는 아이디로 접속할때만 사용될 수 있는 계정 
- ``` 'svc_id'@192.168.0.10'(passcode: 123), 'svc_id'@%'(passcode: abc) ```
    - 192.168.0.10 pc에서 접근시, mysql 서버가 어떤 계정정보를 사용할지에 따라 성공여부가 갈림 
    - mysql 서버는 범위가 가장 작은것을 먼저 선택함 
    - 중첩된 계정을 등록할 때 주의해야할 점 

### 권한 
- 여러가지 권한 (privileges) 이 존재하나 role로 관리하는 기능은 없음 
    - 모든 사용자는 하나하나의 권한을 가지고 있을 뿐
    - 권한을 미리 묶어서 템플릿처럼 만들어 두면 편리함 

#### 로컬 권한 
- alter: 테이블 스키마 변경
- alter routine: 스토어드 프로시져 변경
- create: 테이블 생성
- create routine: 스토어드 프로시저 생성
- create temporary tables: 임시 테이블 생성 
- create view: 뷰 생성
- delete: 레코드 삭제 
- drop: 테이블 삭제
- event: 이벤트 생성 및 변경
- execute: 프로시져 실행 
- index: 인덱스 생성/삭제 
- insert: 레코드 추가 
- lock tables: 테이블 잠금 (inno db 테이블의 레코드 잠금 아님)
- select: 레코드 조회 
- show view: 뷰 생성 스크립트 조회 
- trigger: 트리거 생성 /삭제 
- update: 레코드 갱신 
- grant option: 권한 부여 옵션 

#### 전역 권한
- all [privileges] : grant option 권한을 제외한 여기에 명시된 모든 권한 
- create user : 사용자 생성 
- file : mysql 서버에서 파일 접근
    - select into outfile ... 
    - load data in ...
- process : 프로세스 목록과 각 프로세스 실행 쿼리 조회 
    - show processlist
- reload : 로그 및 권한, 테이블 정보에 대한 flush 명령 사용 권한 
- replication client: show master[slave] status 명령의 사용
- replication slave: 복제를 위해 바이너리 로그를 읽어갈 수 있는 권한 (복제용 계정이 가져야 하는 권한)
- show databases: db 목록 조회 권한 
- shutdown: mysql 서버 종료 권한 
- super: 밑의 설명.
- usage: 아무런 권한이 없는 사용자도 생성 가능, 글로벌 권한을 아무것도 가지지 않는 경우. ..

#### 추가 설명
##### super
- unix root 사용자와 같은 권한을 의미하지는 않음
- 특정한 상황에서 제한을 넘어서는 뭔가 작업을 할 수 있는 권한 
    - read_only 설정하에서 mysql 서버에서 데이터 변경 가능
    - max_connections 상황에서 추가 커넥션 연결 가능 
    - trigger 생성.삭제 하기 위한 권한 
    - 특수한 권한이며, 사용하지 않는게 좋음 

##### 전역 / 로컬 권한 
- 권한의 범위
- 전역 권한: mysql 서버 전역적으로 작동하는 권한
- 로컬 권한: 기본적으로 db단위로 부여하는 권한이지만 테이블.칼럼 단위까지 부여 가능 
- mysql에서 권한을 칼럼 단위로 부여하지 않는게 성능상 좋음 

##### mysql 사용자 계정 초기화 
- root 계정은 기본적으로 모든 권한을 가지고 있는 것으로 초기화됨 
- 처음 설치된 서버는 비밀번호 없는 root계정, 아이디가 없는 계정도 있음
    - 새로운 mysql 설치후, 모든 사용자 계정을 삭제하고 다시 재설정 할 것을 권장 
    - mysql의 관리자 계정은 항상 root 로 생각할 필요는 없음 (단지 처음 초기화된 계정일 뿐 )

#### 권한 부여 
- 다른 사용자한데 권한 부여시 grant sql 문장 사용 
```sql
> grant privilege_list on db.table to 'user'@'host';
> grant privilege_list on db.table to 'user'@'host' identified by 'password' with grant option;
```
- grant 문장 실행ㅇ시 지정된 사용자가 없는 경우, 해당 사용자를 먼저 생성하고 권한 부여
    - identified by 절을 사용해 비밀번호까지 설정 가능
    - grant option 권한은 다른 권한과 다르게 grant 문장 마지막에 ```with grant option``` 으로 지정 

##### 글로벌 권한 
```sql
> grant super on *.* to 'user'@'localhost';
```
- 글로벌 권한은 db/테이블이 부여 될 수 없으므로 on 절에는 항상 *.*를 사용
- *.* -> db의 모든 오브젝트 (테이블, 스토어드 프로시저, 함수 등 )

##### db권한 
```sql
> grant event on *.* to 'user'@'localhost';
> grant event on employees.* to 'user'@'localhost';
```
- 특정 db에 대해서만 권한을 부여하거나, 모든 db에 대해 권한 부여 가능
- 특정 테이블에는 부여할 수 없음 

##### 오프젝트 권한 
```sql
> grant select,insert,update,delete on *.* to 'user'@'localhsot';
> grant select,insert,update,delete on employees.* to 'user'@'localhost';
> grant select,insert,update,delete on employees.department to 'user'@'localhost';
```
- 모든 db 및 특정 db, 특정 테이블에 대해서만 권한 부여 가능
- 테이블의 특정 칼럼에 대해서만 권한 이 부여하는 경우
```sql
> grant select,insert,update(dept_name) on employees,department to 'user'@'localhost';
```
- 여러 레벨로 권한 설정 가능하나, 칼럼에 대해서는 잘 하지 않음 
- 칼럼 단위 권한이 하나라도 설정된 경우, 모든 테이블의 모든 칼럼에 대해서도 권한 체크 하기 때문에 한 컬럼만 추가해도 모든 테이블에 영향을 미침
- 만약 칼럼단위 접근권한 이 필요한 경우, 특정 칼럼만 view를 만들어 사용 

#### mysql 관리자 계정 준비
- mysql 관리자 계정은 잘 알려져 있으므로 별도의 아이디로 관리하느넥 좋음 
```sql
grant all on *.* to 'admin'@'localhost' identified by 'adminpass123!' with grant option;
grant all on *.* to 'admin'@'127.0.0.1' identified by 'adminpass123!' with grant option;
grant all on *.* to 'admin'@'%' identified by 'adminpass123!' with grant option;

update mysql.user set grant_priv = 'Y' where user = 'root';
flush privileges;
```

#### mysql 백업용 계정 
- db에서 백업은 필수고, 관리자 계정으로 실행할 필요 없음 
- 계정의 비번은 스트립트 파일에 노출될 수 밖에 없으므로 최소권한만 준비하는게 좋음 
```sql
grant lock tables, reload, replication client, select, show databases, show view on *.* to 'backup'@'127.0.0.1' identified by 'backuppass';

grant lock tables, reload, replication client, select, show databases, show view on *.* to 'backup'@'localhost' identified by 'backuppass';
```

#### mysql 서비스용 계정 
- 서비스에 필요한 권한만 추가하는게 좋음
- 백업용 계정은 mysql 서버 로컬만 사용되나 서비스용 계정은 웹 서버에서 동작하므로 더 노출되기 쉬움 

```sql
grant file, process, reload, replication client, replication slave, show databases on *.* to 'svc_user'@'%' identified by 'svc_userpass';

grant alter, alter routine, create, create routine, create view, drop, index, show view, create temporary tables, delete, execute, insert, lock tables, select, update on 'svc_db_name'.* to 'svc_user'@'%';
```
- 첫번째 명령을 서비스용 계정 생성
    - file, process 포함, 하지만 서버 프로세스 목록 조회/복제 관련 정보를 조회하지 않을 경우 file 로 충분
    - 절대 super 권한을 넣어선 안됨 
- 두번째 명령으로 db별 권한을 수행 
    - alter, alter routine, create, create routine, create view, drop, index, show view 는 부여하지 않는게 좋음 

### 예제 데이터 적재 
- ref, study/realmysql/data

### mGroonga 설치
```
wget http://packages.groonga.org/centos/5/x86_64/Packages/groonga-2.1.2-0.x86_64.rpm
wget http://packages.groonga.org/centos/5/x86_64/Packages/groonga-debuginfo-2.1.2-0.x86_64.rpm
wget http://packages.groonga.org/centos/5/x86_64/Packages/groonga-devel-2.1.2-0.x86_64.rpm
wget http://packages.groonga.org/centos/5/x86_64/Packages/groonga-doc-2.1.2-0.x86_64.rpm
wget http://packages.groonga.org/centos/5/x86_64/Packages/groonga-httpd-2.1.2-0.x86_64.rpm
wget http://packages.groonga.org/centos/5/x86_64/Packages/groonga-libs-2.1.2-0.x86_64.rpm
wget http://packages.groonga.org/centos/5/x86_64/Packages/groonga-munin-plugins-2.1.2-0.x86_64.rpm
wget http://packages.groonga.org/centos/5/x86_64/Packages/groonga-plugin-suggest-2.1.2-0.x86_64.rpm
wget http://packages.groonga.org/centos/5/x86_64/Packages/groonga-server-common-2.1.2-0.x86_64.rpm
wget http://packages.groonga.org/centos/5/x86_64/Packages/groonga-server-gqtp-2.1.2-0.x86_64.rpm
wget http://packages.groonga.org/centos/5/x86_64/Packages/groonga-server-http-2.1.2-0.x86_64.rpm
wget http://packages.groonga.org/centos/5/x86_64/Packages/groonga-tokenizer-mecab-2.1.2-0.x86_64.rpm

wget https://packages.groonga.org/source/mroonga/mroonga-2.02.tar.gz


$ yum install ruby.x86_64

$ rpm -ivh groonga-doc-
$ rpm -ivh groonga-libs-
-- rpm -ivh groonga-munin-plugins-
$ rpm -ivh groonga-
$ rpm -ivh groonga-devel-


$ ./configure \
    --with-mysql-source=/root/apps/mysql-5.1.73 \
    --with-mysql-config=/usr/local/mysql/bin/mysql_config \
    --without-mecab

$ make
$ make install

> install plugin groonga SONAME 'ha_groonga.so';
> create function last_insert_grn_id returns integer SONAME 'ha_groonga.so';
```

#### groonga test
- p584
```
create table tb_weather (
    id  int primary key,
    content varchar(255),
    fulltext index(content)
) engine=groonga DEFAULT charset utf8;

insert into tb_weather values (1, '내일 서울은 맑을 것입니다.');
insert into tb_weather values (2, '내일 경남 지방은 비가 올 것입니다.');

select * from tb_weather where match(content) against ('서울');
```