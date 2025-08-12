# Setting up the Root CA Key Pair

## Table of Contents
- [Introduction](#from-the-official-openssl-ca-guide)
- [Prep Work](#prep-work)
- [Sample openssl.cnf file](#sample-opensslcnf-file)
- [Create the Root CA Private Key](#create-the-root-ca-private-key)
- [Create the Root CA Certificate](#create-the-root-ca-certificate)
- [Verify the Root CA Certificate](#verify-the-root-ca-certificate)
- [References](#references)

## From the Official OpenSSL CA Guide:
*"Acting as a certificate authority (CA) means dealing with cryptographic pairs of private keys and public certificates. The very first cryptographic pair we’ll create is the root pair. This consists of the root key (ca.key.pem) and root certificate (ca.cert.pem). This pair forms the identity of your CA.*

*Typically, the root CA does not sign server or client certificates directly. The root CA is only ever used to create one or more intermediate CAs, which are trusted by the root CA to sign certificates on their behalf. This is best practice. It allows the root key to be kept offline and unused as much as possible, as any compromise of the root key is disastrous."*

## Prep Work
Running as root, otherwise you'll need to `sudo` a lot of it.
```bash
cd /root/ca
mkdir certs crl newcerts private
chmod 700 private
touch index.txt
echo 1000 > serial
```

## Sample openssl.cnf file
A sample openssl.cnf file can be downloaded from [here](openssl.cnf).  Its path should be `/root/ca/openssl.cnf`  

See the reference links for a more in depth explanation of what each section of the `openssl.cnf` does.

You may wish to alter the `[req_distinguished_name]` sections second half:
```
[ req_distinguished_name ]
# See <https://en.wikipedia.org/wiki/Certificate_signing_request>.
commonName                      = Common Name
countryName                     = Country Name (2 letter code)
stateOrProvinceName             = State or Province Name
localityName                    = Locality Name
0.organizationName              = Organization Name
organizationalUnitName          = Organizational Unit Name
emailAddress                    = Email Address

# Optionally, specify some defaults.
countryName_default             = NZ            # changed
stateOrProvinceName_default     = Northland     # changed
localityName_default            =
0.organizationName_default      = Thats My Tech # changed
#organizationalUnitName_default =               # commented out
#emailAddress_default           =               # commented out
```

## Create the Root CA Private Key
Create and verify a passphrase when prompted:
```bash
cd /root/ca
openssl genrsa -aes256 -out private/ca.key.pem 4096
```

Set some tighter permissions:
```bash
chmod 400 private/ca.key.pem
```


## Create the Root CA Certificate
Create the public Root CA certificate using the Root CA Private Key created in the previous step.  We will set it to expire in about 20 years.  You will be prompted for the passphrase that was set during Root CA Private Key creation and for various attributes related to the new certificate being created:

```bash
cd /root/ca
openssl req -config openssl.cnf -key private/ca.key.pem -new -x509 \
    -days 7300 -sha256 -extensions v3_ca -out certs/ca.cert.pem
```

Set some tighter permissions:
```bash
chmod 444 certs/ca.cert.pem
```

## Verify the Root CA Certificate
If you want to verify what the Root Certificate contains, you can do so with this command:
```bash
openssl x509 -noout -text -in certs/ca.cert.pem
```  
  
# References
https://openssl-ca.readthedocs.io/en/latest/create-the-root-pair.html