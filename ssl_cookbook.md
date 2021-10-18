# SSL cookbook


## Generate certificates and private keys

```bash
# define a prefix for files
f_prefix='myhost'
# create private key and self-signed certificate
openssl req -x509 -newkey rsa:2048 -keyout ${name}-private.key -out ${f_prefix}-public.crt -days 3650 -nodes \
            -subj "/C=FR/L=Lille/O=/CN="
# concatenate certificate and private key into a pem file
cat ${f_prefix}-public.crt ${f_prefix}-private.key > ${name}-private.pem
```
