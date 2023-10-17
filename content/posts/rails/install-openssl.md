---
title: "Install Openssl"
date: 2023-10-12T08:26:29+05:30
draft: true
---

Download the source code of openssl from http://www.openssl.org/source/. Use following command:

```
wget http://www.openssl.org/source/openssl-1.1.1w.tar.gz
```

Change to an installation directory and copy the openssl tar file into it

```
sudo mkdir /opt
cd opt
sudo mv openssl-1.1.1w.tar.gz /opt/
```

Extract the tar file in the directory

```
sudo tar -xvf openssl-1.1.1w.tar.gz
```

Run config to set the installation directory. Note this must be different from where you've extracted the tar file

```
sudo ./config --prefix=/opt/openssl --openssldir=/opt/openssl
```

Make and install openssl

```
sudo make
sudo make install
```

Verify it's working:

```
➜  export LD_LIBRARY_PATH=/opt/openssl/lib
➜  openssl ./bin/openssl version
OpenSSL 1.1.1w  11 Sep 2023
```

Install ruby 2.x with openssl 1.x

```
export LD_LIBRARY_PATH=/opt/openssl/lib
rvm install 2.7.7 --with-openssl-dir="/opt/openssl"
```
