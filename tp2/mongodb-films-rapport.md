# Utilisation de MongoDB pour la gestion d'une base de données de films

Par BENDIDI Wissal
Encadrée par M. Sami YOUSSUF 
Février 2025

## Table des matières
1. Introduction
2. Configuration et importation des données
3. Requêtes de base
4. Requêtes avancées
5. Analyses spécifiques
6. Conclusion

## 1. Introduction

Ce rapport présente l'utilisation pratique de MongoDB dans le contexte d'une base de données de films. MongoDB, en tant que base de données NoSQL orientée document, présente plusieurs avantages clés pour ce type de projet :

- **Flexibilité du schéma** : Particulièrement utile pour les films qui peuvent avoir des attributs variables (certains films n'ont pas de bande-annonce, d'autres ont plusieurs réalisateurs, etc.)
- **Gestion native des données imbriquées** : Idéal pour stocker les informations sur les acteurs, les critiques, et les notes
- **Performances de requête** : Excellentes pour les opérations de lecture fréquentes
- **Évolutivité** : Capacité à gérer de grandes collections de films et leurs métadonnées

## 2. Configuration et importation des données

### 2.1 Importation de la collection

Pour importer vos données de films dans MongoDB :

```javascript
mongoimport --db cinema --collection films --file films.json --jsonArray
```

Cette commande :
- Crée une base de données nommée "cinema" si elle n'existe pas
- Établit une collection "films"
- Importe les données depuis un fichier JSON
- L'option --jsonArray indique que le fichier contient un tableau d'objets JSON

### 2.2 Vérification de l'importation

Après l'importation, il est crucial de vérifier que les données ont été correctement chargées :

```javascript
// Compter le nombre total de documents
db.films.countDocuments()

// Examiner la structure d'un document
db.films.findOne()
```

Ces commandes vous permettent de :
- Confirmer que le nombre de documents importés correspond à vos attentes
- Vérifier que la structure des documents est correcte
- Identifier d'éventuelles anomalies dans les données

## 3. Requêtes de base

### 3.1 Recherche par genre

Les requêtes de genre sont parmi les plus courantes dans une base de films :

```javascript
// Trouver tous les films d'un genre spécifique
db.films.find({ genre: "Science Fiction" })

// Compter le nombre de films par genre
db.films.count({ genre: "Action" })
```

Ces requêtes sont utiles pour :
- Créer des pages de catégories sur un site web
- Générer des statistiques sur la distribution des genres
- Alimenter des systèmes de recommandation basés sur les genres

### 3.2 Filtrage par pays et année

Le filtrage temporel et géographique permet des analyses plus précises :

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

Points clés à noter :
- L'opérateur $gte signifie "greater than or equal to" (supérieur ou égal à)
- L'opérateur $lt signifie "less than" (inférieur à)
- Ces requêtes peuvent être utilisées pour des analyses historiques ou des tendances régionales

## 4. Requêtes avancées

### 4.1 Projection et exclusion de champs

La projection permet d'optimiser les performances en ne récupérant que les données nécessaires :

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

Comprendre la projection :
- 1 indique qu'un champ doit être inclus
- 0 indique qu'un champ doit être exclu
- Par défaut, _id est toujours inclus sauf si explicitement exclu
- La projection améliore les performances en réduisant la quantité de données transférées

### 4.2 Requêtes sur les tableaux

MongoDB excelle dans la gestion des données array :

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

L'opérateur $elemMatch est particulièrement utile car il :
- Permet de chercher dans des tableaux d'objets
- Vérifie plusieurs conditions sur le même élément du tableau
- Est plus précis qu'une simple recherche de valeur dans un tableau

## 5. Analyses spécifiques

### 5.1 Agrégations

Le framework d'agrégation de MongoDB permet des analyses sophistiquées :

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

Le pipeline d'agrégation :
- Regroupe les documents par genre ($group)
- Calcule la moyenne des notes pour chaque groupe ($avg)
- Trie les résultats par note moyenne décroissante ($sort)
- Permet des analyses statistiques complexes

### 5.2 Requêtes distinctes

Pour obtenir des listes de valeurs uniques :

```javascript
// Liste des genres uniques
db.films.distinct("genre")

// Liste des pays de production
db.films.distinct("country")
```

Ces requêtes sont utiles pour :
- Générer des menus de filtres
- Vérifier la cohérence des données
- Obtenir une vue d'ensemble rapide de la diversité des contenus

### 5.3 Analyses conditionnelles

Identification des cas particuliers :

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

Ces requêtes permettent de :
- Identifier les données manquantes ou incomplètes
- Trouver des cas particuliers nécessitant un traitement spécial
- Assurer la qualité des données

## 6. Conclusion

Notre exploration de MongoDB pour la gestion d'une base de données de films démontre plusieurs avantages clés :

- **Flexibilité exceptionnelle** : Adaptation facile aux différentes structures de données des films
- **Requêtes puissantes** : Capacité à effectuer des recherches complexes et des analyses sophistiquées
- **Performance optimale** : Gestion efficace des lectures et écritures fréquentes
- **Évolutivité** : Possibilité d'étendre la base de données sans modifier le schéma

Cette solution s'avère particulièrement adaptée pour :
- Les plateformes de streaming
- Les sites de critique de films
- Les applications de recommandation de films
- Les systèmes d'analyse de tendances cinématographiques

## Annexes

### Bonnes pratiques pour l'administration

```javascript
// Création d'index pour optimiser les recherches
db.films.createIndex({ "title": 1 })
db.films.createIndex({ "genre": 1, "year": 1 })

// Statistiques de la collection
db.films.stats()
```

Points importants pour l'administration :
- Créer des index sur les champs fréquemment recherchés
- Monitorer régulièrement les performances
- Maintenir des sauvegardes régulières
- Optimiser les requêtes fréquentes
