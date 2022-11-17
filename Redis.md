# Ubuntu Redis安装

## 安装配置

```shell
sudo apt update
sudo apt upgrade

# 安装
sudo apt install redis-server

# 查看redis进程
ps -aux | grep redis
ps -ef | grep redis
netstat -nlt|grep 6379

# 查看配置文件 修改绑定地址
sudo vi /etc/redis/redis.conf

# 启动服务器
sudo service redis-server start 
sudo service redis-server restart 
sudo service redis-server stop 

redis-server 

sudo /etc/init.d/redis-server restart
sudo service redis-server restart
sudo redis-server /etc/redis/redis.conf

# 启动redis-client
redis-cli

# 解决可能出现的中文乱码
redis-cli --raw
```

## 设置密码

```bash
# 添加redis访问密码
config set requirepass password

# 查看密码
config get requirepass

# 连接上后输入密码
auth password
```

