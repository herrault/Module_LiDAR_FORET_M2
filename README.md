# Module_LiDAR_FORET_M2

# Analyse de structure de canopée et classification de peuplements à partir de données LiDAR

**Niveau** : Master 2  
**Durée** : 3 séances de 3h  
**Pré-requis** : notions de base en R, connaissance générale des écosystèmes forestiers  
**Logiciels utilisés** : CloudCompare, R (packages `lidR`, `terra`, `sf`, `dplyr`, `SciViews`, `xfun`)  
**Encadrant** : [Nom de l’enseignant]  
**Données** : fournies (fichiers `.LAS`, MNT, grille d’échantillonnage shapefile)  

---

## Objectif général du module

Comprendre et mettre en œuvre une approche complète d’analyse LiDAR pour la **caractérisation et la classification de peuplements forestiers**, depuis l’exploration du nuage de points jusqu’à la production d’une carte de classes structurales.

---

## Organisation des séances

### 🟩 Séance 1 — Exploration et familiarisation avec les données LiDAR (3h)

**Objectif :** Comprendre la nature et la structure des données LiDAR aériennes, visualiser et manipuler un nuage de points, et savoir les importer dans R.

#### 1. Introduction (20 min)
- Rappel du principe du LiDAR aéroporté : hauteur, intensité, retour laser, sol / végétation.  
- Présentation des objectifs du module : de la donnée brute à la classification structurale.  
- Présentation rapide du papier de **Fahey et al. (2022)** pour introduire l’idée de typologie structurale basée sur des variables intégratives.

#### 2. Visualisation dans CloudCompare (1h15)
- **Ouverture d’une tuile LiDAR (.LAS)** : repérage des canaux (X, Y, Z, Intensity, ReturnNumber, Classification).  
- **Colorisation du nuage** : par altitude, intensité et nombre de retours.  
- **Affichage de sections et coupes verticales** pour comprendre la stratification de la canopée.  
- **Exercice pratique : suppression manuelle de points parasites** (ex. points isolés, erreurs de sol).  
  - Utilisation de l’outil “segment” et export du nuage nettoyé au format `.las`.

#### 3. Importation dans R (1h)
```r
library(lidR)
library(sf)
library(terra)
library(dplyr)

ctg <- readLAScatalog("data/LAS/")
mnt <- rast("data/mnt_coteaux_hiatus.tif")
grid <- st_read("data/centre_grille_foret_hiatus.shp")

las <- readLAS("data/example_tile.las")
head(las@data)
summary(las@data$Z)

las_canopy <- filter_poi(las, Z > 1 & ReturnNumber == 1)
```

**Production attendue :**
- Une tuile LiDAR nettoyée et sauvegardée.  
- Un petit script R d’importation, filtrage et résumé des données.  

---

### 🟧 Séance 2 — Extraction des métriques structurales à partir du nuage de points (3h)

**Objectif :** Extraire des variables décrivant la structure de la canopée à partir de données LiDAR normalisées, sur la base du script fourni.

#### 1. Introduction (20 min)
- Discussion sur la typologie structurale de **Fahey et al. (2022)** :  
  - Variables de hauteur (moyenne, max, écart-type).  
  - Variables de rugosité et d’hétérogénéité (Rumple index, Canopy Cover).  
  - Variables liées à la distribution verticale (LAD, PAD, VCI).

#### 2. Mise en place du script (2h)
- Explication et exécution du script fourni pas à pas (voir fichier `extraction_metrics.R`).
- Calcul des métriques par maille : hauteur, rugosité, couverture, densité de scan, etc.
- Vérification de la sortie `results_canopy_metrics.csv` :
```r
data <- read.csv("results_canopy_metrics.csv")
summary(data)
```
- Discussion sur le sens écologique de chaque variable.

#### 3. Discussion finale (40 min)
- Interprétation des variables.  
- Lien avec les typologies structurales de Fahey (axes continus de hauteur, compacité, hétérogénéité).

**Production attendue :**
- Un script R fonctionnel d’extraction des métriques.  
- Un tableau CSV complet de variables structurales par maille.  

---

### 🟦 Séance 3 — Classification et spatialisation des peuplements (3h)

**Objectif :** Réaliser une classification des peuplements selon leurs caractéristiques structurales et représenter les résultats spatialement.

#### 1. Préparation des données (30 min)
```r
data <- read.csv("results_canopy_metrics.csv")
grid <- st_read("data/centre_grille_foret_hiatus.shp")
grid_data <- grid %>% left_join(data, by = "Id")
```

#### 2. Classification (1h30)
```r
library(FactoMineR)
library(factoextra)
res.pca <- PCA(data[, -1], scale.unit = TRUE)
res.hcpc <- HCPC(res.pca, nb.clust = -1)
fviz_pca_biplot(res.pca)
```
- Interprétation des classes : moyennes de variables, signification structurale.  
- Lien avec les types de canopées selon Fahey.

#### 3. Spatialisation et validation (1h)
```r
grid_data$classe <- res.hcpc$data.clust$clust
plot(grid_data["classe"])
```
- Distribution spatiale des classes et cohérence structurale.  
- Comparaison avec des cartes externes (type forestier, âge du peuplement).  
- Discussion sur les limites et perspectives.

**Production attendue :**
- Carte finale de classification structurale.  
- Interprétation qualitative des types obtenus.  

---

## Évaluation (optionnelle)

Rédaction d’un court rapport (2–3 pages) :
- Présentation des principales métriques calculées.  
- Résumé de la classification et interprétation écologique.  
- Discussion critique (limites, pistes d’amélioration).

---

## Références

- **Fahey, R. T. et al. (2022)**. *Defining a spectrum of integrative trait-based vegetation canopy structural types.*  
- **Roussel, J.-R. et al. (2020)**. *lidR: An R package for analysis of Airborne Laser Scanning (ALS) data.* *Remote Sensing of Environment, 251, 112061.*
