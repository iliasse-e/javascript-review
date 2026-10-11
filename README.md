# Promesse, Event loop et design pattern (COURS & Exercices)


## Promesses de base

Quel est l'ordre de l'execution des logs ?

```js
console.log('1');

setTimeout(() => {
  console.log('2');
}, 0);

Promise.resolve().then(() => {
  console.log('3');
});

console.log('4');
```

**REPONSE : Début / Exécution de la promesse / Fin / Succès !**

Exécution de la promesse s'affiche en deuxième : C'est le piège ! 
Le code contenu dans l'exécutable d'une promesse (new Promise(executor)) est 
exécuté de manière synchrone au moment même où la promesse est créée. Ce n'est pas asynchrone.


## Microtask vs macrotask

Quel est l'ordre exact des logs pour ce code ?

```js
console.log('1');

setTimeout(() => {
  console.log('2');
}, 0);

Promise.resolve().then(() => {
  console.log('3');
});

console.log('4');
```

**REPONSE : 1 4 3 2**

En JavaScript, le code est géré par priorité :

Le code synchrone (la pile d'exécution principale).

Les microtâches (les ``.then()`` des promesses, ``queueMicrotask``). Elles ont la priorité absolue dès que le code synchrone est terminé.

Les macrotâches (les ``setTimeout``, ``setInterval``, événements I/O). Elles attendent que toutes les microtâches soient vidées.

## COURS : EVENT LOOP

L'**Event Loop** (ou la *boucle d'événements* en français) est le mécanisme magique qui permet à JavaScript — qui est pourtant un langage à **un seul fil d'exécution (*single-threaded*)** — d'être **asynchrone** et de gérer des tâches longues (comme télécharger un fichier ou attendre une requête serveur) sans bloquer ton interface ou ton application.

Pour comprendre comment elle fonctionne, imagine que JavaScript est un cuisinier unique dans une cuisine.

---

### 1. Les 3 grands acteurs de la cuisine

Pour faire tourner l'Event Loop, le moteur de JavaScript (comme V8 dans Chrome/Node.js) s'appuie sur trois éléments principaux :

