# 로컬 DBMS 도커로 마이그레이션 하기
## mariaDB 가 백그라운드로 동작 중일 때 찾아내기 어려웠던 이유 
`mysqld` 이름으로 동작하고 있다.
`maria`, `mariadb`, `db` 등으로 검색할 것이 아니라, `mysqld`로 검색해야 한다.

```
// 실패
$ pidof maria
$ pidof mariadb
$ pidof db 

// 성공
$ pidof mysqld // 성공
```

동작 중인 mariadb(mysqld)를 종료하고 싶다면 `service mysql stop` 명령어를 사용한다.
(`kill` 명령을 통해 프로세스를 강제 종료 시켜도 곧장 되살아 난다.)

```
$ sudo service mysql stop
```

<br>

## mariaDB, mysqlDB의 데이터 저장 경로
별도의 데이터 저장경로를 지정하지 않았을 경우, default 저장경로는 `/var/lib/mysql/` 이다.
해당 경로로 이동해보면 여러가지 로그파일과 schema 이름과 동일한 디렉토리를 발견할 수 있다.
schema 이름과 동일한 디렉토리로 접근시 테이블들의 `테이블명.frm` 파일과 `테이블명.ibd` 파일을 발견할 수 있다.

```
$ sudo ls -l babble/
total 700
-rw-rw---- 1 mysql mysql     61 Jul  9 05:50 db.opt
-rw-rw---- 1 mysql mysql   1715 Jul 17 03:52 game.frm
-rw-rw---- 1 mysql mysql  98304 Jul 17 03:52 game.ibd
-rw-rw---- 1 mysql mysql   1515 Jul 17 03:52 room.frm
-rw-rw---- 1 mysql mysql 114688 Jul 21 11:06 room.ibd
-rw-rw---- 1 mysql mysql   2730 Jul 17 03:52 room_tag.frm
-rw-rw---- 1 mysql mysql 131072 Jul 19 11:57 room_tag.ibd
-rw-rw---- 1 mysql mysql   2765 Jul 17 03:52 session.frm
-rw-rw---- 1 mysql mysql 131072 Jul 21 11:06 session.ibd
-rw-rw---- 1 mysql mysql   1686 Jul 17 03:52 tag.frm
-rw-rw---- 1 mysql mysql  98304 Jul 17 03:52 tag.ibd
-rw-rw---- 1 mysql mysql   2237 Jul 17 03:52 user.frm
-rw-rw---- 1 mysql mysql 114688 Jul 21 11:06 user.ibd
```

> 이 때 `테이블명.frm` 파일은 innoDB(maria, mysql이 사용하는 엔진)에서 테이블 구조를 다루는 파일이고,
`테이블명.ibd` 파일은 innoDB에서 테이블에 속하는 데이터를 다루는 파일들이다.
간단히 이야기해서 `frm=테이블, ibd=데이터`가 된다.

<br>

## mariaDB, mysqlDB DBMS 설정 저장 경로
별도의 설정이 없을 경우 default 설정 저장 경로는 `/etc/mysql/conf.d`다.
해당 디렉토리에 DBMS 설정 관련 파일이 존재한다.

```
$ ll
total 16
drwxr-xr-x 2 root root 4096 Jul  9 02:30 ./
drwxr-xr-x 4 root root 4096 Jul  9 02:30 ../
-rw-r--r-- 1 root root    8 Aug  3  2016 mysql.cnf
-rw-r--r-- 1 root root   55 Aug  3  2016 mysqldump.cnf
```

<br>

## docker-compose.yml 파일 작성
```yml
# docker-compose.yml

version: '3.7'  # yml 문법 버전 3.7을 사용하겠다.
services:
    mariadb:
        image: mariadb
        restart: always
        environment:
            - MYSQL_ROOT_PASSWORD=[비밀번호]
        volumes:
            # [로컬 디렉토리]:[도커 컨테이너 내부 디렉토리] 매핑
            - ./docker-volume/mariadb/mysql:/var/lib/mysql
            - ./docker-volume/mariadb/conf.d:/etc/mysql/conf.d
        ports:
            - '3306:3306' #포트번호 매핑
        environment:
            TZ: Asia/Seoul # 타임존
```

기존 로컬 데이터베이스에서 사용중이던 2개의 디렉토리를 복제(원본 파일 안전을 위해)한 후 
도커 컨테이너 내부 디렉토리에 매핑시켜줌을 명시한다. 이렇게 하면 기존 데이터를 모두 살린 채로 
데이터베이스를 도커로 마이그레이션 할 수 있다.

