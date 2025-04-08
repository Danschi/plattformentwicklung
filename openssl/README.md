# Zertifizierungsstelle mit OpenSSL

**Ordnerstruktur anlegen**
```console
mkdir -p root-ca/{conf,db,private,certs} 
```
**In den Ordner wechseln**
```console
cd root-ca
```
**Berechtigung ändern**
```console
chmod 700 private
```
**Index Datei erzeugen**
```console
touch db/index
```
**Serial Datei erzeugen**
```console
openssl rand -hex 16 > db/serial
```
**Crlnumber erzeugen**
```console
echo 1001 > db/crlnumber
```
**root-ca.conf erstellen**
```console
vi conf/root-ca.conf
```
**root-ca.conf inhalt**
```console
[default]
name = root-ca
domain_suffix = beispiel.lan
aia_url = http://$name.$domain_suffix/$name.crt
crl_url = http://$name.$domain_suffix/$name.crl
ocsp_url = http://ocsp.$name.$domain_suffix:9080
default_ca = ca_default
name_opt = utf8,esc_ctrl,multiline,lname,align

[ca_dn]
countryName = "CH"
organizationName = "Beispiel GmbH"
commonName = "Beispiel Root CA"

[ca_default]
home = . 
database = $home/db/index
serial = $home/db/serial
crlnumber = $home/db/crlnumber
certificate = $home/$name.crt
private_key = $home/private/$name.key
RANDFILE = $home/private/random
new_certs_dir = $home/certs
unique_subject = no
copy_extensions = copy 
default_days = 3650
default_crl_days = 365
default_md = sha256
policy = policy_c_o_match

[policy_c_o_match]
countryName = match
stateOrProvinceName = optional
organizationName = match
organizationalUnitName  = optional
commonName = supplied
emailAddress = optional

[req]
default_bits = 4096
encrypt_key = yes
default_md = sha256
utf8 = yes
string_mask = utf8only
prompt = no
distinguished_name = ca_dn
req_extensions = ca_ext

[ca_ext]
basicConstraints = critical,CA:true
keyUsage = critical,keyCertSign,cRLSign
subjectKeyIdentifier = hash

[sub_ca_ext]
authorityInfoAccess = @issuer_info
authorityKeyIdentifier = keyid:always
basicConstraints = critical,CA:true,pathlen:0
crlDistributionPoints = @crl_info
extendedKeyUsage = clientAuth,serverAuth
keyUsage = critical,keyCertSign,cRLSign
nameConstraints = @name_constraints
subjectKeyIdentifier = hash

[server_ext]
authorityInfoAccess = @issuer_info
authorityKeyIdentifier = keyid:always
basicConstraints = critical,CA:false
crlDistributionPoints = @crl_info
extendedKeyUsage = clientAuth,serverAuth
keyUsage = critical,digitalSignature,keyEncipherment
subjectKeyIdentifier = hash

[client_ext]
authorityInfoAccess = @issuer_info
authorityKeyIdentifier = keyid:always
basicConstraints = critical,CA:false
crlDistributionPoints = @crl_info
extendedKeyUsage = clientAuth
keyUsage = critical,digitalSignature
subjectKeyIdentifier = hash

[crl_info]
URI.0 = $crl_url

[issuer_info]
caIssuers;URI.0 = $aia_url
OCSP;URI.0 = $ocsp_url

[name_constraints]
excluded;IP.0=0.0.0.0/0.0.0.0
excluded;IP.1=0:0:0:0:0:0:0:0/0:0:0:0:0:0:0:0

[ocsp_ext]
authorityKeyIdentifier = keyid:always
basicConstraints = critical,CA:false
extendedKeyUsage = OCSPSigning
keyUsage = critical,digitalSignature
subjectKeyIdentifier = hash
```
**webserver.conf erstellen**
```console
vi conf/webserver.conf
```
**webserver.conf inhalt**
```console
[req]
prompt = no
distinguished_name = dn
req_extensions = ext
input_password = CHANGEME

[dn]
CN = www.beispiel.lan
emailAddress = info@beispiel.lan
O = Beispiel GmbH
L = Zurich
C = CH

[ext]
subjectAltName = IP:192.168.122.166
```
**Root-CA Schlüssel und CSR erzeugen**
```console
openssl req -new -config conf/root-ca.conf -out root-ca.csr -keyout private/root-ca.key
```
**Root-CA Selbstsigniertes Zertifikat erstellen**
```console
openssl ca -selfsign -config conf/root-ca.conf -in root-ca.csr -out root-ca.crt -extensions ca_ext
```
**Webserver Privater Schlüssel erzeugen**
```console
openssl genpkey -out webserver.key -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -aes-128-cbc
```
**Webserver CSR erzeugen**
```console
openssl req -new --config conf/webserver.conf -key webserver.key -out webserver.csr
```
**Webserverzertifikat ausstellen**
```console
openssl ca -config conf/root-ca.conf -in webserver.csr -out webserver.crt -extensions server_ext
```
