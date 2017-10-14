### hdp-6-security-ssl


> Configuration
- 4 VMs on google compute Engine (3 VM for Hadoop Cluster and 1 VM for data backup and micro services)
- OS: CentOS 7
- RAM: 15 Go
- CPU: 4
- Boot disk: 200Go
- Attached disk: 2000G

> Cluster model

![MetaStore remote database](https://github.com/gamboabdoulraoufou/hdp-1-host-config/blob/master/img/archi_v2.png)

> Overview 

This section contains the following topics:

- Creating an internal CA (OpenSSL)
- Installing Certificates in the Hadoop SSL Keystore Factory (HDFS, MapReduce, and YARN)
- Using an internal CA (OpenSSL)


> Set Up SSL for Ambari

> Set Up Truststore for Ambari Server

> Enable SSL for WebHDFS, MapReduce Shuffle, Tez, and YARN

> Configure SSL for Hue

> Enable SSL on HiveServer2

> Enable SSL for HttpFS

> Enable SSL on Oozie
