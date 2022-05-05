# **Monitoring Via Prometheus, Node_exporter, Grafana sous Docker** #  
## *Projet en cour de dévelopement* ##  
### *De potentiel fichier ne sont pas encore disponible.* ###  
Bonjour,  
Je vais vous montrer dans ce projet, comment crée un monitoring complet sous prometheus.  
## **Information Importante** ##  
1. J'ai réaliser ce projet sous linux (**Ubuntu / 20.04**)  
2. J'ai utiliser des images docker de "**Bitnami**", ce sont donc des images dites "**non root**", vous l'aurez donc compris, quand vous serez dans le docker, vous n'aurait pas d'accès root.  
3. Les images Bitnami, utilisent l'utilisateur **1001** de votre système, il faut donc faire attention pour les droit dans le conteneur qui je vous le rappel n'aura pas d'accès root. Pour remédier a cela, vous verrer que j'ai crée un repertoire de travail racine qui appartient a mon utilisateur 1001 (dockeruser) que j'ai crée pour ce projet.

## **Mise en place de l'infrastructure** ##  
Pour ce projet vous aurez besoin de plusieurs chose:  
* Un OS Linux  
* Accès à internet
* La possibilité de pouvoir communiquez avec plusieur machines sur votre réseaux
* Node_exporter installer sur les instances à monitorées
* Docker et Docker-compose d'installer  
* OPTIONEL PhpMyAdmin

**Détail des images qui vont suivre:**
1. Schéma de l'infrastructure système
2. Repertoire de travail racine (docker) 
3. Détail du repertoire racine
4. Base données MySql pour Grafana

![Schéma de l'infrastructure](https://zupimages.net/up/22/18/3kmj.png)  

![Repertoire de travail](https://zupimages.net/up/22/18/gq85.png)  

![Détail du répertoire](https://zupimages.net/up/22/18/1xsa.png)

![Base de données MySql](https://zupimages.net/up/22/18/j9v4.png)
