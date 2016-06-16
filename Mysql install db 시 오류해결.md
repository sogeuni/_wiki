```bash
sudo /usr/local/mysql/scripts/mysql_install_db --user=mysql

FATAL ERROR: Could not find ./bin/my_print_defaults

If you compiled from source, you need to either run 'make install' to
copy the software into the correct location ready for operation.
If you don't want to do a full install, you can use the --srcddir
option to only install the mysql database and privilege tables

If you are using a binary release, you must either be at the top
level of the extracted archive, or pass the --basedir option
pointing to that location.

The latest information about mysql_install_db is available at
https://mariadb.com/kb/en/installing-system-tables-mysql_install_db
```

mysql 설치 후 기본 database 생성시 위와 같은 오류가 발생할 경우 다음과 같이 `basedir` 옵션을 추가해 준다.

```shell
sudo /usr/local/mysql/scripts/mysql_install_db --user=mysql -basedir=/usr/local/mysql
```

혹은 해당 경로로 이동한 후 `mysql_install_db`를 수행해도 된다.
