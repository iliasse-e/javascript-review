Le **Hoisting** (que l'on traduit souvent par *« le hissage »* ou *« la remonte »* en français) est un comportement fondamental de JavaScript lors de la phase de compilation du code, avant même qu'il ne soit exécuté.

Pour faire simple : **JavaScript donne l'impression de "remonter" (ou de déplacer) les déclarations de variables et de fonctions tout en haut de leur scope (portée) avant l'exécution.**

Mais attention, ce mécanisme piège énormément de développeurs car il ne déplace pas physiquement le code dans ton fichier ! Voyons comment cela fonctionne en détail.

---

### 1. Comment fonctionne le moteur sous le capot ?

Avant d'exécuter ton script, le moteur JavaScript passe par une **phase de création (ou de compilation)** :

1. Il scanne tout ton code à la recherche des déclarations de variables (`var`, `let`, `const`) et de fonctions.
2. Il réserve de l'espace en mémoire pour les stocker.

C'est ce qui crée le phénomène de hoisting, mais la façon dont il traite les éléments dépend entièrement de leur mot-clé.

---

### 2. Le Hoisting avec les variables (`var` vs `let` / `const`)

#### 🔴 Le cas de `var`

Avec `var`, la variable est non seulement hoisted, mais elle est aussi **initialisée automatiquement à `undefined**`.

```javascript
console.log(maVariable); // Affiche 'undefined' (pas d'erreur !)
var maVariable = 'Bonjour';
console.log(maVariable); // Affiche 'Bonjour'

```

*Ce qui se passe réellement en mémoire :* JavaScript traite ton code comme si tu avais écrit ça :

```javascript
var maVariable;          // Déclaration remontée en haut avec undefined
console.log(maVariable); // undefined
maVariable = 'Bonjour';  // Affectation
console.log(maVariable); // 'Bonjour'

```

#### 🟢 Le cas de `let` et `const` (La Temporal Dead Zone)

Avec `let` et `const`, les variables sont bien **hoisted**, mais **elles ne sont PAS initialisées**.

Si tu tentes de les utiliser avant leur ligne de déclaration, JavaScript te jette une erreur critique : la **Temporal Dead Zone (TDZ)**.

```javascript
console.log(monAge); // ReferenceError: Cannot access 'monAge' before initialization
let monAge = 25;

```

*Pourquoi ?* Pour éviter les bugs silencieux liés à `var`. JavaScript préfère bloquer l'exécution s'il détecte qu'on essaie d'utiliser une variable avant qu'elle n'ait réellement été initialisée dans le code.

---

### 3. Le Hoisting avec les fonctions

C'est ici que le hoisting est le plus spectaculaire (et le plus utile) :

#### A. Les déclarations de fonctions (Function Declarations)

Elles sont **totalement hoisted**, y compris leur contenu. Tu peux appeler une fonction *avant* de l'avoir écrite dans le fichier :

```javascript
direBonjour(); // Fonctionne parfaitement ! Affiche "Bonjour !"

function direBonjour() {
  console.log("Bonjour !");
}

```

#### B. Les expressions de fonctions (Function Expressions)

Si tu stockes une fonction dans une variable (`const` ou `let`), le hoisting s'applique à la variable, pas à la fonction elle-même.

```javascript
direCoucou(); // TypeError: direCoucou is not a function (ou ReferenceError selon var/let)

const direCoucou = function() {
  console.log("Coucou !");
};

```

*Ici, `direCoucou` est considéré comme une variable vide (ou en TDZ) au moment de l'appel, d'où l'erreur.*

---

### 💡 À retenir pour tes entretiens ou ton code :

1. **Le hoisting** est un déplacement virtuel des déclarations en haut du scope par le moteur JS.
2. **`var`** est hissé et initialisé à `undefined`.
3. **`let` et `const**` sont hissés mais bloqués dans la *Temporal Dead Zone* (génèrent une erreur si lus trop tôt).
4. **Les fonctions classiques** sont entièrement hissées, ce qui permet de les déclarer n'importe où dans leur scope.
