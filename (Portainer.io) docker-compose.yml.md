# Docker-compose de portainer.io  # 
Avec ce docker-compose, portainer sera accessible sur https://localhost:9443  
Pensez bien à placer vos certificats SSL dans le répertoire mappé, ou alors changer le chemin dans le docker-compose.  
Les options de commande permettent de dire au conteneur quel fichier utiliser comme certificat (.crt et .key).  
Les volumes mappés doivent également être au bon endroit .  

```
version: '2'

services:

  portainer:
    image: portainer/portainer-ce
    container_name: portainer.io
    restart: always
    ports:
      - '8000:8000'
      - '9443:9443'
    volumes:
      - 'portainer_data:/data'
      - '/var/run/docker.sock:/var/run/docker.sock'
                  ##########Certificat SSL###########
      - '/docker/ssl/portainer/portainer.crt:/certs/portainer.crt'
      - '/docker/ssl/portainer/portainer.key:/certs/portainer.key'
    command:
      --ssl
      --sslcert /certs/portainer.crt
      --sslkey /certs/portainer.key
    network_mode: host

volumes:
  portainer_data:
```
