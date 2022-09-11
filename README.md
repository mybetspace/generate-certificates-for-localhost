## Generating Certificates to run HTTPS locally

You can use the same certificate files on front-end and back-end, just copy the files after generate them as the [last topic](https://github.com/mybetspace/generate-certificates-for-localhost/edit/main/README.md#after-generate-the-certificate-for-both-1) says.


### Windows

1. Install [Chocolate](https://chocolatey.org/) using terminal, opening in ADM mode:
  - using CMD: 
  ```bash 
    @"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "[System.Net.ServicePointManager]::SecurityProtocol = 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
   ```
  - using PowerShell: 
  ```bash 
  Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
  ```
3. Run ``choco install mkcert``
4. Run ``mkcert -install``
5. Run ``mkcert localhost ::1 127.0.0.1`` in to the folder that you want the certificate to be.

### Linux

1. Create a file named `ssh.sh` in a directory of your choice. Or download the `ssh.sh` file from this repository and skip step 2 as well;
2. Put this script inside the file

```bash
#! /bin/bash

if [ "$#" -ne 1 ]
then
  echo "Error: No domain name argument provided"
  echo "Usage: Provide a domain name as an argument"
  exit 1
fi

DOMAIN=$1

# Create root CA & Private key

openssl req -x509 \
            -sha256 -days 356 \
            -nodes \
            -newkey rsa:2048 \
            -subj "/CN=${DOMAIN}/C=US/L=San Fransisco" \
            -keyout rootCA.key -out rootCA.crt 

# Generate Private key 

openssl genrsa -out ${DOMAIN}.key 2048

# Create csf conf

cat > csr.conf <<EOF
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[ dn ]
C = US
ST = California
L = San Fransisco
O = MLopsHub
OU = MlopsHub Dev
CN = ${DOMAIN}

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = ${DOMAIN}
DNS.2 = www.${DOMAIN}
IP.1 = 192.168.1.5 
IP.2 = 192.168.1.6

EOF

# create CSR request using private key

openssl req -new -key ${DOMAIN}.key -out ${DOMAIN}.csr -config csr.conf

# Create a external config file for the certificate

cat > cert.conf <<EOF

authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = ${DOMAIN}

EOF

# Create SSl with self signed CA

openssl x509 -req \
    -in ${DOMAIN}.csr \
    -CA rootCA.crt -CAkey rootCA.key \
    -CAcreateserial -out ${DOMAIN}.crt \
    -days 365 \
    -sha256 -extfile cert.conf
```

3. Set permission for the script: 

```bash
chmod +x ssl.sh
```

4. Run the script for the address that you want, in this case `localhost`:

```bash
./ssl.sh localhost
```

5. In the same folder will generate the files `localhost.crt`, `localhost.key`, and you will follow the topic bellow for those two. But, you will use the file `rootCA.crt`, importing this file in to your browser Certificates settings, to the browser recognize your certificate.

### After generate the certificate for both

After generate the files, create in the project root a folder named `certs` and copy the files into the folder. The name of the files must be: `server.pem` for the certificate and `key.pem` for the key, make sure to rename.
