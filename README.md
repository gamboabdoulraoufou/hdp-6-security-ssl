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

Files needed:  
- None  
Files created:  
- ca.key  
- ca.cert  
Files updated:  
- None  
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

> 1-2- Create key and keystore for each node in your cluster    

Files needed:  
- None  
Files created:    
- keystore.jks  
- hdp-1.cert-req.csr
- hdp-2.cert-req.csr
- hdp-3.cert-req.csr  
Files updated:  
- keystore.jks  

```sh
# generate the key and certificate for a component process
keytool -keystore keystore.jks -alias hdp-1 -validity 730 -genkey
keytool -keystore keystore.jks -alias hdp-2 -validity 730 -genkey
keytool -keystore keystore.jks -alias hdp-3 -validity 730 -genkey

# add the generated CA to the server's truststore
keytool -keystore keystore.jks -alias hdp-1 -certreq -file hdp-1-cert-req.csr
keytool -keystore keystore.jks -alias hdp-1 -certreq -file hdp-2-cert-req.csr
keytool -keystore keystore.jks -alias hdp-1 -certreq -file hdp-3-cert-req.csr
```

> 1-3- Create a signed certificate using the client CSR file

Files needed:  
- hdp-1-cert-req.csr (created above)
- hdp-2-cert-req.csr (created above)
- hdp-3-cert-req.csr (created above)
- ca.key (created above)
- ca.cert (created above)
Files created:  
- hdp-1-signed-cert.crt
- hdp-1-signed-cert.crt
- hdp-1-signed-cert.crt
Files updated:  
- None

```sh
openssl x509 -req -CA ca.cert -CAkey ca.key -in hdp-1-cert-req.csr -out hdp-1-signed-cert.crt -d
openssl x509 -req -CA ca.cert -CAkey ca.key -in hdp-2-cert-req.csr -out hdp-2-signed-cert.crt -d
openssl x509 -req -CA ca.cert -CAkey ca.key -in hdp-3-cert-req.csr -out hdp-3-signed-cert.crt -d
```

> 1-4- Acting as the client requesting a signed certificate, import the certificate signed by Signing Authority above into your keystore  

Files needed:  
- ca.cert (created above)
- hdp-1-signed-cert.crt (created above)
- hdp-2-signed-cert.crt (created above)
- hdp-3-signed-cert.crt (created above)
Files created:  
- None
Files updated:  
- keystore.jks

```sh
# Import the trusted CA certificate ca.cert that was created above into the keystore used on each node.  
# This allows any additional certificates to be added to this keystore without confirmation, since they will from that time on be trusted.
keytool -keystore keystore.jks -alias CARoot -import -file ca.cert

# Import the certificate that was created for the key used on each other node
keytool -keystore keystore.jks -alias node-1 -import -file node-1-signed-cert.crt

# Confirm that the CA and signed certificate are in your keystore
keytool -list -v -keystore keystore.jks

```

> Setup

```sh
# set up the CA directory structure
mkdir -m 0700 /root/CA /root/CA/certs /root/CA/crl /root/CA/newcerts /root/CA/private

# Move the CA key to  /root/CA/private  and the CA certificate to  /root/CA/certs
mv ca.key /root/CA/private;mv ca.crt /root/CA/certs

# Add required files:
touch /root/CA/index.txt; echo 1000 >> /root/CA/serial 

# Set permissions on the ca.key
chmod 0400 /root/ca/private/ca.key

# Open the OpenSSL configuration file
vi /etc/pki/tls/openssl.cnf

# Change the directory paths to match your environment
## START ##
## [ CA_default ]
## 
## dir             = /root/CA                  # Where everything is kept
## certs           = /root/CA/certs            # Where the issued certs are kept
## crl_dir         = /root/CA/crl              # Where the issued crl are kept
## database        = /root/CA/index.txt        # database index file.
## #unique_subject = no                        # Set to 'no' to allow creation of
##                                             # several certificates with same subject.
## new_certs_dir   = /root/CA/newcerts         # default place for new certs.
## 
## certificate     = /root/CA/cacert.pem       # The CA certificate
## serial          = /root/CA/serial           # The current serial number
## crlnumber       = /root/CA/crlnumber        # the current crl number
##                                             # must be commented out to leave a V1 CRL
## crl             = $dir/crl.pem               # The current CRL
## private_key     = /root/CA/private/cakey.pem # The private key
## RANDFILE        = /root/CA/private/.rand     # private random number file
## 
## x509_extensions = usr_cert              # The extensions to add to the cert
##
## END ##
# Save the changes and restart OpenSSL

```

> Set Up SSL for Ambari

> Set Up Truststore for Ambari Server

> Enable SSL for WebHDFS, MapReduce Shuffle, Tez, and YARN

> Configure SSL for Hue

> Enable SSL on HiveServer2

> Enable SSL for HttpFS

> Enable SSL on Oozie
