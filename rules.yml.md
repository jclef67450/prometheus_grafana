# Fichier de configuration des règles d'alert prometheus #  
Rien de particulier à dire ici, j'ai créé les alertes qui me semble essentielles à un monitoring PRO.  
Vous pouvez bien sûr crée vos règles personalisées.
```
groups:
##########################################################################
#                                                                        #
#                       CONFIGURATION GLOBAL                             #
#                                                                        #
##########################################################################
  - name: Alertmanager
    rules:
      - alert: Global_MaxSwap
        expr: (1 - (node_memory_SwapFree_bytes / node_memory_SwapTotal_bytes)) * 100 > 40
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "{{$labels.instance}} - SWAP Surchargé"
          description: "SWAP utilisé à + 40%"

      - alert: Global_MaxRAM
        expr: node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100 < 20
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "{{$labels.instance}} - RAM Surchargé"
          description: "Utilisation de la RAM > 80%"

      - alert: Global_MaxCPU
        expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "{{$labels.instance}} - CPU Surchargé"
          description: "Utilisation du CPU >= 80%"

      - alert: Global_MaxTemperature
        expr: node_hwmon_temp_celsius > 90
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "{{$labels.instance}} - Température trop haute"
          description: "Un composant physique est trop chaud"

      - alert: InstanceDown
        expr: up == 0
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "{{$labels.instance}} - Machine Hors-tension"
          description: "La machine c'est éteinte"

      - alert: Global_DiskSpace
        expr: (node_filesystem_avail_bytes * 100) / node_filesystem_size_bytes < 10 and ON (instance, device, mountpoint) node_filesystem_readonly == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "{{$labels.instance}} - Stockage Saturé"
          description: "L'espace système est utilisé à + 90%"
```
