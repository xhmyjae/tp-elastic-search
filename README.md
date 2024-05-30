# tp-elastic-search

## Partie 1 : Installation et Configuration du Cluster Elasticsearch

### 1. Installation de Elasticsearch et configuration du cluster

J'ai décidé d'utiliser Docker pour installer Elasticsearch. J'ai donc créé un fichier `docker-compose.yml` pour définir les services Elasticsearch. Voici le contenu du fichier :

```yaml
version: '3.8'
services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.14.0
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elasticsearch-network
  es02:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.14.0
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata2:/usr/share/elasticsearch/data
    networks:
      - elasticsearch-network
  es03:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.14.0
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata3:/usr/share/elasticsearch/data
    networks:
      - elasticsearch-network
volumes:
  esdata1:
    driver: local
  esdata2:
    driver: local
  esdata3:
    driver: local
networks:
  elasticsearch-network:
```

On définit les services Elasticsearch.
- **image** : On utilise l'image Docker officielle d'Elasticsearch.
- **container_name** : On définit le nom du conteneur.
- **environment** : On définit les variables d'environnement pour chaque service (nom du noeud, nom du cluster, hôtes de découverte, noeuds maîtres initiaux, mémoire de démarrage, options Java).
- **ulimits** : On définit les limites de mémoire pour chaque service.
- **volumes** : On définit les volumes pour stocker les données Elasticsearch.
- **ports** : On définit les ports d'écoute pour Elasticsearch.
- **networks** : On définit le réseau pour les services. (On définit un réseau `elasticsearch-network` pour que les services puissent communiquer entre eux.)

On démarre ensuite les services avec la commande `docker-compose up -d --build`.

### 2. Vérification de l'état du cluster Elasticsearch

On peut vérifier l'état du cluster Elasticsearch avec la commande `curl -X GET "localhost:9200/_cluster/health?pretty"`. On obtient la liste des noeuds du cluster avec leur adresse IP, leur nom, leur rôle, leur ID, leur version, leur type, leur port, leur adresse IP de transport et leur adresse IP de HTTP.

## Partie 2 : Premiers pas avec le Cluster Elasticsearch

### 1. Créer un index nommé `test01`

```bash
curl -X PUT "localhost:9200/test01\
```

### 2. Vérifier que cet index est maintenant présent

```bash
curl -X GET "localhost:9200/_cat/indices?v"
```

### 3. Créer un document dans cet index

On crée un document dans l'index `test01` avec la commande suivante :
```bash
{
  "titre": "Premier document",
  "description": "C'est la première fois que je crée un document dans ES.",
  "quantité": 12,
  "date_creation": "10-05-2024"
}
```

```bash
curl -X POST "localhost:9200/test01/_doc/1" -H 'Content-Type: application/json' -d'
{
  "titre": "Premier document",
  "description": "C\'est la première fois que je crée un document dans ES.",
  "quantité": 12,
  "date_creation": "10-05-2024"
}'
```

### 4. Afficher ce document

```bash
curl -X GET "localhost:9200/test01/_doc/1"
```

### 5. Afficher quel mapping a été appliqué par défaut

```bash
curl -X GET "localhost:9200/test01/_mapping"
```

### 6. Est-il optimal ?

Le mapping appliqué par défaut n'est pas optimal. En effet, le champ `date_creation` est considéré comme un champ de type `text` alors qu'il devrait être de type `date`. De plus, le champ `quantité` est considéré comme un champ de type `long` alors qu'il devrait être de type `integer`.

### 7. Modifier le mapping pour le champ date_creation

Pour adapter le mapping du champ date_creation, nous devons supprimer l'index et le recréer avec le bon mapping, car la modification du mapping d'un champ existant n'est pas permise directement.

```bash
curl -X DELETE "localhost:9200/test01"
```

Et on recrée l'index `test01` avec le bon mapping pour le champ `date_creation` :

```bash
curl -X PUT "localhost:9200/test01" -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "properties": {
      "date_creation": {
        "type": "date",
        "format": "dd-MM-yyyy"
      }
    }
  }
}'
```

```bash
curl -X POST "localhost:9200/test01/_doc/1" -H 'Content-Type: application/json' -d'
{
  "titre": "Premier document",
  "description": "C\'est la première fois que je crée un document dans ES.",
  "quantité": 12,
  "date_creation": "10-05-2024"
}'
```

### 8. Créer un nouvel index test02 avec un mapping adapté

