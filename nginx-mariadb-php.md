# nginx/mariadb/php

## MariaDB

インストール
```
$ sudo dnf install mariadb-server
$ sudo systemctl start mariadb
```

データベースにログイン
```
# root でログイン
$ sudo mariadb -u root
# <user_name> でログイン
$ mariadb -u <user_name> -p
# <user_name> と <db_name> を指定してログイン
$ mariadb -u <user_name> -p <db_name>
```

ユーザ作成
```
MariaDB> grant all privileges on *.* to '<user_name>'@'localhost' identified by '<password>'
```

