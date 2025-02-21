# Utilisation de MongoDB pour la gestion d'une base de données de films

Par [Votre Nom]
Février 2025

## Table des matières
1. Introduction
2. Configuration et importation des données
3. Requêtes de base
4. Requêtes avancées
5. Analyses spécifiques
6. Conclusion

## 1. Introduction

Ce rapport présente l'utilisation pratique de MongoDB dans le contexte d'une base de données de films. MongoDB, en tant que base de données NoSQL orientée document, offre une flexibilité particulièrement adaptée à la gestion de données complexes et hétérogènes comme les informations sur les films.

## 2. Configuration et importation des données

### 2.1 Importation de la collection
```javascript
mongoimport --db cinema --collection films --file films.json --jsonArray
```

### 2.2 Vérification de l'importation
```javascript
// Compter le nombre total de documents
db.films.countDocuments()

// Examiner la structure d'un document
db.films.findOne()
```

## 3. Requêtes de base

### 3.1 Recherche par genre
```javascript
// Trouver tous les films d'un genre spécifique
db.films.find({ genre: "Science Fiction" })

// Compter le nombre de films par genre
db.films.count({ genre: "Action" })
```

### 3.2 Filtrage par pays et année
```javascript
// Films français de 2020
db.films.find({
    country: "FR",
    year: 2020
})

// Films américains des années 90
db.films.find({
    country: "US",
    year: { $gte: 1990, $lt: 2000 }
})
```

## 4. Requêtes avancées

### 4.1 Projection et exclusion de champs
```javascript
// Afficher uniquement les titres et dates
db.films.find(
    { genre: "Action" },
    { title: 1, releaseDate: 1, _id: 0 }
)

// Exclure les critiques
db.films.find(
    { genre: "Drama" },
    { reviews: 0 }
)
```

### 4.2 Requêtes sur les tableaux
```javascript
// Films avec un acteur spécifique
db.films.find({
    "actors": {
        $elemMatch: {
            name: "Morgan Freeman"
        }
    }
})

// Films avec une note minimale
db.films.find({
    "ratings.score": { $gte: 8.0 }
})
```

## 5. Analyses spécifiques

### 5.1 Agrégations
```javascript
// Moyenne des notes par genre
db.films.aggregate([
    {
        $group: {
            _id: "$genre",
            averageRating: { $avg: "$ratings.score" }
        }
    },
    {
        $sort: { averageRating: -1 }
    }
])
```

### 5.2 Requêtes distinctes
```javascript
// Liste des genres uniques
db.films.distinct("genre")

// Liste des pays de production
db.films.distinct("country")
```

### 5.3 Analyses conditionnelles
```javascript
// Films sans synopsis
db.films.find({
    summary: { $exists: false }
})

// Films avec plusieurs réalisateurs
db.films.find({
    directors: { $size: { $gt: 1 } }
})
```

## 6. Conclusion

Notre exploration de MongoDB à travers cette base de données de films démontre la puissance et la flexibilité de ce système de gestion de base de données NoSQL. Les principales observations sont :

- La facilité d'interrogation des données complexes
- La flexibilité du schéma permettant des structures de données variables
- La puissance des opérateurs de requête pour des recherches précises
- La capacité à effectuer des analyses sophistiquées via le framework d'agrégation

Cette expérience pratique confirme l'adéquation de MongoDB pour la gestion de données cinématographiques, où la structure des informations peut varier considérablement d'un film à l'autre.

## Annexes

### Commandes utiles pour l'administration
```javascript
// Création d'index pour optimiser les recherches
db.films.createIndex({ "title": 1 })
db.films.createIndex({ "genre": 1, "year": 1 })

// Statistiques de la collection
db.films.stats()
```
