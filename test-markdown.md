コーポレートサイトサーバー(v2)構築手順
==================================================

サポートバージョン:

| 対象        | バージョン    |
| ------------- |:-------------:|
| corp webapp     | v2.0.1以上　v3.0.0未満 |

変数の定義：

    このドキュメントの参照先の変数、
        - {CORP_VERSION} には corpアプリケーションジバージョン
        - {YYYYMMDD}  には 作業当日の日付
    をそれぞれ代入します。

作成手順
---------------------------------
1. サーバーインスタン作成

    | Item        | Value           | Notes  |
    | ------------- |:-------------:| -----:|
    | AMI      | acaric-vanilla-hvm-2016.03.1-gp2-20160523T2001 |  |
    | Instance Type      | t2.small |  |
    | Subnet	 | corp-private-1c |  |
    | IAM role | ec2_corp_web | 	 |
    | Tag Name | corp-web-{CORP_VERSION}-prod |  |
    | Tag Department | corporate | 	 |
    | Tag Company | acaric | 	 |
    | Security Group | vpc-base-1 corp-app-prod|   |
    | Key pair  | acaric_20160512 |   |

1. ディスク容量、ファイルシステムの確認

    ```bash
    [root@ip-10-3-xx-xx ~]# df -h
    ```

    出力の例
    ```
    Filesystem      Size  Used Avail Use% Mounted on
	/dev/xvda1      7.8G  1.9G  5.9G  24% /
	devtmpfs        994M   56K  994M   1% /dev
	tmpfs          1002M     0 1002M   0% /dev/shm
    ```

1. スワップとメモリーの確認

    ```bash
    [root@ip-10-3-xx-xx ~]# free -m
    ```

    出力の例
    ```
                total       used       free     shared    buffers     cached
	Mem:          2003        620       1382          0         12        546
	-/+ buffers/cache:         61       1942
	Swap:          490          0        490

    ```

1. オーナーとパーミッションの確認

    ```bash
    [root@ip-10-3-xx-xx ~]# ls -l /
    ```

    出力の例
    ```
    total 108
    dr-xr-xr-x  2 root root  4096 Oct 25 11:05 bin
    dr-xr-xr-x  4 root root  4096 Oct 25 11:05 boot
    drwxr-xr-x  2 root root  4096 Mar  1  2014 cgroup
    drwxr-xr-x 16 root root  2700 Oct 25 15:43 dev
    drwxr-xr-x 81 root root  4096 Oct 25 15:43 etc
    drwxr-xr-x  5 root root  4096 Oct 25 10:56 home
    dr-xr-xr-x  7 root root  4096 Apr 30 08:58 lib
    dr-xr-xr-x 10 root root 12288 Oct 25 11:05 lib64
    drwxr-xr-x  2 root root  4096 Apr 30 08:58 local
    drwx------  2 root root 16384 Apr 30 08:58 lost+found
    drwxr-xr-x  2 root root  4096 Jan  6  2012 media
    drwxr-xr-x  2 root root  4096 Jan  6  2012 mnt
    drwxr-xr-x  4 root root  4096 Oct 25 14:03 opt
    dr-xr-xr-x 84 root root     0 Oct 25 15:42 proc
    dr-xr-x---  5 root root  4096 Oct 25 16:24 root
    drwxr-xr-x  3 root root  4096 May 12 15:51 run
    dr-xr-xr-x  2 root root 12288 Oct 25 11:05 sbin
    drwxr-xr-x  2 root root  4096 Jan  6  2012 selinux
    drwxr-xr-x  2 root root  4096 Jan  6  2012 srv
    dr-xr-xr-x 13 root root     0 Oct 25 16:03 sys
    drwxrwxrwt  3 root root  4096 Oct 25 15:43 tmp
    drwxr-xr-x 13 root root  4096 Apr 30 08:58 usr
    drwxr-xr-x 20 root root  4096 Oct 25 11:08 var
    ```

    ```bash
    [root@ip-10-3-xx-xx ~]# ls -l /tmp/
    ```

    出力の例
    ```
    -rw------- 1 root root 514850816 May 12 16:03 swap.img
    ```

