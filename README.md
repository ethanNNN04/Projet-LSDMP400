# Simulation électrostatique FEM 3D — Électrodes coplanaires

## Description

Ce notebook implémente une simulation de l'**équation de Laplace** en électrostatique
dans un domaine 3D, à l'aide de la **méthode des éléments finis (FEM)**.

Le problème modélisé est celui d'un **capteur capacitif à électrodes coplanaires** :
une ou deux électrodes sont posées sur la face supérieure d'un bloc isolant. On calcule
la distribution du **potentiel électrique** φ et du **champ électrique** **E** à l'intérieur
du matériau en résolvant :

```
∇ · (ε ∇φ) = 0
```

avec des conditions aux limites de Dirichlet (potentiel imposé sur les électrodes et sur
la face inférieure du domaine).

---

## Structure du notebook

| Section | Contenu |
|---|---|
| 1. Imports | Chargement des bibliothèques Python |
| 2. Classe `CoplanarElectrodesFEM` | Maillage tétraédrique, assemblage FEM, résolution |
| 3. Paramètres | Géométrie, résolution, matériau, tensions |
| 4. Résolution & visualisation | Exécution de la simulation, tracé des coupes 2D |
| 5. Profil V(z) | Analyse de la variation du potentiel selon la profondeur |
| 6. Sauvegarde | Export des résultats (`.npz`) |

---

## Fonctionnement détaillé

### 1. Génération du maillage
Le domaine 3D (longueur × hauteur × profondeur) est discrétisé en hexaèdres réguliers,
chacun subdivisé en **6 tétraèdres linéaires (P1)**. Les nœuds des électrodes et de la
face inférieure sont identifiés automatiquement.

### 2. Assemblage FEM
Pour chaque tétraèdre, la **matrice de rigidité locale 4×4** est calculée à partir des
gradients des fonctions de forme P1. La matrice globale sparse est assemblée par accumulation.

### 3. Conditions aux limites
Les conditions de Dirichlet sont imposées par substitution directe :
- Électrode 1 → `V_ELECTRODE1` (ex. 1 V)
- Électrode 2 (optionnelle) → `V_ELECTRODE2`
- Face inférieure → `V_BOTTOM = 0 V` (référence)

### 4. Résolution
Le système linéaire sparse `K · φ = F` est résolu avec `scipy.sparse.linalg.spsolve`.

### 5. Champ électrique
Le champ **E = −∇φ** est calculé élément par élément (constant par tétraèdre P1).

### 6. Visualisations produites
- `coplanar_3d_results.png` — maillage, potentiel et lignes équipotentielles (coupe z)
- `coplanar_3d_profiles.png` — profils V(y) et V(x) pour différentes positions
- `coplanar_3d_profile_z.png` — variation du potentiel selon la profondeur z

---

## Prérequis

### Python
Python **3.8 ou supérieur** est requis.

### Bibliothèques Python

Installer toutes les dépendances en une commande :

```bash
pip install -r fonctions_nécéssaires.txt
```

Les bibliothèques utilisées sont :

| Bibliothèque | Usage |
|---|---|
| `numpy` | Calcul matriciel, maillage, algèbre linéaire |
| `matplotlib` | Tracé des résultats (tricontourf, contour, profils) |
| `scipy` | Matrices sparse (`lil_matrix`, `csr_matrix`) et solveur (`spsolve`) |

> **Note :** `matplotlib.tri.Triangulation` et `matplotlib.patches` font partie de
> `matplotlib` et ne nécessitent pas d'installation séparée.

### Jupyter
Pour exécuter le notebook, il faut Jupyter Lab ou Jupyter Notebook :

```bash
pip install jupyterlab
# ou
pip install notebook
```

Lancer ensuite :

```bash
jupyter lab Notebook_SP4_config_3D.ipynb
# ou
jupyter notebook Notebook_SP4_config_3D.ipynb
```

---

## Paramètres configurables

Les principaux paramètres à ajuster se trouvent dans la **Section 3** du notebook :

| Paramètre | Valeur par défaut | Description |
|---|---|---|
| `LENGTH` | 1.0 m | Longueur du domaine (x) |
| `HEIGHT` | 0.5 m | Hauteur du domaine (y) |
| `DEPTH` | 0.3 m | Épaisseur du domaine (z) |
| `NX`, `NY`, `NZ` | 60, 40, 20 | Résolution du maillage (⚠ augmenter = plus lent) |
| `EPSILON_R` | 4.0 | Permittivité relative du matériau isolant |
| `ELECTRODE1_POS` | `(0.4, 0.6, 0.0, 0.0)` | Position (x_start, x_end, z_start, z_end) en m |
| `V_ELECTRODE1` | 1.0 V | Tension appliquée à l'électrode 1 |
| `V_BOTTOM` | 0.0 V | Potentiel de référence (face inférieure) |

---

## Sorties

| Fichier | Description |
|---|---|
| `coplanar_3d_results.png` | Carte du potentiel et équipotentielles |
| `coplanar_3d_profiles.png` | Profils de potentiel selon x et y |
| `coplanar_3d_profile_z.png` | Variation du potentiel selon z |
| `coplanar_data.npz` | Données brutes (nœuds, éléments, φ, E) au format NumPy |

---
## Tests

### Tests unitaires

-   Vérification du maillage
Objectif : vérifier que la discrétisation est correcte

sim = CoplanarElectrodesFEM()

assert sim.n_nodes > 0
assert sim.n_elements > 0
assert sim.elements.shape[1] == 4

___

## Avertissement sur les performances

Le maillage par défaut (60×40×20) génère environ **48 000 nœuds** et **1.3 million de
tétraèdres**. L'assemblage de la matrice de rigidité est l'étape la plus coûteuse
(boucle Python sur les éléments). Pour des tests rapides, réduire `NX`, `NY`, `NZ`
(ex. 20, 15, 10).
