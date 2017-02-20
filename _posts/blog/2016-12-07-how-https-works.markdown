---
layout: post
title:  "How HTTPS works?"
date:   2016-12-07
categories: blog
tags: [web,server,nginx,security,general]
author: shobhit_garg
share: true
comments: true
excerpt: ""
---


These days when you open a popular website you see a symbol of lock beside your browser address bar. It is assumed that this lock is notifying some security but have you ever wondered what's behind the scene? This lock actually denotes that your communication with that website is secured with HTTPS. 

HTTPS is basically secured HTTP or HTTP over SSL/TLS. In HTTPS, TLS(Transport Layer Security)/SSL(Secure Sockets Layer) protocol is used to send the data from client to server and vice versa.

(TLS is the new name for SSL. Namely, SSL protocol got to version 3.0; TLS 1.0 is SSL 3.1.)

## Certificates, their generation and use of asymmetric communication:

 __<u>Certificate Authorities(CA):</u>__ 

Certificate Authorities are companies which manage certificates for browser and server. __CAs sign server's public key with their private key__ , which is called `Server Certificate` . Server certificate also contains data like expiry date, issued date, domain name etc. Browsers store CAs certificates in their local memory. A `CA Certificate` basically contains a CA public key and some other information like expiry date etc. When server certificate is sent to client, browser decrypts it using CA's public key(key present in CAs certificate) as it is signed by CA's private key at the time of creation. Also by decrypting it, browser checks multiple things like domain for which the certificate is issued, it's validity etc. which also helps in controlling Man in Middle Attack. Many of the popular companies like Symantec, Digicert etc. works as CA.

__Why can't a server send it's public key directly to client?__

In that case client can't be sure that received public key belongs to server because anyone can send a public key and claims to be a server. With the help of CA's issued certificate, server makes sure that client gets the authentic public key of server.

 __<u>How to get a website using https?</u>__ 

 On a machine you can generate a .csr file with a private key file(.key file) with few commands. csr file is a certificate request file which contains information like public key, domain name, organization address and email etc. and .key file is file which contains a private key for the public key present in csr file. For example:

{% highlight javascript %}

openssl req -new -newkey rsa:2048 -nodes -keyout yourdomain.key -out yourdomain.csr

{% endhighlight %}

When you run this command in terminal you have to provide all the information like domain name, organization address, email id etc. 
One important point to notice is that .csr file and private key file(.key file) are generated in pair. This private key acts as server private key and you have to keep it as a secret.

Now you have to send .csr file to CA which later issues a valid Server Certificate(basically by encrypting csr file data with CA's private key).CA issues certificate generally as .crt file which is basically a file signed by CA's private key. You don't need to send private key(.key file) to CA, but you have to keep that file at your server. 


Now as client knows the server public key so it can send data to client by encrypting it with this public key and server can send data to client by encrypting with it's private key(key present in .key file).
This way a secure communication can be achieved. But this type of encryption would be asymmetric encryption which has many flaws like:

1.It is much much slower than symmetric encryption.

2.Given the same key length, asymmetric is much weaker than symmetric. Therefore, you need a much larger key to provide equivalent protection. This also contributes to the slowness.

3.Asymmetric encryption carries with it an increase in size of output. For instance, if you use RSA, encrypted data is at least 10% larger than the clear text. Symmetric encryption, on the other hand, has a fixed size overhead even when encrypting gigabytes of data.

So Asymmetric conversation is just used in initial conversation when client and server try to validate each other. After that symmetric conversation is used to send the message from client to server and vice versa.
 
-----------------------------

## How TLS/SSL works?

In ssl term, validating the identity of client and server and generating the symmetric keys is called a handshake. So after handshake client and server can communicate using symmetric encryption.

__STEP 1(client to server)__

At the starting of handshake client sends:

1.A random number which is called Client.random

2.Session ID to resume the session if it is there

3.Info like time etc.

__STEP 2(server to client)__

Server after receiving this info sends this info to client:

1.A random number which is called Server.random

2.Server certificate(issued by CA)        

3.Encryption method it is going to support. For example if it sends TLS_RSA_WITH_3DES_EDE_CBC_SHA means that the session key will be transmitted with RSA (asymmetric encryption, using the RSA public key from the server certificate), the data will be symmetrically encrypted with 3DES, and the integrity check will use the SHA-1 hash function.

__STEP 3(client to server)__

Client validates server using the server certificate and generate a random number which is called pre_master_secret and send it to server by encrypting it with server public key.
(Client now have server public key as CA certificate can decrypt the server certificate.)


__STEP 4(for both client and server)__

Using the pre_master_secret, Client.random and Server.random both client and server can generate master_secret on their respective sides. Function to generate master_secret is like:

{% highlight javascript %}

master_secret = PRF(pre_master_secret, 
                    "master secret", 
                    ClientHello.random + ServerHello.random)

{% endhighlight %}                    

This `master_secret is used to generate different encryption and MAC keys` on both sides.

If client wants to resume the previous session it can just send the Session ID in Step 1. In that case server replies with a Server.random and both can generate master_secret on their respective machines. In this case client doesn't need to send the pre_master_secret to server again and client doesn't need to validate server certificates.





__Why does the SSL/TLS handshake have a client and server random?__

Say someone try to replay the attack, he can send the same encrypted packets to server. So in that case pre_master_secret and Client.random would be same. Server would generate it's master_secret after that using Server.random and Client.random. That means it's master_secret would be different than the one which client has used because in the replay case Client.random would be same as same packets are being forwarded but Server.random would not be. So if someone forwards the same packets,  server can't validate and decrypt the data as it had been generated using master_secret of other client.

Say someone replays the same packets just after the transfer of original packets without doing the handshake. In that case too it would not work as with every packet sent to server it uses sequence number which changes according to previous packets so server would not respond to the replay request.

------------------------


If you want to go in more details, please go through [first-few-milliseconds-of-https][https-blog].This blog is very informative and descriptive.


[https-blog]: http://www.moserware.com/2009/06/first-few-milliseconds-of-https.html

