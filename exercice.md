Rien ne vaut un bon exercice de lecture de code pour valider que la *Temporal Dead Zone* et le hoisting n'ont plus aucun secret pour toi !

---

### 🏋️ Exercice : Le piège des scopes et du Hoisting

Regarde ce code attentivement. Certains blocs utilisent `var`, d'autres `let`, et il y a une subtilité avec les fonctions.

```javascript
var x = 10;
let y = 20;

const testHoisting = () => {
  console.log(x);
  console.log(y);

  if (true) {
    var x = 30;
    let y = 40;
    console.log(x);
    console.log(y);
  }

  console.log(x);
  console.log(y);
};

testHoisting();

```

**La question :** Que va afficher exactement ce code dans la console ? (Attention aux erreurs potentielles ou aux modifications de variables de portée !).

Prends ton temps pour décortiquer le scope de la fonction et celui du bloc `if`. À toi de jouer !



## REPONSE


Voici ce que la console affiche réellement :

```javascript
undefined
20
30
40
30
20

```

---

### Pourquoi ce résultat ? Décomposons pas à pas :

#### 1. Les premiers `console.log(x)` et `console.log(y)` :

* **Pour `y` (affiche `20`) :** `y` vaut `20` (la variable globale). Comme il n'y a pas de `let y` au début de la fonction, la fonction va chercher `y` dans le scope parent (global).
* **Pour `x` (affiche `undefined`) — LE GROS PIÈGE :**
Tu pourrais te dire qu'il va afficher `10` (la variable globale `var x = 10`). **Mais non !**
À cause du **Hoisting**, la déclaration `var x = 30;` située plus bas *à l'intérieur du bloc `if*` est hissée... **tout en haut du scope de la fonction `testHoisting**`.
Du coup, au tout début de la fonction, JavaScript sait que `x` existe localement, mais sa valeur n'est pas encore initialisée $\rightarrow$ elle vaut `undefined`.

#### 2. À l'intérieur du bloc `if` :

* `var x = 30;` écrase la variable `x` locale de la fonction $\rightarrow$ affiche **`30`**.
* `let y = 40;` crée une *nouvelle* variable `y` propre au bloc `if` (grâce à la portée de bloc des `let`) $\rightarrow$ affiche **`40`**.

#### 3. Après le bloc `if` :

* **Pour `x` (affiche `30`) :** Le `x` modifié dans le `if` a modifié la variable locale `x` de toute la fonction.
* **Pour `y` (affiche `20`) :** Le `let y = 40;` du bloc `if` est mort en sortant du bloc. Le `console.log(y)` de la fin va donc réutiliser le `y` global qui valait `20`.

---

### 💡 La grande leçon à retenir :

1. **`var` n'a PAS de portée de bloc :** Il est hissé soit au niveau global, soit tout en haut de la **fonction** qui l'entoure. C'est pour cela qu'il crée des bugs de "shadowing" (masquage) involontaires.
2. **`let` et `const` ont une portée de bloc :** Ils restent strictement cantonnés aux accolades `{}` dans lesquelles ils sont déclarés.
