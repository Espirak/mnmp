# mnmp
nginx+mysql+php for macos

# Install
1. 把: https://github.com/Espirak/mnmp 的脚本下载下来.

```bash
  git clone https://github.com/Espirak/mnmp
```
* 安装前请确认安装了homebrew, 就不提了. 开始安装:
* bash切换到setup-mnmp.sh目录, 然后执行: sh setup-mnmp.sh 就等着自动安装吧.

必要的一些配置:

2. nginx:

* vim /usr/local/etc/nginx/nginx.conf
* http {...} 里面最后面加上: include vhost/*.conf; (大概是倒数第三行的位置)
* vim /usr/local/etc/nginx/vhost/default.conf , 添加类似下面的内容:
```ini
server {
    listen       80;
    server_name  dev.local;
    root   /your_direcotry;
    index  index.html index.htm index.php;

    autoindex on;
    autoindex_exact_size off;
    autoindex_localtime on;
    #error_page  404 /404.html;
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {root html;}
    location ~ .*\.(php|php5)?$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        include fastcgi.conf;
    }
    location / {
        if (!-e $request_filename){
            rewrite ^/sitemap.xml$ /sitemap.php; # rewrite
        }
    }
    access_log off;
}
```
* sudo sh -c "echo '127.0.0.1 yourdomain.com' >> /etc/hosts"
* 上面的nginx配置和命令是绑定http://yourdomain.com作为域名, 绑定到/your_direcotry目录(域名和目录根据自己的需要修改吧).

3. php-fpm:

* vim /usr/local/etc/php/5.6/php-fpm.conf, 找到并修改下面3行, 后面两个是要注释掉的:
```ini
error_log = /tmp/php-fpm.log
;user = _www
;group = _www
```
* 另外php.ini的路径: /usr/local/etc/php/5.6/php.ini, 如果有额外需求自己修改, 比如:
```ini
date.timezone = Asia/Shanghai
error_reporting = E_ALL
```
4. mysql:

* 基本不需要配置了, 配置文件在/usr/local/opt/mysql/my-new.cnf 如果没有就是 my.cnf.
* 默认不需密码, 如果需要可以执行: mysql_secure_installation 一步步来, 本地开发意义不大.

5. 启动重启service脚本:

* 设置权限: chmod +x ./mnmp.sh
* 然后运行: ./mnmp.sh start | stop | restart 即可.
* 推荐加到profile里面, 比如: echo "alias mnmp='/Users/你的路径/mnmp.sh'" >> ~/.bash_profile
* source ~/.bash_profile
* 因为我不需要开机启动, 如果有需要开机启动的可以参考gist.github.com/mystix/3041577最下面的那几行.
* 然后就可以直接: mnmp start 这样用了.

* Ps: 如果遇到类似这样的错误: "xxx" failed (13: Permission denied), 需要设定下权限:
* chmod 755 $HOME && chmod 755 $HOME/Documents/

6. phpmyadmin:(可选)
* 下载: www.phpmyadmin.net/home_page/downloads.php
* 解压到nginx的conf指定的目录, config.sample.inc.php重命名为config.inc.php, 修改$cfg['Servers'][$i]['AllowNoPassword'] = true;
* Ps: 如果phpmyadmin报错:The mcrypt extension is missing. 可以关掉mnmp, 执行下:brew install php54-mcrypt
* 然后打开: http://youdomain.com/phpmyadmin 应该就OK了.
