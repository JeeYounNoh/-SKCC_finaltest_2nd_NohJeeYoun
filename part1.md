##create a CDH Cluster on AWS

###Linux setup

```
$ sudo useradd training
$ sudo passwd training
passwd: all authentication tokens updated successfully.
$ sudo groupadd skcc
$ sudo usermod -a -G skcc training
$ sudo visudo
```
![image](screenshot/Image 2.png)
![image](screenshot/Image 4.png)
![image](screenshot/Image 5.png)




###Install a MySQL server

```
$ sudo yum install -y mariadb-server
