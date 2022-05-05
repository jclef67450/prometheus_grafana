# Fichier de configuration web-config.yml  
Vous devez biensur changer le chemin si ce n'est pas le bon sur votre système.  
### Fichier de prometheus
```
tls_server_config:
  cert_file: /opt/bitnami/prometheus/ssl/prometheus.crt
  key_file: /opt/bitnami/prometheus/ssl/prometheus.key
```  
### Fichier d'alertmanager
```
tls_server_config:
  cert_file: /opt/bitnami/alertmanager/ssl/alertmanager.crt
  key_file: /opt/bitnami/alertmanager/ssl/alertmanager.key
```
