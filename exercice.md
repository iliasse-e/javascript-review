C'est parti pour un exercice complet qui combine la **Scope Chain** (la recherche de variables) et le piège classique des blocs (`let` vs `var`).

---

### 🏋️ Exercice : Le labyrinthe des portées

Observe bien ce code et essaie de deviner ce que chaque `console.log` va afficher, ou s'il va y avoir une erreur :

```javascript
let globalVal = "Global";

const outerFunction = () => {
  let outerVal = "Outer";
  let globalVal = "Shadow Global"; // Attention au masquage !

  if (true) {
    let innerVal = "Inner";
    var leakedVar = "Leak";
    let globalVal = "Block Global";
    
    console.log(globalVal); 
    console.log(outerVal);  
    console.log(innerVal);  
  }

  console.log(leakedVar); 
  // console.log(innerVal); // Que se passerait-il ici ? (Réfléchis à celle-ci aussi)
  console.log(globalVal); 
};

outerFunction();

```

**Les questions :**

1. Que vont afficher les trois premiers `console.log` à l'intérieur du bloc `if` ?
2. Que va afficher le `console.log(leakedVar)` juste après le bloc `if` ?
3. Que va afficher le dernier `console.log(globalVal)` tout à la fin de la fonction ?

Prends ton temps, décompose les scopes (global, fonction, bloc) et donne-moi tes réponses !


### REPONSES

 "Block Global", "Outer", "Inner"
"Leak"
"Shadow Global" 
