# Zertifizierungsstelle
## OpenSSL
### Debian 12.8
1. **System aktualisieren**
```console
ca@sollca:~# sudo apt update
ca@sollca:~# sudo apt upgrade
```
1. **Privater Schlüssel erzeugen**
```console
ca@sollca:~# openssl genpkey \
> -out foo.key \
> -algorithm RSA \
> -pkeyopt rsa_keygen_bits:2048 \
> -aes-256-cbc
```
1. **Ordner anlegen**
```console
ca@sollca:~# mkdir sollit-ca
ca@sollca:~# cd sollit-ca
ca@sollca:~# mkdir db private certs
```
1. **root-ca.conf mit folgendem Inhalt anlegen**
```
[req]
prompt = no
distinguished_name = dn
req_extensions = ext
input_password = PASSPHRASE
[dn]
CN = vpn.sollit.lan
emailAddress = webmaster@sollit.lan
O = SOLLIT
L = Zürich
C = CH
[ext]
subjectAltName = DNS:vpn.sollit.lan,IP:172.16.0.3
```
