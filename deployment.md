# deployment

## 部署前修改

1. 修改加载脚本中的指令

   ```python
   # filename:configs.py
   import os
   from dotenv import load_dotenv
   
   basedir = os.path.abspath(os.path.dirname(__name__))
   
   dot_env_path = os.path.join(basedir, '.env')
   flask_env_path = os.path.join(basedir, '.flaskenv')
   if os.path.exists(dot_env_path):
       load_dotenv(dot_env_path)
   if os.path.exists(flask_env_path):
       load_dotenv(flask_env_path)
   ```

2. 修改配置文件

   ```python
   class BaseConfig:
       """基本的配置文件"""
   	...
   	
       LOG_LEVEL = logging.WARN
       
   class ProductionConfig(BaseConfig):
       SQLALCHEMY_DATABASE_URI = os.getenv('SQLALCHEMY_DATABASE_URI')
   ```

## linux部署操作

1. 更新apt

2. 安装数据库

3. 设置防火墙

   ```bash
   $ sudo ufw allow 22
   $ sudo ufw allow 80
   $ sudo ufw allow 443
   
   # 如果使用SMTP发送邮件，需要使用的端口，25或465或587
   $ sudo ufw allow 25
   $ sudo ufw allow 265
   $ sudo ufw allow 587
   
   # 查看防火墙状态
   $ sudo ufw status
   ```

4. 初始化环境(安装与开发时对应版本的环境)

5. 克隆仓库代码

6. 进入仓库目录，创建虚拟环境并安装

7. 创建.env环境变量，并修改Flask_ENV与FLASK_CONFIG

8. 初始化数据迁移

## Gunicorn操作

1. 安装依赖

   ```bash
   # 1. 安装
   $ pip install gunicorn
   
   # 测试时临时允许8000端口，部署后关闭
   $ sudo ufw allow 8000
   ```

2. 创建gunicorn配置文件

   ```python
   # filename: gunicorn.conf.py
   import os
   import multiprocessing
   
   bind = '127.0.0.1:8000'  # 绑定ip和端口号
   backlog = 512  # 监听队列
   chdir = os.path.dirname(os.path.abspath(__file__))  # gunicorn要切换到的目的工作目录
   timeout = 30  # 超时
   worker_class = 'sync'  # 使用gevent模式，还可以使用sync 模式，默认的是sync模式
   
   workers = multiprocessing.cpu_count() * 2 + 1  # 进程数
   threads = 2  # 指定每个进程开启的线程数
   loglevel = 'info'  # 日志级别，这个日志级别指的是错误日志的级别，而访问日志的级别无法设置
   access_log_format = '%(t)s %(p)s %(h)s "%(r)s" %(s)s %(L)s %(b)s %(f)s" "%(a)s"'  # 设置gunicorn访问日志格式，错误日志无法设置
   
   """
   其每个选项的含义如下：
   h          remote address
   l          '-'
   u          currently '-', may be user name in future releases
   t          date of the request
   r          status line (e.g. ``GET / HTTP/1.1``)
   s          status
   b          response length or '-'
   f          referer
   a          user agent
   T          request time in seconds
   D          request time in microseconds
   L          request time in decimal seconds
   p          process ID
   """
   accesslog = os.path.join(chdir, "logs/gunicorn_access.log")  # 访问日志文件
   errorlog = os.path.join(chdir, "logs/gunicorn_error.log")  # 访问日志文件
   
   ```

3. 测试程序是否可以启动

   ```bash
   gunicorn -c gunicorn.conf.py "application:create_app()"
   ```

## nginx操作

1. 安装依赖

   ```bash
   # 安装依赖
   $ sudo apt install nginx
   
   # 移除默认配置 新建个人配置
   $ sudo rm /etc/nginx/sites-enabled/default
   $ sudo vi /etc/nginx/sites-enabled/xxx
   ```

2. 个人配置中添加文本

   ```
   /etc/nginx/sites-enabled/xxx
   
   server {
       listen 80 default_server;
       access_log  /var/log/nginx/access.log;
       error_log  /var/log/nginx/error.log;
   
      location / {
               proxy_pass http://127.0.0.1:5000;  # 转发的地址，即Gunicorn运行的地址
               proxy_redirect     off;
   
               proxy_set_header   Host                 $host;
               proxy_set_header   X-Real-IP            $remote_addr;
               proxy_set_header   X-Forwarded-For      $proxy_add_x_forwarded_for;
               proxy_set_header   X-Forwarded-Proto    $scheme;
       }
   }
   ```

3. 检查配置文件语法正确性，并重启，关闭防火墙

   ```bash
   # 检查
   $ sudo nginx -t
   
   # 重启服务
   $ sudo service nginx restart
   
   # 关闭8000端口防火墙
   $ sudo ufw delete allow 8000
   ```

## Supervisor管理进程

1. 安装依赖

   ```bash
   $ sudo apt install supervisor
   ```

2. 创建配置文件

   > 这个全局配置默认会将/etc/supervisor/conf.d目录下的配置文件也包含在全局配置文件中，所以我们创建一个bluelog.conf存储程序配置：这个文件可以放在/etc/supervisord.conf路径下。

   ```bash
   $ sudo vim /etc/supervisor/conf.d/xxx.conf
   ```

3. 编辑配置文件

   ```
   [program:xxxx]
   command=bash /home/ubuntu/xxxx/start.sh
   user=root 
   autostart=true
   autorestart=true
   stopasgroup=true
   killasgroup=true
   ```

4. 切换到项目仓库目录下，创建start.sh文件

   ```bash
   # filename:start.sh
   cd /home/ubuntu/xxxx
   source venv/bin/activate
   exec gunicorn -c gunicorn.conf.py application:create_app("production")
   ```

5. 重启服务

   ```bash
   $ sudo service supervisor restart
   ```

```bash
# 进入suervisorctl
sudo supervisorctl

# 停止
supervisor > stop xxx

# 启动
supervisor > start xxx

# 查看错误日志
supervisor > tail infomation stderr  

# 查看所有可用的命令
supervisor > help  
```



