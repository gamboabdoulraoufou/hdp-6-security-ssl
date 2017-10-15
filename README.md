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


> 1- Create and Set Up an Internal CA (OpenSSL)
> 1-1- Create a signing CA Trust key and certificate _master node: hdp-1_  
Create a:
- Self-Signed Signing CA certificate file called ca.cert 
- and an associated private key file called ca.key
The certificate will have an expiration date 10 years (3650 days) into the future. Keep these files in this secure directory as they are the basis of the trust structure.

```sh
# log as root 
sudo su - root

# create certificte folder
mkdir SelfSignCA
chmod 700 SelfSignCA
cd SelfSignCA

# install open SSL
yum install openssl

# create the key and certificate that will be used to sign all server keys used on your development and test environments
openssl req -new -x509 -keyout ca-key -out ca-cert -days 3650

```

> 1-2- Create client key
Acting as the client requesting a signed certificate, create your key and keystore for each node in your cluster

```sh
# generate the key and certificate for a component process
keytool -keystore keystore-1.jks -alias hdp-1 -validity 730 -genkey
keytool -keystore keystore-2.jks -alias hdp-2 -validity 730 -genkey
keytool -keystore keystore-3.jks -alias hdp-3 -validity 730 -genkey

# add the generated CA to the server's truststore
keytool -keystore server.truststore.jks -alias CARoot -import -file ca-cert

# add the generated CA to the client's truststore, so that clients know that they can trust this CA
keytool -keystore client.truststore.jks -alias CARoot -import -file ca-cert




> Set Up SSL for Ambari

> Set Up Truststore for Ambari Server

> Enable SSL for WebHDFS, MapReduce Shuffle, Tez, and YARN

> Configure SSL for Hue

> Enable SSL on HiveServer2

> Enable SSL for HttpFS

> Enable SSL on Oozie
