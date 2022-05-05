# Fichier de configuration de prometheus #  
1. On indique le chemin de fichier de règles avec l'option "rule_files:".
2. On initialise la liaison alertmanager sur le port 9093.
3. On créer les "job" d'instance à monitorer.  

Vous pouvez constater que j'ai mis l'option de service sur les job, je trouve cela bien plus pratique pour la configuration d'alertmanager avec les routes et les match.  

Vous pouvez également voir comment indiquez les instance, la syntaxe est comme ceci:  
`['machine1:9100', machine2:9100']`  

```
global:
  scrape_interval: 30s
  evaluation_interval: 15s
#  external_labels:
#    foo: bar

rule_files:
  - '/opt/bitnami/prometheus/conf/rules.yml'

alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - localhost:9093
    scheme: https
    tls_config:
      insecure_skip_verify: true
    timeout: 10s

scrape_configs:
###############################################################################

  - job_name: 'Worker'
    scrape_interval: 30s
    static_configs:
      - targets: ['instance1:9100', 'instance2:9100', instance3:9100', instance4:9100']
        labels:
          service: '1'
          
###############################################################################

  - job_name: 'Data'
    scrape_interval: 30s
    static_configs:
      - targets: ['instance1:9100', 'instance2:9100', instance3:9100', instance4:9100']
        labels:
          service: '2'

###############################################################################

  - job_name: 'Services'
    scrape_interval: 30s
    static_configs:
      - targets: ['instance1:9100', 'instance2:9100', instance3:9100', instance4:9100']
        labels:
          service: '3'

###############################################################################

  - job_name: 'UI'
    scrape_interval: 30s
    static_configs:
      - targets: ['instance1:9100', 'instance2:9100', instance3:9100', instance4:9100']
        labels:
          service: '4'

##############################################################################

  - job_name: 'Localhost'
    scrape_interval: 30s
    static_configs:
      - targets: ['localhost:9100']
        labels:
          service: '5'

##############################################################################

  - job_name: 'Serveur_Web'
    scrape_interval: 30s
    static_configs:
      - targets: ['instance1:9100', 'instance2:9100', instance3:9100', instance4:9100']
        labels:
          service: '6'
```