1. **La Call Stack (La Pile d'exécution) :**
C'est le plan de travail du cuisinier. Il n'y a **qu'une seule** pile. Le cuisinier fait les choses une par une, dans l'ordre, de haut en bas (c'est le code synchrone).
2. **Les Web APIs (ou l'Environnement d'exécution) :**
Ce sont les commis de cuisine (fournis par le navigateur ou Node.js). Quand tu lances un `setTimeout`, une requête HTTP (`fetch`) ou un écouteur d'événement (`click`), JavaScript délègue la tâche à ces commis pour libérer le cuisinier principal. Pendant que le commis attend que le temps soit écoulé ou que le serveur réponde, le cuisinier continue de faire autre chose.
3. **Les Files d'attente (Queues) :**
C'est le passe-plat où les commis déposent les plats prêts à être servis :
* **La Microtask Queue (Priorité VIP) :** C'est là que vont les `.then()` des promesses, `queueMicrotask`, etc.
* **La Macrotask Queue (Priorité normale) :** C'est là que vont les `setTimeout`, `setInterval`, événements DOM, etc.



---

### 2. Comment fonctionne l'Event Loop (La Boucle) ?

L'Event Loop est un surveillant infatigable qui tourne en boucle pour vérifier deux choses :

1. **Est-ce que la Call Stack (la pile du cuisinier) est totalement vide ?**
2. **S'il y a des tâches dans les files d'attente, est-ce que je peux les faire remonter ?**

Dès que la Call Stack est vide, l'Event Loop applique une règle de priorité absolue :

* **Étape 1 :** Elle vide **toute** la `Microtask Queue` (les promesses). S'il y a de nouvelles microtâches générées pendant qu'elle la vide, elle les traite aussi immédiatement.
* **Étape 2 :** Dès que la `Microtask Queue` est complètement vide, elle prend **une seule** macrotâche de la `Macrotask Queue` (par exemple, un `setTimeout`) et la pose dans la Call Stack.
* **Étape 3 :** On recommence la boucle !

---

### 3. Résumé en image avec ton avant-dernier exercice

Reprenons ton code :

```javascript
console.log('1');                 // 1. Synchrone -> Direct dans la Call Stack -> Affiche '1'

setTimeout(() => {                // 2. Le setTimeout est délégué aux Web APIs (commis)
  console.log('2');               //    Quand le timer expire, le commis met ça dans la Macrotask Queue
}, 0);

Promise.resolve().then(() => {    // 3. La promesse est résolue immédiatement :
  console.log('3');               //    Le `.then` est mis dans la Microtask Queue (VIP)
});

console.log('4');                 // 4. Synchrone -> Direct dans la Call Stack -> Affiche '4'
```

* **Fin du code synchrone :** La Call Stack est vide. Les logs affichés sont **1** et **4**.
* **L'Event Loop regarde les files :**
* Elle voit le `3` dans la **Microtask Queue** (VIP). Elle l'exécute $\rightarrow$ Affiche **3**.
* La Microtask Queue est vide. Elle regarde la **Macrotask Queue**, y trouve le `2`. Elle l'exécute $\rightarrow$ Affiche **2**.



C'est grâce à ce système de files d'attente et de priorités que JavaScript ne se bloque jamais, même s'il jongle avec des milliers de tâches asynchrones !

## Async await

```js
const f = async () => {
  console.log('1');
  
  try {
    const res = await Promise.reject('Erreur critique');
    console.log('2', res);
  } catch (err) {
    console.log('3', err);
  } finally {
    console.log('4');
  }
  
  console.log('5');
};

console.log('A');
f();
console.log('B');
```

#### Décortiquons pas à pas pour comprendre où s'est glissé le piège :

#### 1. ``A`` s'affiche : C'est du code synchrone classique au niveau global.

#### 2. On appelle ``f()`` : La fonction async démarre immédiatement... de manière synchrone jusqu'au premier await.
- Elle affiche donc 1.
- Elle arrive sur ``await Promise.reject('Erreur critique')``. La promesse rejette, ce qui déclenche l'interruption de la fonction ``f()``. **Le reste de f() est mis en pause et envoyé dans la file des microtâches.**

#### 3. B s'affiche : La fonction ``f()`` étant mise en pause, le code global synchrone reprend la main. Le ``console.log('B')`` s'exécute.(Fin du code global synchrone. Les logs affichés pour l'instant sont A, 1, B).

#### 4. L'Event Loop vide les microtâches :

- La promesse rejetée réveille la fonction ``f()``. 
- L'erreur est interceptée par le catch.Le ``console.log('3', err)`` s'exécute $\rightarrow$ Affiche 3 Erreur critique.
- Le bloc finally s'exécute ensuite $\rightarrow$ Affiche 4.
- Enfin, la suite de la fonction s'exécute $\rightarrow$ Affiche 5.

**La grande règle à retenir sur async/await :**

    Tout le code situé au début d'une fonction async avant le premier await est totalement synchrone. Dès qu'on croise un await, la fonction se met en pause, rend la main au reste du script, et attend que la promesse soit réglée pour reprendre plus tard dans les **microtâches**.


## Séquentiel vs Parallèle

```js
const delay = (ms, val) => new Promise(resolve => setTimeout(() => resolve(val), ms));

const processData = async () => {
  console.log('Début');

  // Cas 1 : Exécution séquentielle
  const res1 = await delay(100, 'A');
  const res2 = await delay(100, 'B');
  console.log(res1, res2);

  // Cas 2 : Exécution en parallèle
  const p1 = delay(100, 'C');
  const p2 = delay(100, 'D');
  const res3 = await delay(100, 'E'); // Petit piège ici !
  const [res4, res5] = await Promise.all([p1, p2]);
  console.log(res3, res4, res5);

  console.log('Fin');
};

processData();
```

**Questions **

Quel est le comportement du Cas 1 (les await l'un après l'autre) ? Sont-ils lancés en même temps ou l'un après l'autre ?

Dans le Cas 2, quand est-ce que les promesses p1 et p2 commencent-elles réellement à s'exécuter par rapport à res3 ?

Quel est l'ordre global des logs affichés à la fin ?

**Correction détaillée**
#### 1. Pour les await successifs (Cas 1) :

  Ta réponse : "A, B et non ils sont lancés un après l'autre, il y a un respect de pile je pense ?"

  Verdict : Exact pour le "l'un après l'autre" !

  Explication : On appelle cela une exécution séquentielle. Le premier await delay(100, 'A') bloque l'exécution de la fonction pendant 100ms. Le second await ne se lance que lorsque le premier est terminé. Au total, le Cas 1 prend 200ms.

#### 2. Pour le Cas 2 (p1, p2, res3 et Promise.all) :

  **Ta réponse :** "Comme il n'y a pas de await, je sais pas si Promise.all répond... le log affiche E et peut être des promesse pending..."

  **Verdict :** C'est là que se trouve le piège le plus important de l'exercice !

  **Explication :**

    Regarde bien comment p1 et p2 sont créées : const p1 = delay(100, 'C');. Dès qu'on appelle delay(), la promesse est créée et se lance immédiatement en arrière-plan (dans les Web APIs / Macrotâches), même sans await.

    Ensuite, on lance const res3 = await delay(100, 'E');. Celui-ci est séquentiel.

    Pendant ces 100ms où on attend res3, les 100ms de p1 et p2 se déroulent en même temps (en parallèle) !

    Quand on arrive à await Promise.all([p1, p2]), les promesses p1 et p2 sont déjà terminées (ou presque, puisqu'elles ont démarré en même temps que res3). Le Promise.all ne renvoie donc pas des objets pending, mais bien les vraies valeurs résolues ('C' et 'D').


#### 3. Pour l'ordre global des logs :

  **Ta réponse :** Debut Fin A B E (avec une petite confusion sur l'ordre de Fin et E).

  La vraie chronologie des logs :

      Début (synchrone au début de la fonction).

      A, B (après les 200ms du Cas 1).

      E, C, D (après l'attente du Cas 2).

      Fin (tout à la fin).

💡 Ce qu'il faut absolument retenir :

  ``await`` met en pause : Il bloque la suite de la fonction jusqu'à ce que la promesse soit résolue.

  Créer une promesse sans ``await`` la lance tout de suite : const p = delay(...) démarre l'action asynchrone immédiatement. C'est l'astuce pour lancer des tâches en parallèle avant d'attendre leur résultat groupé avec Promise.all.


## COURS : Design patterns asynchrones

Optimiser les performances grâce à l'exécution en parallèle est l'un des meilleurs réflexes à acquérir en JavaScript, surtout lorsqu'on fait des requêtes réseau (appels API, base de données, etc.).

Voyons comment fonctionne `Promise.all` et comment il peut transformer un code lent en un code ultra-rapide.

---

### 1. Le problème : Le piège de l'attente séquentielle (Waterfall)

Imagine que tu doives récupérer le profil d'un utilisateur, ses amis et ses messages. Si tu fais tes requêtes l'une après l'autre avec `await` :

```javascript
const fetchUserData = async () => {
  console.time('Temps total');

  const profile = await fetch('/api/profile'); // Prend 300ms
  const friends = await fetch('/api/friends'); // Prend 300ms
  const posts = await fetch('/api/posts');     // Prend 300ms

  console.timeEnd('Temps total');
  // Résultat : ~900ms (300ms + 300ms + 300ms)
};
```

**Le problème :** Tes trois requêtes n'ont aucun rapport entre elles, mais tu attends que la première finisse pour lancer la deuxième, et ainsi de suite. C'est ce qu'on appelle un effet *waterfall* (cascade). Ton utilisateur attend 3 fois plus longtemps que nécessaire !

---

### 2. La solution : `Promise.all` pour tout lancer en même temps

Pour corriger cela, tu lances toutes tes promesses **en même temps** (sans `await`), puis tu les passes à `Promise.all` qui va attendre que *toutes* soient terminées :

```javascript
const fetchUserDataOptimized = async () => {
  console.time('Temps total');

  // 1. On lance les trois requêtes en parallèle (immédiatement)
  const profilePromise = fetch('/api/profile');
  const friendsPromise = fetch('/api/friends');
  const postsPromise = fetch('/api/posts');

  // 2. On attend que tout soit prêt en même temps
  const [profile, friends, posts] = await Promise.all([
    profilePromise,
    friendsPromise,
    postsPromise
  ]);

  console.timeEnd('Temps total');
  // Résultat : ~300ms (le temps de la requête la plus longue des trois !)
```

**Gain de performance :** Ton code est 3 fois plus rapide, car les requêtes se font en même temps.

---

### 3. Les subtilités et pièges de `Promise.all`

Bien que puissant, `Promise.all` a une règle stricte qu'il faut connaître : **C'est "tout ou rien" (fail-fast).**

* **Si toutes les promesses réussissent :** `Promise.all` te renvoie un tableau avec tous les résultats dans le même ordre que tu les as fournis.
* **Si *une seule* promesse échoue (rejette) :** `Promise.all` plante immédiatement. Il rejette la promesse globale, même si les deux autres requêtes avaient réussi en arrière-plan.

#### Comment gérer les échecs partiels ? (`Promise.allSettled`)

Si tu veux récupérer les résultats de toutes tes promesses, même si certaines échouent, utilise **`Promise.allSettled`** :

```javascript
const results = await Promise.allSettled([
  fetch('/api/profile'),
  fetch('/api/bad-url') // Va échouer
]);

results.forEach((result) => {
  if (result.status === 'fulfilled') {
    console.log('Succès :', result.value);
  } else {
    console.log('Échec :', result.reason);
  }
});
```

`Promise.allSettled` ne rejette jamais : il te donne un tableau d'objets qui détaillent le statut (`fulfilled` ou `rejected`) de chaque promesse individuellement.


**Il existe aussi d'autres méthodes :**

- ``Promise.race``	La première qui finit (succès ou échec)	Renvoie le résultat de la plus rapide, qu'elle ait réussi ou planté.

- ``Promise.any``	La première qui réussit	Renvoie le succès de la plus rapide (ignore les échecs des autres). Plante uniquement si tout le monde échoue.
