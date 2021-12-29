---
title: 'Compile latest OpenSSL for CentOS 5'
date: "2018-05-10"
draft: false
#url: /2018/05/compile-latest-openssl-for-centos-5.html
tags: 
  - tools
  - script
  - openssl
  - compile
  - centos
---

  
Sometimes we are still managing very old hardware or OS versions and that we have nothing in our hands to change it till it phases out.  
  
Here is an example from CentOS 5 where the  latest OpenSSL package is 0.9.8e and of course it [does not support TLS 1.2](https://access.redhat.com/articles/1462353)  
  
Let's start with the download and uncompressing the OpenSSL package. The latest package supporting CentOS at the time of this writing from the OpenSSL [webpage](https://www.openssl.org/source/) is 1.0.2o.  
  
$ cd /usr/src/  
$ curl -O -L https://www.openssl.org/source/openssl-1.0.2o.tar.gz  
$ tar zxvf openssl-1.0.2o.tar.gz  
$ cd openssl-1.0.2o  
  
Some libraries that we need in our system in order for a successful compilation of the package are below. allow also the dependencies to be installed in this process such as kernel-headers, cpp, cvs etc.  
  
$ yum install expat-devel gettext-devel zlib-devel gcc autoconf gcc libtool perl-core zlib-devel  
  
  
Now it's time to configure and compile OpenSSL. It is worth to run the tests to see if there are any unexpected errors.  
  
$ ./config --prefix=/usr/local/openssl --openssldir=/usr/local/openssl shared zlib  
$ make  
$ make test  
  
_prefix_ and _openssldir_ sets the output paths for OpenSSL. _shared_ will force crating shared libraries and _zlib_ means that compression will be performed by using **zlib** library  
  
In order to install the libraries and the new binary you need to execute:  
  
$ make install  
  
...  
OpenSSL shared libraries have been installed in:  
  /usr/local/openssl  
...  
Sources of OpenSSL are required to compile other tools such us curl, Apache, Nginx etc., so I don't remove them.  
  
To test the new binary you can initially check the version and then try the connectivity with a TLS1.2 only website such as GitHub:  
  
$ /usr/local/openssl/bin/openssl version  
OpenSSL 1.0.2o  27 Mar 2018  
  
$ /usr/local/openssl/bin/openssl s\_client -connect github.com:443 -tls1\_2 | grep Protocol  
    Protocol  : TLSv1.2  
  
Success!  
  

### Add new version to PATH

After the installation you will probably want to check the version of OpenSSL but it will print out old version. Why? Because it's also installed on your server. I rarely override packages installed via yum. The reason is that when there is new version of OpenSSL and you will install it via yum, it will simply override compiled version, and you will have to recompile it again.  
  
Instead of overriding files I personally like to create new profile entry and force the system to use compiled version of OpenSSL.  
  
In order to do that, create following file:  
  
$ vi /etc/profile.d/openssl.sh  
and paste there following content:  
  
$ cat /etc/profile.d/openssl.sh  
pathmunge /usr/local/openssl/bin  
  
Save the file and reload your shell, for instance log out and log in again. Then you can check the version of your OpenSSL client. If you have errors loading shared libraries continue reading  

#### Link libraries

In order to fix the problem with loading shared libraries we need to create an entry in ldconfig.  
  
Create following file:  
  
$ vi /etc/ld.so.conf.d/openssl-1.0.2o.conf  
  
And paste the following contents:  
  
$ cat /etc/ld.so.conf.d/openssl-1.0.2o.conf  
/usr/local/openssl/lib  
  
We simply told the dynamic linker to include new libraries. After creating the file you need to reload linker by using following command:  
  
$ ldconfig -v  
  
Check the version of your OpenSSL now. It should print out  
OpenSSL 1.0.2o  27 Mar 2018  
  

Curl
----

In order to have full HTTPS functionality in most cases we need to use curl to access HTTP data. The defaultcurl version in CentOS 5 of course is compiled to use an outdated version of OpenSSL so we need to recompile a new version supporting the latest OpenSSL version we just compiled  
  
$ cd /usr/src/  
$ curl -O -L https://curl.haxx.se/download/curl-7.59.0.tar.gz  
$ tar zxvf curl-7.59.0.tar.gz  
$ cd curl-7.59.0  
$ ./configure --prefix=/usr/local/curl --with-ssl=/usr/local/openssl --enable-http --enable-ftp LDFLAGS=-L/usr/local/opensssl/lib CPPFLAGS=-I/usr/local/openssl/include  
$ make  
  
$ make install  
$ libtool --finish /usr/local/curl --with-ssl=/usr/local/openssl/lib  
  
and that will do your curl compile process to allow browsing sites with latest TLS1.2 encryption crypto.