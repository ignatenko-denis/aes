# Advanced Encryption Standard (AES)

[Advanced Encryption Standard)](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard), is a specification for the encryption of electronic data established by the U.S. 

### Public-key cryptography, or asymmetric cryptography
Public-key cryptography, or asymmetric cryptography, is a cryptographic system that uses pairs of keys. 
Each pair consists of a public key (which may be known to others) and a private key 
(which may not be known by anyone except the owner). 
The generation of such key pairs depends on cryptographic algorithms which are based on 
mathematical problems termed one-way functions. Effective security requires keeping the private key private; 
the public key can be openly distributed without compromising security.

In such a system, any person can encrypt a message using the intended receiver's public key, 
but that encrypted message can only be decrypted with the receiver's private key. 
This allows, for instance, a server program to generate a cryptographic key 
intended for a suitable symmetric-key cryptography, 
then to use a client's openly-shared public key to encrypt that newly generated symmetric key. 
The server can then send this encrypted symmetric key over an insecure channel to the client; 
only the client can decrypt it using the client's private key 
(which pairs with the public key used by the server to encrypt the message). 
With the client and server both having the same symmetric key, 
they can safely use symmetric key encryption (likely much faster) to communicate over otherwise-insecure channels. 
This scheme has the advantage of not having to manually pre-share symmetric keys 
(a fundamentally difficult problem) while gaining the higher data throughput advantage of symmetric-key cryptography.

### How to encrypt file with AES in terminal?
#### Before you start
1. install the latest JDK with `keytool`.
2. install `openssl`
```shell
brew install openssl
```
#### Steps
First we need to create a private, public key pair for asymmetric encryption.
We can generate a keystore using keytool with a private key.
```shell
keytool -genkey -alias testaes -storetype JKS -keystore keystore.jks -keyalg RSA -keysize 2048
```
For asymmetric encryption and decryption we need to extract the private key and the public key from this keystore.
```shell
keytool -importkeystore -srckeystore keystore.jks -srcalias testaes -destalias testaes -destkeystore keystore.p12 -deststoretype PKCS12
```
This command will convert our JKS keystore to a PKCS12 Keystore.
```shell
openssl pkcs12 -in keystore.p12 -nodes -nocerts -out private_key.pem
```
This will save our private key as a private_key.pem file.
Then we can export the certificate for the private key from the keystore.
```shell
keytool -export -alias testaes -keystore keystore.jks -file cert.pem
```
This certificate contains the public key for the generated private key. 
We can export the public key from the certificate using openssl.
```shell
openssl x509 -inform der -pubkey -noout -in cert.pem > public_key.pem
```
Now we have public key (public_key.pem) and private key (private_key.pem) for our asymmetric encryption.
Generate a 256 bit (32 byte) random key.
```shell
openssl rand -base64 32 > key.bin
```
Encrypt the key
```shell
openssl rsautl -encrypt -inkey public_key.pem -pubin -in key.bin -out key.bin.enc
```
We use file with test data - README.md.
Actually Encrypt our large file
```shell
openssl enc -aes-256-cbc -salt -in README.md -out README.md.enc -pass file:./key.bin
```
Send/Decrypt the files
Send the *.enc files to the other person and have them do:
```shell
openssl rsautl -decrypt -inkey private_key.pem -in key.bin.enc -out key_result.bin

openssl enc -d -aes-256-cbc -in README.md.enc -out README_result.md -pass file:./key_result.bin

diff README.md README_result.md 
```
