# About Server Certificate:

A server certificate is a digital certificate that proves the identity of server and enables encrypted communication over HTTPS.

## When will a server sends its certificate to prove its identity?

When someone/something(person, program) via browser/software hits the server address -> 
DNS resolution(Got IP Address) -> 
tcp handshake(Server side OS stack creates SOCKET<5 tuple -  SourceIp, SourcePort, DestinationIp, DestinationPort, ProtocolNumber such as TCP-6 UDP-17 or ICMP - 1 > for communication) -> 
now server and client can communicate -> 
TLS Handshake happens (Here server sends its Certificate + the certificate chain(intermediate certificates but not rootCA, server assumes you have rootCA), Client validating the certificate and key exchange happens)

server certificate -> proves "This is really the server you wanted"
intermediate certificates -> help your device verify that the server certificate is trustworthy


## What certificates a server sends?
Goal: When you hit google.com (or any HTTPS server), see the server certificate + intermediate chain.
You will learn:
how to fetch a sites certificate
how to view full certificate details
what the certificate chain looks like

simulation commands:
```bash
openssl s_client -connect google.com:443 -showcerts
```

Simulation:
Generally server only sends leaf+intermediate certificates but not rootCA.

in corporate pcs you can get proxies(zscaler) rootCA will be automatically appended to OS level certificate store by GPO or client applications.

You can get that certificate from browser

Without the root CA, HTTPS interception breaks because certificate chain cannot be validated.


Get certificates of a server
This below command does three things
1. Connects to google.com on port 443
2. Shows the TLS Handshake output
3. Prints all certificates the server sends

```bash
openssl s_client -connect google.com:443 -showcerts
```


Append them to a file

```bash
openssl s_client -connect google.com:443 -showcerts > allCerts
```

Copy one certificate to a file 'google.crt' and then run below command to view detailed
like Subject (google.com), Issuer, Expiry Date, Public Key, Extentions

```bash
openssl x509 -in google.crt -text -noout
```

## At first how certificates come with OS? Where is certificate store located at OS level? How they are being managed?


**Note: only root CA certificates are stored in the system. server certificate + intermediate certificates will be sent by the server we are getting connected to**


1. When you install Ubuntu/WSL -> the ca-certificates package is installed
    + This package includes
        * /usr/share/ca-certificates (official CA files)
        * The configuration /etc/ca-certificates.conf
        * the tool `update-ca-certificates`
2. Ubuntu updates your CA list through normal system updates
    + Whenever there is a trusted CA added/removed by Mozilla or Ubuntu security teams
        * Ubuntu releases a new ca-certificates package version
        * you get it when you run

```bash
sudo apt update && sudo apt upgrade
```
This automatically refreshes your trust store
3. The tool update-ca-certifictes builds the final list
    + Each time the package updates, it runs:

      ```bash
      update-ca-certificates
      ```
      
    + This tool:
        * Reads the config file /etc/ca-certificates.conf
        * Takes certifif=cates enabled in that file
        * Copies them into /etc/ssl/certs/
        * Generates hashed links
        * Builds the big trust bundle: **/etc/ssl/certs/ca-certificates.crt**
Thats the bundle your apps use.

SUMMARY:

Ubuntu gets its trusted root certificates from the **ca-certificate** package(built from mozillas CA list), and the helper script `update-ca-certificates` installs them into `/etc/ssl/certs` and the combined bundle `/etc/ssl/certs/ca-certificates.crt`

verify ubuntu packages
```bash
dpkg -l | grep ca-certificates
or 
dpkg -s ca-certificates
or 
dpkg -L ca-certificates   #to list files from the package
```

in ubuntu, below command updates certificates 
```bash
update-ca-certificates
```


✅ Where does it pull certificates from?

Primary source:
/usr/share/ca-certificates/
This directory contains subfolders with .crt files (trusted CA certificates).


Custom additions:
/usr/local/share/ca-certificates/
If you add a .crt file here (e.g., Zscaler root CA), update-ca-certificates will include it.

✅ What does the command do?

Scans /usr/share/ca-certificates/ and /usr/local/share/ca-certificates/ for .crt files.
Updates the master bundle:
/etc/ssl/certs/ca-certificates.crt

This file is a concatenated PEM bundle of all trusted CAs.
Creates hashed symlinks in:
/etc/ssl/certs/

These are used by OpenSSL for fast lookup.

✅ How to verify after update

Check if your certificate is included:

grep -A2 "Zscaler" /etc/ssl/certs/ca-certificates.crt

Count total certificates:

grep -c "BEGIN CERTIFICATE" /etc/ssl/certs/ca-certificates.crt

certificates can also be enabled or disabled using #-comment or not commenting in `/etc/ca-certificates.conf`




## As a admin how can you place a custom certificate to trust server
Just copy the file.crt to /usr/local/share/ca-certificates/ and then run
```bash
sudo update-ca-certificates 
```

## How to verify the certificate i have with certificate store in ubuntu?

Get the server or intermediate certificate.

Copy only the base64 encoded data, from ---begin certificate--- to ---end certificate---

run the below command:
```bash
openssl verify -CAfile /ect/ssl/certs/ca-certificates.crt my_example_cert.pem
```
output might be success
```
my_example_cert.pem: Ok
```
or errors like
```
+ unable to verify first certificate
+ self-signed certificate in certificate chain
+ hostname mismatch
+ certificate has expired
+ unable to get issuer certificate
```
