# Importing existing keys into a PIV slot on a yubikey:
(examples from OSX)

Follow the directions to get PIV-AGENT working https://github.com/arekinath/piv-agent

Download and Install [yubikey-manager](https://www.yubico.com/products/services-software/download/yubikey-manager/). I tried and failed to install certs via ykman command line too . I -had- to use the gui. If you generate keys using openssl the cli will most likely work. If you generated your keys with ssh-keygen like I did (and not open ssl) the certs won't import via command line.

### Install yubikey mananager cli tool:
```
$ brew install ykman

Rainier:github ryankitchen$ ykman piv info
PIV version: 5.1.2
PIN tries remaining: 3
CHUID:    3019000000000000000000000000000000000000000000000000003410920951864af7e698bb9acb93a03ba88e350832303530303130313e00fe00
CCC:     f015a000000116ff3d142070a9aad144dc20f0f235e6f6f10121f20121f300f400f50110f600f700fa00fb00fc00fd00fe00
Slot 9a:
    Algorithm:    ECCP256
    Subject CN:    920951864AF7E698BB9ACB93A03BA88E
    Issuer CN:    920951864AF7E698BB9ACB93A03BA88E
    Fingerprint:    b106936a4b2e9d8a874dc4d411c663d2e780374dd27895c81adcb86a88975fad
    Not before:    2019-03-20 21:03:40
    Not after:    2029-03-17 21:03:40
Slot 9c:
    Algorithm:    RSA2048
    Subject CN:    920951864AF7E698BB9ACB93A03BA88E
    Issuer CN:    920951864AF7E698BB9ACB93A03BA88E
    Fingerprint:    d0c4e216495c3a69415d21c9cbb7b1d04841c362a123ed3e8f1b2caef11307d6
    Not before:    2019-03-20 21:03:44
    Not after:    2029-03-17 21:03:44
Slot 9d:
    Algorithm:    ECCP256
    Subject CN:    920951864AF7E698BB9ACB93A03BA88E
    Issuer CN:    920951864AF7E698BB9ACB93A03BA88E
    Fingerprint:    4921e0aebce645b09a1e14cd0eda4ea6279a9bd98daa3b08c91b14c1442db09d
    Not before:    2019-03-20 21:03:45
    Not after:    2029-03-17 21:03:45
Slot 9e:
    Algorithm:    ECCP256
    Subject CN:    920951864AF7E698BB9ACB93A03BA88E
    Issuer CN:    920951864AF7E698BB9ACB93A03BA88E
    Fingerprint:    138c150e5c75c57036a1e62e5a2ae0f739d666153f1cb1b20d8b93c5ed7e67a3
    Not before:    2019-03-20 21:03:40
    Not after:    2029-03-17 21:03:40
```

### Remove the generated cert in slots 9a/c:
```
Rainier:piv-agent ryankitchen$ ykman piv delete-certificate 9a
Enter a management key [blank to use default key]:
Touch your YubiKey...
Rainier:piv-agent ryankitchen$ ykman piv delete-certificate 9c
Enter a management key [blank to use default key]:
Touch your YubiKey...
```
### Add Private Key used to gen cert:
```
Rainier:.ssh ryankitchen$ ykman piv import-key 9c id_ecdsa
Enter a management key [blank to use default key]:
Touch your YubiKey...
Enter password to decrypt key:
Rainier:.ssh ryankitchen$ ykman piv import-key 9c id_rsa
Enter a management key [blank to use default key]:
Touch your YubiKey...
Enter password to decrypt key:
```
### Create CSRs for each private key we need a cert for:
```openssl x509 -req -days 3650 -in yubi.csr -signkey id_ecdsa -out yubiecdsa.crt --subject="C=US/ST=WA/L=Enumclaw/O=Joyent/OU=QA/ CN=qa/emailAddress=ryan.kitchen@joyent.com"```

### Create .crts for each private key:
```
Rainier:.ssh ryankitchen$ openssl x509 -req -days 3650 -in yubiecdsa.csr -signkey id_ecdsa -out yubiecdsa.crt
Signature ok
subject=/C=US/ST=WA/L=Enumclaw/O=Joyent/OU=QA/CN=qa/emailAddress=ryan.kitchen@joyent.com
Getting Private key
Enter pass phrase for id_ecdsa:
```
### Import the Certs you Generated
Fire up the yubikey manager gui and select the application:<br>
![alt text](https://raw.githubusercontent.com/joyfulrabbit/qasoft/master/img/appselect.png)

Import the cert into the correct corresponding slot:<br>
![imp](https://raw.githubusercontent.com/joyfulrabbit/qasoft/master/img/import.png)

### Update your bash profile:
```
ssh-add -l
256 SHA256:PJ6ucGKUqlQhiJdArDaF65+AVImg8SVq77vL6nVE/ME PIV_slot_9C /CN=Digital Signature (ECDSA)
256 SHA256:U86TVxP/gxVk4CQibIWit3Q+/5i4aZuXa2NALIahjww PIV_slot_9E /CN=Card Authentication (ECDSA)
```
Select the appropriate fingerprint and add it to you .bash_profile:
```
#Triton 
export SDC_KEY_ID="SHA256:PJ6ucGKUqlQhiJdArDaF65+AVImg8SVq77vL6nVE/ME"

#Manta
export MANTA_KEY_ID="SHA256:PJ6ucGKUqlQhiJdArDaF65+AVImg8SVq77vL6nVE/ME"
```

### Try it out:
Open a new window and unlock your agent:
```
Rainier:~ ryankitchen$ ssh-add -X
Enter lock password:
Agent unlocked.
Rainier:~ ryankitchen$ mls
kafka/
out/
pub_key/
wordpress/

Rainier:~ ryankitchen$ ssh -v staging-1
...
debug1: Will attempt key: PIV_slot_9C SHA256:PJ6ucGKUqlQhiJdArDaF65+AVImg8SVq77vL6nVE/ME agent
debug1: Offering public key: PIV_slot_9C SHA256:PJ6ucGKUqlQhiJdArDaF65+AVImg8SVq77vL6nVE/ME agent
debug1: Authentication succeeded (publickey).
...
- SmartOS (build: 20180921T223234Z)
You have new mail.
[root@headnode (staging-1) ~]#

```


