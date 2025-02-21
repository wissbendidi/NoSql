# Opérations Matricielles Distribuées avec MapReduce
Par BENDIDI Wissal
Février 2025

## Table des matières
1. Introduction et contexte
2. Modélisation des données
3. Implémentation des opérations vectorielles
4. Calcul de similarité entre documents
5. Conclusion

## 1. Introduction et contexte

Dans le contexte du traitement de données massives, nous nous intéressons à l'utilisation de MapReduce pour effectuer des opérations matricielles distribuées. Notre cas d'étude porte sur une matrice S de dimension N × M représentant les similarités entre N documents et M termes, où chaque élément Sij représente l'importance du terme j dans le document i.

## 2. Modélisation des données

### 2.1 Structure de la matrice
La matrice S est représentée comme une collection de documents où chaque document correspond à un vecteur ligne :

```javascript
{
    "_id": "doc_1",
    "terms": {
        "term1": 0.8,
        "term2": 0.5,
        "term3": 0.3
    }
}
```

### 2.2 Propriétés mathématiques
Pour un document i, son vecteur de termes Vi est défini comme :
Vi = (Si1, Si2, ..., SiM)

## 3. Implémentation des opérations vectorielles

### 3.1 Calcul de magnitude vectorielle

#### Fonction Map
```javascript
function map() {
    let sumSquares = 0;
    for (let term in this.terms) {
        sumSquares += Math.pow(this.terms[term], 2);
    }
    emit(this._id, sumSquares);
}
```

#### Fonction Reduce
```javascript
function reduce(key, values) {
    return Math.sqrt(Array.sum(values));
}
```

### 3.2 Exemple de calcul de magnitude
Pour un document avec les termes :
```javascript
{
    "term1": 0.5,
    "term2": 0.3,
    "term3": 0.4
}
```

Magnitude = √(0.5² + 0.3² + 0.4²) = √(0.25 + 0.09 + 0.16) = √0.5 = 0.707

## 4. Calcul de similarité entre documents

### 4.1 Produit scalaire distribué

#### Fonction Map
```javascript
function map() {
    const queryVector = globalQueryVector; // Vecteur de référence
    let dotProduct = 0;
    
    for (let term in this.terms) {
        if (queryVector[term]) {
            dotProduct += this.terms[term] * queryVector[term];
        }
    }
    
    emit(this._id, {
        dotProduct: dotProduct,
        magnitude: Math.sqrt(Object.values(this.terms)
            .reduce((sum, val) => sum + val * val, 0))
    });
}
```

#### Fonction Reduce
```javascript
function reduce(key, values) {
    const queryMagnitude = Math.sqrt(
        Object.values(globalQueryVector)
            .reduce((sum, val) => sum + val * val, 0)
    );
    
    return values[0].dotProduct / (values[0].magnitude * queryMagnitude);
}
```

### 4.2 Algorithme de similarité cosinus

La similarité cosinus entre deux documents est calculée comme :

sim(A,B) = (A·B)/(||A||·||B||)

où :
- A·B est le produit scalaire
- ||A|| et ||B|| sont les magnitudes des vecteurs

### 4.3 Exemple de calcul

Soit deux documents :
```javascript
Doc1 = {
    "term1": 0.5,
    "term2": 0.3
}

Doc2 = {
    "term1": 0.4,
    "term2": 0.6
}
```

Calcul :
1. Produit scalaire = (0.5 × 0.4) + (0.3 × 0.6) = 0.38
2. ||Doc1|| = √(0.5² + 0.3²) = 0.583
3. ||Doc2|| = √(0.4² + 0.6²) = 0.721
4. Similarité = 0.38/(0.583 × 0.721) = 0.904

## 5. Optimisations

### 5.1 Réduction de la taille des données émises
```javascript
function map() {
    // N'émettre que les termes non nuls
    Object.entries(this.terms)
        .filter(([_, value]) => value > 0.001)
        .forEach(([term, value]) => 
            emit(this._id, { [term]: value }));
}
```

### 5.2 Parallélisation des calculs
```javascript
const options = {
    mapReduce: "documents",
    map: mapFunction,
    reduce: reduceFunction,
    out: { merge: "results" },
    scope: { threshold: 0.001 },
    parallel: true
};
```

## 6. Conclusion

Cette approche MapReduce pour les opérations matricielles présente plusieurs avantages :
1. Scalabilité horizontale efficace
2. Traitement parallèle naturel
3. Tolérance aux pannes intégrée
4. Flexibilité dans la structure des données

Les principales limitations sont :
1. Latence pour les petits ensembles de données
2. Complexité de débogage
3. Surcharge de communication réseau

## Annexe: Complexité algorithmique

Pour une matrice N × M :
- Complexité temporelle : O(N × M) en parallèle
- Complexité spatiale : O(N + M) par nœud
- Communication réseau : O(N) messages
