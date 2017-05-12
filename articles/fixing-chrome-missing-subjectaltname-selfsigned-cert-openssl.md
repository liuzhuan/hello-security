# Fixing Chrome 58+ [missing_subjectAltName] with openssl when using self signed certificates

https://alexanderzeitler.com/articles/Fixing-Chrome-missing_subjectAltName-selfsigned-cert-openssl/

Author: Alexander Zeitler

Date: April 23, 2017

> As a passionated developer Alexander Zeitler loves to create outstanding solutions for the web and mobile platforms since 1994.

Since version 58, Chrome requires SSL certificates to use **SAN** (Subject Alternative Name) instead of the popular Common Name (CN), thus CN support has been removed.

> **SAN**: 

If you're using self signed certificates (but not only!) having only CN defined, you get an error like this when calling a website using the self signed certificate:

![missing Subject Alternative Name](https://alexanderzeitler.com/articles/Fixing-Chrome-missing_subjectAltName-selfsigned-cert-openssl/missing_subjectAltName.png)

Here's how to create a self signed certificate with SAN using openssl

First, let's create a root CA cert using `createRootCA.sh`:

```
#!/usr/bin/env bash
mkdir ~/ssl/
openssl genrsa -des3 -out ~/ssl/rootCA.key 2048
openssl req -x509 -new -nodes -key ~/ssl/rootCA.key -sha256 -days 1024 -out ~/ssl/rootCA.pem
```

Next, create a file `createselfsignedcretificate.sh`:

```
#!/usr/bin/env bash
sudo openssl req -new -sha256 -nodes -out server.csr -newkey rsa:2048 -keyout server.key -config <( cat server.csr.cnf )

sudo openssl x509 -req -in server.csr -CA ~/ssl/rootCA.pem -CAkey ~/ssl/rootCA.key -CAcreateserial -out server.crt -days 500 -sha256 -extfile v3.ext
```

Then, create the openssl configuration file `server.csr.cnf` referenced in the openssl command above:

```
[req]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn

[dn]
C=US
ST=New York
L=Rochester
O=End Point
OU=Testing Domain
emailAddress=your-administrative-address@your-awesome-existing-domain.com
CN = localhost
```

Now we need to create the `v3.ext` file in order to create a X509 v3 certificate instead of a v1 which is the default when not specifying a extension file:

```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = localhost
```

In order to create your cert, first run `createRootCA.sh` which we created first. Next, run `createselfsignedcertificate.sh` to create the self signed cert using localhost as the SAN and CN.

After adding the `rootCA.pem` to the list of your trusted root CAs, you can use the `server.key` and `server.crt` in your web server and browse https://localhost using Chrome 58 or later:

![validsan](https://alexanderzeitler.com/articles/Fixing-Chrome-missing_subjectAltName-selfsigned-cert-openssl/validsan.jpg)

You can also verify your certificate to contain the SAN by calling

```
openssl x509 -text -in server.crt -noout
```

Watch for this line `Version: 3 (0x2)` as well as `X509v3 Subject Alternative Name:` (and below).