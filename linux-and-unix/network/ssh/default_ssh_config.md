# How to configure ssh package

## First: Stop ssh server

```bash
systemctl stop ssh
```

OpenSSH supports 11 key exchange protocols:

1. curve25519-sha256: ECDH over Curve25519 with SHA2
1. diffie-hellman-group1-sha1: 1024 bit DH with SHA1
1. diffie-hellman-group14-sha1: 2048 bit DH with SHA1
1. diffie-hellman-group14-sha256: 2048 bit DH with SHA2
1. diffie-hellman-group16-sha512: 4096 bit DH with SHA2
1. diffie-hellman-group18-sha512: 8192 bit DH with SHA2
1. diffie-hellman-group-exchange-sha1: Custom DH with SHA1
1. diffie-hellman-group-exchange-sha256: Custom DH with SHA2
1. ecdh-sha2-nistp256: ECDH over NIST P-256 with SHA2
1. ecdh-sha2-nistp384: ECDH over NIST P-384 with SHA2
1. ecdh-sha2-nistp521: ECDH over NIST P-521 with SHA2

If you chose to enable 8,
open `/etc/ssh/moduli` if exists,
and delete lines where the 5th column is less than 2000.

```bash
awk '$5 > 2000' /etc/ssh/moduli > "${HOME}/moduli"
wc -l "${HOME}/moduli" # make sure there is something left
mv "${HOME}/moduli" /etc/ssh/moduli
```

If it does NOT exist,
create it:
```bash
ssh-keygen -G /etc/ssh/moduli.all -b 4096
ssh-keygen -T /etc/ssh/moduli.safe -f /etc/ssh/moduli.all
mv /etc/ssh/moduli.safe /etc/ssh/moduli
rm /etc/ssh/moduli.all
```
This will take a while so continue while it's running.

## OpenSSH Server

```sshdconfig
Protocol 2
HostKey /etc/ssh/ssh_host_ed25519_key
HostKey /etc/ssh/ssh_host_rsa_key
KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-512,hmac-sha2-256,umac-128@openssh.com

PermitRootLogin no
ChallengeResponseAuthentication no
PermitEmptyPasswords no
UsePAM no

PrintMotd no
AcceptEnv LANG LC_*

PasswordAuthentication no
PubkeyAuthentication yes
X11Forwarding no

ClientAliveInterval 300
ClientAliveCountMax 0

IgnoreRhosts yes

HostbasedAuthentication no
```

## OpenSSH Client

```sshconfig
HashKnownHosts yes
Host github.com
	MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512
Host *
	PasswordAuthentication no
	ChallengeResponseAuthentication no

	UseRoaming no
	SendEnv LANG LC_*

	ConnectTimeout 30
	KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256
	MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-512,hmac-sha2-256,umac-128@openssh.com
	Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
	ServerAliveInterval 10
	ControlMaster auto
	ControlPersist yes
	ControlPath ~/.ssh/socket-%r@%h:%p
```

## End: Reload ssh server

Reload the ssh server:

```bash
sudo systemctl reload ssh
```

## References

- https://cipherli.st/
- https://stribika.github.io/2015/01/04/secure-secure-shell.html