`docker-compose.yml` 파일이 존재하는 위치에서 도커 컨테이너를 올려보자.

```
$ sudo docker-compose up -d --build
Creating [DB 컨테이너 이름] ... done
```

```JSON
# HTTP GET 요청 찔러본 결과
{
    "roomId": 3,
    "createdDate": "2021-07-17T04:25:14.568",
    "game": {
        "id": 3,
        "name": "Apex Legend"
    },
    "tags": [
        "2시간",
        "솔로랭크",
        "실버"
    ]
}
```

한글 인코딩 설정도 함께 포함되어 있기 때문에 한글이 깨지지 않고 정상적으로 잘 반환됨을 확인할 수 있다!
그러나 DBMS 툴로 접근해서 테이블을 조회해보면 한글 인코딩이 전부 깨진 것을 볼 수 있다.

```
$ sudo docker exec -it [DB 컨테이너 이름] bash
$ root@8d577f8c631d:/# mysql -u root -p
Enter password: [설정해둔 패스워드]

MariaDB [(none)]> use babble;
MariaDB [babble]> select * from user;
+----+------+---------+
| id | name | room_id |
+----+------+---------+
|  1 | ??   |    NULL |
|  2 | ???  |    NULL |
|  3 | ???  |    NULL |
|  4 | ??   |    NULL |
|  5 | ???  |    NULL |
|  6 | ??   |    NULL |
+----+------+---------+

MariaDB [babble]> show variables like 'char%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | latin1                     |
| character_set_connection | latin1                     |
| character_set_database   | utf8mb3                    |
| character_set_filesystem | binary                     |
| character_set_results    | latin1                     |
| character_set_server     | utf8mb4                    |
| character_set_system     | utf8mb3                    |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
```

`web-server - was - db` 를 거치는 HTTP GET 요청에는 정상적으로 한글 결과를 받을 수 있었으나, 
DBMS 툴로 직접 테이블을 조회한 경우에만 한글이 깨졌다. 즉, DBMS 자체도 한글 인코딩을 설정해주어야 한다.

도커 볼륨 매핑을 위해 복제했던 로컬 디렉토리 `conf.d`의 `mysql.cnf` 파일에 한글 인코딩 설정을 추가한다.

```cnf
# docker-volume/mariadb/conf.d/mysql.cnf

[mysqld]
skip-character-set-client-handshake
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[client]
default-character-set = utf8

[mysql]
default-character-set = utf8

[mysqldump]
default-character-set = utf8
```

이후 도커 컨테이너를 내린 뒤 다시 올려보자.

```
$ sudo docker stop [DB 컨테이너 이름]
$ sudo docker rm [DB 컨테이너 이름]

$ sudo docker-compose up -d --build
Creating [DB 컨테이너 이름] ... done
```

도커의 데이터베이스에 다시 접근해서 테이블을 확인하자.

```
MariaDB [babble]> select * from user;
+----+-----------+---------+
| id | name      | room_id |
+----+-----------+---------+
|  1 | 루트      |    NULL |
|  2 | 와일더    |    NULL |
|  3 | 현구막    |    NULL |
|  4 | 포츈      |    NULL |
|  5 | 그루밍    |    NULL |
|  6 | 피터      |    NULL |
+----+-----------+---------+

MariaDB [babble]> show variables like 'char%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8mb4                    |
| character_set_connection | utf8mb4                    |
| character_set_database   | utf8mb3                    |
| character_set_filesystem | binary                     |
| character_set_results    | utf8mb4                    |
| character_set_server     | utf8mb4                    |
| character_set_system     | utf8mb3                    |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
```

한글이 정상적으로 잘 출력됨을 확인할 수 있다.

<br>

## References
- [[MySQL || MariaDB] InnoDB 총 정리 - .java의 개발일기 - Tistory](https://java119.tistory.com/75)
- [InnoDB에 대하여 (MyISAM과의 차이점) - 삵 (sarc.io)](https://sarc.io/index.php/mariadb/346-innodb-myisam)
- [[Docker] mariadb 및 utf-8 설정 - 정리 중... - 티스토리](https://royleej9.tistory.com/entry/Docker-mariadb-%EB%B0%8F-utf-8-%EC%84%A4%EC%A0%95)