1. 起動スクリプトの確認・更新

    ```bash
    [root@ip-10-3-xx-xx ~]# svn export http://vcs.acaric.com/svn/tools/aws/ec2-templ/base/init.d/acaric-init
    [root@ip-10-3-xx-xx ~]# diff -u ./acaric-init /etc/init.d/acaric-init
    [root@ip-10-3-xx-xx ~]# mv acaric-init /etc/init.d/
    [root@ip-10-3-xx-xx ~]# reboot
    ```

1. sudo ユーザー追加

    ```bash
    [root@ip-10-3-xx-xx ~]# mkdir bin
    [root@ip-10-3-xx-xx ~]# svn export http://vcs.acaric.com/svn/tools/account/setup-users.sh ./bin/
    [root@ip-10-3-xx-xx ~]# ls -l bin/setup-users.sh
    [root@ip-10-3-xx-xx ~]# setup-users.sh zhao-x nakanishi-m
    [root@ip-10-3-xx-xx ~]# ls -l /home/
    [root@ip-10-3-xx-xx ~]# visudo -f /etc/sudoers.d/acaric
    ```

    差分
    ```
    + nakanishi-m ALL = NOPASSWD: /bin/su -
    + zhao-x ALL = NOPASSWD: /bin/su -
    +
    ```

1. logwatch 通知メール送信元の確認

    ```bash
    [root@ip-10-3-xx-xx ~]# cat /etc/logwatch/conf/logwatch.conf
    ```

    出力の例
    ```
    MailFrom = logwatch corp-webapp-{CORP_VERSION}-prod <system_maintenance@acaric.co.jp>
    MailTo = system_maintenance@acaric.co.jp
    ```

1. hostnameの確認

    ```bash
    [root@ip-10-3-xx-xx ~]# cat /etc/hosts
    ```

    出力の例
    ```
    127.0.0.1   localhost localhost.localdomain        ip-10-3-xx-xxx
    ```

    ip-10-3-xx-xxxがあることを確認する。

1. 時刻確認

    ```bash
    [root@ip-10-3-xx-xx ~]# date
    ```

    出力の例
    ```
    Tue Oct 25 16:04:47 JST 2016
    ```

    日本時間であり、現在時刻があっているかを確認します。

1. yumアップデート

    ```bash
    [root@ip-10-3-xx-xx ~]# yum clean all
    [root@ip-10-3-xx-xx ~]# yum update
    ```

1. プロンプト

    ```bash
    [root@ip-10-3-xx-xx ~]# vim /etc/bashrc
    ```

    差分
    ```
    - #[ "$PS1" = "\\s-\\v\\\$ " ] && PS1="[\u@\h \W]\\$ "
    + [ "$PS1" = "\\s-\\v\\\$ " ] && PS1="[\u@corp-web-prod-${HOSTNAME#ip-10-*-} \W]\\$ "
    ```

    ```bash
    [root@ip-10-3-xx-xx ~]# reboot
    ```

1. ミドルウェアのインストール

    ```bash
    [root@corp-web-prod-xx-xx ~]# yum install httpd24 mod24_ssl
    [root@corp-web-prod-xx-xx ~]# yum install php56 php-pear php56-cli php56-common php56-gd php56-mbstring php56-mcrypt php56-mysqlnd php56-pdo php56-process php56-xml
    [root@corp-web-prod-xx-xx ~]# yum install mysql55-server
    ```

1. cronの設定

    ```bash
    [root@corp-web-prod-xx-xx ~]# cd
    [root@corp-web-prod-xx-xx ~]# vim /etc/crontab
    ```

    差分
    ```
    @@ -13,4 +13,5 @@
    # |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
    # |  |  |  |  |
    # *  *  *  *  * user-name command to be executed
    +# 22  5  *  *  * root run-parts /opt/corp-webapp/apps/app-latest/bash/cron.daily
    +# 22  1  *  *  6 root run-parts /opt/corp-webapp/apps/app-latest/bash/cron.weekly
    ```