```bash
curl -X PUT "localhost:9200/test02" -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "properties": {
      "titre": { "type": "text" },
      "description": { "type": "text" },
      "quantité": { "type": "integer" },
      "date_creation": {
        "type": "date",
        "format": "dd-MM-yyyy"
      }
    }
  }
}'
```

### 9. Analyser le texte de la description

```bash
curl -X GET "localhost:9200/test02/_analyze" -H 'Content-Type: application/json' -d'
{
  "text": "C\'est la première fois que je crée un document dans ES."
}'
```

### 10. Créer un index `test03` avec différents traitements de `description`

```bash
curl -X PUT "localhost:9200/test03" -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "properties": {
      "titre": { "type": "text" },
      "description": {
        "type": "text",
        "fields": {
          "raw": { "type": "keyword" },
          "no_stopwords": {
            "type": "text",
            "analyzer": "stop"
          }
        }
      },
      "quantité": { "type": "integer" },
      "date_creation": {
        "type": "date",
        "format": "dd-MM-yyyy"
      }
    }
  }
}'
```

### 11. Importer plusieurs documents en une seule requête

On utilise le format de requête `bulk` pour importer plusieurs documents en une seule requête :

```bash
curl -X POST "localhost:9200/test03/_bulk" -H 'Content-Type: application/json' -d'
{ "index": { "_id": "1" } }
{ "titre": "Document 1", "description": "C\'est le premier document.", "quantité": 10, "date_creation": "01-01-2024" }
{ "index": { "_id": "2" } }
{ "titre": "Document 2", "description": "Voici le second document.", "quantité": 20, "date_creation": "02-02-2024" }
{ "index": { "_id": "3" } }
{ "titre": "Document 3", "description": "Un autre document pour ES.", "quantité": 30, "date_creation": "03-03-2024" }
{ "index": { "_id": "4" } }
{ "titre": "Document 4", "description": "Le dernier document de la série.", "quantité": 40, "date_creation": "04-04-2024" }
'
```

## Partie 3 : Intégration de Kibana

### 1. Intégration des Données Factices Fournies

Supposons que les données factices fournies sont stockées dans un fichier `data.json`. On peut les importer dans Elasticsearch avec la commande suivante :

```bash
curl -X POST "localhost:9200/_bulk" -H 'Content-Type: application/json' --data-binary "@dataset1.json"
curl -X POST "localhost:9200/_bulk" -H 'Content-Type: application/json' --data-binary "@dataset2.json"
curl -X POST "localhost:9200/_bulk" -H 'Content-Type: application/json' --data-binary "@dataset3.json"
```
On utilise `curl` pour importer les données dans Elasticsearch.

### 2. Utilisation de Kibana pour les Requêtes

On accède à Kibana en allant sur `http://localhost:5601` dans un navigateur web. On peut ensuite effectuer des requêtes sur les données importées.
On crée un index pattern qui correspond aux données importées (par exemple `dataset*`) et on peut ensuite effectuer des requêtes sur ces données.

Lister les vols dont le prix moyen est entre 300€ et 450€
```bash
GET /flights/_search
{
  "query": {
    "range": {
      "average_price": {
        "gte": 300,
        "lte": 450
      }
    }
  }
}
```

Lister les vols annulés
```bash
GET /flights/_search
{
  "query": {
    "term": {
      "status": "cancelled"
    }
  }
}
```

Lister les vols où il pleut à l'arrivée ou au départ. Les vols avec de la pluie à l'arrivée devront sortir avant les autres.
```bash
GET /flights/_search
{
  "query": {
    "bool": {
      "should": [
        { "term": { "weather_departure": "rain" } },
        { "term": { "weather_arrival": "rain" } }
      ]
    }
  },
  "sort": [
    { "weather_arrival": { "order": "desc" } }
  ]
}
```

Lister les vols partant d'Allemagne et à destination de France
```bash
GET /flights/_search
{
  "query": {
    "bool": {
      "must": [
        { "term": { "departure_country": "Germany" } },
        { "term": { "arrival_country": "France" } }
      ]
    }
  }
}
```

Lister les vols ayant eu lieu entre le 1er avril 2024 et le 20 mai 2024
```bash
GET /flights/_search
{
  "query": {
    "range": {
      "flight_date": {
        "gte": "2024-04-01",
        "lte": "2024-05-20"
      }
    }
  }
}
```

Les résultats de chaque requête montrent les documents correspondant aux critères définis. Par exemple, la première requête renvoie tous les vols dont le prix moyen est compris entre 300€ et 450€, affichant les détails de chaque vol répondant à ce critère.