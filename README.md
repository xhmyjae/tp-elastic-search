# tp-elastic-search

## Partie 1 : Installation et configuration du cluser Redis

### 1. Installation de Redis et configuration du cluser

J'ai décidé d'utiliser Docker pour installer Redis. J'ai donc créé un fichier `docker-compose.yml` pour définir le service Redis. Voici le contenu du fichier :

```yaml
version: '3.8'

services:
  redis-node-1:
    image: redis:latest
    container_name: redis-node-1
    command: ["redis-server", "--cluster-enabled", "yes", "--cluster-config-file", "/data/nodes.conf", "--cluster-node-timeout", "5000", "--appendonly", "yes"]
    volumes:
      - redis-node-1-data:/data
    ports:
      - "6379:6379"
    networks:
        redis-cluster:
            ipv4_address: 172.28.0.2

  redis-node-2:
    image: redis:latest
    container_name: redis-node-2
    command: ["redis-server", "--cluster-enabled", "yes", "--cluster-config-file", "/data/nodes.conf", "--cluster-node-timeout", "5000", "--appendonly", "yes"]
    volumes:
      - redis-node-2-data:/data
    ports:
      - "6380:6379"
    networks:
        redis-cluster:
            ipv4_address: 172.28.0.3

  redis-node-3:
    image: redis:latest
    container_name: redis-node-3
    command: ["redis-server", "--cluster-enabled", "yes", "--cluster-config-file", "/data/nodes.conf", "--cluster-node-timeout", "5000", "--appendonly", "yes"]
    volumes:
      - redis-node-3-data:/data
    ports:
      - "6381:6379"
    networks:
        redis-cluster:
            ipv4_address: 172.28.0.4
            
  redis-node-4:
    image: redis:latest
    container_name: redis-node-4
    command: ["redis-server", "--cluster-enabled", "yes", "--cluster-config-file", "/data/nodes.conf", "--cluster-node-timeout", "5000", "--appendonly", "yes"]
    volumes:
      - redis-node-4-data:/data
    ports:
      - "6382:6379"
    networks:
        redis-cluster:
            ipv4_address: 172.28.0.5
            
  redis-node-5:
    image: redis:latest
    container_name: redis-node-5
    command: ["redis-server", "--cluster-enabled", "yes", "--cluster-config-file", "/data/nodes.conf", "--cluster-node-timeout", "5000", "--appendonly", "yes"]
    volumes:
      - redis-node-5-data:/data
    ports:
      - "6383:6379"
    networks:
        redis-cluster:
            ipv4_address: 172.28.0.6
  
  redis-node-6:
    image: redis:latest
    container_name: redis-node-6
    command: ["redis-server", "--cluster-enabled", "yes", "--cluster-config-file", "/data/nodes.conf", "--cluster-node-timeout", "5000", "--appendonly", "yes"]
    volumes:
      - redis-node-6-data:/data
    ports:
      - "6384:6379"
    networks:
        redis-cluster:
            ipv4_address: 172.28.0.7

networks:
  redis-cluster:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16

volumes:
    redis-node-1-data:
    redis-node-2-data:
    redis-node-3-data:
    redis-node-4-data:
    redis-node-5-data:
    redis-node-6-data:
```

On définit trois services Redis, chacun avec une adresse IP différente. On définit également un réseau `redis-cluster` pour que les services puissent communiquer entre eux.
On démarre ensuite les services avec la commande `docker-compose up`.

### 2. Démarrage du cluser Redis

On accède à l'un des conteneurs Docker pour initialiser le cluster Redis avec la commande `docker exec -it redis-node-1 redis-cli --cluster create 172.28.0.2:6379 172.28.0.3:6379 172.28.0.4:6379 172.28.0.5:6379 172.28.0.6:6379 172.28.0.7:6379 --cluster-replicas 1`.

On peut vérifier que le cluster a bien été créé avec la commande `docker exec -it redis-node-1 redis-cli cluster nodes`.

## Partie 2 : Premiers pas avec le cluster Redis

### 1. Injection de données

Insertion String :
```shell
docker exec -it redis-node-1 redis-cli -c SET keyString1 "valeur de key1"
```
Cette commande permet d'insérer une clé `keyString1` avec la valeur `valeur de key1` dans le cluster Redis.
_`docker exec -it redis-node-1 redis-cli` permet d'exécuter une commande redis dans le conteneur redis-node-1 en mode intéractif._
_Le flag `-c` permet de spécifier que l'on veut utiliser le mode cluster et gérer la répartition des données automatiquement._
> La commande renvoie : `OK`

Insertion Lists : 
```shell
docker exec -it redis-node-1 redis-cli -c LPUSH keyList1 "valeur1" "valeur2" "valeur3"
```
Cette commande permet d'insérer une liste `keyList1` avec les valeurs `valeur1`, `valeur2` et `valeur3` dans le cluster Redis.
> La commande renvoie : `(integer) 3`

Insertion Sets :
```shell
docker exec -it redis-node-1 redis-cli -c SADD keySet1 "valeurSet1" "valeurSet2" "valeurSet3"
```
Cette commande permet d'insérer un set `keySet1` avec les valeurs `valeurSet1`, `valeurSet2` et `valeurSet3` dans le cluster Redis.
> La commande renvoie : `(integer) 3`

Insertion Hashes :
```shell
docker exec -it redis-node-1 redis-cli -c HMSET keyHash1 field1 "valeurHash1" field2 "valeurHash2" field3 "valeurHash3"
```
Cette commande permet d'insérer un hash `keyHash1` avec les champs `field1`, `field2` et `field3` et les valeurs `valeurHash1`, `valeurHash2` et `valeurHash3` dans le cluster Redis.
> La commande renvoie : `OK`

