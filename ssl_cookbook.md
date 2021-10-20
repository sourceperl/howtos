# SSL cookbook


## Browser certficate manager shortcut

### Chrome

chrome://settings/certificates

### Firefox

about:preferences#privacy


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

```bash
# init custom authority: CA-private.key/CA-public.cert
# create a private key (RSA 4096 bit long) PEM encode
openssl genrsa -out CA-private.key 4096
# create a self-signed certicate for this private key
openssl req -new -key CA-private.key -x509 -days 3650 -out CA-public.cert \
            -subj "/C=FR/ST=Haut-de-France/L=Lille/O=My Custom Authority/OU=/CN="
```
