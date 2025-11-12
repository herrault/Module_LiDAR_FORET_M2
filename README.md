# Analyse de structure de canopée et classification de peuplements à partir de données LiDAR

**Niveau** : Master 2  
**Durée** : 3 séances de 3h  
**Pré-requis** : notions de base en R, connaissance générale des écosystèmes forestiers  
**Logiciels utilisés** : CloudCompare, R (packages `lidR`, `terra`, `sf`, `dplyr`, `SciViews`, `xfun`)  
**Encadrant** : Herrault Pierre-Alexis
**Données** : fournies (fichiers `.LAS`, MNT, grille d’échantillonnage shapefile)  

---

## Objectif général du module

Ce module appartient à l'UE Traitement de Nuages de Points et dure 12h. Les objectifs seront de comprendre et mettre en œuvre une approche complète d’analyse LiDAR pour la **caractérisation et la classification de peuplements forestiers**, depuis l’exploration du nuage de points jusqu’à la production d’une carte de classes structurales.

---

## Organisation des séances

### 🟩 Séance 1 — Exploration et familiarisation avec les données LiDAR (3h)

**Objectif :** Comprendre la nature et la structure des données LiDAR aériennes, visualiser et manipuler un nuage de points, et savoir les importer dans R.

#### 1. Introduction (20 min)

Un point de cours rapide vous sera proposé afin de reprendre les points suivants : 

- Rappel du principe du LiDAR aéroporté : hauteur, intensité, retour laser, sol / végétation.  
- Présentation des objectifs du module : de la donnée brute à la classification structurale de la végétation 
- Présentation rapide du papier de **Fahey et al. (2022)** pour introduire l’idée de typologie structurale basée sur des variables LiDAR. 

#### 2. Visualisation dans CloudCompare (1h15)

Votre premier objectif consiste à prendre en main une tuile .las et à l'importer dans CloudCompare. Prenez le temps d'explorer la donnée, ses spécificités, son hétérogénéité. 

- **Ouverture d’une tuile LiDAR (.LAS)** : repérage des canaux (X, Y, Z, Intensity, ReturnNumber, Classification).  
- **Colorisation du nuage** : par altitude, intensité et nombre de retours. NB : n'hésitez pas à visualiser la distribution des variables et à modifier les bornes min-max pour mieux saisir l'amplitude des valeurs. 
- **Affichage de sections et coupes verticales** pour comprendre la stratification de la canopée. Par exemple, visualisez les points dans les deux mètres supérieurs. Ceux situés dans le premier mètre en partant du sol. Pour ces derniersq, qu'observez vous ?   
- **Exercice pratique : suppression manuelle de points parasites** (ex. points isolés, erreurs de sol).  
  - Pour cela, utilisez l’outil “segment” et export du nuage nettoyé au format `.las`.

#### 3. Importation dans R 

Dans un second temps, nous allons passer sur R pour visualiser cette même donnée. L'objectif est à terme d'utiliser cet environnement pour faciliter l'extraction de variables à partir de packages déjà développés. 

```r
library(lidR)
library(sf)
library(terra)
library(dplyr)

ctg <- readLAScatalog("data/LAS/")    ## votre chemin vers les tuiles LAS
mnt <- rast("data/mnt_coteaux_hiatus.tif")  ## votre chemin vers le MNT
grid <- st_read("data/plot_fabas.shp")   ## votre chemin vers la grille de plots

las <- readLAS("data/example_tile.las")    ## lire votre tuile 
head(las@data)                             ## visualiser la structure de la table attributaire
summary(las@data$Z)                        ## checker les statistiques de hauteur

las_canopy <- filter_poi(las, Z > 1 & ReturnNumber == 1)       ## filtrer les points constituant la canopée

plot(las, color = "Intensity") ## Reproduisez la même ligne en appliquant une palette viridis à la hauteur des points
```
---

### 🟧 Séance 2 — Extraction des métriques structurales à partir du nuage de points (3h)

**Objectif :** Extraire des variables décrivant la structure de la canopée à partir de données LiDAR normalisées, sur la base du script fourni.

#### 1. Introduction 

- Nous allons premièrement revenir sur la proposition de classification structurale proposée par **Fahey et al. (2022)**. Quels sont les trois dimensions proposées ? En quoi reflètent-elles la structure globale d'un environnement forestier ?
- Dans le script fourni, repérez les 3 groupes de variables suivants. 
  
  - Variables de hauteur 
  - Variables liées à l'hétérogénéité horizontale 
  - Variables liées à la distribution verticale 
 
- Pour chacune des variables utilisées pour chaque groupe, dites en quoi elles peuvent être complémentaires ? Quel intérêt peut-il y avoir à calculer ces différentes variables ?

#### 2. Mise en place du script

Pour cette section, l'objectif va être de faire fonctionner le script pour calculer les variables sur la totalité de vos plots (plot_fabas)

- Explication et exécution du script fourni pas à pas (voir fichier `extraction_metrics.R`).
- Calcul des métriques par maille : hauteur, rugosité, couverture, densité de scan, etc.
- Vérification de la sortie `results_canopy_metrics.csv` :
```r
data <- read.csv("results_canopy_metrics.csv")
summary(data)
```
- Tracez la distibution de chaque variable à l'aide de boxplots ou d'histogramme. Qu'observez vous sur la distribution statistique des variables calculées ?

#### 3. Discussion finale 

- Interprétation des variables.  
- Lien avec les typologies structurales de Fahey (axes continus de hauteur, hétérogénéité verticale et horizontale).
  
---

### 🟦 Séance 3 — Classification et spatialisation des peuplements (3h)

**Objectif :** Réaliser une classification des peuplements selon leurs caractéristiques structurales et représenter les résultats spatialement.

#### 1. Préparation des données (30 min)
```r
data <- read.csv("results_canopy_metrics.csv")
grid <- st_read("data/plot_fabas.shp")
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
- Observez la distribution spatiale des classes. Appliquez une orthophoto IRC (=> grâce au flux de l'IGN). Commentez la cohérence entre la classification et la texture de l'orthophotographie. 
- Comparez ensuite votre avec des cartes externes issus des Plans de Gestion Simple (cf Documentation). 
- Notez vos résultats

**Production attendue :**
- Interprétation des types obtenus du point de vue de la structure et de la gestion 

---

## Références

- **Fahey, R. T. et al. (2019)**. *Defining a spectrum of integrative trait-based vegetation canopy structural types.*  
- **Roussel, J.-R. et al. (2020)**. *lidR: An R package for analysis of Airborne Laser Scanning (ALS) data.* *Remote Sensing of Environment, 251, 112061.*
