# Cloudera Manager 安装

* 官方教程
https://www.cloudera.com/documentation/enterprise/latest/topics/install_software_cm_wizard.html

* 下载tarball 安装文件（文件较大 1.3G）
https://archive.cloudera.com/cm6/6.3.0/epo-as-tarball/cm6.3.0-redhat6.tar.gz

* 此处可根据不同的操作系统来选择不同的包
https://archive.cloudera.com/cm6/6.3.0/repo-as-tarball/

## 安装MySQL

~~~shell
wget http://repo.mysql.com/mysql-community-release-el6-5.noarch.rpm

rpm -ivh mysql-community-release-el7-5.noarch.rpm

yum install mysql-server

service mysqld start
~~~

~~~shell
/usr/bin/mysql_secure_installation
~~~

### 安装MySQL驱动
~~~shell
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.46.tar.gz
tar xf mysql-connector-java-5.1.46.tar.gz

mkdir -p /usr/share/java/
cd mysql-connector-java-5.1.46
cp mysql-connector-java-5.1.46-bin.jar /usr/share/java/mysql-connector-java.jar
~~~

### 建库与用户
~~~sql
CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY 'wzlinux';
CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON amon.* TO 'amon'@'%' IDENTIFIED BY 'wzlinux';
CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON rman.* TO 'rman'@'%' IDENTIFIED BY 'wzlinux';
CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON hue.* TO 'hue'@'%' IDENTIFIED BY 'wzlinux';
CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON metastore.* TO 'hive'@'%' IDENTIFIED BY 'wzlinux';
CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON sentry.* TO 'sentry'@'%' IDENTIFIED BY 'wzlinux';
CREATE DATABASE nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON nav.* TO 'nav'@'%' IDENTIFIED BY 'wzlinux';
CREATE DATABASE navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON navms.* TO 'navms'@'%' IDENTIFIED BY 'wzlinux';
CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON oozie.* TO 'oozie'@'%' IDENTIFIED BY 'wzlinux';

GRANT ALL ON scm.* TO 'scm'@'localhost' IDENTIFIED BY 'wzlinux';
~~~

## 启动cloudera manager

~~~shell
service cloudera-scm-server start

tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
~~~

### 访问

~~~properties
http://localhost:7180
admin / admin
~~~
