
# Simulateur numérique 1D de l’équation d’advection

Ce projet implémente un simulateur pour l’équation d’advection linéaire en une dimension.
Il inclut plusieurs schémas numériques (Upwind et Lax–Wendroff), une solution exacte, une version parallèle utilisant des threads C++, des mesures de temps d’exécution, ainsi que des tests unitaires et un script Gnuplot pour la visualisation.

---

## 1. Modèle mathématique

Le phénomène étudié est décrit par l’équation :

$$
\frac{\partial u}{\partial t} + a,\frac{\partial u}{\partial x} = 0, \qquad a>0
$$

où :

* (u(x,t)) est la quantité transportée,
* (a) est la vitesse d’advection (constante).

### 1.1 Domaine et maillage

Le domaine est défini par :

* un intervalle temporel ([t_{\text{ini}},t_{\text{final}}]) avec un pas (\Delta t),
* un intervalle spatial ([x_{\min},x_{\max}]) avec un pas (\Delta x).

Le nombre total de points spatiaux est :

$$
N_x = \frac{x_{\max} - x_{\min}}{\Delta x} + 1.
$$

Un maillage uniforme est utilisé pour la discrétisation.

### 1.2 Condition initiale

La condition initiale choisie est une gaussienne centrée au milieu du domaine :

$$
u(x,0) = \frac{1}{\sigma\sqrt{2\pi}}
\exp\left(-\frac{(x-\mu)^2}{2\sigma^2}\right),
$$

avec :

* (\mu = \dfrac{x_{\min} + x_{\max}}{2}),
* (\sigma = 10 \Delta x).

Cette fonction sert de profil initial transporté par l’équation.

### 1.3 Solution exacte

La solution analytique est une simple translation de la condition initiale :

$$
u_{\text{exact}}(x,t) = u_0(x - at),
$$

ce qui permet une comparaison directe avec les solutions numériques.

---

## 2. Schémas numériques

Les méthodes employées sont explicites et utilisent un maillage uniforme.

### 2.1 Condition CFL

La stabilité des schémas est gouvernée par le nombre de Courant :

$$
\text{CFL} = \frac{a,\Delta t}{\Delta x}.
$$

Une valeur proche de 0.5 est adoptée dans le projet.

### 2.2 Schéma Upwind (ordre 1)

Le schéma d’ordre 1 repose sur une discrétisation décentrée amont :

$$
u_i^{n+1} = u_i^n - \text{CFL},(u_i^n - u_{i-1}^n).
$$

Ce schéma est stable et simple, mais souffre de diffusion numérique.

### 2.3 Schéma de Lax–Wendroff (ordre 2)

Le schéma d’ordre 2 améliore la précision et limite la diffusion :

$$
u_i^{n+1} = u_i^n

* \frac{\text{CFL}}{2}(u_{i+1}^n - u_{i-1}^n)

- \frac{\text{CFL}^2}{2}(u_{i+1}^n - 2u_i^n + u_{i-1}^n).
  $$

Il conserve mieux la forme de la gaussienne, mais peut générer des oscillations en présence de forts gradients.

---

## 3. Architecture du projet

### 3.1 Maillage

* **IMesh** : interface décrivant les accès au maillage (positions, temps, pas, tailles).
* **UniformMesh** : maillage uniforme utilisé dans toutes les simulations.

### 3.2 Stockage des variables

* **Variable** contient les valeurs discrétisées de chaque champ aux points du maillage.
* Les valeurs peuvent être écrites dans des fichiers `.data` (un fichier par variable et par itération).

### 3.3 Équation et schémas

* **Equation** :

  * calcule la condition initiale gaussienne,
  * calcule la solution exacte,
  * met en œuvre le schéma Upwind,
  * fournit un schéma d’ordre 2 (méthode interne ou via templates).

* **Upwind** et **LaxWendroff** :

  * fournissent les mises à jour spécifiques pour chaque schéma.

### 3.4 Mesure de temps

* **Timer** mesure la durée d’une itération et du calcul complet.

### 3.5 Résolution

La classe **Problem** réalise :

* la création des variables,
* la boucle temporelle,
* l’appel aux schémas numériques,
* l’écriture des résultats,
* l’exécution séquentielle ou parallèle.

### 3.6 Programme principal

* configuration du maillage et de la vitesse d’advection,
* création des objets Equation et Problem,
* lancement du calcul.

### 3.7 Tests unitaires

Basés sur GoogleTest, ils vérifient :

* la cohérence du maillage,
* le comportement des variables,
* l’absence d’erreurs dans Equation et Problem.

### 3.8 Visualisation

Le script Gnuplot `plot.gp` permet de tracer :

* la condition initiale,
* la solution numérique (Upwind),
* la solution exacte,
* la solution d’ordre 2 si souhaité.

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
* GoogleTest installé
* Threads C++

### Commandes

```bash
mkdir -p build
cd build
cmake ..
cmake --build .
```

Les exécutables générés sont :

* l’exécutable principal de simulation,
* l’exécutable de tests unitaires.

---

## 6. Exécution de la simulation

### Mode séquentiel

Depuis `build` :

```bash
./main.exe
```

Des fichiers de sortie sont générés :

```
Variable_<nom>_<iteration>.data
```

Ils contiennent les valeurs obtenues à chaque pas de temps.

### Mode parallèle

Si la méthode `solve_parallel()` est activée :

```bash
./main.exe
```

La mise à jour des solutions est alors répartie sur plusieurs threads.

---

## 7. Visualisation des résultats

Depuis la racine du projet :

```bash
gnuplot plot.gp
```

Une image `plot.png` est générée et montre :

* la condition initiale,
* les solutions numériques,
* la solution exacte.

---

## 8. Tests unitaires

Depuis `build` :

### Avec CTest

```bash
ctest
```

### Ou en lançant directement l’exécutable

```bash
./unit_tests
```

---

## 9. Paramètres modifiables

Dans le fichier principal, les paramètres ajustables sont :

* temps initial et final,
* pas de temps,
* domaine spatial,
* pas en espace,
* vitesse d’advection,
* choix du schéma (ordre 1 ou ordre 2),
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

Ce README présente le fonctionnement du simulateur, les formules mathématiques, l’organisation du projet, les commandes d’exécution et les outils disponibles pour analyser les résultats.
