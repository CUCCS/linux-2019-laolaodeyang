### 《Linux系统与网络管理》实验五：web服务器
#### 软件环境
+ Ubuntu 18.04 Server 64bit
+ 软件环境建议
  + Nginx
  + VeryNginx
  + Wordpress
  + WordPress 4.7
  + Damn Vulnerable Web Application (DVWA)
#### 实验内容
###### 基本要求
+ 在一台主机（虚拟机）上同时配置Nginx和VeryNginx
  + VeryNginx作为本次实验的Web App的反向代理服务器和WAF
  + PHP-FPM进程的反向代理配置在nginx服务器上，VeryNginx服务器不直接配置Web站点服务
+ 使用Wordpress搭建的站点对外提供访问的地址为： https://wp.sec.cuc.edu.cn
+ 使用Damn Vulnerable Web Application (DVWA)搭建的站点对外提供访问的地址为： http://dvwa.sec.cuc.edu.cn
###### 安全加固要求
+ 使用IP地址方式均无法访问上述任意站点，并向访客展示自定义的友好错误提示信息页面-1
+ Damn Vulnerable Web Application (DVWA)只允许白名单上的访客来源IP，其他来源的IP访问均向访客展示自定义的友好错误提示信息页面-2
+ 在不升级Wordpress版本的情况下，通过定制VeryNginx的访问控制策略规则，热修复WordPress < 4.7.1 - Username Enumeration
+ 通过配置VeryNginx的Filter规则实现对Damn Vulnerable Web Application (DVWA)的SQL注入实验在低安全等级条件下进行防护
###### VERYNGINX配置要求
+ VeryNginx的Web管理页面仅允许白名单上的访客来源IP，其他来源的IP访问均向访客展示自定义的友好错误提示信息页面-3
+ 通过定制VeryNginx的访问控制策略规则实现：
  + 限制DVWA站点的单IP访问速率为每秒请求数 < 50
  + 限制Wordpress站点的单IP访问速率为每秒请求数 < 20
  + 超过访问频率限制的请求直接返回自定义错误提示信息页面-4
  + 禁止curl访问
#### 实验步骤
##### 环境准备
一、安装nginx
1. 从源直接安装nginx
```bash 
sudo apt-get update
sudo apt-get install nginx
```
2. 调整防火墙，以免出现问题
```bash 
sudo ufw app list
```
获得可用应用程序：
+ Nginx Full
+ Nginx HTTP
+ Nginx HTTPS
+ OpenSSH
```bash
sudo ufw allow 'Nginx HTTP'
sudo ufw allow 'Nginx HTTPS'
sudo ufw allow 'OpenSSH'
#输入命令启动防火墙
sudo ufw enable
#输入命令查看防火墙状态
sudo ufw status
```
可以看到允许通过的服务有哪些：
3. 检查web服务器是否正在运行
```bash
sudo systemctl status nginx
```
4. 访问默认网页
```bash
192.168.56.101
```

二、安装verynginx
```bash
#安装git
sudo apt-get install git 

#安装依赖
sudo apt-get install libssl-dev libpcre3 libpcre3-dev build-essential

#安装verynginx
git clone https://github.com/alexazhou/VeryNginx.git
cd VeryNginx
sudo python install.py install
```

```bash
#更改端口，以免同时占用80端口
sudo vim /opt/verynginx/openresty/nginx/conf/nginx.conf
#将监听端口改为8080
```

相关命令
```bash
#启动服务
/opt/verynginx/openresty/nginx/sbin/nginx

#停止服务
/opt/verynginx/openresty/nginx/sbin/nginx -s stop

#重启服务
/opt/verynginx/openresty/nginx/sbin/nginx -s reload
```

在浏览器中输入```https://192.168.56.101/verynginx/index.html```出现如下界面访问成功
初始账号密码为```verynginx/verynginx```

三、基于LEMP安装wordpress

参考资料：
+ https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-lemp-on-ubuntu-18-04

1. 安装mysql
```bash
sudo apt-get install mysql-server
sudo service mysql start
mysql -u root -p

#进入mysql

#创建wordpress独立数据库
mysql>CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;

#创建账户，设置密码
mysql>GRANT ALL ON wordpress.* TO 'wordpressuser'@'localhost' IDENTIFIED BY 'password';

#刷新更改
mysql>FLUSH PRIVILEGES;

mysql>EXIT;
```

2. 安装php以及相关依赖
```bash
sudo apt-get update

#安装php7.2
sudo apt-get install php7.2

#安装相关依赖
sudo apt-get install php-curl php-gd php-intl php-mbstring php-soap php-xml php-xmlrpc php-zip

#重启php
sudo systemctl restart php7.2-fpm
```

3. 下载wordpress
```bash
cd /tmp

#下载wordpress-5.1.1
sudo wget https://wordpress.org/wordpress-5.1.1.tar.gz

#解压
tar xzvf wordpress-5.1.1.tar.gz

# 复制配置文件到wordpress实际读取的文件中
cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php

#创建wordpress根目录
mkdir /var/www/html/wq.sec.cuc.edu.cn

#将tmp目录下文件复制到根目录
sudo cp -a /tmp/wordpress/. /var/www/html/wq.sec.cuc.edu.cn

#分配文件所有权给www-data用户和组
sudo chown -R www-data:www-data /var/www/wordpress
```

