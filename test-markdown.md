test
===========================


test
---------------------

1. cronの設定

    ```bash
    [root@corp-web-prod-xx-xx ~]# cd
    [root@corp-web-prod-xx-xx ~]# vim /etc/crontab

    @@ -13,4 +13,5 @@
    # |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
    # |  |  |  |  |
    # *  *  *  *  * user-name command to be executed
    +# 22  5  *  *  * root run-parts /opt/corp-webapp/apps/app-latest/bash/cron.daily
    +# 22  1  *  *  6 root run-parts /opt/corp-webapp/apps/app-latest/bash/cron.weekly


1. apacheの設定

    ```bash
    [root@corp-web-prod-xx-xx ~]# cd
    [root@corp-web-prod-xx-xx ~]# svn export http://vcs.acaric.com/svn/corporate/corporate/branches/develop/2_x_x/infra/httpd24/etc/httpd/conf/httpd.conf
    [root@corp-web-prod-xx-xx ~]# cd /etc/httpd/conf/
    [root@corp-web-prod-xx-xx conf]# mv httpd.conf httpd.conf.orig
    [root@corp-web-prod-xx-xx conf]# mv ~/httpd.conf ./httpd.conf
    ```
