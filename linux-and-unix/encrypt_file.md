# Use ssh-rsa public key to encrypt a text

## Convert your ssh public key to PEM format (that `openssl rsautl` can read it):

```bash
ssh-keygen -f ~/.ssh/id_rsa.pub -e -m PKCS8 > id_rsa.pem.pub
```

## Generate a One-Time-Use Password to Encrypt the File

The passwords used to encrypt files should be reasonably long 32+ characters, random, and never used twice.
To do this we'll generate a random password which we will use to encrypt the file.

```bash
openssl rand -out key.txt 192
```

## Encrypt
Assuming `myMessage.txt` is your message which should be public-key encrypted.
+ **Encrypt your message**
  ```bash
  openssl aes-256-cbc -in myMessage.txt -out secret.txt.enc -pass file:key.txt
  ```
+ **Encrypt the key**
  ```bash
  openssl rsautl -encrypt -pubin -inkey id_rsa.pem.pub -ssl -in key.txt -out key.enc
  ```

> The result is your encrypted message in `secret.txt.enc`

+ **Compress all of it**
  ```bash
  tar -zcvf secret.tgz key.enc secret.txt.enc
  ```

Now remove the `myMessage.txt` and `key.txt` by:
`rm myMessage.txt key.txt`

## Decrypt

```bash
tar -xzvf secret.tgz
openssl rsautl -decrypt -inkey ~/.ssh/id_rsa -in key.enc -out key.txt
openssl aes-256-cbc -d -in secret.txt.enc -out secret.txt -pass file:key.txt
```
