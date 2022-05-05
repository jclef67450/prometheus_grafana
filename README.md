# **Monitoring Via Prometheus, Node_exporter, Grafana sous Docker** #  
## Pour des raison de sécurité, je cacherais les nom de machine et/ou adresse IP, car le projet à été rélaiser dans un cadre pro ##
Bonjour,  
Je vais vous montrer dans ce projet, comment crée un monitoring complet sous prometheus.  
## **Information Importante** ##  
1. J'ai réaliser ce projet sous linux (**Ubuntu / 20.04**)  
2. J'ai utiliser des images docker de "**Bitnami**", ce sont donc des images dites "**non root**", vous l'aurez donc compris, quand vous serez dans le docker, vous n'aurait pas d'accès root.  
3. Les images Bitnami, utilisent l'utilisateur **1001** de votre système, il faut donc faire attention pour les droit dans le conteneur qui je vous le rappel n'aura pas d'accès root. Pour remédier a cela, vous verrer que j'ai crée un repertoire de travail racine qui appartient a mon utilisateur 1001 (dockeruser) que j'ai crée pour ce projet.  
   *Il est quand meme possible d'avoir l'utilisateur root via une commande d'environement a préciser à la création du docker*
4. **Les fichier annexes comme les docker-compose, et les fichiers de conf seront dans le projet comme le README actuel.**

## **Mise en place de l'infrastructure** ##  
Pour ce projet vous aurez besoin de plusieurs chose:  
* Un OS Linux  
* Accès à internet
* La possibilité de communiquez avec plusieur machines sur votre réseaux
* Node_exporter installer sur les instances à monitorées
* Docker et Docker-compose d'installer  
* Une BDD MySql ou Postgre
* AU CHOIX, Grafana en natif sur linux ou un docker grafana
* OPTIONEL PhpMyAdmin

**Détail des images qui vont suivre:**
1. Schéma de l'infrastructure système
2. Repertoire de travail racine (docker) 
3. Détail du repertoire racine
4. Base données MySql pour Grafana

