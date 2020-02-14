#### 使用docker 构建mysql镜像，并在容器初次创建时初始化数据



#### Dockerfile

```dockerfile
FROM mysql:5.7.23

MAINTAINER gradyjiang "jiangzhongjin@hotmail.com"

ENV LANG C.UTF-8

# 当前父目录
ENV PARENT_DIR .

# 容器内 mysql 工作目录
ENV CONTAINER_WORK_PATH /usr/local

# 被容器自动执行的目录
ENV AUTO_RUN_DIR /docker-entrypoint-initdb.d

# 初始化数据库的SQL
ENV DDL_SQL V1.0_DDL.sql
ENV DML_SQL V1.0_DML.sql

# 执行SQL的脚本
ENV INSTALL_DATA_SHELL install_data.sh

COPY $PARENT_DIR/$DDL_SQL $CONTAINER_WORK_PATH/

COPY $PARENT_DIR/$DML_SQL $CONTAINER_WORK_PATH/

COPY $PARENT_DIR/$INSTALL_DATA_SHELL $AUTO_RUN_DIR/

RUN chmod a+x $AUTO_RUN_DIR/$INSTALL_DATA_SHELL
```



#### /docker-entrypoint-initdb.d 是 ==mysql==镜像的自动执行目录，会自动执行内部的SQL脚本和linux脚本；但==仅在第一次创建时执行==

```shell
查看docker-entrypoint.sh （即执行docker-entrypoint-initdb.d 内的文件的脚本）可知，一旦数据库创建(即数据库数据目录已存在)，将不会执行该目录中的文件；

一下是其中重点代码

# Loads various settings that are used elsewhere in the script
# This should be called after mysql_check_config, but before any other functions
docker_setup_env() {
	# Get config
	declare -g DATADIR SOCKET
	DATADIR="$(mysql_get_config 'datadir' "$@")"
	SOCKET="$(mysql_get_config 'socket' "$@")"

	# Initialize values that might be stored in a file
	file_env 'MYSQL_ROOT_HOST' '%'
	file_env 'MYSQL_DATABASE'
	file_env 'MYSQL_USER'
	file_env 'MYSQL_PASSWORD'
	file_env 'MYSQL_ROOT_PASSWORD'

	declare -g DATABASE_ALREADY_EXISTS
	if [ -d "$DATADIR/mysql" ]; then
		DATABASE_ALREADY_EXISTS='true'
	fi
}

_main() {
		...
		...
		# there's no database, so it needs to be initialized
		if [ -z "$DATABASE_ALREADY_EXISTS" ]; then
			docker_verify_minimum_env
			docker_init_database_dir "$@"

			mysql_note "Starting temporary server"
			docker_temp_server_start "$@"
			mysql_note "Temporary server started."

			docker_setup_db
			docker_process_init_files /docker-entrypoint-initdb.d/*

			mysql_expire_root_user

			mysql_note "Stopping temporary server"
			docker_temp_server_stop
			mysql_note "Temporary server stopped"

			echo
			mysql_note "MySQL init process done. Ready for start up."
			echo
		fi
	fi
	exec "$@"
}
```







#### 由于有两个脚本，可能在执行顺序上不一样，所以需要用一个脚本来执行

```shell
#!/bin/bash
mysql -uroot -p$MYSQL_ROOT_PASSWORD <<EOF
source $CONTAINER_WORK_PATH/$DDL_SQL;
source $CONTAINER_WORK_PATH/$DML_SQL;
```



##### 构建镜像脚本 build-image-fim-mysql.sh

```shell
echo "开始构建 fim-mysql 镜像..."

docker build -t fim-mysql:1.0 ./fim-mysql
```



##### 执行创建容器命令 docker-run-fim-mysql.sh

```shell
docker run --name fim-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d fim-mysql:1.0
```

==MYSQL_ROOT_PASSWORD=123456，指定 root 用户名密码 123456==

==MYSQL_DATABASE=my_db 创建数据库实例 my_db==