4. 配置nginx
```bash
#查看站点服务器块文件
sudo vim /etc/nginx/sites-available/default

#更改文件名
sudo mv /etc/nginx/sites-available/default /etc/nginx/sites-available/wp.sec.cuc.edu.cn

#进入文件并修改
sudo vim /etc/nginx/sites-available/wp.sec.cuc.edu.cn

#在主server块中，添加几个location块
server {
    . . .

    location = /favicon.ico { log_not_found off; access_log off; }
    location = /robots.txt { log_not_found off; access_log off; allow all; }
    location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
    }
    location ~ /\.ht{
            deny all;
    }
    location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
        expires max;
        log_not_found off;
    }
    . . .
}

#在现有location/块内部，调整try_files列表，以便不是将404错误作为默认选项返回，而是index.php使用请求参数将控制传递给文件
server {
    . . .
    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }
    . . .
}

#修改root
root /var/www/html/wp.sec.cuc.edu.cn;

#修改server name
server_name 192.168.56.101;

#保存退出
```

+ 创建软连接
```bash
sudo ln -s /etc/nginx/sites-available/wp.sec.cuc.edu.cn /etc/nginx/sites-enabled/
```
+ 取消链接默认配置文件
```bash
sudo unlink /etc/nginx/sites-enabled/default
```

+ 查看配置文件是否有错误
```bash
sudo nginx -t
```
无误

+ 重新加载nginx
```bash
sudo systemctl reload nginx
```
报错
```bash
#查看错误原因
sudo systemctl status nginx.service
```
原来是空格害人，更正错误后重启成功

5. 设置wordpress配置文件
+ 从wordpress密钥生成器中获取安全值
```bash
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```

获得如下

+ 打开wordpress配置文件
```bash
sudo nano /var/www/html/wp.sec.cuc.edu.cn/wp-config.php
```

+ 将文件中相应部分替换
+ 修改开头部分数据
```bash
define('DB_NAME', 'wordpress');

define('DB_USER', 'wordpressuser');

define('DB_PASSWORD', 'password');
···
define('FS_METHOD', 'direct');
```

6. 通过web界面完成安装
+ 在浏览器中，输入```192.168.56.103:8080```
+ 选择语言，进行设置，完成登录

四、安装dvwa
+ 参考资料：https://kifarunix.com/how-to-setup-damn-vulnerable-web-app-lab-on-ubuntu-18-04-server/

* 下载dvwa
```bash
# 在/var/www/html下为DVWA创建目录
sudo mkdir /var/www/html/DVWA

# 将安装仓库克隆到临时目录下
git clone https://github.com/ethicalhack3r/DVWA /tmp/DVWA

# 将安装文件拷贝到/var/www/html/DVWA网站根目录下
sudo rsync -avP /tmp/DVWA/ /var/www/html/DVWA
```

* 配置dvwa
+ 配置数据库
```bash
#复制config.inc.php.dist到config.inc.php
cp /var/www/html/DVWA/config/config.inc.php.dist /var/www/html/DVWA/config/config.inc.php
```
+ 在mysql中为dvwa新建用户名密码
```bash
# 新建数据库dvwa
CREATE DATABASE dvwa DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;

# 创建用户名dvwauser,分配管理权限并设置密码
GRANT ALL ON dvwa.* TO 'dvwauser'@'localhost' IDENTIFIED BY 'p@ssw0rd';

# 刷新配置
FLUSH PRIVILEGES;

# 退出
exit；

# 重启mysql
sudo systemctl restart mysql
```

+ 修改dvwa配置文件
```bash
sudo sudo vim /var/www/html/DVWA/config/config.inc.php

#进行修改
$_DVWA[ 'db_server' ]   = '127.0.0.1';
$_DVWA[ 'db_database' ] = 'dvwa';
$_DVWA[ 'db_user' ]     = 'dvwauser';
$_DVWA[ 'db_password' ] = 'p@ssw0rd';
```

+ 配置php文件
```bash
#修改/etc/php/7.2/fpm/php.ini
vim /etc/php/7.2/fpm/php.ini

#修改
allow_url_include = On
allow_url_fopen = On
display_errors = Off

#修改DVWA文件访问权限
sudo chown -R www-data.www-data /var/www/html/
```

+ 配置nginx
```bash
# 将/etc/nginx/sites-available下的wp.sec.cuc.edu.cn复制新文件改名为dvwa
cp /etc/nginx/sites-avaliable/wp.sec.cuc.edu.cn /etc/nginx/sites-avaliable/dvwa

# 将监听端口改为8088
listen 8088;
listen [::]:8088 ipv6only=on;

#将root目录进行更改
root /var/www/html/DVWA;

# 创建软连接
sudo ln -s /etc/nginx/sites-available/dvwa /etc/nginx/sites-enabled/

# 重启nginx
sudo systemctl restart nginx
```
