# 도커 위의 DBMS를 로컬로 마이그레이션 하기

이전에 [로컬에서 잘 돌아가고 있는 데이터베이스를 굳이 도커 위로 마이그레이션 시킨 경험](https://hyeon9mak.github.io/migrate-local-database-to-docker/)이 있었다. 그 후로 큰 이슈 없이 서비스가 잘 동작했기 때문에 도커 위에 데이터베이스를 운영하는 것에 만족하고 있었는데, 얼마전 브라운이 초청해주신 홍정민 DBA님의 특별 강연에서 
DBA님이 도커 위에 데이터베이스를 운영하는 것에 대해 부정적인 말씀을 하셔서, [질문을 드리고 답변을 받게 되었다.](https://hyeon9mak.github.io/why-does-not-run-database-on-docker/) 그리고 답변 받은 내용을 베이스로 확신을 얻어서 도커 위에 올렸던 데이터베이스를 다시 로컬로 내려보았다!

<br>

## 도커에서 사용중인 데이터베이스 덤프 백업하기
도커에서 동작하고 있는 데이터베이스를 덤프로 추출하여 백업한다.
도커 컨테이너의 bash로 접속한 다음, mysqldump 명령어를 통해 덤핑한다.

```
$ sudo docker exec -it {컨테이너 이름} bash

# dockershell에 접속된 상태
& sudo mysqldump -u {DB 계정 이름} -p {덤프할 스키마 이름} > {덤프 파일명}.sql
```

명령을 수행하면 도커 컨테이너 내부에 덤프 파일이 생성된다.
컨테이너 내부 덤프 파일을 로컬로 꺼내오면 된다.
(혹시나 dockershell 에서 나오지 않았다면 `exit` 하자.)

```
$ sudo docker cp {컨테이너 이름}:{덤프 파일경로} {복제할 로컬 경로}
```

명령을 수행하면 도커 컨테이너 내부에 있는 덤프 파일이 로컬 경로로 복제된다.
덤프 파일을 성공적으로 추출했다고 확신이 들면, 도커 컨테이너를 일시중지 시킨다.
일시중지 하지 않을 경우 mariadb 도커 컨테이너와 로컬 mariadb-server가 같은 포트번호(3306)를 사용하려 하기 때문에 충돌이 발생해서 동작하지 않는다. (`/var/run/mysqld/mysqld.sock` 관련 에러가 발생할 것이다.)

```
$ sudo docker stop {컨테이너 이름}
```

<br>

## 로컬 mariadb-server 버전업
> 최근 버전업을 시도하면 더 이상 IP를 찾을 수 없다고 에러가 출력되며, `$sudo apt install mariadb-server` 명령어를 입력하면 자동으로 10.5 버전이 설치된다. 따라서 이 부분은 참고용으로 지나가도 될 것 같다.

로컬에서 `apt install` 명령을 통해 mariadb-server를 설치할 경우 10.1 버전이 설치된다. 
보통 문제가 없겠지만, Flyway를 사용하는 경우 10.1 버전을 더 이상 사용할 수 없다.

> FlywayEditionUpgradeRequiredException: Flyway Teams Edition or MariaDB upgrade required: MariaDB 10.1 is no longer supported by Flyway Community Edition, but still supported by Flyway Teams Edition.

도커를 통해 사용하는 mariadb-server는 lastest 버전 설치시 자동으로 10.5 버전이 설치되기 때문에 문제가 없었지만, [로컬 설치에는 별도로 apt 설정](https://www.linuxbabe.com/mariadb/install-mariadb-10-5-ubuntu)을 해주어야 한다.

```
$ sudo apt-get install software-properties-common

$ sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8

$ sudo add-apt-repository 'deb [arch=amd64,arm64,ppc64el] http://mirror.lstn.net/mariadb/repo/10.5/ubuntu {bionic 또는 focal} main'
```

이 때 `add-apt-repository` 명령 입력시 Ubuntu 18.04 버전은 `binoic`을, 20.04 버전은 `focal`을 이용하면 된다.

```
$ sudo apt update

$ sudo apt install mariadb-server
```

설정을 마치고 새로 설치를 진행할 때, 기존 mariadb-server가 설치되어 있는 경우 설치 마지막 과정에서 새 버전으로 설치할 것인지, 기존 버전을 유지할 것인지 질문이 나온다. `Y` 입력으로 새로운 10.5 버전으로 설치를 진행하면 된다.

설치에 성공한지 확인하고 싶다면 아래 명령을 수행하면 된다.

```
$ sudo mysql -u root -p
(초기엔 비밀번호 없음)

> SELECT VERSION();
+----------------------------------------+
| VERSION()                              |
+----------------------------------------+
| 10.5.12-MariaDB-1:10.5.12+maria~bionic |
+----------------------------------------+
```

<br>

## IP 주소 허용 수정
새로운 버전을 설치할 경우 기존 mariadb-server 설정 값이 모두 초기화 된다.
bind-address 설정을 다시 바꾸기 위해 `/etc/mysql/mariadb.conf.d/50-server.cnf` 에 접근한다.

```
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address            = 0.0.0.0
```

bind-address 설정 기본값이 127.0.0.1 일텐데, 0.0.0.0으로 바꿔서 모든 ip를 허용해주면 된다.

<br>

## 덤프 불러오기
설정을 모두 마쳤다면 덤프 파일을 다시 불러와야 한다.

mariadb-server에 접속한 뒤 애플리케이션 서버에서 사용할 계정과 스키마를 생성하고 계정에 스키마 관리 권한을 부여해주자. (권한은 각 프로젝트 정책에 맞게 잘 부여하자.)

```
$ mysql -u root -p 

> create user {사용 계정명}@'{애플리케이션 서버 ip}' identified by '{비밀번호}';
> create database {스키마 이름};
> grant all privileges on {DB 이름}.* to {사용 계정명}@'{애플리케이션 서버 ip}';
```

설정을 모두 끝냈다면 exit 명령으로 mariadb-server에서 빠져 나온뒤, 아래 명령으로 덤프 파일을 읽게 하자.

```
$ sudo mysql -u root -p {스키마 이름} < {덤프 파일명}.sql
```

성공적으로 덤프가 불러와졌을 경우, 아래와 같이 결과를 확인할 수 있다.

```
> show databases;
+--------------------+
| Database           |
+--------------------+
| babble             |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.000 sec)

> use babble
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

> show tables;
+-----------------------+
| Tables_in_babble      |
+-----------------------+
| administrator         |
| alternative_game_name |
| alternative_tag_name  |
| flyway_schema_history |
| game                  |
| room                  |
| session               |
| tag                   |
| tag_registration      |
| user                  |
+-----------------------+
```

<br>

## References
- [과연 도커(Docker) 컨테이너를 통해 데이터베이스를 운영하는 게 좋은 방법일까? - 규도자 개발 블로그](https://this-programmer.tistory.com/entry/%EA%B3%BC%EC%97%B0-%EB%8F%84%EC%BB%A4Docker-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%EB%A5%BC-%ED%86%B5%ED%95%B4-%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4%EB%A5%BC-%EC%9A%B4%EC%98%81%ED%95%98%EB%8A%94-%EA%B2%8C-%EC%A2%8B%EC%9D%80-%EB%B0%A9%EB%B2%95%EC%9D%BC%EA%B9%8C)
- [[MariaDB] Docker Container에 올렸던 DB를 도커 바깥으로 빼내기 - 기록기록](https://parkadd.tistory.com/119)
- [How to Install MariaDB 10.5 on Ubuntu 18.04, Ubuntu 20.04 - LinuxBabe](https://www.linuxbabe.com/mariadb/install-mariadb-10-5-ubuntu)
