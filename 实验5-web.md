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
一、安装nginx
1.从源直接安装nginx
```bash sudo apt-get update
sudo apt-get install nginx
```
2.调整防火墙，以免出现问题
```bash sudo ufw app list
```
获得可用应用程序：
