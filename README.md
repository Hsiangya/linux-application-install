## git clone github 无法连接

1. 查询git公网ip(https://www.ipaddress.com/),获得github.com、github.global.ssl.fastly.net、codeload.github.com三个域名的公网ip

2. sudo vim /etc/hosts,加入公网ip与域名

   ```
   140.82.113.4 github.com
   199.232.5.194 github.global.ssl.Fastly.net
   140.82.114.9 codeload.github.com
   ```

3. 保存并退出