# CouchDB et MapReduce : Analyse de données distribuées
Par BENDIDI Wissal
Février 2025

## Table des matières
1. Présentation de CouchDB
2. Configuration et déploiement
3. Manipulation des données
4. Implémentation de MapReduce
5. Cas d'usage avancés
6. Conclusion

## 1. Présentation de CouchDB

CouchDB est une base de données NoSQL orientée document qui se distingue par :
- Une architecture RESTful native
- Un format de données JSON
- Des capacités de réplication bidirectionnelle
- Un moteur de vue MapReduce intégré

### 1.1 Caractéristiques principales
- Stockage document JSON
- API HTTP/REST
- Réplication master-master
- Gestion de conflits intégrée

## 2. Configuration et déploiement

### 2.1 Installation via Docker
```bash
docker run -d --name couchdb-instance \
  -e COUCHDB_USER=admin \
  -e COUCHDB_PASSWORD=secret \
  -p 5984:5984 \
  couchdb:latest
```

### 2.2 Vérification de l'installation
```bash
# Test de connexion
curl -X GET http://admin:secret@localhost:5984

# Création d'une base
curl -X PUT http://admin:secret@localhost:5984/mediadb
```

## 3. Manipulation des données

### 3.1 Insertion de documents
```javascript
// Document unique
{
    "_id": "movie_001",
    "title": "The Matrix",
    "year": 1999,
    "directors": ["Lana Wachowski", "Lilly Wachowski"],
    "genres": ["Science Fiction", "Action"],
    "ratings": [
        {"user": "user_1", "score": 9.5},
        {"user": "user_2", "score": 8.8}
    ]
}
```

### 3.2 Opérations CRUD via REST
```bash
# Création
curl -X PUT \
  http://admin:secret@localhost:5984/mediadb/movie_001 \
  -H "Content-Type: application/json" \
  -d @movie.json

# Lecture
curl -X GET \
  http://admin:secret@localhost:5984/mediadb/movie_001
```

## 4. Implémentation de MapReduce

### 4.1 Calcul de moyennes des notes

#### Fonction Map
```javascript
function(doc) {
    if (doc.ratings && doc.ratings.length > 0) {
        doc.ratings.forEach(rating => {
            emit(doc.title, rating.score);
        });
    }
}
```

#### Fonction Reduce
```javascript
function(keys, values, rereduce) {
    if (rereduce) {
        let totalSum = values.reduce((a, b) => a + b.sum, 0);
        let totalCount = values.reduce((a, b) => a + b.count, 0);
        return {
            sum: totalSum,
            count: totalCount,
            average: totalSum / totalCount
        };
    } else {
        let sum = values.reduce((a, b) => a + b, 0);
        return {
            sum: sum,
            count: values.length,
            average: sum / values.length
        };
    }
}
```

### 4.2 Analyse des genres

#### Fonction Map
```javascript
function(doc) {
    if (doc.genres) {
        doc.genres.forEach(genre => {
            emit([genre, doc.year], 1);
        });
    }
}
```

#### Fonction Reduce
```javascript
function(keys, values) {
    return sum(values);
}
```

## 5. Cas d'usage avancés

### 5.1 Analyse temporelle des sorties

```javascript
// Map : Distribution par année
function(doc) {
    if (doc.year) {
        emit([doc.year, doc.quarter], {
            count: 1,
            budget: doc.budget || 0
        });
    }
}

// Reduce : Agrégation statistique
function(keys, values, rereduce) {
    return {
        count: values.reduce((a, b) => a + b.count, 0),
        totalBudget: values.reduce((a, b) => a + b.budget, 0),
        averageBudget: values.reduce((a, b) => a + b.budget, 0) / 
                      values.reduce((a, b) => a + b.count, 0)
    };
}
```

### 5.2 Analyse des collaborations

```javascript
// Map : Réseaux d'acteurs
function(doc) {
    if (doc.actors && doc.actors.length > 1) {
        for (let i = 0; i < doc.actors.length; i++) {
            for (let j = i + 1; j < doc.actors.length; j++) {
                emit([doc.actors[i], doc.actors[j]], {
                    movie: doc.title,
                    year: doc.year
                });
            }
        }
    }
}
```

## 6. Optimisations et bonnes pratiques

### 6.1 Gestion de la mémoire
- Émission contrôlée des clés
- Utilisation de vues temporaires vs permanentes
- Pagination des résultats

### 6.2 Indexation
```javascript
// Création d'index composite
{
    "index": {
        "fields": ["year", "genre", "rating"]
    },
    "name": "year-genre-rating-index",
    "type": "json"
}
```

## 7. Conclusion

CouchDB, combiné avec MapReduce, offre une solution puissante pour :
- Le traitement distribué de données
- L'analyse statistique de grands ensembles
- La génération de rapports complexes
- La réplication et la synchronisation de données

Les principaux avantages sont :
1. Simplicité d'utilisation
2. Flexibilité du schéma
3. Capacités de réplication natives
4. Performances en lecture

## Annexe : Commandes utiles

```bash
# Compactage de base
curl -X POST http://admin:secret@localhost:5984/mediadb/_compact

# Surveillance des tâches
curl -X GET http://admin:secret@localhost:5984/_active_tasks
```
