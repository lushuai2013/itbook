# MySQL重设root密码

参考；https://segmentfault.com/a/1190000000412194
幸运地是，重设密码很容易。

注意：MySQL的root用户和服务器操作系统的root用户是两个不同的用户，不要搞混了。

基本的思路是，以安全模式启动mysql，这样不需要密码可以直接以root身份登录，然后重设密码。

首先，我们停掉MySQL服务：

sudo service mysql stop
以上命令适用于Ubuntu和Debian。CentOS、Fedora和RHEL下使用mysqld替换mysql。

以安全模式启动mysql：

sudo mysqld_safe --skip-grant-tables --skip-networking &
注意我们加了--skip-networking，避免远程无密码登录MySQL。（感谢 RobberPhex指出。）

这样我们就可以直接用root登录，无需密码：

mysql -u root
接着重设密码：

mysql> use mysql;
mysql> update user set password=PASSWORD("mynewpassword") where User='root';
mysql> flush privileges;
注意，命令后需要加分号。

重设完毕后，我们退出，然后启动mysql服务：

mysql > quit
quit不需要分号。

重启服务：

sudo service mysql restart
同样，以上命令适用于Ubuntu和Debian，Centos、Fedora和RHEL需要用mysqld替换mysql。

现在可以尝试用新密码登录了：

mysql -u root -pmynewpassword
注意，-p和密码间不能有空格。

其他方案