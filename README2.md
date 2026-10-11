
---

### 1. Pourquoi dit-on que les fonctions sont des "citoyens de première classe" ?

En JavaScript, une fonction n'est pas juste un bloc de code figé : **c'est un objet comme les autres**.
Cela signifie que tu peux :

* Les stocker dans des variables ou des constantes.
* Les passer en arguments à d'autres fonctions (on appelle ça des **fonctions de rappel** ou *callbacks*).
* Les faire retourner par d'autres fonctions (ce qui est le secret magique des **closures** !).

---

### 2. Les différentes façons d'écrire des fonctions

Il y a trois grandes syntaxes en JavaScript moderne, et elles ont chacune leurs particularités :

#### A. La Déclaration de Fonction (Function Declaration)

Celle qu'on a le plus vue, entièrement **hoisted** (hissée).

```javascript
function direBonjour(nom) {
  return `Bonjour ${nom}`;
}

```

#### B. L'Expression de Fonction (Function Expression / Anonyme ou Nommée)

On stocke une fonction dans une variable. Elle n'est **pas** hoisted de la même manière (sujette à la TDZ si `let`/`const`).

```javascript
const direBonjour = function(nom) {
  return `Bonjour ${nom}`;
};

```

#### C. Les Fonctions Fléchées (Arrow Functions - ES6)

Plus courtes, très utilisées, mais elles cachent un piège subtil (le fameux `this`).

```javascript
const direBonjour = (nom) => `Bonjour ${nom}`;

```

---

### 3. Le grand piège des fonctions fléchées : Le mot-clé `this`

C'est LA différence fondamentale entre une fonction classique (`function()`) et une fonction fléchée (`() => {}`).

* **La fonction classique :** Elle a son **propre `this**`, qui change dynamiquement selon *comment* elle est appelée.
* **La fonction fléchée :** Elle n'a **pas de `this` propre**. Elle "hérite" (ou capture) le `this` du scope parent dans lequel elle a été écrite (lexical `this`).

Regarde cet exemple classique avec un objet :

```javascript
const utilisateur = {
  nom: "Alice",
  
  // Fonction classique
  direBonjourClassique: function() {
    console.log(`Bonjour, je m'appelle ${this.nom}`); // 'this' pointe vers l'objet 'utilisateur'
  },
  
  // Fonction fléchée
  direBonjourFlechee: () => {
    console.log(`Bonjour, je m'appelle ${this.nom}`); // 'this' pointe vers le scope global (undefined) !
  }
};

utilisateur.direBonjourClassique(); // "Bonjour, je m'appelle Alice"
utilisateur.direBonjourFlechee();   // "Bonjour, je m'appelle undefined" (ou global)

```

*Règle d'or :* Évite d'utiliser des fonctions fléchées pour créer des méthodes à l'intérieur d'objets, car elles perdent le contexte de l'objet.

---

### 4. Le lien avec les Closures (qu'on a survolées)

Les fonctions en JavaScript sont par nature des **closures** : elles se souviennent toujours de leur environnement de création (leur scope lexical).

C'est grâce à cela qu'on peut créer de l'encapsulation (des variables privées), comme dans notre compteur tout à l'heure :

```javascript
const creerPocheDeMonnaie = () => {
  let pieces = 10; // Variable privée, inaccessible de l'extérieur

  return {
    ajouter: (montant) => pieces += montant,
    obtenirTotal: () => pieces
  };
};

const maPoche = creerPocheDeMonnaie();
maPoche.ajouter(5);
console.log(maPoche.obtenirTotal()); // 15
// Impossible de faire direktement `maPoche.pieces` ! C'est protégé par la closure.

```
