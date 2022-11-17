# Ubuntu安装MySQL

## 安装

```bash
sudo apt update

sudo apt upgrade

# 安装mysql
sudo apt-get install mysql-server mysql-client

# 查看默认密码
sudo cat /etc/mysql/debian.cnf

# 使用默认密码登录mysql
mysql -u XXXXX -p

# 修改root远程登录的ip为%，意思是允许所有的ip远程访问
use mysql;
update user set host = '%' where user = 'root' limit 1;
flush privileges;

# 查看用户表
select host,user from user;

# 修改root 密码
grant all privileges on *.* to 'root'@'%' identified by 'root';##

ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'xy159951';

# 重启mysql
mysql start
mysql stop
service mysql restart

# 卸载mysql
sudo rm /var/lib/mysql/ -R
sudo rm /etc/mysql/ -R
sudo apt-get autoremove mysql* --purge
sudo apt-get remove apparmor

# 查看mysql状态
sudo systemctl status mysql
```

## 使用外部程序访问

```bash
# 方法一是将身份验证方法从更改auth_socket为mysql_native_password
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'very_strong_password';

FLUSH PRIVILEGES;

#方法二是创建一个新的专用管理用户，该用户可以访问所有数据库：
GRANT ALL PRIVILEGES ON *.* TO 'administrator'@'localhost' IDENTIFIED BY 'very_strong_password';
```

## 创建新用户

```shell
# 使用mysql数据库
USE mysql；

# 创建用户
CREATE USER username IDENTIFIED BY 'password';

# 查看用户
SELECT user, host, authentication_string FROM USER WHERE USER='username';

# 修改用户密码
update user set authentication_string='' where user='username';
ALTER USER 'windows'@'%' IDENTIFIED BY '123456';

# 删除用户
DROP USER username;

# 修改绑定地址
sudo vim /etc/mysql/my.cnf ## 或者其他配置文件

# 查看权限
SHOW GRANTS FOR username;

# 授予用户全局级全部权限：
GRANT ALL PRIVILEGES ON *.* TO 'hsiangya'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;

# 授予用户针对database数据库全部权限：
GRANT ALL PRIVILEGES ON database.* TO 'hsiangya';

# 生效(刷新权限)
FLUSH PRIVILEGES;

# 撤销权限
# revoke privileges on databasename.tablename from 'username'@'host';
REVOKE ALL PRIVILEGES FROM windows;
```

## GRANT命令说明

- priveleges(权限列表)，可以是 all priveleges , 表示所有权限，也可以是 select、update 等权限，多个权限的名词,相互之间用逗号分开。
- on用来指定权限针对哪些库和表。
- *.* 中前面的 * 号用来指定数据库名，后面的 * 号用来指定表名。
- to 表示将权限赋予某个用户，@后面接限制的主机，可以是IP，IP段，域名以及%，%表示任何地方。注意：这里%有的版本不包括本地，可能会出现某个用户设置了%允许任何地方登录，但是在本地登录不了，这个和版本有关系，遇到这个问题再加一个localhost的用户就可以了。
- identifified by 指定用户的登录密码,该项可以省略。
- WITH GRANT OPTION 这个选项表示该用户可以将自己拥有的权限授权给别人。注意：经常有人在创建操作用户的时候不指定 WITH GRANT OPTION 选项导致后来该用户不能使用 GRANT 命令创建用户或者给其它用户授权。

**备注**：可以使用GRANT重复给用户添加权限，权限叠加，比如你先给用户添加一个select权限，然后又给用户添加一个insert权限，那么该用户就同时拥有了select 和 insert 权限。

## **授权原则说明：**

权限控制主要是出于安全因素，因此需要遵循一下几个经验原则：

1. 只授予能满足需要的最小权限，防止用户干坏事。比如用户只是需要查询，那就只给select权限就可以了，不要给用户赋予update、insert或者delete权限。

2. 创建用户的时候限制用户的登录主机，一般是限制成指定IP或者内网IP段。

3. 初始化数据库的时候删除没有密码的用户。安装完数据库的时候会自动创建一些用户，这些用户默认没有密码。

4. 为每个用户设置满足密码复杂度的密码。

5. 定期清理不需要的用户。回收权限或者删除用户。









