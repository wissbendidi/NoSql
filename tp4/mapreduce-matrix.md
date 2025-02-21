# Opérations Matricielles Distribuées avec MapReduce

Par BENDIDI Wissal  
Février 2025  

## Table des matières
1. Introduction et contexte
2. Modélisation des données
3. Implémentation des opérations vectorielles
4. Calcul de similarité entre documents
5. Optimisations et bonnes pratiques
6. Conclusion
7. Annexe : Explication des commandes

---

## 1. Introduction et contexte

Dans le contexte du traitement de données massives, nous explorons l'utilisation de **MapReduce** pour effectuer des opérations matricielles distribuées. Notre cas d'étude porte sur une matrice **S** de dimension **N × M**, représentant les similarités entre **N documents** et **M termes**, où chaque élément **Sij** représente l'importance du terme **j** dans le document **i**.

---

## 2. Modélisation des données

### 2.1 Structure de la matrice
La matrice **S** est représentée comme une collection de documents où chaque document correspond à un vecteur ligne :

```json
{
    "_id": "doc_1",
    "terms": {
        "term1": 0.8,
        "term2": 0.5,
        "term3": 0.3
    }
}
```

Chaque document possède un identifiant unique (`_id`) et une liste de termes associés à des pondérations.

### 2.2 Propriétés mathématiques
Pour un document **i**, son vecteur de termes **Vi** est défini comme :

\[ V_i = (S_{i1}, S_{i2}, ..., S_{iM}) \]

---

## 3. Implémentation des opérations vectorielles

### 3.1 Calcul de magnitude vectorielle
La **magnitude** d'un vecteur représente sa norme en espace euclidien et est définie par :

\[ ||V|| = \sqrt{\sum_{j=1}^{M} S_{ij}^2} \]

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
Cette fonction **Map** parcourt tous les termes d'un document et émet la somme des carrés des pondérations.

#### Fonction Reduce
```javascript
function reduce(key, values) {
    return Math.sqrt(Array.sum(values));
}
```
La fonction **Reduce** applique la racine carrée pour obtenir la magnitude.

### 3.2 Exemple de calcul
Pour un document avec les termes :
```json
{
    "term1": 0.5,
    "term2": 0.3,
    "term3": 0.4
}
```
La magnitude est :
\[ \sqrt{0.5^2 + 0.3^2 + 0.4^2} = \sqrt{0.5} \approx 0.707 \]

---

## 4. Calcul de similarité entre documents

La **similarité cosinus** entre deux documents **A** et **B** est définie par :

\[ \text{sim}(A,B) = \frac{A \cdot B}{||A|| \times ||B||} \]

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
La fonction **Map** calcule le produit scalaire entre un document et un vecteur de requête.

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
La fonction **Reduce** applique la formule de la similarité cosinus.

### 4.2 Exemple de calcul

Soit deux documents :
```json
Doc1 = {
    "term1": 0.5,
    "term2": 0.3
}

Doc2 = {
    "term1": 0.4,
    "term2": 0.6
}
```

1. Produit scalaire = \( (0.5 \times 0.4) + (0.3 \times 0.6) = 0.38 \)
2. Magnitude de **Doc1** = \( 0.583 \)
3. Magnitude de **Doc2** = \( 0.721 \)
4. Similarité = \( 0.38 / (0.583 \times 0.721) = 0.904 \)

---

## 5. Optimisations et bonnes pratiques

### 5.1 Réduction de la taille des données émises
```javascript
function map() {
    Object.entries(this.terms)
        .filter(([_, value]) => value > 0.001)
        .forEach(([term, value]) =>
            emit(this._id, { [term]: value }));
}
```
Cette optimisation permet de réduire la taille des documents émis en filtrant les termes peu significatifs.

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
L'option `parallel: true` optimise l'exécution des tâches.

---

## 6. Conclusion

L'approche **MapReduce** pour les opérations matricielles offre :
1. **Scalabilité horizontale**
2. **Traitement parallèle**
3. **Tolérance aux pannes**
4. **Flexibilité**

Cependant, elle peut engendrer une **latence élevée** et une **complexité accrue** dans les petits jeux de données.

---

## 7. Annexe : Explication des commandes

- **Émission contrôlée des termes** : Réduit l'encombrement réseau
- **Parallélisation** : Accélère le traitement
- **Filtrage des valeurs faibles** : Réduit la mémoire utilisée

Ces techniques améliorent les performances et optimisent le flux de données dans un cadre distribué.


