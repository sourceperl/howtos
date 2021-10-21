# SSL cookbook


## Generate a private key and a self-signed certificate 

```bash
# define a prefix for files
f_prefix='myhost'
# create private key and self-signed certificate
openssl req -x509 -newkey rsa:2048 -keyout ${f_prefix}-private.key -out ${f_prefix}-public.crt -days 3650 -nodes \
            -subj "/C=FR/ST=Haut-de-France/L=Lille/O=/CN="
# concatenate certificate and private key into a pem file
cat ${f_prefix}-public.crt ${f_prefix}-private.key > ${f_prefix}-private.pem
```


## Create a custom authority to sign certificates

Create a root CA.

```bash
# init custom authority: CA-private.key/CA-public.cert
# create a private key (RSA 4096 bit long) PEM encode
openssl genrsa -out CA-private.key 4096
# create a self-signed certicate for this private key
openssl req -new -key CA-private.key -x509 -days 9132 -out CA-public.cert \
            -subj "/C=FR/ST=Haut-de-France/L=Lille/O=My Custom Authority/CN=My Custom Authority Root CA"
```

Sign a certficate with this CA, for host with a FQDN (here myserver.example.com)

```bash
# init custom authority: CA-private.key/CA-public.cert
# create a private key (RSA 2048 bit long) PEM encode
openssl genrsa -out myserver-private.key 2048
# create a CSR (Certificate Signing Request) for this private key
openssl req -new -key myserver-private.key -out myserver-public.csr \
            -subj "/C=FR/ST=Haut-de-France/L=Lille/O=My Little Company/OU=MLC server division/CN=myserver.example.com"
# sign this CSR, produce a CERT
openssl x509 -req -days 1825 -in myserver-public.csr -CA CA-public.cert -CAkey CA-private.key -CAcreateserial -out myserver-public.cert
```


## Some useful commands

Read a PEM encode certficate as text

```bash
openssl x509 -in myserver-public.cert -text -noout
```

Extract subject from a certficate

```bash
openssl crl2pkcs7 -nocrl -certfile myserver-public.cert | openssl pkcs7 -print_certs -noout | grep subject
```


## Browser certficate manager shortcut

### Chrome

chrome://settings/certificates

### Firefox

about:preferences#privacy