![Schéma de l'infrastructure](https://zupimages.net/up/22/18/3kmj.png)  

![Repertoire de travail](https://zupimages.net/up/22/18/qjat.png)  

![Détail du répertoire](https://zupimages.net/up/22/18/1xsa.png)

![Base de données MySql](https://zupimages.net/up/22/18/j9v4.png)

Par rapport a l'infrastructure système, j'ai préférer utiliser une version linux native de grafana ainsi qu'une BDD Mysql reliée à Grafana.  
*Vous pouvez utiliser un grafana sous docker mais il faudra dans ce cas la "mapper" tous les fichiers de config liée au parametrage.*
  
**Pourquoi ?**  

Je trouve que le fait d'avoir une BDD fixe au système est plus sécurisé en cas de coupure du docker et des services. De ce fait, les données de grafana seront toujours présente et ceci meme si docker est désinstaller.  

Pour remedier au problème de l'utilisateur 1001, j'ai créer un repertoire "docker" à la racine du système qui sera notre environement de travail.  

J'ai également donnée les droit à ce repertoire a notre utilisateur 1001 qui dans mon cas s'apelle "dockeruser".  
Rapelle de comande pour attribuer les droits au repertoire:  
`chown -R dockeruser:dockeruser *`  

Pour la BDD, vous etes assez libre. Créer simplement une BDD sous mysql, mariadb ou postrge. Il faudra liée la BDD dans les fichier de configuration de grafana. *Je vous montre biensur cela plustard*.  

### **Dans la pratique, sa donne quoi ?** ###  
#### Le principe :  ####
1. Vous aurez un docker-compose.yml dédié a portainer.io qui sera indépendant de celui de prometheus.  
2. Vous avez votre docker-compose.yml qui créera vos conteneurs Prometheus, Node_exporter et Alertmanager  
3. De ce fait, quand une modification est faites dans un fichier de configuration, il suffirat de redémarer le conteneur en question.  
le tous est facilité via portainer.io avec l'interface graphique.  
4. Vous recevrez les alertes, liée à vos règles d'alerte prédéfinit, de ce fait vous pourez aller soit sur prometheus soit sur grafana et visulaiser via les dashboard ce qui cloche sur votre infrastructure.

## Configuration des rôles ##  
**Rappel: Vous avez les fichiers de configuration type dans le projet GitHub**  
### Prometheus ###  
1. Prometheus va "scraper" les instance cibles. Dans mon cas, je vais crée 6 job (groupe de machine).
2. Il sera acessible via HTTPS. (via un fichier web-config.yml).
3. Envera les instructions d'alerte à Alertmanager sur le port 9093.  

### Alertmanager ###
1. Envera les alertes par mail au destinataire voulue (destination multiple possible).
2. Sera accessible via l'interface WEB en HTTPS (via un fichier web-config.yml également).

### Grafana ###
1. Grafana sera accessible via le port 3000 en HTTPS et sera liée a la base de données MySql.
2. Vous pourrez visualiser les graphique de métrique de vos instance en direct, en somme, en moyenne etc.
3. Vous pourrez vous connecter via un LDAP.

 ### Portainer.io ###
1. Portainer sera accessible via le port 9443 en HTTPS
2. Il vous permettra de gérer les volumes, images conteneurs et autres via l'interface graphique.
3. Connexion via LDAP possible.  

### MySql ###
1. Libre à vous, le but et simplement d'avoir une BDD vierge.  
   
### Node_exporter ###
1. Node_exporter ne nécéssite aucun parametrage.  
   
### Certificat SSL
1. Pensez soit à crée des certificat SSL avec openssl, nginx ou autre.
2. Utiliser un conteuneur qui gère les certificat comme traefik ou caddy.

## Après l'installation des rôles ##  
Après avoir installer les rôles, vous devez encore faire quelque petit parametrage.  
1. Sur l'interface WEB de Grafana, il faut liée la BDD prometheus pour pouvoir lire les données sur les graphique.
2. Crée des dashboard pour lire les données prometheus.  

### Liaison à la BDD ###
Pour liée la BDD prometheus c'est très simple:  
1. Sur votre espace WEB, cliquer sur l'écrou à gouche et sur "Data Sources".
2. Cliquez sur "Add data source" et selectionner prometheus.
3. Remplisser les informations comme ceci :

![Configuration de prometheus](https://zupimages.net/up/22/18/x6vk.png)

**Info**: Si vous etes en HTTPS, cocher la case "Skip TLS Verify", sinon laiser la décocher.  
### Création de Dashboard ###  
Pour crée des Dashboard vous avez 3 solutions :  
1. Utiliser un dashboard de [grafana dashboard](https://grafana.com/grafana/dashboards/).
2. Crée vous même un dashboard en JS.
3. Utilisé les dashboard [grafana dashboard](https://grafana.com/grafana/dashboards/) pour crée des dashboard personalisé. (il faudra surement modifé un peux les requete mais je pense que cela est quand meme plus simple que de les écrire de A a Z.)  

Une foix votre methode choisie, cliquer sur le logo "+" à gauche et "Import".  
1. Si vous utiliser un dashboard préfait, il faut indiquer le numéro de dashboard comme ceci, est cliquer sur load : 
   ![Dashboard importé](https://zupimages.net/up/22/18/x881.png "Cliquer sur load quand le code est rentrer") 
2. Si vous crée votre dashboard, cliquez sur "Upload JSON file".  

Voila, vous pouvez mainteant lire vos données, ou modifier les dashboard à votre sauce.  
## Autres Infos utiles  
1. J'ai créer des alias de commande docker pour se faciliter la taches, les voici, libre à vous de les utilisés : 
```
# Mes alias
alias update='apt -y update && apt -y upgrade'
# Exécuter le docker-compose du repertoire actuel
alias dockercompose='docker-compose up -d'
# Aller dans le repertoire de travail de docker
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
2. Si vous avez trop d'alerte d'un coup, il se peut que alertmanager stop l'envoie de mail, ou les envoyes plus lentement.
3. Prometheus garde nativement les données sur une periode de 15j, cela peut etre augmenter avec le parametre comme indiquer dans le docker-compose mais cela augmentera grandement l'espace disque nécéssaire.
4. Le problème de rétention des données longues, peut etre résolu en installant "thanos", mais cela nécéssite un déploiement assez gros et long à installer / paramètrer.
5. Vous pouvez biensur utiliser une 2 ème BDD dans grafana en plus de prometheus. J'ai moi meme une source Prometheus et une source InfluxDB sur le meme Grafana.
6. Quand vous changer un fichier de configuration d'un services, il faut redémarer le conteneur. 
7. Si un fichiers de configuration est fautou indique une erreur, le conteneur ne démarre pas ou va tenter de redémarrer en boucle. Vous pouvez regarder les log via portainer.
