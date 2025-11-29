

# Simulateur numérique 1D de l’équation d’advection

Ce projet implémente un simulateur pour l’équation d’advection linéaire en une dimension.
Il inclut plusieurs schémas numériques (Upwind et Lax–Wendroff), une solution exacte, une version parallèle utilisant des threads C++, des mesures de temps d’exécution, ainsi que des tests unitaires et un script Gnuplot pour la visualisation.

---

## 1. Modèle mathématique

Le phénomène étudié est décrit par l’équation :

[
\frac{\partial u}{\partial t} + a,\frac{\partial u}{\partial x} = 0,
\qquad a > 0
]

où :

* ( u(x,t) ) représente la quantité transportée,
* ( a ) désigne la vitesse d’advection (constante).

### 1.1 Domaine et maillage

Le domaine est défini par :

* un intervalle temporel ([t_{\text{ini}}, t_{\text{final}}]) avec un pas (\Delta t),
* un intervalle spatial ([x_{\min}, x_{\max}]) avec un pas (\Delta x).

Le nombre total de points spatiaux est :

[
N_x = \frac{x_{\max} - x_{\min}}{\Delta x} + 1.
]

Un maillage uniforme est utilisé pour la discrétisation.

### 1.2 Condition initiale

La condition initiale est une gaussienne centrée au milieu du domaine :

[
u(x,0) =
\frac{1}{\sigma\sqrt{2\pi}}
\exp\left(-\frac{(x-\mu)^2}{2\sigma^2}\right),
]

avec :

* ( \mu = \dfrac{x_{\min} + x_{\max}}{2} ),
* ( \sigma = 10,\Delta x ).

### 1.3 Solution exacte

La solution analytique correspondant à l’équation d’advection est :

[
u_{\text{exact}}(x,t) = u_0(x - a t),
]

ce qui permet une comparaison directe avec la solution numérique.

---

## 2. Schémas numériques

Les méthodes employées sont explicites et utilisent un maillage uniforme.

### 2.1 Condition CFL

La stabilité des schémas est assurée par le nombre de Courant :

[
\text{CFL} = \frac{a,\Delta t}{\Delta x}.
]

Une valeur proche de 0.5 est utilisée.

### 2.2 Schéma Upwind (ordre 1)

Le schéma Upwind d’ordre 1 s’écrit :

[
u_i^{n+1}
= u_i^n - \text{CFL},(u_i^n - u_{i-1}^n).
]

Ce schéma est simple et stable, mais introduit une diffusion numérique perceptible.

### 2.3 Schéma de Lax–Wendroff (ordre 2)

Le schéma de Lax–Wendroff améliore la précision :

[
u_i^{n+1}
= u_i^n

* \frac{\text{CFL}}{2}(u_{i+1}^n - u_{i-1}^n)

- \frac{\text{CFL}^2}{2}(u_{i+1}^n - 2u_i^n + u_{i-1}^n).
  ]

Ce schéma conserve mieux la forme du signal mais peut produire des oscillations.

---

## 3. Architecture du projet

### 3.1 Maillage

* **IMesh** : interface pour les accès au maillage.
* **UniformMesh** : maillage uniforme utilisé pour toutes les simulations.

### 3.2 Stockage des variables

* **Variable** : stockage des valeurs discrétisées du champ.
* Export possible vers des fichiers `.data`.

### 3.3 Équation et schémas

* **Equation** :

  * calcule la condition initiale,
  * calcule la solution exacte,
  * implémente le schéma Upwind,
  * fournit un schéma d’ordre 2.
* **Upwind** et **LaxWendroff** : classes dédiées aux mises à jour.

### 3.4 Mesure de temps

* **Timer** : temps d’exécution des itérations et du calcul complet.

### 3.5 Résolution

La classe **Problem** effectue :

* la création des variables,
* la boucle temporelle,
* l’appel aux schémas,
* l’écriture des résultats,
* l’exécution séquentielle ou parallèle.

### 3.6 Programme principal

* configuration du maillage et de la vitesse,
* création des objets Equation et Problem,
* lancement du calcul.

### 3.7 Tests unitaires

Les tests GoogleTest vérifient :

* la cohérence du maillage,
* le comportement des variables,
* l’absence d’erreurs dans les méthodes de Equation et Problem.

### 3.8 Visualisation

Le script `plot.gp` permet de tracer :

* la condition initiale,
* la solution numérique Upwind,
* la solution exacte,
* la solution d’ordre 2.

---

## 4. Organisation du projet

* `main.cpp`
* `Problem.*`
* `Equation.*`
* `UniformMesh.*`
* `IMesh.h`
* `Variable.*`
* `Upwind.*`
* `LaxWendroff.*`
* `Timer.*`
* `tests.cpp`
* `plot.gp`
* `CMakeLists.txt`

---

## 5. Compilation

### Prérequis

* CMake ≥ 3.10
* Compilateur C++17
* GoogleTest
* Threads C++

### Commandes

```bash
mkdir -p build
cd build
cmake ..
cmake --build .
```

Les exécutables générés sont :

* l’exécutable principal,
* l’exécutable des tests unitaires.

---

## 6. Exécution de la simulation

### Mode séquentiel

Depuis `build` :

```bash
./main.exe
```

Des fichiers de sortie sont produits :

```
Variable_<nom>_<iteration>.data
```

### Mode parallèle

Si la fonction `solve_parallel()` est activée :

```bash
./main.exe
```

---

## 7. Visualisation des résultats

Depuis la racine du projet :

```bash
gnuplot plot.gp
```

Le fichier `plot.png` est généré.

---

## 8. Tests unitaires

Depuis `build` :

### Avec CTest

```bash
ctest
```

### Ou exécution directe

```bash
./unit_tests
```

---

## 9. Paramètres modifiables

Les paramètres suivants peuvent être ajustés :

* temps initial et final,
* pas de temps,
* domaine spatial,
* pas en espace,
* vitesse d’advection,
* choix du schéma (ordre 1 ou 2),
* mode séquentiel ou parallèle.

---

## 10. Résumé

1. Compilation :

```bash
mkdir build
cd build
cmake ..
cmake --build .
```

2. Simulation :

```bash
./main.exe
```

3. Visualisation :

```bash
gnuplot plot.gp
```

4. Tests :

```bash
ctest
```

---

Si une version optimisée pour l’affichage GitHub ou une version avec formules simplifiées est nécessaire, elle peut être générée.


Ce README présente le fonctionnement du simulateur, les formules mathématiques, l’organisation du projet, les commandes d’exécution et les outils disponibles pour analyser les résultats.