1. apache24の設定

    ```bash
    [root@corp-web-prod-xx-xx ~]# cd /etc/httpd/conf/
    [root@corp-web-prod-xx-xx conf]# mv httpd.conf httpd.conf.orig
    [root@corp-web-prod-xx-xx conf]# svn export http://vcs.acaric.com/svn/corporate/corporate/branches/develop/2_x_x/infra/httpd24/etc/httpd/conf/httpd.conf
    [root@corp-web-prod-xx-xx conf]# ls -l

    [root@corp-web-prod-xx-xx ~]# cd
    [root@corp-web-prod-xx-xx ~]# svn export http://vcs.acaric.com/svn/corporate/corporate/branches/develop/2_x_x/infra/httpd24/etc/httpd/conf.d/
    [root@corp-web-prod-xx-xx ~]# ls -l ~/conf.d/
    [root@corp-web-prod-xx-xx ~]# cd /etc/httpd/conf.d/
    [root@corp-web-prod-xx-xx conf.d]# rename ".conf" ".conf.orig" autoindex.conf userdir.conf welcome.conf
    [root@corp-web-prod-xx-xx conf.d]# mv ~/conf.d/* ./
    [root@corp-web-prod-xx-xx conf.d]# rm -rf ~/conf.d

    [root@corp-web-prod-xx-xx ~]# cd
    [root@corp-web-prod-xx-xx ~]# svn export http://vcs.acaric.com/svn/corporate/corporate/branches/develop/2_x_x/infra/httpd24/etc/httpd/conf.modules.d/
    [root@corp-web-prod-xx-xx ~]# ls -l ~/conf.modules.d/
    [root@corp-web-prod-xx-xx ~]# cd /etc/httpd/conf.modules.d/
    [root@corp-web-prod-xx-xx conf.modules.d]# rename ".conf" ".conf.orig"  00-base.conf 00-dav.conf 00-lua.con 00-proxy.conf 01-cgi.conf
    [root@corp-web-prod-xx-xx conf.modules.d]# mv ~/conf.modules.d/* ./
    [root@corp-web-prod-xx-xx conf.modules.d]# rm -rf ~/conf.modules.d

    [root@corp-web-prod-xx-xx conf.modules.d]# chown apache. /var/log/httpd
    ```

