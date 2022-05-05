# Docker-compose de Prometheus  #
Avec ce docker-compose, vous aurez:
1. Prometheus disponible sur https://localhost:9090
2. Alertmanager disponible sur https://localhost:9093
3. Node_exporter installer

**Attention**  
Les volumes qui sont mapper ainsi que les fichier et autre parametres doivent etre dans les repertoire cité dans le docker-compose.  
Si vous chnager un fichier d'emplacement ou de chemin, il faut également le changer de le docker-compose.    
Vous remarquerai que les conteneur sont en mode "host" cela veut dire qu'il vont utiliser la carte réseau de l'hote comme si c'étais leur carte propre, il seront donc accessible comme si le role etait en native sur linux.  
L'option "user: root" présente sur prometheus et alertmanager permet d'etre en root dans le conteneur. Personelement, elle est désactiver.

## Spécificité des rôles ##  
### Prometheus ###  
Pour accéder en HTTPS à prometheus, il faut crée un fichier "web-config.yml" (également dans le projet GitHub) qui indique les chemin des certificat.  
En plus de cela, il faut rentre les parametre de commande "--config.file=", et "--web.config.file=".  
Les options "--storage.tsdb.retention.time=" et "--storage.tsdb.wal-compression" permetent d'étendre la rétention des données > 15j. **Faites attention, cela peut prendre beaucoup d'espace disque**.

### Le fameux docker-compose ###
```
version: '2'

services:


########################Prometheus################################
  prometheus:
    image: bitnami/prometheus
    container_name: prometheus
    ports:
      - '9090:9090'
#    user: root
    volumes:
      - prometheus00:/opt/bitnami/prometheus/data/:rw
      - /docker/prometheus/prometheus.yml:/opt/bitnami/prometheus/conf/prometheus.yml
      - /docker/prometheus/rules.yml:/opt/bitnami/prometheus/conf/rules.yml
      - /docker/prometheus/web-config.yml:/opt/bitnami/prometheus/conf/web-config.yml
                 ###########Certificat SSL Prometheus##########
      - /docker/ssl/prometheus/prometheus.crt:/opt/bitnami/prometheus/ssl/prometheus.crt
      - /docker/ssl/prometheus/prometheus.key:/opt/bitnami/prometheus/ssl/prometheus.key
    command:
      --config.file=/opt/bitnami/prometheus/conf/prometheus.yml
      --web.config.file=/opt/bitnami/prometheus/conf/web-config.yml
#      --storage.tsdb.retention.time=1y
#      --storage.tsdb.wal-compression
    network_mode: host
    restart: always

######################Node_exporter################################
  node_exporter:
    image: bitnami/node-exporter
    ports:
      - '9100:9100'
    container_name: node_exporter
    network_mode: host
    restart: always

########################Alertmanager################################
  alertmanager:
    image: bitnami/alertmanager
    ports:
      - '9093:9093'
#    user: root
    container_name: alertmanager
    volumes:
      - /docker/alertmanager/config.yml:/opt/bitnami/alertmanager/conf/config.yml
      - /docker/alertmanager/web-config.yml:/opt/bitnami/alertmanager/conf/web-config.yml
                ##########Certificat SSl Alertmanager###############
      - /docker/ssl/alertmanager/alertmanager.crt:/opt/bitnami/alertmanager/ssl/alertmanager.crt
      - /docker/ssl/alertmanager/alertmanager.key:/opt/bitnami/alertmanager/ssl/alertmanager.key
    command:
      --web.config.file=/opt/bitnami/alertmanager/conf/web-config.yml
      --config.file=/opt/bitnami/alertmanager/conf/config.yml
    environment:
      - ALERTMANAGER_TLS_INSECURE_SKIP_VERIFY=true
      - ALERTMANAGER_TLS_INSECURESKIPVERIFY=true
    network_mode: host
    restart: always

volumes:
  prometheus00:
```
