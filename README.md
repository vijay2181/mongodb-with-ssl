# Local Mongodb setup with SSL

MongoDB is an open source NoSQL database that uses a document-oriented data model. MongoDB is built on architecture of collection and document, whereas MySQL uses tables and rows. MongoDB uses JSON-like documents with schemas.

Key features of MongoDB:

    High performance
    Rich query language
    High availability
    Horizontal scalability
    Support for multiple storage engines

## Installing Mongodb

Login into AmazonLinux-2 Machine in AWS  

If we want install MongoDB directly using yum, then we have to add MongoDB repository. It is recommended to follow the installation steps as a root user
```bash
sudo -i 
```
```bash
yum update -y
```
Create a  /etc/yum.repos.d/mongodb-org-3.4.repo
```bash
vim /etc/yum.repos.d/mongodb-org-3.4.repo
```
Add the following lines in the repository file [mongodb-org-3.4]
```bash
[mongodb-org-3.4]

name=MongoDB Repository

baseurl=https://repo.mongodb.org/yum/amazon/2013.03/mongodb-org/3.4/x86_64/

gpgcheck=1

enabled=1

gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc

```
Use the following command to install MongoDB
```bash
yum install -y mongodb-org
```
## Start MongoDB
```bash
service mongod start
```
## Verify if MongoDB has started

You can verify if MongoDB process has started or not by checking the content of MongoDB log file which is located at 
 /var/log/mongodb/mongod.log

You should see following line in log file:

[initandlisten] waiting for connections on port 27017

Where 27017 is default port. You can configure another port in configuration file located at /etc/mongod.conf

If you want to start the MongoDB after every system reboot, you can use following command to do the same
```bash
/sbin/chkconfig mongod on
```
```bash
cat /var/log/mongodb/mongod.log
```
## For Stopping MongoDB
```bash
service mongod stop
```
## For Restarting MongoDB
```bash
service mongod restart
```
By Default MongoDB stores its data file in /var/lib/mongo 
and its log file in /var/log/mongoDB

We can specify alternate path for data and log directory by modifying /etc/mongod.conf

## check MongoDB service status
![image](https://user-images.githubusercontent.com/66196388/184007361-e08d7faa-42f4-4aa0-a94c-4f130576f6e7.png)

Enter 'mongo' on CLI to get into mongodb shell

![image](https://user-images.githubusercontent.com/66196388/184007854-6e756b31-ace4-4adb-a297-9eb4e749d6b2.png)

>use admin

>db.createUser(
      {
          user: "vijayDB",
          pwd: "vijayDB",
          roles: [ "root" ]
      }
  )


![image](https://user-images.githubusercontent.com/66196388/184008417-b90a6f70-5c02-4648-b2e0-62e354d5e50b.png)

>exit

```bash
mongo -u vijayDB -p --authenticationDatabase "admin" 

```
 -- asks password in commandline

>show dbs

![image](https://user-images.githubusercontent.com/66196388/184008968-65d1c7ef-b68d-432d-8e51-3cac89c5dc0e.png)


>Now we connected to Local Mongodb directly without Security, so we need to Connect it Securely

## Secure your MongoDB connections - SSL/TLS

```bash
sudo -i
cd /opt
mkdir mongodb
cd mongodb
mkdir ssl

```
```bash
openssl genrsa -out ca.key 2048
```
```bash
openssl req -x509 -new -nodes -key ca.key -sha256 -days 1024 -out ca.pem
```
 Common Name (e.g. server FQDN or YOUR name) []:127.0.0.1
```bash
openssl genrsa -out mongodb.key 2048
```
```bash
openssl req -new -key mongodb.key -out mongodb.csr -subj "/CN=127.0.0.1"
```
```bash
openssl x509 -req -in mongodb.csr -CA ca.pem -CAkey ca.key -CAcreateserial -out mongodb.crt -days 500 -sha256
```
```bash
cat mongodb.key mongodb.crt > mongodb.pem
```
```bash
openssl verify -CAfile ca.pem mongodb.pem
```

>open /etc/mongod.conf file

```bash
vi /etc/mongod.conf
```
>Add below Lines where Necessary
```bash
net:
  port: 27017
  bindIp: 127.0.0.1  # Listen to local interface only, comment to listen on all interface
  ssl:
    mode: requireSSL
    PEMKeyFile: /opt/mongodb/ssl/mongodb.pem
    CAFile: /opt/mongodb/ssl/ca.pem
security:
  authorization: enabled

```
![image](https://user-images.githubusercontent.com/66196388/184013423-40b97469-8769-4ce3-b587-a30ed78812c1.png)

```bash
chown -R mongod:mongod /opt/mongodb/ssl/mongodb.pem
chown -R mongod:mongod /opt/mongodb/ssl/ca.pem
```
```bash
service mongod restart
service mongod status
```
```bash
cat /var/log/mongodb/mongod.log
```
![image](https://user-images.githubusercontent.com/66196388/184012830-f0bf8ca6-7366-400c-ac55-b9f59c72346f.png)

>Our local mongodb is on ssl, so it will now accepts connections using pem file only
```bash
mongo --host 127.0.0.1:27017 -u "vijayDB" -p  --authenticationDatabase "admin" --ssl --sslCAFile /opt/mongodb/ssl/ca.pem --sslPEMKeyFile /opt/mongodb/ssl/mongodb.pem
```
![image](https://user-images.githubusercontent.com/66196388/184013869-6f1034fe-ff79-4f63-a5f8-fc4ebf8b0be6.png)

