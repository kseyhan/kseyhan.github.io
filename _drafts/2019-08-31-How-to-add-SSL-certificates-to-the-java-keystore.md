---
date: 2019-08-31
layout: post
title: "How to add certificates to the java keystore"
description: How to add certificates to the java keystore
categories: [java]
tags: [java, truststore, keytool, openssl, ssl]
---

Heres a quick guide to add certificates to your truststore in java that you'll probably will need when you start to use some kinda trusted ssl connection on your services like for example ssl protected smtp connections etc..

Its probably a good idea to simply just list the certificates in the keystore to start with so you can be sure that you're working with the correct file and password.

```keytool -keystore $JAVA_HOME/jre/lib/security/cacerts -storepass changeit -list```

After that you want to get a copy of the cert you want to import with the following command.


```openssl s_client -servername <server-address> -connect <server-address>:<server-port> | openssl x509 -outform pem```

So if your domain is example.com and you want to get a certificate for a ssl protected smtp service and save that file to tmp the command would be as follows.

```openssl s_client -servername example.com -connect example.com:465 < /dev/null 2>/dev/null | openssl x509 -outform pem > /tmp/example.crt```


After that we import that certificate into our java truststore by using the following command.

```keytool -import -alias example -file /tmp/example.crt -keystore $JAVA_HOME/jre/lib/security/cacerts -storepass changeit```

and verify after that that everything worked out by listing the certificates again and searching for our certificate.

```keytool -keystore $JAVA_HOME/jre/lib/security/cacerts -storepass changeit -list | grep example```

happy trusted connecting!