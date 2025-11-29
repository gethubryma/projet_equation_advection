# Simulateur numérique 1D de l’équation d’advection

Ce projet propose un simulateur de l’équation d’advection linéaire en une dimension.
Il intègre plusieurs schémas numériques (Upwind d’ordre 1 et Lax–Wendroff d’ordre 2), une solution exacte permettant l’évaluation de la précision, ainsi que des outils de mesure du temps d’exécution et une version parallèle reposant sur les threads C++.
Des tests unitaires (GoogleTest) assurent la validité des composants, et un script Gnuplot facilite la visualisation des résultats.

---

## 1. Modèle mathématique

Le phénomène étudié est décrit par l’équation d’advection :

[
\frac{\partial u}{\partial t} + a , \frac{\partial u}{\partial x} = 0, \quad a > 0
]

où :

* ( u(x,t) ) représente la quantité transportée,
* ( a ) est la vitesse constante de propagation.

### 1.1 Domaine et maillage

Le domaine est discrétisé à l’aide d’un maillage uniforme construit sur :

* un intervalle temporel : ( t \in [t_{\text{ini}}, t_{\text{final}}] ) avec le pas ( \Delta t ),
* un intervalle spatial : ( x \in [x_{\min}, x_{\max}] ) avec le pas ( \Delta x ).

Le nombre total de points spatiaux est donné par :

[
N_x = \frac{x_{\max} - x_{\min}}{\Delta x} + 1.
]

### 1.2 Condition initiale

La condition initiale correspond à une gaussienne centrée au milieu du domaine :

[
u(x,0) = \frac{1}{\sigma\sqrt{2\pi}}
\exp\left(-\frac{(x-\mu)^2}{2\sigma^2}\right),
]

avec :

* ( \mu = (x_{\min} + x_{\max})/2 ),
* ( \sigma = 10\Delta x ).

Cette fonction sert de profil initial à transporter.

### 1.3 Solution exacte

La solution analytique est obtenue simplement par translation :

[
u_{\text{exact}}(x,t) = u_0(x - at),
]

ce qui permet de comparer la précision des schémas numériques.

---

## 2. Schémas numériques employés

Les méthodes utilisées sont explicites et reposent sur un maillage uniforme.

### 2.1 Condition CFL

La stabilité est régie par le nombre de Courant :

[
\text{CFL} = \frac{a\Delta t}{\Delta x}.
]

Une valeur proche de 0.5 est adoptée dans le projet.

### 2.2 Schéma Upwind (ordre 1)

Le schéma d’ordre 1 utilise une approche décentrée amont :

[
u_i^{n+1} = u_i^n - \text{CFL} (u_i^n - u_{i-1}^n).
]

Ce schéma est robuste mais introduit de la diffusion numérique.

### 2.3 Schéma de Lax–Wendroff (ordre 2)

Le schéma de Lax–Wendroff améliore l’ordre de précision et limite la diffusion :

[
u_i^{n+1} = u_i^n

* \frac{\text{CFL}}{2}(u_{i+1}^n - u_{i-1}^n)

- \frac{\text{CFL}^2}{2}(u_{i+1}^n - 2u_i^n + u_{i-1}^n).
  ]

Il réduit la perte de forme mais peut générer des oscillations en présence de gradients abrupts.

---

## 3. Structure générale du projet

### 3.1 Maillage

* **IMesh** : interface définissant les accès au maillage.
* **UniformMesh** : implémentation d’un maillage uniforme en temps et espace.

### 3.2 Stockage des variables

* **Variable** gère un tableau de valeurs correspondant aux points du maillage et propose une fonction d’écriture dans des fichiers `.data`.

### 3.3 Équation et schémas numériques

* **Equation** rassemble :

  * le calcul de la condition initiale,
  * la solution exacte,
  * le schéma Upwind,
  * le schéma d’ordre 2 (implémentation directe ou via template).

* **Upwind** et **LaxWendroff** fournissent les méthodes statiques de mise à jour pour chaque schéma.

### 3.4 Mesure du temps

* **Timer** permet la mesure du temps d’une itération et du temps total du calcul.

### 3.5 Résolution

* **Problem** coordonne l’ensemble :

  * initialisation,
  * boucle temporelle,
  * application des schémas,
  * écriture des fichiers résultats,
  * version séquentielle ou version parallèle (via `std::thread`).

### 3.6 Programme principal

* **main.cpp** configure le maillage, la vitesse d’advection, instancie le problème et lance la simulation.

### 3.7 Tests unitaires

* Des tests, basés sur GoogleTest, verifient :

  * la cohérence du maillage,
  * le fonctionnement de `Variable`,
  * l’absence d’erreurs dans `Equation` et `Problem`.

### 3.8 Visualisation

* **plot.gp** contient les instructions Gnuplot pour tracer les solutions :

  * condition initiale,
  * solution numérique upwind,
  * solution exacte,
  * solution d’ordre 2 si nécessaire.

---

## 5. Compilation

### Prérequis

* CMake ≥ 3.10
* Compilateur compatible C++17
* GoogleTest installé
* Threads (gérés automatiquement par CMake)

### Étapes

```bash
mkdir -p build
cd build
cmake ..
cmake --build .
```

La compilation produit principalement :

* l’exécutable de simulation,
* l’exécutable de tests unitaires.

---

## 6. Exécution de la simulation

### Version séquentielle

Depuis `build` :

```bash
./main.exe
```

La simulation génère à chaque itération des fichiers :

```
Variable_<nom>_<iteration>.data
```

Ces fichiers contiennent les valeurs de la solution numérique et exacte à chaque pas de temps.

### Version parallèle

Si la méthode parallèle (`solve_parallel`) est activée dans le code principal, l’exécutable s’utilise de la même manière :

```bash
./main.exe
```

Le calcul de chaque étape est alors réparti sur plusieurs threads.

---

## 7. Visualisation des résultats

Le script Gnuplot `plot.gp` permet de générer un graphique :

```bash
gnuplot plot.gp
```

Le fichier image produit compare :

* la condition initiale,
* la solution numérique,
* la solution exacte au même instant.

---

## 8. Tests unitaires

Les tests sont exécutés depuis `build` :

### Via ctest

```bash
ctest
```

### Ou directement

```bash
./unit_tests
```

Les tests vérifient le bon comportement du maillage, de la gestion des variables et des composants de calcul.

---

## 9. Paramètres modifiables

Plusieurs choix numériques peuvent être ajustés dans `main.cpp` :

* durée de la simulation,
* pas de temps,
* domaine spatial,
* pas en espace,
* vitesse d’advection,
* choix du schéma (ordre 1 ou ordre 2),
* activation ou non de la version parallèle.

---

## 10. Résumé rapide

1. **Compilation** :

```bash
mkdir -p build
cd build
cmake ..
cmake --build .
```

2. **Exécution** :

```bash
./main.exe
```

3. **Visualisation** :

```bash
gnuplot plot.gp
```

4. **Tests** :

```bash
ctest
```

Ce document présente l’essentiel du fonctionnement, des équations, de l’organisation et de l’utilisation du simulateur d’advection numérique.
