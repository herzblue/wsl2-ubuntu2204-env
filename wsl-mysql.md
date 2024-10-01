
# WSL - MYSQL 설치 및 설정 가이드

## MYSQL 설치
```bash
sudo apt install mysql-server //mysql을 설치
```

## MYSQL 시작
WSL은 systemctl을 지원하지 않으므로 service 명령어로 mysql을 시작합니다.
```bash
sudo service mysql start 
```
만약 `/nonexistent` 폴더를 찾을 수 없다는 에러가 발생하면, 해당 폴더를 만들어주면 됩니다.
```bash
sudo mkdir /nonexistent
```

## MYSQL 환경 설정
```bash
sudo mysql_secure_installation
```
위 명령어를 입력하면 비밀번호를 입력하라는 메시지가 출력됩니다.

## MYSQL 사용
mysql 쉘로 이동합니다.
```bash
sudo mysql -u root -p
# Enter password: (그냥 enter)
```

## MYSQL DB 목록 확인
```sql
show databases;
-- +--------------------+
-- | Database           |
-- +--------------------+
-- | information_schema |
-- | mysql              |
-- | performance_schema |
-- | sys                |
-- +--------------------+
-- 4 rows in set (0.01 sec)
```


# MySQL Workbench 연결

## 시도 1
MySQL Workbench에 연결을 시도해봅니다. 그러면 아래에 '데이터베이스 서버에 연결할 수 없습니다' 오류가 표시됩니다.

```bash
hostname: 127.0.0.1
port: 3306
username: root
```

- 이유: 로컬 호스트 IP(예: 127.0.0.1)는 WSL의 IP와 매핑되지 않습니다. 이 로컬 호스트는 Windows의 로컬 호스트일 뿐입니다.

## 시도 2
그러면 WSL의 IP 주소를 사용해보겠습니다. PowerShell에서 다음과 같이 가져옵니다:

```bash
$ wsl hostname -I
172.28.255.242
```

Workbench 창에서 호스트 이름으로 IP 주소를 입력하고 연결을 시도해봅니다. 그러나 "연결할 수 없음" 또는 "연결이 설정되지 않음"이라고 표시되며 모든 UI 요소가 회색으로 표시됩니다.

- 이유: 기본 MySQL 구성은 127.0.0.1의 연결만 허용하고 다른 IP 주소의 연결은 허용하지 않습니다. </br> 이 문제를 해결하려면 모든 IP 주소의 바인딩(허용) 연결을 허용하도록 구성을 업데이트해야 합니다. </br> 특정 IP 주소를 설정할 수 있지만 WSL의 IP 주소는 변경되므로 이것이 더 쉬운 대안입니다.

## 시도 3
/etc/mysql/mysql.conf.d/mysqld.cnf에 있는 MySQL 구성 파일에서 바인드 주소 속성을 127.0.0.1에서 0.0.0.0으로 업데이트하고 MySQL 서버를 다시 시작합니다.

```bash
$ sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
# In the above file, 
# change 
#     bind-address=127.0.0.1 
# to
#     bind-address=0.0.0.0

$ sudo service mysql restart
#  * Stopping MySQL database server mysqld
#  * Starting MySQL database server mysqld
```

그리고 다시 연결해 보세요. 그러나 '데이터베이스 서버에 연결할 수 없습니다'라는 오류 메시지가 표시되면서 다시 실패합니다. 하지만 힌트가 있습니다. 사용자 root@172.28.255.242에 대한 액세스가 거부되었습니다.

- 이유: 루트는 기본적으로 localhost 연결만 수신합니다.

## 시도 4
해결책으로, MySQL 셸로 이동하여 루트로 로그인하고 모든 IP 연결을 수신할 수 있는 duck라는 새 사용자를 생성하세요.

```sql
sudo mysql -u root -p
mysql> create user if not exists `duck`@`%` identified with mysql_native_password by 'password';
Query OK, 0 rows affected (0.03 sec)
-- mysql> create user if not exists `root`@`%` identified with mysql_native_password by 'root1234';
```

% 와일드카드는 duck 사용자가 모든 IP를 수신해야 함을 나타냅니다.

그리고 Workbench를 재구성하고 연결을 다시 시도하세요.

짜잔🎉🎉!! 이제 작동하는 MySQL Workbench가 표시됩니다.

```sql
show databases;
```

위 명령을 실행하면 MySQL 서버의 모든 데이터베이스를 나열합니다.

## 마지막 문제
duck 사용자는 다른 일을 할 수 없다는 사실을 곧 깨닫게 될 것입니다 🦆!

이유: 사용자만 생성했지만 액세스 권한은 부여하지 않았습니다.

MySQL 셸에서 루트 사용자로 로그인하고 아래 명령을 실행하여 duck 사용자에게 모든 액세스 권한을 부여합니다.

```sql
mysql> GRANT CREATE, ALTER, DROP, INSERT, UPDATE, DELETE, SELECT, REFERENCES, RELOAD on *.* TO 'duck'@'%' WITH GRANT OPTION;
mysql> FLUSH PRIVILEGES;
-- 확인
mysql> SELECT user, authentication_string, plugin, host FROM mysql.user;
```

root 계정 만들어 사용하기.
```sql
mysql> CREATE USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root1234';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
mysql> GRANT CREATE, ALTER, DROP, INSERT, UPDATE, DELETE, SELECT, REFERENCES, RELOAD on *.* TO 'root'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
-- 확인
mysql> SELECT user, authentication_string, plugin, host FROM mysql.user;
```

WSL에서 실행되는 MySQL 서버에 대한 원활한 MySQL Workbench 연결이 가능해졌습니다.
