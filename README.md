# **Monitoring Via Prometheus, Node_exporter, Grafana sous Docker** #  
## Pour des raisons de sécurité, je cacherais les noms de machine et/ou adresse IP, car le projet à été réalisé dans un cadre pro ##
## **Information Importante** ##  
1. J'ai réalisé ce projet sous linux (**Ubuntu / 20.04**)  
2. J'ai utilisé des images docker de "**Bitnami**", ce sont donc des images dites "**non root**", vous l'aurez donc compris, quand vous serez dans le docker, vous n'aurez pas d'accès root.  
3. Les images Bitnami, utilisent l'utilisateur **1001** de votre système, il faut donc faire attention pour les droits dans le conteneur qui je vous le rappelle n'aurai pas d'accès root. Pour remédier à cela, vous verrez que j'ai crée un répertoire de travail racine qui appartient a mon utilisateur 1001 (dockeruser) que j'ai crée pour ce projet.  
   *Il est quand même possible d'avoir l'utilisateur root via une commande d'environnement à préciser à la création du docker*
4. **Les fichiers annexes comme le docker-compose, et les fichiers de conf seront dans le projet GitHub**

## **Mise en place de l'infrastructure** ##  
Pour ce projet vous aurez besoin de plusieurs choses:  
* Un OS Linux  
* Accès à internet
* La possibilité de communiquer avec plusieur machines sur votre réseau
* Node_exporter installé sur les instances à monitorer
* Docker et Docker-compose d'installer  
* Une BDD MySql ou Postgre
* AU CHOIX, Grafana en natif sur linux ou un docker grafana
* OPTIONNEL PhpMyAdmin

**Détail des images qui vont suivre:**
1. Schéma de l'infrastructure système
2. Répertoire de travail racine (docker) 
3. Détail du répertoire racine
4. Base de données MySql pour Grafana

