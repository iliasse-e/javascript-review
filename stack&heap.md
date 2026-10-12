Comprendre comment JavaScript stocke les données en mémoire est un secret de fabrication précieux. C'est ce qui évite les bugs bizarres où un objet se modifie tout seul alors qu'on ne lui a rien demandé !

En gros, JavaScript sépare la mémoire en deux zones distinctes : la **Call Stack (la Pile)** et la **Memory Heap (le Tas)**, selon le type de ta donnée (les *primitifs* d'un côté, les *références* de l'autre).

---

### 1. La Stack (La Pile) : Pour les types primitifs

La Stack est une zone de mémoire structurée, rapide, et organisée "au compte-gouttes" (en pile d'assiettes : le dernier arrivé est le premier servi).

* **Qu'est-ce qu'on y stocke ?** Les **types primitifs** : `string`, `number`, `boolean`, `undefined`, `null`, `symbol`, et `bigint`.
* **Comment ça fonctionne ?** Ces valeurs ont une taille fixe et connue à l'avance. Quand tu crées une variable primitive, la valeur est stockée **directement dans la Stack**.

```javascript
let age = 25;       // Stocké dans la Stack
let nom = "Alice";  // Stocké dans la Stack
let estMajeur = true; // Stocké dans la Stack

```

#### Ce qui se passe lors d'une copie :

Quand tu copies un primitif, JavaScript crée une **copie exacte** de la valeur. Les deux variables sont totalement indépendantes :

```javascript
let a = 10;
let b = a; // On copie la valeur 10

b = 20;
console.log(a); // 10 (a n'a pas bougé !)

```

---

### 2. La Heap (Le Tas) : Pour les objets et complexes

La Heap est une zone de mémoire immense et non structurée (un grand entrepôt de stockage).

* **Qu'est-ce qu'on y stocke ?** Les **objets**, les **tableaux (arrays)**, et les **fonctions** (qui sont des objets).
* **Pourquoi ?** Parce qu'ils peuvent contenir énormément de données, grandir, rétrécir, ou contenir d'autres objets. Leur taille est variable.

#### Le secret des variables d'objets (La Référence) :

Quand tu crées un objet, l'objet lui-même est jeté dans la **Heap**. Mais dans ta **Stack**, ta variable ne contient pas l'objet : elle contient **une référence (une adresse mémoire, un pointeur)** qui pointe vers l'objet dans la Heap.

```javascript
const utilisateur = { prenom: "Bob" };

```

* **Dans la Heap :** L'objet `{ prenom: "Bob" }` est créé à une adresse (ex: `#001`).
* **Dans la Stack :** La variable `utilisateur` contient la valeur `#001`.

---

### 3. Le grand piège : La copie par référence

C'est ici que beaucoup de développeurs se font piéger. Regarde ce code :

```javascript
const user1 = { nom: "Alice" };
const user2 = user1; // On ne copie PAS l'objet, on copie L'ADRESSE (#001) !

user2.nom = "Charlie";

console.log(user1.nom); // Qu'est-ce qui s'affiche ? "Charlie" !

```

* **Pourquoi ?** Parce que `user1` et `user2` pointent tous les deux vers **exactement le même objet** dans la Heap (`#001`). Modifier `user2`, c'est modifier l'objet commun.

#### Comment faire une vraie copie (un clone) ?

Si tu veux dupliquer un objet sans lier les deux variables, il faut créer un nouvel objet dans la Heap (par exemple avec l'opérateur de propagation `...` ou *spread operator*) :

```javascript
const user1 = { nom: "Alice" };
const user2 = { ...user1 }; // On crée un NOUVEL objet dans la Heap avec les mêmes valeurs

user2.nom = "Charlie";

console.log(user1.nom); // "Alice" (user1 est sain et sauf !)
console.log(user2.nom); // "Charlie"

```

---

### 💡 Résumé mémo :

* **Stack (Pile) = Primitifs (nombres, chaînes, booléens)** $\rightarrow$ Stockage direct, copie par valeur (indépendants).
* **Heap (Tas) = Objets, Tableaux, Fonctions** $\rightarrow$ Stockage en vrac, la variable ne garde qu'une référence, copie par référence (attention aux modifications partagées !).
