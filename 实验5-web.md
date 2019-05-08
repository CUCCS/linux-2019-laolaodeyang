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

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E8%B0%83%E6%95%B4%E9%98%B2%E7%81%AB%E5%A2%99.png )

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

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E9%98%B2%E7%81%AB%E5%A2%99%E7%8A%B6%E6%80%81.png )

3. 检查web服务器是否正在运行
```bash
sudo systemctl status nginx
```

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E6%A3%80%E6%9F%A5web%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%98%AF%E5%90%A6%E5%9C%A8%E8%BF%90%E8%A1%8C.png )

4. 访问默认网页
```bash
192.168.56.101
```

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E6%88%90%E5%8A%9F%E8%AE%BF%E9%97%AEnginx.png )


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

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E6%88%90%E5%8A%9F%E4%B8%8B%E8%BD%BDverynginx.png)

```bash
#更改端口，以免同时占用80端口
sudo vim /opt/verynginx/openresty/nginx/conf/nginx.conf
#将监听端口改为8080
```

+ 相关命令
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

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E6%88%90%E5%8A%9F%E8%AE%BF%E9%97%AEverynginx.png)
![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/verynginx%E6%97%B6%E4%BA%8B%E4%BA%A4%E4%BA%92.png)

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

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E5%88%9B%E5%BB%BAwordpress%E6%95%B0%E6%8D%AE%E5%BA%93.png)


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

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E4%B8%8B%E8%BD%BDphp.png)


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

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E4%B8%8B%E8%BD%BDwordpress.png)
![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E6%8E%88%E6%9D%83.png)


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
server_name 192.168.56.103;

#保存退出
```

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E5%A2%9E%E5%8A%A0location.png)
![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E5%A2%9E%E5%8A%A0location2.png)
![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E6%9B%B4%E6%94%B9.png)
![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E4%BF%AE%E6%94%B9%E6%A0%B9%E7%9B%AE%E5%BD%95.png)

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

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E9%AA%8C%E8%AF%81%E6%98%AF%E5%90%A6%E9%94%99%E8%AF%AF.png)

+ 重新加载nginx
```bash
sudo systemctl reload nginx
```
报错

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/fail%E4%BF%A1%E6%81%AF.png)

```bash
#查看错误原因
sudo systemctl status nginx.service
```
原来是空格害人，更正错误后重启成功

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E5%8E%BB%E6%8E%89%E7%A9%BA%E6%A0%BC.png)


5. 设置wordpress配置文件
+ 从wordpress密钥生成器中获取安全值
```bash
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```

获得如下

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E7%94%9F%E6%88%90%E7%A7%98%E9%92%A5.png)

+ 打开wordpress配置文件
```bash
sudo nano /var/www/html/wp.sec.cuc.edu.cn/wp-config.php
```

+ 将文件中相应部分替换

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E5%A4%8D%E5%88%B6%E7%A7%98%E9%92%A5.png)

+ 修改开头部分数据
```bash
define('DB_NAME', 'wordpress');

define('DB_USER', 'wordpressuser');

define('DB_PASSWORD', 'password');
···
define('FS_METHOD', 'direct');
```

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%B8%AD%E4%BF%A1%E6%81%AF.png)


6. 通过web界面完成安装
+ 在浏览器中，输入```192.168.56.103:8080```

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E6%88%90%E5%8A%9F%E8%AE%BF%E9%97%AEwordpress.png)

+ 选择语言，进行设置，完成登录

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/wp%E7%99%BB%E5%BD%95%E7%95%8C%E9%9D%A2.png)
![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E6%88%90%E5%8A%9F%E7%99%BB%E5%BD%95wp.png)


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

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E4%B8%8B%E8%BD%BDdvwa.png)


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

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E5%88%9B%E5%BB%BAdvwa%E6%95%B0%E6%8D%AE%E5%BA%93.png)


+ 修改dvwa配置文件
```bash
sudo sudo vim /var/www/html/DVWA/config/config.inc.php

