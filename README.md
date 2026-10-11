
Le scope, c'est tout simplement **la zone de visibilité d'une variable** : où a-t-on le droit de l'utiliser, et d'où est-elle inaccessible ?

---

### 1. Les 3 grands types de scope en JavaScript

En JavaScript, il existe trois niveaux principaux de portée :

#### A. Le Scope Global (Global Scope)

* **Où ?** En dehors de toute fonction ou de tout bloc.
* **Visibilité :** Partout dans ton fichier JavaScript (et même accessible globalement via l'objet `window` dans le navigateur).
* **Exemple :**
```javascript
const appName = "MonSuperApp"; // Global

const logApp = () => {
  console.log(appName); // Accessible ici sans problème !
};

```


* ⚠️ **Attention :** Évite au maximum les variables globales, car n'importe quel script ou fonction peut les modifier par erreur (pollution du scope global).

#### B. Le Scope de Fonction (Function Scope)

* **Où ?** À l'intérieur d'une fonction.
* **Visibilité :** Les variables créées avec `var`, `let` ou `const` à l'intérieur d'une fonction ne sont **accessibles que dans cette fonction**. Elles sont totalement invisibles à l'extérieur.
* **Exemple :**
```javascript
const calculer = () => {
  const resultat = 42; // Scope de fonction
  console.log(resultat); // OK
};

calculer();
// console.log(resultat); -> ReferenceError: resultat is not defined !

```



#### C. Le Scope de Bloc (Block Scope)

* **Où ?** À l'intérieur de n'importe quelles accolades `{}` (comme un bloc `if`, un `for`, un `while`, ou juste des accolades isolées).
* **Visibilité :** C'est la grande révolution introduite avec ES6 (2015) grâce à **`let`** et **`const`**.
* ⚠️ **Rappel crucial :** `var` **ne respecte pas** le scope de bloc ! Il traverse les accolades pour se retrouver dans la fonction parente (ou en global). `let` et `const`, eux, restent bien sagement enfermés dans leur bloc.
* **Exemple :**
```javascript
if (true) {
  let secret = "Mot de passe";
  var publicVar = "Je suis partout";
}

// console.log(secret); -> ReferenceError (let est confiné au if)
console.log(publicVar); -> "Je suis partout" (var s'est échappé du if)

```



---

### 2. La règle d'or : La chaîne de portées (Scope Chain)

Comment fait JavaScript pour trouver une variable ? Il applique le principe de **la recherche du plus proche au plus lointain** (la *Scope Chain*).

1. Il regarde d'abord dans le scope **local** (la fonction ou le bloc actuel).
2. S'il ne la trouve pas, il remonte d'un cran dans le scope parent.
3. Il continue de remonter de proche en proche jusqu'à atteindre le scope global.
4. S'il ne la trouve nulle part tout en haut, il déclenche une `ReferenceError`.

*Analogie :* C'est comme chercher un outil. Tu regardes d'abord dans ta poche (scope local), puis dans la caisse à outils de ta pièce (scope parent), puis dans le garage de la maison (scope global). Si l'outil n'est nulle part, tu l'annonces (erreur).

---

### 3. Le cas particulier des Closures (Fermetures)

Puisqu'on parle de scope, impossible de ne pas mentionner la notion reine des entretiens JS : la **Closure**.

Une closure, c'est lorsqu' **une fonction interne se "souvient" et conserve l'accès au scope de sa fonction parente**, même une fois que la fonction parente a fini de s'exécuter.

```javascript
const creerCompteur = () => {
  let compteur = 0; // Variable dans le scope parent

  return () => {
    compteur++; // La fonction interne "ferme" (clôt) sur la variable compteur
    return compteur;
  };
};

const maFois = creerCompteur();
console.log(maFois()); // 1
console.log(maFois()); // 2 (la variable compteur est préservée grâce à la closure !)

```
