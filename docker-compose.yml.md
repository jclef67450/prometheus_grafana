# Docker-compose de Prometheus  #
Avec ce docker-compose, vous aurez:
1. Prometheus disponible sur https://localhost:9090
2. Alertmanager disponible sur https://localhost:9093
3. Node_exporter installé

**Attention**  
Les volumes qui sont mappés ainsi que les fichiers et autre parametres doivent être dans les répertoire cités dans le docker-compose.  
Si vous chnagez un fichier d'emplacement ou de chemin, il faut également le changer dans le docker-compose.    
Vous remarquerez que les conteneurs sont en mode "host" cela veut dire qu'ils vont utiliser la carte réseau de l'hote comme si c'étais leur carte propre, ils seront donc accessibles comme si le rôle était en native sur linux.  
L'option "user: root" présente sur prometheus et alertmanager permet d'être en root dans le conteneur. Personnellement, elle est désactivée.

## Spécificité des rôles ##  
### Prometheus ###  
J'ai mappé un volume fixe de la machine pour ne pas perdre toutes les données à chaque redémarrage.  
Pour accéder en HTTPS à prometheus, il faut créer un fichier "web-config.yml" (également dans le projet GitHub) qui indique les chemins des certificats.  
En plus de cela, il faut rentré les parametres de commande "--config.file=", et "--web.config.file=".  
Les options "--storage.tsdb.retention.time=" et "--storage.tsdb.wal-compression" permet d'étendre la rétention des données > 15j. **Faites attentions, cela peut prendre beaucoup d'espaces disque**.  

### Node_exporter ###  
Comme indiquer dans le fichier texte, rien de particulier sur ce rôle.  

### Alertmanager ###  
Comme pour prometheus, alertmanager à besoin d'un fichier "web-config.yml" pour l'accès en HTTPS et également les commandes de lancement.  
Dans mon cas, je vais utiliser un serveur SMTP sans SSL/TLS, il faut donc rajouter les 2 variable d'environement dans le docker-compose.  

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
