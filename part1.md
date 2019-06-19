###Linux setup

```
$ sudo useradd training
$ sudo passwd training
passwd: all authentication tokens updated successfully.
$ sudo groupadd skcc
$ sudo usermod -a -G skcc training
$ sudo visudo
```
-sudo capability 주는 것, ip list, lunux release, fs capacity list, yum output, /etc/ list, getent output 은 모두 image에 별도 첨부
![allow group sudo](https://user-images.githubusercontent.com/48976454/59750017-1d518e80-92b9-11e9-984d-12e8792f0429.png)
![show ip](https://user-images.githubusercontent.com/48976454/59750081-2f333180-92b9-11e9-80d3-8a0bda422c81.png)
![yum](https://user-images.githubusercontent.com/48976454/59750090-31958b80-92b9-11e9-83df-d81c914bb476.png)
![show file](https://user-images.githubusercontent.com/48976454/59750057-29d5e700-92b9-11e9-9977-54c31bfc71c9.png)
![yum](https://user-images.githubusercontent.com/48976454/59750090-31958b80-92b9-11e9-83df-d81c914bb476.png)
![getent](https://user-images.githubusercontent.com/48976454/59750041-24789c80-92b9-11e9-918f-3e8d33746eae.png)

###Install a MySQL server

```
$ sudo yum install -y mariadb-server
```

### System Configuration Checks

  - Run **_yum update_** and install **wget**
```
$ sudo yum update -y
$ sudo yum install -y wget
```

  - Create a user '**training**' with the gid '**wheel**' and set the password
```Bash
$ sudo useradd training -g wheel
$ sudo passwd training
```


  - Set **NOPASSWD** option for the group '**wheel**'
```Bash
$ sudo visudo
$ sudo cat /etc/sudoers | grep wheel
## Allows people in group wheel to run all commands
#%wheel ALL=(ALL)       ALL
%wheel  ALL=(ALL)       NOPASSWD: ALL
```


  - Generate RSA private/public key for the user 'training'


  - Append the public key to '~training/.ssh/authorized_keys'
```Bash
$ su - training
Password:
$ mkdir .ssh
$ vi .ssh/authorized_keys
$ chmod 600 .ssh/authorized_keys
$ cat .ssh/authorized_keys
```


  - Append **OpenSSH formatted** private key to '~training/.ssh/id_rsa'
```Bash
$ vi .ssh/id_rsa
$ chmod 600 .ssh/id_rsa
$ cat .ssh/id_rsa
```


  1. Check vm.swappiness on all your nodes
  - Set the value to 1 if necessary
```Bash
$ cat /proc/sys/vm/swappiness
30
$ sudo sysctl -w vm.swappiness=1
vm.swappiness = 1
$ cat /proc/sys/vm/swappiness
1
```
```Bash
$ echo vm.swappiness = 1 | sudo tee -a /etc/sysctl.conf
$ cat /etc/sysctl.conf
(중략)
vm.swappiness = 1
```

```Bash
$ cat /proc/mounts
```


  3. If you have ext-based volumes, list the reserve space setting
  - XFS volumes do not support reserve space **_=> No ext-based volumes found_**
```Bash
$ cat /etc/fstab
(중략)
UUID=f41e390f-835b-4223-a9bb-9b45984ddf8d /                       xfs     defaults        0 0
```


  4. Disable transparent hugepage support
  - Check [disable-transparent-hugepages](files/disable-transparent-hugepages)
  - Check [tuned.conf](files/tuned.conf)
```Bash
$ sudo vi /etc/init.d/disable-transparent-hugepages
$ sudo chmod 755 /etc/init.d/disable-transparent-hugepages
$ sudo chkconfig --add disable-transparent-hugepages
$ sudo mkdir /etc/tuned/no-thp
$ sudo vi /etc/tuned/no-thp/tuned.conf
$ sudo tuned-adm profile no-thp
```

```Bash
$ cat /sys/kernel/mm/transparent_hugepage/enabled
always madvise [never]
$ cat /sys/kernel/mm/transparent_hugepage/defrag
always madvise [never]
```


  5. List your network interface configuration
```Bash
$ ifconfig -a
```


  6. Show that forward and reverse host lookups are correctly resolved
  - For /etc/hosts, use getent
  - For DNS, use nslookup
```Bash
$ sudo vi /etc/hosts
$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
# AWS servers
172.31.2.32     cm1
172.31.3.52     m1
172.31.12.190   w1
172.31.7.57     w2
172.31.1.53     w3
```

```Bash
$ sudo hostnamectl set-hostname m1
$ sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
$ sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1
```

```Bash
$ echo net.ipv6.conf.all.disable_ipv6 = 1 | sudo tee -a /etc/sysctl.conf
$ echo net.ipv6.conf.default.disable_ipv6 = 1 | sudo tee -a /etc/sysctl.conf
$ cat /etc/sysctl.conf | grep net.ipv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
```

```Bash
$ getent hosts cm1
172.31.2.32     cm1
$ getent hosts 172.31.2.32
172.31.2.32     cm1
```


  7. Show the nscd service is running
```Bash
$ sudo systemctl status nscd
Unit nscd.service could not be found.
$ sudo yum install -y nscd
$ sudo systemctl start nscd
$ sudo systemctl status nscd
$ sudo systemctl enable nscd
```


  8. Show the ntpd service is running
```Bash
$ sudo systemctl status ntp
Unit ntp.service could not be found.
$ sudo yum install -y ntp
$ sudo systemctl start ntpd
$ sudo systemctl status ntpd
$ sudo systemctl disable chronyd
Removed symlink /etc/systemd/system/multi-user.target.wants/chronyd.service.
$ sudo systemctl enable ntpd
$ ntpq -p
```

### Cloudera Manager Install Lab
#### Path B install using CM 5.15.x
[The full rundown](https://www.cloudera.com/documentation/enterprise/5-15-x/topics/install_cm_cdh.html) is here. You will have to modify your package repo to get the right release. The default repo download always points to the latest version.

Use the documentation to complete the following objectives:

  - Import Cloudera manager repository on **_cm1_**
```Bash
$ sudo wget https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo -P /etc/yum.repos.d/
$ sudo sed -i 's/x86_64\/cm\/5/x86_64\/cm\/5.15.2/g' /etc/yum.repos.d/cloudera-manager.repo
$ sudo rpm --import https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPM-GPG-KEY-cloudera
```

  - **Install a supported Oracle JDK on your first node**
```Bash
$ sudo yum install -y oracle-j2sdk1.7
```
  - **Install a supported JDBC connector on all nodes**
  1. Install from the node _**cm1**_
```Bash
$ sudo wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.46.tar.gz ~/
$ tar xvfz ~/mysql-connector-java-5.1.46.tar.gz
$ sudo mkdir -p /usr/share/java
$ sudo cp ~/mysql-connector-java-5.1.46/mysql-connector-java-5.1.46-bin.jar /usr/share/java/mysql-connector-java.jar
```
  2. Copy to all other nodes
```Bash
$ scp /usr/share/java/mysql-connector-java.jar m1:~/
$ scp /usr/share/java/mysql-connector-java.jar w1:~/
$ scp /usr/share/java/mysql-connector-java.jar w2:~/
$ scp /usr/share/java/mysql-connector-java.jar w3:~/
$ ssh m1 "sudo mkdir -p /usr/share/java; sudo mv ~/mysql-connector-java.jar /usr/share/java/"
$ ssh w1 "sudo mkdir -p /usr/share/java; sudo mv ~/mysql-connector-java.jar /usr/share/java/"
$ ssh w2 "sudo mkdir -p /usr/share/java; sudo mv ~/mysql-connector-java.jar /usr/share/java/"
$ ssh w3 "sudo mkdir -p /usr/share/java; sudo mv ~/mysql-connector-java.jar /usr/share/java/"
```
  - **Create the databases and access grants you will need**
  1. Install **MariaDB**
```
$ sudo yum install -y mariadb-server
```
  2. Enable and run MariaDB
```
$ sudo systemctl enable mariadb
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.
$ sudo systemctl start mariadb
```
  3. Run **/usr/bin/mysql_secure_installation** for security
```
$ sudo /usr/bin/mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] Y
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] Y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] N
 ... skipping.

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!

```
  4. Create Databases and users for Cloudera manager and eco systems
```Bash
$ mysql -u root -p
CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci; GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY 'training';
CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci; GRANT ALL ON amon.* TO 'amon'@'%' IDENTIFIED BY 'training';
CREATE DATABASE rmon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci; GRANT ALL ON rmon.* TO 'rmon'@'%' IDENTIFIED BY 'training';
CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci; GRANT ALL ON hue.* TO 'hue'@'%' IDENTIFIED BY 'training';
CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci; GRANT ALL ON metastore.* TO 'metastore'@'%' IDENTIFIED BY 'training';
CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci; GRANT ALL ON oozie.* TO 'oozie'@'%' IDENTIFIED BY 'training';
FLUSH PRIVILEGES;
EXIT;
```
  - **Configure Cloudera Manager to connect to the database**
  1. Install **cloudera-manager-daemons** **cloudera-manager-server**
```Bash
$ sudo yum install -y cloudera-manager-daemons cloudera-manager-server
```
  2. Run initialize script for SCM
```Bash
$ sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql scm scm training
JAVA_HOME=/usr/java/jdk1.7.0_67-cloudera
Verifying that we can write to /etc/cloudera-scm-server
Creating SCM configuration file in /etc/cloudera-scm-server
Executing:  /usr/java/jdk1.7.0_67-cloudera/bin/java -cp /usr/share/java/mysql-connector-java.jar:/usr/share/java/oracle-connector-java.jar:/usr/share/java/postgresql-connector-java.jar:/usr/share/cmf/schema/../lib/* com.cloudera.enterprise.dbutil.DbCommandExecutor /etc/cloudera-scm-server/db.properties com.cloudera.cmf.db.
[                          main] DbCommandExecutor              INFO  Successfully connected to database.
All done, your SCM database is configured correctly!
```
  - **Start your Cloudera Manager server -- debug as necessary**
```Bash
$ sudo systemctl start cloudera-scm-server
$ sudo tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
```
  - **Do not continue until you can browse your CM instance at port 7180**
  ### Cloudera Manager Install Lab
  #### Install a cluster and deploy CDH
  Adhere to the following requirements while creating your cluster:
    - Do not use Single User Mode. Do not. Don't do it.
    - Ignore any steps in the CM wizard that are marked (Optional)
    - Install the Data Hub Edition
    - Install CDH using parcels
    - Deploy only the Core set of CDH services.
    - Deploy three ZooKeeper instances.
      - CM does not tell you to do this but complains if you don't

  1. Open Cloudera Manager Console from a browser
  -이미지별도첨부(install-cdh-001.png)
  ![install-cdh-001](https://user-images.githubusercontent.com/48976454/59750619-0a8b8980-92ba-11e9-8b7a-5ddf76d91455.png)

  2. Put target nodes' FQDN
  -이미지별도첨부(install-cdh-002.png)
![install-cdh-002](https://user-images.githubusercontent.com/48976454/59750620-0a8b8980-92ba-11e9-8f12-c5e5d4bcd8f3.png)

  3. Select nodes on the list
  -이미지별도첨부(install-cdh-003.png)
![install-cdh-003](https://user-images.githubusercontent.com/48976454/59750621-0a8b8980-92ba-11e9-83f1-97a5be202fd3.png)

  4. Put account information for agent installation
  -이미지별도첨부(install-cdh-012.png)
![install-cdh-012](https://user-images.githubusercontent.com/48976454/59750618-09f2f300-92ba-11e9-91b6-dff5c4c51658.png)

  5. Check the installation steps on each nodes
  -이미지별도첨부(install-cdh-004.png)
![install-cdh-004](https://user-images.githubusercontent.com/48976454/59750625-0b242000-92ba-11e9-8527-ec70263c15b4.png)

  6. Set roles on each nodes [(Reference Link)](https://www.cloudera.com/documentation/enterprise/5-8-x/topics/cm_ig_host_allocations.html)
  -이미지별도첨부(install-cdh-005.png)
![install-cdh-005](https://user-images.githubusercontent.com/48976454/59750626-0b242000-92ba-11e9-8eeb-a7f6290d4918.png)

  7. Check roles on each nodes
  -이미지별도첨부(install-cdh-006.png)
![install-cdh-006](https://user-images.githubusercontent.com/48976454/59750627-0b242000-92ba-11e9-8385-a688183bb3e2.png)

  8. Set Database information for each services
  -이미지별도첨부(install-cdh-007.png)
![install-cdh-007](https://user-images.githubusercontent.com/48976454/59750629-0c554d00-92ba-11e9-8a5d-b6d515083c26.png)

  9. Check cluster setting
  -이미지별도첨부(install-cdh-008.png)
![install-cdh-008](https://user-images.githubusercontent.com/48976454/59750631-0c554d00-92ba-11e9-97a3-8e4b7a05d672.png)

  10. Check cluster setting steps on each services
  -이미지별도첨부(install-cdh-009.png)
![install-cdh-009](https://user-images.githubusercontent.com/48976454/59750633-0cede380-92ba-11e9-980b-d12cba3b4438.png)

  11. Final message of cluster installation
  -이미지별도첨부(install-cdh-010.png)
![install-cdh-010](https://user-images.githubusercontent.com/48976454/59750634-0cede380-92ba-11e9-9256-6f90ea6305e4.png)

  12. Cloudera Manager main page
  -이미지별도첨부(install-cdh-011.png)
![install-cdh-011](https://user-images.githubusercontent.com/48976454/59750617-09f2f300-92ba-11e9-8bff-d8ad2153a7d0.png)
