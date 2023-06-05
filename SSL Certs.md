---
title: SSL Certs
lastmod: 2023-06-03T00:33:18-05:00
---
# SSL Certs
## The Gist
__TODO__: explain probably with a diagram the idea of how an authority binds a physical server to a domain. Maybe the basics of asymmetric encryption like RSA and how the public key is served to the client.
## Local Cert for Development
OK seriously, just use [mkcert](https://mkcert.dev/). It's way too complicated to do manually.
```powershell
# this will add the root CA to your system's trusted authorities
.\mkcert.exe -install

# this will generate a new cert from the installed root CA for the localhost domain
.\mkcert.exe localhost

# if you want access to the raw .pem file for the root CA, you can get them here
explorer.exe (.\mkcert.exe -CAROOT)
```
## The old way
Old script to generate certificate. It worked decent but had issues with different browsers if I recall correctly. Didn't really dig further.
```powershell
#create ca key
openssl genrsa -out ca.key 4096
#create ca
openssl req -new -x509 -days 365 -key ca.key -out ca.cert.pem -config .\openssl-custom.cnf
#create server key (private key)
openssl genrsa -out server.key 4096
#create server certificate signing request (csr)
openssl req -new -key server.key -out server.csr -config .\openssl-custom.cnf
#create certificate (public key)
openssl x509 -req -days 365 -in server.csr -CA ca.cert.pem -CAkey ca.key -CAcreateserial -out server.crt

rm ca.cert.pem
rm ca.cert.srl
rm ca.key
rm server.csr

Write-Host ">>>Import server.crt into your browser under the certificate authorities category."
Write-Host ">>>In Firefox, you will need to go to about:config and set security.enterprise_roots.enabled to true"
```