![Schéma de l'infrastructure](https://zupimages.net/up/22/18/3kmj.png)  

![répertoire de travail](https://zupimages.net/up/22/18/qjat.png)  

![Détail du répertoire](https://zupimages.net/up/22/18/1xsa.png)

![Base de données MySql](https://zupimages.net/up/22/18/j9v4.png)

Par rapport à l'infrastructure système, j'ai préféré utiliser une version linux native de grafana ainsi qu'une BDD Mysql reliée à Grafana.  
*Vous pouvez utiliser un grafana sous docker mais il faudra dans ce cas la "mapper" tous les fichiers de config liée au paramétrage.*
  
**Pourquoi ?**  

Je trouve que le fait d'avoir une BDD fixe au système est plus sécurisé en cas de coupure du docker et des services. De ce fait, les données de grafana seront toujours présent et ceci même si docker est désinstallé.  

Pour remédier au problème de l'utilisateur 1001, j'ai crée un répertoire "docker" à la racine du système qui sera notre environnement de travail.  

J'ai également donné les droits à ce répertoire à notre utilisateur 1001 qui dans mon cas s'apelle "dockeruser".  
Rappel de comande pour attribuer les droits au répertoire:  
`chown -R dockeruser:dockeruser *`  

Pour la BDD, vous êtes assez libre. Créer simplement une BDD sous mysql, mariadb ou postrge. Il faudra lier la BDD dans les fichiers de configuration de grafana. *Je vous montre bien sûr cela plus tard*.  

### **Dans la pratique, sa donne quoi ?** ###  
#### Le principe :  ####
1. Vous aurez un docker-compose.yml dédié à portainer.io qui sera indépendant de celui de prometheus.  
2. Vous avez votre docker-compose.yml qui créera vos conteneurs Prometheus, Node_exporter et Alertmanager  
3. De ce fait, quand une modification est faite dans un fichier de configuration, il suffira de redémarrer le conteneur en question.  
Le tous est facilité via portainer.io avec l'interface graphique.  
4. Vous recevrez les alertes, liée à vos règles d'alerte prédéfinit, de ce fait vous pourrez aller soit sur prometheus soit sur grafana et visualiser via les dashboard ce qui cloche sur votre infrastructure.

## Configuration des rôles ##  
**Rappel: Vous avez les fichiers de configuration type dans le projet GitHub**  
### Prometheus ###  
1. Prometheus va "scraper" les instances cibles. Dans mon cas, je vais créer 6 jobs (groupe de machine).
2. Il sera accessible via HTTPS. (via un fichier web-config.yml).
3. Donnera les instructions d'alertes à Alertmanager sur le port 9093.  

### Alertmanager ###
1. Envera les alertes par mail au destinataire voulu (destination multiple possible).
2. Sera accessible via l'interface WEB en HTTPS (via un fichier web-config.yml également).

### Grafana ###
1. Grafana sera accessible via le port 3000 en HTTPS et sera liée à la base de données MySql.
2. Vous pourrez visualiser les graphiques de métriques de vos instances en direct, en somme, en moyenne etc.
3. Vous pourrez vous connecter via un LDAP.

 ### Portainer.io ###
1. Portainer sera accessible via le port 9443 en HTTPS
2. Il vous permettra de gérer les volumes, images conteneurs et autres via l'interface graphique.
3. Connexion via LDAP possible.  

### MySql ###
1. Libre à vous, le but est simplement d'avoir une BDD vierge.  
   
### Node_exporter ###
1. Node_exporter ne nécessite aucun paramétrage.  
   
### Certificat SSL
1. Pensez soit à crée des certificats SSL avec openssl, nginx ou autre.
2. Utiliser un conteneur qui gère les certificats comme traefik ou caddy.

## Après l'installation des rôles ##  
Après avoir installé les rôles, vous devez encore faire quelque petit paramétrage.  
1. Sur l'interface WEB de Grafana, il faut lier la BDD prometheus pour pouvoir lire les données sur les graphiques.
2. Crée des dashboard pour lire les données prometheus.  

### Liaison à la BDD ###
Pour lier la BDD prometheus c'est très simple:  
1. Sur votre espace WEB, cliquer sur l'écrou à gouche et sur "Data Sources".
2. Cliquez sur "Add data source" et sélectionner prometheus.
3. Remplissez les informations comme ceci :

![Configuration de prometheus](https://zupimages.net/up/22/18/x6vk.png)

**Info**: Si vous êtes en HTTPS, cocher la case "Skip TLS Verify", sinon laisser la décocher.  
### Création de Dashboard ###  
Pour crée des Dashboard vous avez 3 solutions :  
1. Utiliser un dashboard de [grafana dashboard](https://grafana.com/grafana/dashboards/).
2. Créé vous-même un dashboard en JS.
3. Utilisé les dashboard [grafana dashboard](https://grafana.com/grafana/dashboards/) pour crée des dashboard personalisé. (il faudra surement modifié un peut les requetes mais je pense que cela est quand même plus simple que de les écrires de A a Z.)  

Une foix votre méthode choisie, cliquer sur le logo "+" à gauche et "Import".  
1. Si vous utilisez un dashboard pré-fait, il faut indiquer le numéro de dashboard comme ceci, est cliqué sur load : 
   ![Dashboard importé](https://zupimages.net/up/22/18/x881.png "Cliquer sur load quand le code est rentrer") 
2. Si vous crée votre dashboard, cliquez sur "Upload JSON file".  

Voilà, vous pouvez maintenant lire vos données, ou modifier les dashboard à votre sauce.  
## Autres Infos utiles  
1. J'ai créé des alias de commande docker pour se faciliter la tâche, les voici, libre à vous de les utiliser : 
```
# Mes alias
alias update='apt -y update && apt -y upgrade'
# Exécuter le docker-compose du répertoire actuel
alias dockercompose='docker-compose up -d'
# Aller dans le répertoire de travail de docker
alias cdd='cd /docker'
# Stoper tous les docker du docker-compose
alias dockerstop='docker-compose stop'
# Redemarer tous les docker du docker-compose
alias dockerrestart='docker-compose stop && docker-compose up -d'
# Visualiser les port du par-feu ouvert
alias ufwlist='ufw status numbered | grep v6'
# Purger tous les  volumes, images et autre non utiliser (a utiliser avec les docker allumé)
alias dockerpurge='docker system prune -a'
```  
2. Si vous avez trop d'alerte d'un coup, il se peut qu'alertmanager stop l'envoie de mail, ou les envoyés plus lentement.
3. Prometheus garde nativement les données sur une période de 15j, cela peut être augmenté avec le paramètre comme indiquer dans le docker-compose mais cela augmentera grandement l'espace disque nécessaire.
4. Le problème de rétention des données longues, peut être résolu en installant "Thanos", mais cela nécessite un déploiement assez gros et long à installer / paramétrer.
5. Vous pouvez bien sûr utiliser une 2nd BDD dans grafana en plus de prometheus. J'ai moi même une source Prometheus et une source InfluxDB sur le même Grafana.
6. Quand vous changer un fichier de configuration d'un service, il faut redémarrer le conteneur. 
7. Si un fichier de configuration est faux ou indique une erreur, le conteneur ne démarre pas ou va tenter de redémarrer en boucle. Vous pouvez regarder les logs via portainer.
