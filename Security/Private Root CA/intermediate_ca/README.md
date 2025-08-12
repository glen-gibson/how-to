# Setting up the Intermediate CA Key Pair

## Table of Contents
- [Prep Work](#prep-work)
- [Sample openssl.cnf file](#sample-opensslcnf-file)
- [Create the Intermediate CA Private Key](#create-the-intermediate-ca-private-key)
- [Create a CSR for the Intermediate CA](#create-a-csr-for-the-intermediate-ca)
- [Create the Intermediate CA Certificate](#create-the-intermediate-ca-certificate)
- [Verify the Intermediate CA Certificate](#verify-the-intermediate-ca-certificate)
- [The `index.txt` file](#the-indextxt-file)
- [Create the Certificate Chain file](#create-the-certificate-chain-file)
- [References](#references)

## From the Official OpenSSL CA Guide:
>*"We will be signing certificates using our intermediate CA. You can use these signed certificates in a variety of situations, such as to secure connections to a web server or to authenticate clients connecting to a service."*  

## Prep Work
> The steps below are from your perspective as the certificate authority. A third-party, however, can instead create their own private key and certificate signing request (CSR) without revealing their private key to you. They give you their CSR, and you give back a signed certificate. In that scenario, skip the `genrsa` and `req` commands.

Running as root, otherwise you'll need to `sudo` a lot of it.
```bash
mkdir /root/ca/intermediate
cd /root/ca/intermediate
mkdir certs crl csr newcerts private
chmod 700 private
touch index.txt
echo 1000 > serial
echo 1000 > /root/ca/intermediate/crlnumber
```

## Sample openssl.cnf file
A sample openssl.cnf file can be downloaded from [here](openssl.cnf).  Its path should be `/root/ca/intermediate/openssl.cnf`  

It is very similar to the `openssl.cnf` used for the Root CA Key Pair apart from the following changes:
```bash
[ CA_default ]
dir             = /root/ca/intermediate
private_key     = $dir/private/intermediate.key.pem
certificate     = $dir/certs/intermediate.cert.pem
crl             = $dir/crl/intermediate.crl.pem
policy          = policy_loose
copy_extensions = copy
```

You can put some defaults for certificate generation like you did in the root version of that [openssl.cnf](../root_ca/README.md#sample-opensslcnf-file).

## Create the Intermediate CA Private Key
Create and verify a passphrase when prompted:
```bash
cd /root/ca
openssl genrsa -aes256 -out intermediate/private/intermediate.key.pem 4096
```

Set some tighter permissions:
```bash
chmod 400 intermediate/private/intermediate.key.pem
```

## Create a CSR for the Intermediate CA
We will create a certificate signing request (CSR) for the Intermediate CA's certificate.

```bash
cd /root/ca
openssl req -config intermediate/openssl.cnf -new -sha256 \
    -key intermediate/private/intermediate.key.pem \
    -out intermediate/csr/intermediate.csr.pem
```
Follow the prompts.

## Create the Intermediate CA Certificate
We'll issue and sign an Intermediate CA Certificate using the Root CA and the CSR from the previous step.  The `openssl.cnf` used in this command is the one for the Root CA, not the Intermediate.  You will be prompted for the passphrase for the Root CA's key that was set during root key creation.

```bash
cd /root/ca
openssl ca -config openssl.cnf -extensions v3_intermediate_ca \
    -days 3650 -notext -md sha256 -in intermediate/csr/intermediate.csr.pem \
    -out intermediate/certs/intermediate.cert.pem
```

When asked "Sign the certificate" choose `y` to create the certificate.

Set some tighter permissions:
```bash
chmod 444 intermediate/certs/intermediate.cert.pem
```

## Verify the Intermediate CA Certificate
If you want to verify what the Intermediate CA Certificate contains, you can do so with this command:
```bash
openssl x509 -noout -text -in certs/ca.cert.pem
```  

To verify against the Root CA certificate:
```bash
cd /root/ca
openssl verify -CAfile certs/ca.cert.pem intermediate/certs/intermediate.cert.pem
```

## The `index.txt` file
The index.txt file is where the OpenSSL ca tool stores the certificate database. Do not delete or edit this file by hand. It should now contain a line that refers to the intermediate certificate.

## Create the Certificate Chain file
To verify a certificate's authenticity a certicicate must be verified against the Intermediate CA, which in turn must be verified against the Root CA.

To complete the chain of trust, we should create a CA certificate chain to present to applications that consume our certificates.

```bash
cd /root/ca
cat intermediate/certs/intermediate.cert.pem certs/ca.cert.pem > \
    intermediate/certs/ca-chain.cert.pem
chmod 444 intermediate/certs/ca-chain.cert.pem
```

From the OpenSSL CA Official Documentation:  
>*Our certificate chain file must include the root certificate because no client application knows about it yet. A better option, particularly if you’re administrating an intranet, is to install your root certificate on every client that needs to connect. In that case, the chain file need only contain your intermediate certificate.*
  
# References
https://openssl-ca.readthedocs.io/en/latest/create-the-intermediate-pair.html