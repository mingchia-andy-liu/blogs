---
title: "Openssl with ssl certificates"
date: 2021-06-12T19:56:38-07:00
description: Learn how to manage ssl certificate with openssl. A cheat sheet of common commands to deal with certificate.
slug: Openssl with ssl certificates
tags:
  - ssl
  - certificate
  - command line
---

[OpenSSL](https://www.openssl.org/) is a versatile command line toolkit for the TLS and SSL protocols. It can do so much more than what I am covering here. I am only compiling the certificate related commands.

#### View CSR
```bash
openssl req -in domain.csr -text -noout -verify
```
#### View Certificate
This command allows you to view the contents of a certificate (`domain.crt`) in plain text:
```bash
openssl x509 -in domain.crt -text -noout
```
#### Verify a Certificate was Signed by a CA
Use this command to verify that a certificate (`domain.crt`) was signed by a specific CA certificate (`ca.crt`):
```bash
openssl verify -verbose -CAFile ca.crt domain.crt
```

#### Convert PEM to DER
```bash
openssl x509 -inform PEM -in domain.crt -out domain.der -outform DER
```

#### Convert DER to PEM
```bash
openssl x509 -inform DER -in domain.der -out domain.crt -outform PEM
```

#### Check md5 with key
```bash
openssl x509 -noout -modulus -in domain.crt | openssl md5
openssl rsa -noout -modulus -in domain.key | openssl md5
openssl req -noout -modulus -in domain.csr | openssl md5
```

&nbsp;

---

## Generate certificates

#### Generate a Private Key

```bash
openssl genrsa -des3 -out private.key 2048
```

### Generate a Private Key and a CSR
If you want to use a Certificate Authority (CA) to issue the SSL certificate. The Certificate Signing Request (CSR) is generated and can be sent to a CA to request the issuance of a CA-signed SSL certificate.

This command creates a private key (`private.key`) and a CSR (`domain.csr`) from scratch:
```bash
openssl req -newkey rsa:2048 -nodes -keyout private.key -out domain.csr
```
* `-newkey rsa:2048`: the new key should be 2048-bit, generated using the RSA algorithm.
* `-nodes`: the new key should not be encrypted with a passphrase.

### Generate a CSR from an Existing Key

This command creates a new CSR (`domain.csr`) based on an existing private key (`private.key`):
```bash
openssl req -key private.key -new -out domain.csr
```

## Self-signed

### Generate a CA Key and Certificate:

We create a local self-sign CA to sign other certificates.
```bash
openssl req -x509 -newkey rsa:4096 -nodes -keyout ca.key -days 365 -out ca.crt
```

### Generate the Server Key, and Certificate and Sign with the CA Certificate:

```bash
openssl req -new -newkey rsa:4096 -keyout server.key -out server.csr -nodes -subj '/CN=mydomain.com'
openssl x509 -req -sha256 -days 365 -in server.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out server.crt
```

### Generate the Client Key, and Certificate and Sign with the CA Certificate:
```bash
openssl req -new -newkey rsa:4096 -keyout client.key -out client.csr -nodes -subj '/CN=My Client'
openssl x509 -req -sha256 -days 365 -in client.csr -CA ca.crt -CAkey ca.key -set_serial 02 -out client.crt
```