1. mod24_sslの設定

   ```bash
    [root@corp-web-prod-xx-xx ~]# cd /etc/httpd/conf.d/
    [root@corp-web-prod-xx-xx conf.d]# mv ./ssl.conf ./ssl.conf.orig
    [root@corp-web-prod-xx-xx conf.d]# svn export http://vcs.acaric.com/svn/corporate/corporate/branches/develop/2_x_x/infra/mod24_ssl/etc/httpd/conf.d/ssl.conf
    [root@corp-web-prod-xx-xx conf.d]# ls -l
    ```

    [サーバ証明書入れ替え手順] (https://github.com/acaric-system/docs/blob/master/infrastructure/manual/tls-certificate-installation-apache24.md)

1.  mysql55-serverの設定

    ```bash
    [root@corp-web-prod-xx-xx ~]# cd
    [root@corp-web-prod-xx-xx ~]# mv /etc/my.cnf /etc/my.cnf.orig
    [root@corp-web-prod-xx-xx ~]# svn export http://vcs.acaric.com/svn/corporate/corporate/branches/develop/2_x_x/infra/mysql55-server/etc/my.cnf /etc/my.cnf

    [root@corp-web-prod-xx-xx ~]# mkdir /var/log/mysqld
    [root@corp-web-prod-xx-xx ~]# chown mysql. /var/log/mysqld
    [root@corp-web-prod-xx-xx ~]# chmod 700 /var/log/mysqld
    [root@corp-web-prod-xx-xx ~]# chkconfig mysqld on
    ```

1.  mysql55アカウントの設定

    ```bash
    [root@corp-web-prod-xx-xx ~]# service mysqld start

    [root@corp-web-prod-xx-xx ~]# mysql_secure_installation
    ```

    options
    ```
    Set root password? [Y/n] Y
    New password:
    Re-enter new password:
    ....
    Remove anonymous users? [Y/n] Y
    ....
    Disallow root login remotely? [Y/n] Y
    ....
    Remove test database and access to it? [Y/n] n
    ....
    Reload privilege tables now? [Y/n] Y
    ```

    ```bash
    [root@corp-web-prod-xx-xx ~]# mysql -uroot -p
    ```

    ```mysql
    mysql> select user, host from mysql.user;
    ```

    出力の例
    ```
    +------+-----------+
    | user | host      |
    +------+-----------+
    | root | 127.0.0.1 |
    | root | ::1       |
    | root | localhost |
    +------+-----------+
    ```

    ```mysql
    # dump user
    mysql> grant select,reload,lock tables on *.* to dump@localhost identified by '******' ;

    # reload user
    mysql> grant reload on *.* to db_reload@localhost identified by '******' ;

    mysql> create database corporate DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_bin ;
    mysql> grant all on corporate.* to corporate_admin@localhost identified by '******' ;
    mysql> SHOW GRANTS for corporate_admin@localhost;
    ```

    出力の例
    ```
    +------------------------------------------------------------------------------------------------------------------------+
    | Grants for corporate_admin@localhost                                                                                   |
    +------------------------------------------------------------------------------------------------------------------------+
    | GRANT USAGE ON *.* TO 'corporate_admin'@'localhost' IDENTIFIED BY PASSWORD '******' |
    | GRANT ALL PRIVILEGES ON `corporate`.* TO 'corporate_admin'@'localhost'                                                 |
    +------------------------------------------------------------------------------------------------------------------------+
    ```

    ```mysql
    mysql> select user, host from mysql.user;
    ```

    出力の例
    ```
    +-----------------+-----------+
    | user            | host      |
    +-----------------+-----------+
    | root            | 127.0.0.1 |
    | root            | ::1       |
    | corporate_admin | localhost |
    | db_reload       | localhost |
    | dump            | localhost |
    | root            | localhost |
    +-----------------+-----------+
    ```

    ```mysql
    mysql> exit
    ```

    ```bash
    [root@corp-web-prod-xx-xx ~]# service mysqld stop
    [root@corp-web-prod-xx-xx ~]# cd /var/log/mysqld
    [root@corp-web-prod-xx-xx mysqld]# mv mysqld.log mysqld.log-{YYYYMMDD}
    [root@corp-web-prod-xx-xx mysqld]# mv mysqld-slow-query.log mysqld-slow-query.log-{YYYYMMDD}
    ```

1. php5.6設定

    ```bash
    [root@corp-web-prod-xx-xx ~]# cd
    [root@corp-web-prod-xx-xx ~]# mv /etc/php-5.6.ini /etc/php-5.6.ini.orig
    [root@corp-web-prod-xx-xx ~]# svn export http://vcs.acaric.com/svn/corporate/corporate/branches/develop/2_x_x/infra/php56/etc/php-5.6.ini /etc/php-5.6.ini
    [root@corp-web-prod-xx-xx ~]# ls -l /etc/php-5.6.ini*

    [root@corp-web-prod-xx-xx ~]# mv /etc/httpd/conf.d/php-conf.5.6 /etc/httpd/conf.d/php-conf.5.6.orig
    [root@corp-web-prod-xx-xx ~]# svn export http://vcs.acaric.com/svn/corporate/corporate/branches/develop/2_x_x/infra/php56/etc/httpd/conf.d/php-conf.5.6 /etc/httpd/conf.d/php-conf.5.6
    [root@corp-web-prod-xx-xx ~]# ls -l /etc/httpd/conf.d/php-conf.5.6*
    ```

1. logrotate設定

    ```bash
    [root@corp-web-prod-xx-xx ~]# cd
    [root@corp-web-prod-xx-xx ~]# svn export http://vcs.acaric.com/svn/corporate/corporate/branches/develop/2_x_x/infra/logrotate/etc/logrotate.d
    [root@corp-web-prod-xx-xx ~]# ls -l ./logrotate.d/
    [root@corp-web-prod-xx-xx ~]# cd /etc/logrotate.d/
    [root@corp-web-prod-xx-xx logrotate.d]# mv ~/logrotate.d/httpd ./httpd
    [root@corp-web-prod-xx-xx logrotate.d]# mv ~/logrotate.d/mysqld ./mysqld
    [root@corp-web-prod-xx-xx logrotate.d]# rm -rf ~/logrotate.d

    [root@corp-web-prod-xx-xx logrotate.d]# vim ~/.my.cnf
    ```

    差分
    ```
    +[mysqladmin]
    +user= db_reload
    +password = ******
    ```

    ```bash
    [root@corp-web-prod-xx-xx logrotate.d]# chmod 400 ~/.my.cnf
    ```

1. ssmtp設定

    ```bash
    [root@corp-web-prod-xx-xx ~]# cd
    [root@corp-web-prod-xx-xx ~]# ls -l /etc/httpd/conf/
    [root@corp-web-prod-xx-xx ~]# svn export http://vcs.acaric.com/svn/corporate/corporate/branches/develop/2_x_x/infra/ssmtp/etc/httpd/conf/ssmtp.conf /etc/httpd/conf/ssmtp.conf
    [root@corp-web-prod-xx-xx ~]# chown apache. /etc/httpd/conf/ssmtp.conf
    [root@corp-web-prod-xx-xx ~]# chmod 400 /etc/httpd/conf/ssmtp.conf

    [root@corp-web-prod-xx-xx ~]# vim /etc/httpd/conf/ssmtp.conf
    ```

    差分
    ```
    @@ -3,5 +3,5 @@
    RewriteDomain=acaric.co.jp
    FromLineOverride=YES
    AuthUser=noreply
    -AuthPass=
    +AuthPass= ******
    AuthMethod=cram-md5

    ```

1. logwatch設定

    ```bash
    [root@corp-web-prod-xx-xx ~]# cd
    [root@corp-web-prod-xx-xx ~]# svn export http://vcs.acaric.com/svn/corporate/corporate/branches/develop/2_x_x/infra/logwatch/etc/logwatch/
    [root@corp-web-prod-xx-xx ~]# ls -l logwatch
    [root@corp-web-prod-xx-xx ~]# cd /etc/logwatch/conf/services/
    [root@corp-web-prod-xx-xx ~]# mv ~/logwatch/conf/services/* ./
    [root@corp-web-prod-xx-xx ~]# cd /etc/logwatch/conf/logfiles/
    [root@corp-web-prod-xx-xx ~]# mv ~/logwatch/conf/logfiles/* ./
    [root@corp-web-prod-xx-xx ~]# ls -l

    [root@corp-web-prod-xx-xx ~]# cd /etc/logwatch/scripts/services/
    [root@corp-web-prod-xx-xx ~]# mv ~/logwatch/scripts/services/* ./
    [root@corp-web-prod-xx-xx ~]# ls -l
    ```

1. アプリケーション用ディレクトリ作成

    ```bash
    [root@corp-web-prod-xx-xx ~]# mkdir -p /opt/corp-webapp/assets
    [root@corp-web-prod-xx-xx ~]# mkdir -p /opt/corp-webapp/apps
    [root@corp-web-prod-xx-xx ~]# mkdir -p /opt/corp-webapp/apps/corp-dummy/etc/httpd/
    ```

1. apache設定確認

    ```bash
    [root@corp-web-prod-xx-xx ~]# ln -sn /opt/corp-webapp/apps/corp-dummy /opt/corp-webapp/apps/app-latest
    [root@corp-web-prod-xx-xx ~]# service httpd configtest
    [root@corp-web-prod-xx-xx ~]# service httpd start
    [root@corp-web-prod-xx-xx ~]# openssl s_client -connect {10.3.xx.xx}:443 < /dev/null | less
    [root@corp-web-prod-xx-xx ~]# service httpd stop
    [root@corp-web-prod-xx-xx ~]# cd /var/log/httpd
    [root@corp-web-prod-xx-xx httpd]# rm *
    ```
