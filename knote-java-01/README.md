# knote-java
Simple Spring Boot app to take notes

Créer une image docker avec la commande suivante:
docker build -t knote-java .

Pour connecter les conteneurs on doit créer un réseau docker grâce à la commande suivante:
docker network create knote

Maintenant on execute mongodb grace a la commande suivante:
docker run --name=mongo --rm --network=knote mongo
 
Pour exécuter l'application on exécute la commande suivante:
docker run --name=knote-java --rm --network=knote -p 8080:8080 -e MONGO_URL=mongodb://mongo:27017/dev knote-java

Charger l'image du conteneur dans un registre de conteneurs
docker login

Renommer l'image
docker tag knote-java miage2022/knote-java:1.0.0

Télécharger l'image sur docker hub
docker push miage2022/knote-java:1.0.0 

Créer un cluster Kubernetes local (avec minikube)
installer kubectl
https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/#install-kubectl-binary-with-curl-on-windows
curl -LO "https://dl.k8s.io/release/v1.23.0/bin/windows/amd64/kubectl.exe"

Ajouter le path
$oldPath = [Environment]::GetEnvironmentVariable('Path', [EnvironmentVariableTarget]::Machine)
if ($oldPath.Split(';') -inotcontains 'C:\minikube'){ `
  [Environment]::SetEnvironmentVariable('Path', $('{0};C:\minikube' -f $oldPath), [EnvironmentVariableTarget]::Machine) `
}

Créer un cluster
minikube start

Vérifier le cluster avec
kubectl cluster-info

Créer un dossier pour le déploiement 
mkdir kube


Ajout du fichier knote.yaml pour le déploiement

Les quatre premières lignes définissent le type de ressource (Déploiement), 
la version de ce type de ressource ( apps/v1), et le nom de cette ressource spécifique ( knote) 
Ensuite, nous disposons du nombre souhaité de réplicas de votre conteneur

La troisième partie associe la ressource de déploiement aux répliques de pod

Le template.metadata.labels champ définit une étiquette pour les pods qui enveloppent votre conteneur Knote ( app: knote).

Le selector.matchLabels champ sélectionne les pods avec une app: knoteétiquette pour appartenir à cette ressource de déploiement.

La dernière partie du déploiement définit le conteneur réel qu'on souhaite exécuter

Il définit les éléments suivants :

Un nom pour le conteneur (knote)
Le nom de l'image Docker (miage2022/knote-java:1.0.0).
Le port sur lequel le conteneur écoute (8080)
Une variable d'environnement (MONGO_URL) qui sera mise à la disposition du processus dans le conteneur


Définir un service

La première partie est le sélecteur.
Il sélectionne les Pods à exposer en fonction de leurs labels.
Dans ce cas, tous les pods portant l'étiquette app: knote seront exposés par le service.
La prochaine partie est le port
Dans ce cas, le service écoute les requêtes sur le port 80 et les transmet au port 8080 des pods cibles.
La dernière partie est le type de Service
Dans ce cas, le type est LoadBalancer, ce qui rend les pods exposés accessibles depuis l'extérieur du cluste

Définition du niveau de la base de données
dans le fichier mongo.yaml

En principe, un pod MongoDB peut être déployé de la même manière que notre application, c'est-à-dire en définissant une ressource de déploiement et de service.

Cependant, le déploiement de MongoDB nécessite une configuration supplémentaire.



MongoDB nécessite un stockage persistant.
Le déploiement a une structure similaire à l'autre déploiement.

Cependant, il contient un champ supplémentaire que vous n'avez pas encore vu : volumes.

Le volumeschamp définit un volume de stockage nommé storage, qui fait référence à PersistentVolumeClaim.

De plus, le volume est référencé depuis le volumeMountschamp dans la définition du conteneur MongoDB.

Le volumeMountchamp monte le volume référencé sur le chemin spécifié dans le conteneur, qui dans ce cas est /data/db.

Et /data/dbc'est là que MongoDB enregistre ses données.

En d'autres termes, les données de la base de données MongoDB sont stockées 
dans un volume de stockage persistant qui a un cycle de vie indépendant du conteneur MongoDB.

Déploiement de l'application 
kubectl apply -f kube

Démarrer le pod
kubectl get pods --watch
Démarrage du service
minikube service knote --url