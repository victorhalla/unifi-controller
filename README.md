# unifi-controller
This is a Docker for Unifi Controller version 5.6.36 (AMD64). It is necessary to support some devices not suported on newer versions.

# Building locally
If you want to make local modifications to these images for development purposes or just to customize the logic:

```
git clone https://github.com/victorhalla/unifi-controller.git
cd unifi-controller
docker build \
  --no-cache \
  --pull \
  -t victorhalla/unifi-controller:5.6.36 .
```

# Usage
Here are some example snippets to help you get started creating a container.
Create a user and group called unifi with PUID & PGID 1000 and give access to this user and group to directory /etc/unifi.

docker-compose (recommended)
Compatible with docker-compose v2 schemas.

```
---
version: "2.1"
services:
  unifi-controller:
    image: victorhalla/unifi-controller
    container_name: unifi-controller
    environment:
      - PUID=1000
      - PGID=1000
      - MEM_LIMIT=1024M #optional /etc/unifi
    volumes:
      - <path to data>:/config #/etc/unifi
      - <path to letsencrypt>:/config #optional /etc/letsencrypt
    ports:
      - 3478:3478/udp
      - 10001:10001/udp
      - 8080:8080
      - 8443:8443
      - 1900:1900/udp #optional
      - 8843:8843 #optional
      - 8880:8880 #optional
      - 6789:6789 #optional
      - 5514:5514 #optional
    restart: unless-stopped
```

docker cli

```
docker run -d \
  --name=unifi-controller \
  -e PUID=1000 \
  -e PGID=1000 \
  -e MEM_LIMIT=1024M `#optional` \
  -p 3478:3478/udp \
  -p 10001:10001/udp \
  -p 8080:8080 \
  -p 8443:8443 \
  -p 1900:1900/udp `#optional` \
  -p 8843:8843 `#optional` \
  -p 8880:8880 `#optional` \
  -p 6789:6789 `#optional` \
  -p 5514:5514 `#optional` \
  -v /etc/unifi:/config \
  -v /etc/letsencrypt:/etc/letsencrypt `#optional` \
  --restart unless-stopped \
  victorhalla/unifi-controller
  ```

# Use Letsencrypt Certificate
Make sure you already have letsencrypt installed and the certificate generated for your server.
It is also mandatory that container has accees to the directory /etc/letsencrypt:
```
-v /etc/letsencrypt:/etc/letsencrypt `#optional`
```

## Convert cert to PKCS #12 format
```
openssl pkcs12 -export -in /etc/letsencrypt/live/[YOUR SERVER]/cert.pem -inkey /etc/letsencrypt/live/[YOUR SERVER]/privkey.pem -out /etc/letsencrypt/archive/[YOUR SERVER]/cert.p12 -name unifi -CAfile /etc/letsencrypt/live/[YOUR SERVER]
use passowrd as temppass
```

## Back up current keystore file
```
mv -f /etc/unifi/data/keystore /etc/unifi/data/keystore.backup
```

## Load it into the java keystore that UBNT understands
```
keytool -importkeystore -deststorepass aircontrolenterprise -destkeypass aircontrolenterprise -destkeystore /etc/unifi/data/keystore -srckeystore /etc/letsencrypt/archive/plutao.theparker.viottohalla.com.br/cert.p12 -srcstoretype PKCS12 -srcstorepass temppass -alias unifi -noprompt
```

## Clean up and use new cert
```
docker restart unifi-controller
```

# License
GNU GENERAL PUBLIC LICENSE Version 3, 29 June 2007

# Author
Victor Halla - victorhalla@hotmail.com