#进行修改
$_DVWA[ 'db_server' ]   = '127.0.0.1';
$_DVWA[ 'db_database' ] = 'dvwa';
$_DVWA[ 'db_user' ]     = 'dvwauser';
$_DVWA[ 'db_password' ] = 'p@ssw0rd';
```

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%BF%A1%E6%81%AF.png)

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

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E7%9B%91%E5%90%AC%E7%AB%AF%E5%8F%A3%E4%BF%AE%E6%94%B9.png)
![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/dvwa%E6%A0%B9%E7%9B%AE%E5%BD%95%E4%BF%AE%E6%94%B9.png)

+ 访问dvwa

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E6%88%90%E5%8A%9F%E8%AE%BF%E9%97%AEdvwa.png)
![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E7%99%BB%E5%BD%95.png)

以admin/password登录

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/adminpassword%E7%99%BB%E5%BD%95.png)

##### 基本要求
+ 使用Wordpress搭建的站点对外提供访问的地址为： https://wp.sec.cuc.edu.cn
```bash
#修改/etc/nginx/sites-available/wp.sec.cuc.edu.cn的server_name
server_name wp.sec.cuc.edu.cn
```

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/server_name%20wp.png)

```bash
#在/etc/hosts中添加
192.168.56.103 wp.sec.cuc.edu.cn
```

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/hosts_wp.png)

```bash
#在本机C:\windows\system32\dirvers\etc\hosts中添加
192.168.56.103 wp.sec.cuc.edu.cn
```
   + 通过域名访问成功

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/wp.sec.cuc.edu.cn8080.png)

+ 使用Damn Vulnerable Web Application (DVWA)搭建的站点对外提供访问的地址为： http://dvwa.sec.cuc.edu.cn
```bash
#修改/etc/nginx/sites-available/dvwa的server_name
server_name dvwa.sec.cuc.edu.cn
```

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/server_name%20dvwa.png)

```bash
#在/etc/hosts中添加
192.168.56.103 dvwa.sec.cuc.edu.cn
```

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/hosts_dvwa.png)

```bash
#在本机C:\windows\system32\dirvers\etc\hosts中添加
192.168.56.103 dvwa.sec.cuc.edu.cn
```

![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/windows.png)

   + 通过域名访问成功
   
![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/dvwa.sec.cuc.edu.cn8088.png)

+ 反向代理
   +设置如图

   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/dvwa%E5%92%8Cwp%E5%8F%8D%E5%90%91.png)
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/wp%E5%92%8Cdvwa.png)

   + 再次尝试，不添加端口也可访问

   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E9%80%9A%E8%BF%87%E5%9F%9F%E5%90%8D%E8%AE%BF%E9%97%AEwp%E6%88%90%E5%8A%9F.png)
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E9%80%9A%E8%BF%87%E5%9F%9F%E5%90%8D%E8%AE%BF%E9%97%AEdvwa%E6%88%90%E5%8A%9F.png)


##### 安全加固要求
+ 使用IP地址方式均无法访问上述任意站点，并向访客展示自定义的友好错误提示信息页面-1
   + matcher
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E9%98%BB%E6%AD%A2ip.png)
   
   + response
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/ip%E5%9B%9E%E5%BA%94.png)
   
   + filter
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/ip%E8%BF%9E%E6%8E%A5.png)
   
   + 实验结果
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/%E4%BD%BF%E7%94%A8ip%E8%AE%BF%E9%97%AE%E9%98%BB%E6%8C%A1.png)
   
+ Damn Vulnerable Web Application (DVWA)只允许白名单上的访客来源IP，其他来源的IP访问均向访客展示自定义的友好错误提示信息页面-2
   + matcher
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/dvwa_wl-matcher.PNG)
   
   + response
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/dvwa_wl-response.PNG)
   
   + filter
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/dvwa_wl-filter.PNG)
   
   + 实验结果
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/whitelist-no%20permission.PNG)
   
+ 在不升级Wordpress版本的情况下，通过定制VeryNginx的访问控制策略规则，热修复WordPress < 4.7.1 - Username Enumeration
   + 配置访问/wp-json/wp/v2/users/可以获取wordpress用户信息的json数据，禁止访问·wp.sec.cuc.edu.cn站点的/wp-json/wp/v2/users/路径
   + matcher
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/wp_matcher.png)
   
   + response
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/wp_response.png)
   
   + filter
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/wp_filter.png)
   
   + 实验结果 访问```wp.sec.cuc.edu.cn/wp-json/wp/v2/users/```

+ 通过配置VeryNginx的Filter规则实现对Damn Vulnerable Web Application (DVWA)的SQL注入实验在低安全等级条件下进行防护
   + 将dvwa安全等级调整为low
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/dvwa%E5%AE%89%E5%85%A8%E7%AD%89%E7%BA%A7.png)
   
   + matcher
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/dvwa_sql_matcher.png)
   
   + resopnse
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/dvwa_sql_respnse.png)
   
   + filter
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/dvwa_sql_filter.png)
   
   + sql注入实验结果
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/dvwa_sql%E5%AE%9E%E9%AA%8C%E7%BB%93%E6%9E%9C.png)
   
##### VERYNGINX配置要求
+ VeryNginx的Web管理页面仅允许白名单上的访客来源IP，其他来源的IP访问均向访客展示自定义的友好错误提示信息页面-3
   + matcher
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/vn_whitelist_matcher.PNG)
   
   + reponse
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/vn_whitelist_matcher.PNG)
   
   + filter
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/vn_whitelist_filter.PNG)
   
   + 不在白名单的ip地址可以访问，在白名单的地址不可以访问
   
+ 通过定制VeryNginx的访问控制策略规则实现：
   + 限制DVWA站点的单IP访问速率为每秒请求数 < 50
   + 限制Wordpress站点的单IP访问速率为每秒请求数 < 20
   + 超过访问频率限制的请求直接返回自定义错误提示信息页面-4
   （1）frenquency limit
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/frequencylimit.png)
   
   （2）response
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/frequencylimitres.png)
   
   （3）结果
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/dvwalenonsucceed.png)
   
   （4）```sudo apt install apache2-utils```安装apache2-utils，执行```ab -c 30 -n 53 http://dvwa.sec.cuc.edu.cn/```,执行```ab -c 10 -n 23 http://wp.sec.cuc.edu.cn/```
   
   + 禁止curl访问
   （1）matcher
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/curldisabledm_matcher.png)
   
   （2）filter
   
   ![](https://github.com/CUCCS/linux-2019-laolaodeyang/blob/%E5%AE%9E%E9%AA%8C5%E2%80%94web/curldisabled_filter.png)
   
   （3）结果执行```curl -v http://dvwa.sec.cuc.edu.cn```
   
##### 参考资料
+ https://github.com/CUCCS/linux-2019-MrCuihi/blob/chap0x05/Linux%E7%B3%BB%E7%BB%9F%E4%B8%8E%E7%BD%91%E7%BB%9C%E7%AE%A1%E7%90%86/chap0x05/chap0x05%20web%20server.md
+ https://github.com/CUCCS/linux-2019-luyj/blob/Linux_exp0x05/Linux_exp0x05/Linux_exp0x05.md

   
   
   
   
