# CouchDB et MapReduce : Analyse de données distribuées

Par BENDIDI Wissal 
Février 2025  

## Table des matières
1. Introduction
2. Présentation de CouchDB
3. Configuration et déploiement
4. Manipulation des données
5. Implémentation de MapReduce
6. Cas d'usage avancés
7. Optimisations et bonnes pratiques
8. Conclusion
9. Annexe : Explication des commandes

---

## 1. Introduction

CouchDB est une base de données NoSQL orientée documents qui repose sur un modèle distribué et intègre MapReduce pour l’indexation et le traitement de données volumineuses. Ce guide explore son fonctionnement et son implémentation avec MapReduce.

---

## 2. Présentation de CouchDB

CouchDB se distingue par :
- Une architecture **RESTful** native
- Un stockage des données en **JSON**
- Une réplication **master-master**
- Un moteur de vue **MapReduce** pour les requêtes complexes

### 2.1 Caractéristiques principales
- API HTTP/REST
- Gestion des conflits intégrée
- Indexation avec MapReduce

---

## 3. Configuration et déploiement

### 3.1 Installation via Docker
```bash
docker run -d --name couchdb-instance \
  -e COUCHDB_USER=admin \
  -e COUCHDB_PASSWORD=secret \
  -p 5984:5984 \
  couchdb:latest
```
**Explication** : Lance un conteneur CouchDB avec un utilisateur et un mot de passe définis.

### 3.2 Vérification de l'installation
```bash
curl -X GET http://admin:secret@localhost:5984
```
**Explication** : Vérifie que CouchDB est bien accessible via l'API REST.

---

## 4. Manipulation des données

### 4.1 Insertion de documents
```json
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

### 4.2 Opérations CRUD via REST
```bash
curl -X PUT \
  http://admin:secret@localhost:5984/mediadb/movie_001 \
  -H "Content-Type: application/json" \
  -d @movie.json
```
**Explication** : Ajoute un document dans la base `mediadb`.

---

## 5. Implémentation de MapReduce

### 5.1 Calcul des moyennes des notes

#### Fonction Map
```javascript
function(doc) {
    if (doc.ratings) {
        doc.ratings.forEach(rating => {
            emit(doc.title, rating.score);
        });
    }
}
```
**Explication** : Associe chaque film à ses notes.

#### Fonction Reduce
```javascript
function(keys, values, rereduce) {
    let sum = values.reduce((a, b) => a + b, 0);
    return { sum: sum, count: values.length, average: sum / values.length };
}
```
**Explication** : Calcule la moyenne des scores pour chaque film.

---

## 6. Cas d'usage avancés

### 6.1 Analyse des genres
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
**Explication** : Émet une clé **(genre, année)** avec la valeur 1.

#### Fonction Reduce
```javascript
function(keys, values) {
    return sum(values);
}
```
**Explication** : Compte le nombre de films par genre et année.

---

## 7. Optimisations et bonnes pratiques

- **Limiter l’émission des clés** pour éviter la surcharge mémoire.
- **Utiliser des vues permanentes** pour accélérer les requêtes.
- **Indexer les données** pour améliorer les performances :

```javascript
{
    "index": {
        "fields": ["year", "genre", "rating"]
    },
    "name": "year-genre-rating-index",
    "type": "json"
}
```

---

## 8. Conclusion

CouchDB avec MapReduce permet :
- Un **traitement efficace des grandes bases de données**
- Une **réplication et synchronisation robustes**
- Une **modélisation flexible sans schéma fixe**

Ses principaux atouts sont la **scalabilité horizontale** et la **simplicité d'utilisation**.

---

## 9. Annexe : Explication des commandes

### Compactage de la base
```bash
curl -X POST http://admin:secret@localhost:5984/mediadb/_compact
```
**Explication** : Réduit la taille de la base en supprimant les anciennes révisions des documents.

### Surveillance des tâches
```bash
curl -X GET http://admin:secret@localhost:5984/_active_tasks
```
**Explication** : Affiche les tâches en cours dans CouchDB (réplication, indexation, etc.).


