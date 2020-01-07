# Certificate Basics

Today we look at certificate Basics and how the trust works generally, and how certain details are working in Windows vs Linux. And maybe some macOS.

## Prerequisites

```bash
mark@aragorn:~$ caddy version
v2.0.0-beta11 h1:NVHnPAdZPt6OUBMltUMe2DWVsyYRbeE6NxhCm3AjGT8=
mark@aragorn:~$ openssl version
OpenSSL 1.1.0l  10 Sep 2019
mark@aragorn:~$
```

OpenSSL is used to create certificates and Caddy is used to simply have a versatile WebServer on your localhost.

Caddy is now using a json format that you should adapt from the provided caddy.json:

If you have installed the tools, run the "create_certs.sh" to create your certs. 
![Certificate Creation](images/CertificateCreation.png)

Now you can start the Caddy Server with the local TLS config (Details in the caddy.json)

```bash
To start Caddy:
$ caddy start --config caddy.json

To stop Caddy:
$ caddy stop
```

When the start was successful, you should see something like this:

```bash
mark@aragorn:~/Git/CertificateBasics$ caddy start --config caddy.json 
2020/01/07 22:53:05.468 INFO    admin   admin endpoint started  {"address": "localhost:2019", "enforce_origin": false, "origins": ["localhost:2019"]}
2020/01/07 23:53:05 [INFO][cache:0xc000265540] Started certificate maintenance routine
2020/01/07 23:53:05 [WARNING] Stapling OCSP: no OCSP stapling for [localhost]: no OCSP server specified in certificate
2020/01/07 22:53:05.480 INFO    tls     cleaned up storage units
2020/01/07 22:53:05.481 INFO    admin   Caddy 2 serving initial configuration
Successfully started Caddy (pid=12056)
```

## Test Cases

Now try to see, whether a standard curl request does, what it should do...

First, lets try the insecure request:

```bash
curl https://localhost:2020 --insecure
Hello world!
```

Secondly, lets try the secure method:

```bash
curl https://localhost:2020
2020/01/08 00:01:51 http: TLS handshake error from [::1]:40426: remote error: tls: unknown certificate authority
curl: (60) SSL certificate problem: self signed certificate
More details here: https://curl.haxx.se/docs/sslcerts.html

curl performs SSL certificate verification by default, using a "bundle" of Certificate Authority (CA) public keys (CA certs). If the default bundle file isn't adequate, you can specify an alternate file using the --cacert option.
If this HTTPS server uses a certificate signed by a CA represented in the bundle, the certificate verification probably failed due to a problem with the certificate (it might be expired, or the name might not match the domain name in the URL).
If you'd like to turn off curl's verification of the certificate, use the -k (or --insecure) option.
```

So as we can see, curl told us there is a certificate problem and specifically even a self-signed one...  
It seems for some odd reason, that my system does not trust this self signed certificate...
And for good reason, if a malicious party could put themselves in between my browser and the web server, they could practically decrypt all the traffic.  
Hence every system that can speak http/tls has provided CA certs in a bundle file. Some browsers like Firefox bring their own one, Chrome and IE take the one from the system and Java is so os-idependent it uses also it's own keystore. The very same is true for python, which uses the "certifi" package.

```bash
curl https://localhost:2020 --cacert cert_localhost.pem 
Hello world!
```

Another way would have been to add the cert to my ca path in my os, or the key store of my programming language.

## Certificate Handling

Certificates are all about trust. Achieved through cryptography and the private/public key approach.

This means, that for each public certificate there is a corresponding private key to use. The key is being used do decrypt packets, that have been encrypted via the public certificate.

Furthermore, each certificate ususally exists in a chain. For publicly available sites/endpoints, this starts with a CA (certificate authority) and a subsequent IA (Intermediate authority).  
The main difference is the validity timeframe and the airgapness of the root certificate versus the intermediate certificate.  
Reason is, that under no circumstances a 3rd party can gains access to the root CA.
The intermediate CA is used on connected servers to actually run the service in an automated fashion and that, when a IA has been compromised, it can be replaced fast.  
Otherwise each cert signing would need a manual effort and copy/paste actions (god forbid...) ;)

This can be simply observed in your browser settings:

![Certificate Main](images/CertificateMain.png)

![Certificate Details](images/CertificateDetails.png)

## Enterprise Environments

In internal enterprise environments, this is somewhat the same, with the difference that you do not have to rely on the "trusted" 3rd parties by the browser OEMs, but on your enterprise PKI (Public Key Infrastructure).  
This PKI is used to create your root and subordinate CAs and can create your needed certificates.  
Indeed, many cloud providers, like Azure or AWS, do have PKI services backed by HSMs (Hardware Security Module) to achieve the very same in cloud environments.

Thx!
