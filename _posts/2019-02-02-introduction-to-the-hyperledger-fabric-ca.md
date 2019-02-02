---
title: "Introduction to the Hyperledger Fabric Certificate Authority \U0001F1EC\U0001F1E7"
layout: post
---

Hyperledger Fabric CA is a certificate authority for Hyperledger Fabric, a CA is an entity that issues digital certificates, Hyperledger Fabric CA provides X.509 certificates, these type of certificates are especified in more detail in the *RFC 5280*

### RFC 5280

- CA (Certificate Authority)
- CRLs (Certificate revocation list): Timestamped list identifying revoked certificates is identified in a CRL by its certificate serial number
- RA (Registration authority)

We have three types of PEM CA

1. **Internet policity registration authority (IPRA)** that corresponds to the level 1
2. **Policity certification authorities (PCAs)** that corresponds to the level 2
3. **Certificate Authorities**
    3.1 **Cross certifies:** Issue and subject are differents, there is a trust relationship between 2 CAs
    3.2 **Self issues:** Support changes in policity or operations
    3.3 **Self signed:** Convey a public key for use to begin certificate paths

## Working with the Hyperledger Fabric CA

This docker-compose file is pulling the Hyperledger Fabric CA image and starting a server (Hyperledger Fabric CA Server) at 7054 port.

{% highlight dockerfile %}
version: '3'

services:
  fabric-ca-server:
    image: hyperledger/fabric-ca:1.4
    ports:
      - "7054:7054"
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
    volumes:
      - "./fabric-ca-server:/etc/hyperledger/fabric-ca-server"
    command: sh -c 'fabric-ca-server start -b admin:youpasswd'
{% endhighlight %}

Now let's start the **fabric-ca-server** ```docker-compose up -d```

If you inspect your current directory you'll have a new folder called **fabric-ca-server/** this folder contains all the basic configuration for the Hyperledger Fabric CA, a `ca-cert.pem` with the basic info defined in the configuration file (**fabric-ca-server-config.yml**).

The **fabric-ca-server-config.yml** file is the key piece to config our CA, you can modify this file as you want, for example we'll change the SQLite database by a PostgreSQL database hosted in AWS RDS (previously created, more info at [AWS Documentation](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.PostgreSQL.html)).


{% highlight dockerfile %}
db:
  type: postgres
  datasource: host=mydbinstance.mx9avwd8tsu9.us-east-1.rds.amazonaws.com port=5432 user=msflore password=youdbpassword dbname=fabric_ca sslmode=disable
  tls:
      enabled: false
      certfiles:
      client:
        certfile:
        keyfile:
{% endhighlight %}

You have to remove the ```ca-cert-pem``` and restart the docker-compose ```docker-compose restart```

The last step for this basic configuration is to enroll us in the Fabric CA Server, for this we have to use the **fabric-ca-client**, you can think in the client / server model, there is a client making peticions to a server.

Enter to the docker container  and exec the enrollment

```
docker exec -it <container-id> /bin/bash

fabric-ca-client enroll -u http://0.0.0.0:7054
```

Now you can inspect your database to see the tables that created this enrollment.

```
psql -h mydbinstance.mx9avwd8tsu9.us-east-1.rds.amazonaws.com

fabric_ca=# SELECT * FROM affiliations;
```

## Conclusion

In all the examples above we show how to configurate a basic Fabric Certificate Athority storing the information in a PostgreSQL database hosted in AWS, there is another ways to configure your Fabric CA like LDAP, MySQL or connect to your own CA.