# SSL cookbook


## Generate a private key and a self-signed certificate

```bash
# define a prefix for files
f_prefix='myhost'
# create private key (RSA 2048 bits long) and a self-signed certificate
openssl req -x509 -newkey rsa:2048 -days 3650 -nodes \
            -subj "/C=FR/ST=Haut-de-France/L=Lille/O=/CN=" \
            -keyout ${f_prefix}-private.key \
            -out ${f_prefix}-public.crt
```

```bash
# if need, concatenate certificate and private key into a pem file
cat ${f_prefix}-public.crt ${f_prefix}-private.key > ${f_prefix}-private.pem
```


## Create a custom authority to sign certificates

Create a root CA.

```bash
# init custom authority: CA-private.key/CA-public.cert
# create a private key (RSA 4096 bits long) PEM encode
openssl genrsa -out CA-private.key 4096
# create a self-signed certicate for this private key
openssl req -new -x509 -days 9132 \
            -subj "/C=FR/ST=Haut-de-France/L=Lille/O=My Custom Authority/CN=My Custom Authority Root CA" \
            -key CA-private.key \
            -out CA-public.cert
```

Sign a certficate with this CA for host with a FQDN (here www.example.com)

```bash
# init custom authority: CA-private.key/CA-public.cert
# create aa private key and a CSR (Certificate Signing Request)
openssl req -new -nodes \
            -newkey rsa:2048 \
            -subj "/C=FR/ST=Haut-de-France/L=Lille/O=My Little Company/OU=MLC server division/CN=www.example.com" \
            -keyout www-private.key \
            -out www-public.csr
# sign this CSR with CA certficate and its private key, produce a signed certficate
openssl x509 -req -days 1825 \
             -CA CA-public.cert -CAkey CA-private.key -CAcreateserial \
             -in www-public.csr \
             -out www-public.cert
```


Sign a certficate with this CA for host with an IPv4 address (here private 192.168.1.45)

```bash
# init custom authority: CA-private.key/CA-public.cert
# build ssl.conf file
cat <<EOF >ssl.conf
[ req ]
prompt             = no
distinguished_name = req_dn
req_extensions     = req_v3

[ req_dn ]
C=FR
ST=Haut-de-France
L=Lille
O=My Little Company
OU=MLC server division
CN=my server name

[ req_v3 ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names

[ alt_names ]
IP.0 = 192.168.1.45
EOF
# create a private key (RSA 2048 bit long) PEM encode
openssl genrsa -out myserver-private.key 2048
# create a CSR (Certificate Signing Request) for this private key
openssl req -new -config ssl.conf \
            -key myserver-private.key \
            -out myserver-public.csr
# sign this CSR, produce a CERT
openssl x509 -req -days 1825 \
             -in myserver-public.csr \
             -CA CA-public.cert -CAkey CA-private.key -CAcreateserial \
             -extfile ssl.conf -extensions req_v3 \
             -out myserver-public.cert
```


## Some useful commands

Read a PEM encode certficate as text

```bash
openssl x509 -text -noout -in myserver-public.cert
```

Extract subject from a certficate

```bash
openssl crl2pkcs7 -nocrl -certfile myserver-public.cert | openssl pkcs7 -print_certs -noout | grep subject
```


## Browser certficate manager shortcut

### Chrome

chrome://settings/security

### Firefox

about:preferences#privacy