Insertion Sorted Sets :
```shell
docker exec -it redis-node-1 redis-cli -c ZADD keySortedSet1 1 "valeurSorted1" 2 "valeurSorted2" 3 "valeurSorted3"
```
Cette commande permet d'insérer un sorted set `keySortedSet1` avec les valeurs `valeurSorted1`, `valeurSorted2` et `valeurSorted3` dans le cluster Redis.
> La commande renvoie : `(integer) 3`

### 2. Requêtage des données

Requête String :
```shell
docker exec -it redis-node-1 redis-cli -c GET keyString1
```
Cette commande permet de récupérer la valeur de la clé `keyString1`.
> La commande renvoie : `"valeur de key1"`

Requête Lists :
```shell
docker exec -it redis-node-1 redis-cli -c LRANGE keyList1 0 -1
```
Cette commande permet de récupérer toutes les valeurs de la liste `keyList1` (de l'index 0 à la fin).
> La commande renvoie :
> `1) "valeur3"`
> `2) "valeur2"`
> `3) "valeur1"`

Requête Sets :
```shell
docker exec -it redis-node-1 redis-cli -c SMEMBERS keySet1
```
Cette commande permet de récupérer toutes les valeurs du set `keySet1`.
> La commande renvoie :
> `1) "valeurSet3"`
> `2) "valeurSet1"`
> `3) "valeurSet2"`

Requête Hashes :
```shell
docker exec -it redis-node-1 redis-cli -c HGETALL keyHash1
```
Cette commande permet de récupérer tous les champs et valeurs du hash `keyHash1`.
> La commande renvoie :
> `1) "field1"`
> `2) "valeurHash1"`
> `3) "field2"`
> `4) "valeurHash2"`
> `5) "field3"`
> `6) "valeurHash3"`

Requête Sorted Sets :
```shell
docker exec -it redis-node-1 redis-cli -c ZRANGE keySortedSet1 0 -1
```
Cette commande permet de récupérer toutes les valeurs du sorted set `keySortedSet1`.
> La commande renvoie :
> `1) "valeurSorted1"`
> `2) "valeurSorted2"`
> `3) "valeurSorted3"`

## Partie 3 : Intégration de Redis dans un projet

### 1. Création d'une application d'annuaire en Python

On commence par installer le package `redis` avec la commande `pip install redis`.
La connection à Redis en localhost se fait avec la commande `redis.Redis(host='localhost', port=6379, db=0)`.

_Le code entier est disponible dans le fichier `app.py`._

### 2. Tests de l'application

On peut tester l'application avec la commande `python app.py`.

## Partie 4 : Introspection sur l'intégration de Redis

L'intégration de Redis peut apporter des avantages significatifs à divers types de projets, en particulier ceux nécessitant une manipulation rapide de données, une mise en cache efficace, une scalabilité horizontale et une haute disponibilité. Voici une analyse détaillée des avantages potentiels de l'intégration de Redis dans différents types de projets :

1. **Applications Web** :
   - **Sessions Utilisateur** : Stocker les sessions utilisateur en mémoire avec Redis permet d'améliorer les performances en réduisant les temps de chargement et les temps de réponse.
   - **Mise en Cache** : Utiliser Redis comme cache pour les requêtes fréquemment effectuées peut réduire la charge sur la base de données principale et accélérer les temps de réponse pour les utilisateurs.

2. **Applications de Jeux Vidéo en Temps Réel** :
   - **Gestion de l'État du Jeu** : Stocker l'état du jeu et les données de session en mémoire avec Redis permet une communication rapide entre les différents serveurs et joueurs, améliorant ainsi la fluidité et la réactivité du jeu.
   - **Classements et Statistiques** : Maintenir les classements et les statistiques des joueurs en mémoire avec Redis permet des mises à jour en temps réel et une analyse rapide des données de jeu.

3. **Applications de Commerce Électronique** :
   - **Paniers d'Achat** : Stocker temporairement les paniers d'achat en mémoire avec Redis permet de garantir une expérience utilisateur fluide et rapide, tout en assurant la persistance des données en cas de panne.
   - **Gestion des Stocks** : Utiliser Redis pour gérer les informations sur le stock et les niveaux de produits peut améliorer la gestion des commandes en temps réel et prévenir les situations de survente.

4. **Applications de Traitement de Données en Temps Réel** :
   - **File d'Attente de Messages** : Utiliser Redis comme système de messagerie ou de file d'attente pour le traitement asynchrone des données peut réduire les goulots d'étranglement et améliorer la scalabilité du système.
   - **Analyse en Temps Réel** : Stocker les données d'analyse en mémoire avec Redis permet des calculs rapides et en temps réel sur de grands ensembles de données, facilitant ainsi la prise de décision en temps opportun.

5. **Applications IoT (Internet des Objets)** :
   - **Gestion des Dispositifs** : Utiliser Redis pour gérer les connexions et l'état des dispositifs IoT permet une communication rapide et fiable entre les appareils et la plateforme IoT.
   - **Traitement des Flux de Données** : Stocker et traiter les flux de données générés par les appareils IoT en mémoire avec Redis permet des analyses en temps réel et des actions basées sur les événements.

En résumé, l'intégration de Redis peut fournir des avantages significatifs en termes de performances, de scalabilité, de réactivité et de disponibilité pour une large gamme de projets, allant des applications web aux systèmes IoT en passant par les jeux vidéo en temps réel et les applications de commerce électronique. Cependant, il est crucial de bien comprendre les besoins spécifiques du projet et de planifier soigneusement l'intégration de Redis pour maximiser ses avantages.