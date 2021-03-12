---
title: '[Java]后台线上环境部署笔记'
date: 2020-03-31
categories: 
  - 运维
tags:
  - 技巧
---
##### 安装Docker

```shell
# 安装docker
yum install docker
# 启动docker
systemctl start docker
# 设置开机自启动
systemctl enable docker
```

##### Docker安装及配置MySQL

###### 1.安装MySQL

```shell
docker run -d -p 3306:3306 -v /etc/conf:/etc/mysql/mysql.conf.d \
-v /etc/mysql/data:/var/lib/mysql  \
-e MYSQL_ROOT_PASSWORD=your-password --name mysql-docker mysql:5.7 
```

命令解析:

1. -d 参数表示后台运行
2. -p 主机端口:docker端口
3. -v 主机目录:docker目录 卷映射 防止docker删除重新运行数据丢失
4. -v /etc/conf:/etc/mysql/mysql.conf.d 将服务器/etc/conf文件夹下的配置映射到Docker内的/etc/mysql/mysql.conf.d目录(形成映射关系)
5. -v /etc/mysql/data:/var/lib/mysql    将服务器/etc/mysql/data 与Docker内的/var/lib/mysql目录形成映射关系
6. -e MYSQL_ROOT_PASSWORD=your-password 设置root的密码
7. --name mysql-docker 设置容器的名称
8. mysql:5.7 从镜像中启动一个容器
[CentOS 中利用docker安装MySQL](https://juejin.cn/post/6844904142167605256)

###### 2.新增MySQL的用户

```
>进入 MySQL 容器 其中mysql-docker是镜像名称
$ sudo docker exec -it mysql-docker /bin/bash
>登录 MySQL，执行后输入密码进入 MySQL
$ mysql -uroot -p 
>选择使用 mysql 数据库
$ use mysql; 

# 创建一个账号，用来进行远程访问；
# {username} 是远程访问登录的用户名，不建议用 root;
# {password} 是远程访问的登录密码;
# '%'代表的是所有IP，如果可以尽量设置指定 IP 或 IP 段
CREATE USER 'username'@'%' IDENTIFIED BY 'password';

# 赋予所有权限给之前创建的账号
GRANT ALL ON *.* TO 'username'@'%';

# 确认使用这里的密码登录此账号
ALTER USER 'username'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
# 刷新权限
FLUSH PRIVILEGES;
```

###### 3.Expression #1 of ORDER BY clause is not in GROUP BY 问题处理

>主要是sql_mode 中有一个 only_full_group_by 的值，用sql在服务器上查了一下里面的确有这个参数
将以下文件拷贝到 步骤1【1.安装MySQL】指定的 主机配置文件夹下->/etc/conf/mysqld.cnf
重启mysql容器即可【docker restart mysql-docker】

[mysqld.cnf]

```conf
# but not limited to OpenSSL) that is licensed under separate terms,
# as designated in a particular file or component or in included license
# documentation.  The authors of MySQL hereby grant you an additional
# permission to link the program and your derivative works with the
# separately licensed software that they have included with MySQL.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License, version 2.0, for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file = /var/run/mysqld/mysqld.pid
socket  = /var/run/mysqld/mysqld.sock
datadir  = /var/lib/mysql
#log-error = /var/log/mysql/error.log
# By default we only accept connections from localhost
#bind-address = 127.0.0.1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
sql_mode = STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

##### 错误处理:Data truncation: Incorrect datetime value: '0000-00-00 00:00:00' for column xxx

```sql
# 查看全局sql_mode
select @@global.sql_mode;
# 修改sql_mode(将上述查询到的sql_mode中的NO_ZERO_DATE和NO_ZERO_IN_DATE删除即可)
set @@global.sql_mode = 'STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
```

##### Docker安装Jenkins

```shell
#拉取镜像
docker pull jenkins/jenkins:lts
# 创建外部目录
mkdir -p /home/jenkins_home
# 运行容器
docker run -d --name jenkins -p 8181:8080 -v /home/jenkins_home:/home/jenkins_home jenkins/jenkins:lts
# 进入容器内部
docker exec -it jenkins /bin/bash
# 获取初始密码
cat /var/jenkins_home/secrets/initialAdminPassword
```

> 特别注意:如果访问不了,使用的是阿里云的服务器器,请确保在安全组放行特定的端口
