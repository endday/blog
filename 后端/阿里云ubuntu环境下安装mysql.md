---
title: 阿里云ubuntu环境下安装mysql
url: https://www.yuque.com/endday/blog/hyh23r
---

<a name="424a2ad8"></a>

## 准备

登录远程服务器，Ubuntu环境

<a name="9d7a175c"></a>

## 更新apt源

`apt-get update`
`apt-get upgrade`

<a name="511fea56"></a>

## apt操作

删除软件及其配置文件
`apt-get --purge remove <包名>`

删除没用的依赖包
`apt-get autoremove <包名>`

列出已安装的含有该名称的包
`dpkg --get-selections | grep <包名>`

<a name="54ab1937"></a>

## 安装mysql

`apt-get install mysql-server`

这个包已经包含了mysql-common，mysql-client，mysql-server

mysql mysql 客户端
mysql-devel 开发用到的库以及包含文件
mysql-server 数据库服务器

<a name="fda5bd07"></a>

## 检查mysql

`netstat -tap | grep mysql`
检查是否安装成功

<a name="5763bd17"></a>

## mysql基本命令

启动：`service mysql start`
停止：`service mysql stop`
重启：`service mysql restart`
查看状态：`service mysql status`
查看状态：`systemctl status mysql.service`

我看很多教程基于centos，和ubuntu不同，大部分教程都是service mysqld start
我这里尝试了，用mysql才行，不是mysqld，应该是版本问题

<a name="a98022dd"></a>

## 登录root

`mysql -u root -p`
\-u 表示选择登陆的用户名， -p 表示登陆的用户密码，上面命令输入之后会提示输入密码，此时输入密码就可以登录到mysql

<a name="e3f00bf5"></a>

## 查看当前数据库

通过 show databases; 就可以查看当前的数据库
记得要加 ; 分号结束语句
如果你打命令时，回车后发现忘记加分号，你无须重打一遍命令，只要打个分号回车就可以了。
也就是说你可以把一个完整的命令分成几行来打，完后用分号作结束标志就OK

<a name="ac5b586c"></a>

## 创建用户

`CREATE USER 'user_name'@'host' IDENTIFIED BY 'password'；`

user\_name：要创建用户的名字。

host：表示要这个新创建的用户允许从哪台机登陆，如果只允许从本机登陆，则 填　‘localhost’ ，如果允许从远程登陆，则填 ‘%’

password：新创建用户的登陆数据库密码，如果没密码可以不写。

例子：`CREATE USER 'endday'@'%' IDENTIFIED BY '123456'；`
创建一个名称为endday的用户，允许远程登录，密码为123456

<a name="e2143a89"></a>

## 远程链接数据库

我用的是navicat
1、用ssh连接登录服务器，在本地登录数据库

![](..\assets\hyh23r\44840452-a557a080-ac73-11e8-870f-42a059528786.png)

主机如上，localhost，端口为mysql默认的3306，帐号为你数据库的帐号

![](..\assets\hyh23r\44840508-c0c2ab80-ac73-11e8-9c50-4583e7d7062a.png)

这里就按照正常ssh连接的方式，主机填写远程服务器的公网地址，用户名是远程服务器帐号

2、直接通过远程登录服务器

![](..\assets\hyh23r\44840550-d637d580-ac73-11e8-833c-b0490e607811.png)

主机填写远程服务器的公网地址，用户名是刚刚在数据库新建的允许远程连接的帐号
或者你自己将已有的帐号的权限改为允许远程连接

如果你遇到这个提示，2003 – Can't connect to MySQL server on ‘ubuntu'(10061)
那是因为mysql默认绑定了本地地址
编辑这个配置文件 /etc/mysql/mysql.conf.d/mysqld.cnf

`# bind-address = 127.0.0.1`
注释掉这行配置

重启mysql
`service mysql restart`
