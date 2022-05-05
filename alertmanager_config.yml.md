# Fichier de configuration d'alertmanager #  
Ce fichier permet d'envoyer les alertes à différentes personnes en fonction du service prometheus (voir fichier de configuration "prometheus.yml").
```
global:
  resolve_timeout: 2m
  smtp_require_tls: false
  smtp_smarthost: "SERVEURSMTP:25" 
  smtp_from: "alertmanager@local.fr"

############################################################################

route:
  group_by: ['alertname', 'instance', 'severity']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 6h
  receiver: "email_me"

  routes:
    - receiver: "1"
#      group_wait: 30s
      match_re:
        service: "6"
      continue: true

    - receiver: "2"
#      group_wait: 30s
      match_re:
        service: "1, 2, 3, 4, 5"
      continue: true

############################################################################

receivers:
- name: '2'
  email_configs:
  - to: "XXXX@XXXX.fr"

- name: 'eric'
  email_configs:
  - to: "XXXX@XXXX.fr"
```
