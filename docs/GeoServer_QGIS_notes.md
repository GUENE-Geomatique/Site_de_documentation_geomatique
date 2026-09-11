# GeoServer & QGIS — Notes de formation

> Document de synthèse sur l'utilisation de GeoServer, la publication de couches SIG et la connexion à QGIS.

---

## 1. Introduction

Les **serveurs cartographiques** (ou serveurs web carto) permettent de mettre à disposition, via Internet, des données issues d'un Système d'Information Géographique (SIG). Ils servent d'intermédiaire entre les données stockées (fichiers, bases de données) et les clients qui les consomment (logiciels SIG comme QGIS, interfaces web cartographiques, etc.), sous forme de **flux** (WMS, WFS...).

<img width="606" height="573" alt="Image2" src="https://github.com/user-attachments/assets/dfbe40c7-ab95-4a3a-9051-da21137a3785" />

### Les principaux serveurs cartographiques

Ce sont des serveurs connectés en **permanence** aux données du SIG, qui fournissent ces flux de données.

| Serveur | Type | Remarque |
|---|---|---|
| **MapServer** | Open source | Partie intégrante du serveur web (voir schéma ci-dessous) |
| **GeoServer** | Open source | Servlet Java indépendante, exécutée sur le serveur |
| **ArcGIS Server** | Propriétaire (Esri) | — |
| **QGIS Server** | Open source | Plus récent, se rapproche du fonctionnement de MapServer |

Les deux principaux serveurs open source sont **MapServer** et **GeoServer**. **QGIS Server**, apparu plus récemment, se rapproche du fonctionnement de MapServer et gagne en popularité.

### Différence de conception

- **MapServer** : fonctionne comme une partie intégrante du serveur web (client web/intranet → serveur web → MapServer → WMS/WFS).
- **GeoServer** : fonctionne comme une **servlet Java indépendante**, hébergée dans une JVM via un conteneur comme **Tomcat**, séparée du serveur web (client web/intranet → serveur web → Tomcat/JVM → GeoServer → WMS/WFS).

### Le mode transactionnel

Le mode transactionnel permet à quiconque dispose d'une connexion Internet et des droits appropriés de **mettre à jour des données partagées à distance**, y compris en situation de mobilité.

> ⚠️ **GeoServer permet de faire du transactionnel, contrairement à MapServer.**

### Serveur de tuiles et cache

Les images sont découpées en **dalles (tuiles)** de plus en plus petites à mesure que la résolution augmente :
- grande tuile → faible résolution
- petite tuile → forte résolution

---

## 2. Accès et identifiants

### 2.1 Serveur GeoServer local (2026)

- **Connexion anonyme** (droit visualisation uniquement) :
  `http://192.168.10.52:8080/geoserver/`

- **Compte admin/lecteur personnel** :
  - `user : lag`
  - `mdp : Sou..#`

- **Compte alternatif** :
  - `user : cpgeom`
  - `mdp : cp...m2026`

> ℹ️ Ces identifiants peuvent être modifiés par l'administrateur.

### 2.2 QGIS Server (obsolète)

```
http://qgis.demo/cgi-bin/qgis_mapserv.fcgi?SERVICE=WMS&VERSION=1.3.0&REQUEST=GetCapabilities
```

### 2.3 Serveur de formation en ligne

- URL : `http://formationgeomatique.fr:8094/geoserver/web/?0`
- Documentation associée : [supports.idgeo.fr – module GeoServer](https://supports.idgeo.fr/malt/geoserver_2025/co/GEOSERVER_module_16.html)

| Formation | Login | Mot de passe |
|---|---|---|
| **MALT 2024-2025** | `stagiaire` | `stagiaire` |
| **CPGEOM 2025-2027** | `cpgeom` | `cpgeom2026` |
| **CPGEOM — accès admin** | `admin` | `geoserver` |

### 2.4 Installation

Téléchargement de GeoServer pour Mac : [docs.geoserver.org – installation OSX](https://docs.geoserver.org/2.19.x/en/user/installation/osx_binary.html)

---

## 3. Organisation des données dans GeoServer

GeoServer structure les données selon une hiérarchie à trois niveaux :

```
Espace de travail (workspace)
   └── Entrepôt (data store)  →  lié à un type de données (shp, gpkg, PostGIS…)
          └── Couche (layer)  →  publiée avec un ou plusieurs styles
```

**Exemple illustré :**

```
Espace de travail
 ├── Entrepôt A
 │     ├── Couche 1 → Style 1, Style 2
 │     └── Couche 2 → Style 6, Style 5, Style 4
 ├── Entrepôt B
 │     └── Couche 3 → Style 7, Style 8
 └── Entrepôt C
       └── Couche 4 → Style 1, Style 2, Style 3
```

> ⚠️ **Point important : GeoServer ne crée pas de données.** Il permet uniquement de **prendre des données existantes** (shapefile, GeoPackage, base PostGIS…) et de les **publier** sous forme de flux **WMS**, **WFS**, etc.

---

## 4. Publier une couche — étape par étape

### Étape 1 — Créer un espace de travail

Depuis la barre latérale de GeoServer, ajouter un **espace de travail** (workspace) qui regroupera les données d'un projet.

### Étape 2 — Créer un entrepôt (data store)

1. Ajouter un **entrepôt** et choisir le type de source de données :
   - **Shapefile** (répertoire de fichiers `.shp`)
   - **GeoPackage** (`.gpkg`)
   - **PostGIS** (pour une base de données / un schéma)
2. Renseigner les paramètres de connexion :
   - **Espace de travail** associé
   - **Nom de la source de données**
   - **Répertoire des fichiers** (chemin vers les shapefiles ou le `.gpkg`)
   - **Jeu de caractères** (ex. `UTF-8`)
3. **Sauvegarder**.

<img width="606" height="301" alt="Image3" src="https://github.com/user-attachments/assets/45487739-989b-4d86-9f2c-29c6272c9ac1" />

### Étape 3 — Publier la couche

1. Depuis l'entrepôt créé, cliquer sur **Publier** en face de la ressource souhaitée.
2. Sur la page **Nouvelle couche**, régler tous les paramètres nécessaires (nom, système de coordonnées, emprise, etc.).
3. *(Optionnel)* Avant de sauvegarder, il est possible de **retirer ou rajouter des attributs** dans le type d'objet (feature type).
4. **Sauvegarder** pour publier la couche.

<img width="606" height="289" alt="Image4" src="https://github.com/user-attachments/assets/dbc598b9-d1f8-4b58-9085-eede0862d6de" />

### Étape 4 — Visualiser la couche publiée

1. Aller dans **Prévisualisation de la couche** (menu en haut à gauche).
2. Rechercher la couche publiée dans la liste.
3. Cliquer sur **OpenLayers** pour l'afficher dans le navigateur.

<img width="606" height="297" alt="Image5" src="https://github.com/user-attachments/assets/02be3fd4-4788-47a1-8a3f-c5d44d3841c8" />

### Étape 5 — Récupérer le flux pour QGIS

<img width="606" height="313" alt="Image6" src="https://github.com/user-attachments/assets/235b2c17-1654-451c-8d43-a2ce5b88bdde" />

Sur la page de prévisualisation OpenLayers, copier l'**URL** affichée jusqu'à `…wms?` :

- Utiliser cette URL telle quelle dans QGIS pour un flux **WMS** (image, raster).
- Remplacer `wms` par `wfs` dans l'URL pour obtenir les données en **vectoriel** (WFS).

Dans QGIS : *Couche → Ajouter une couche → Ajouter une couche WMS/WFS* → coller l'URL comme nouvelle connexion.

<img width="606" height="328" alt="Image7" src="https://github.com/user-attachments/assets/2f2d51d6-4c29-4842-8a53-afc53071f765" />

---

## 5. Publier un GeoPackage

Le principe est identique à un shapefile :

1. Créer un **nouvel entrepôt** de type **GeoPackage** (au lieu de Shapefile).
2. Dans les paramètres, sélectionner dans le répertoire le fichier `.gpkg` contenant les données.
3. **Sauvegarder**.
4. Publier la couche comme décrit à l'étape 3 ci-dessus.

---

## 6. Gérer la symbologie (styles SLD)

GeoServer permet de créer des **styles symbologiques** (format SLD) et de les associer aux couches publiées.

### Créer un style

Menu **Données → Styles → Ajouter un nouveau style**, puis définir la symbologie souhaitée.

<img width="602" height="301" alt="Image8" src="https://github.com/user-attachments/assets/25cf5829-6955-47c2-8d51-128af85c5c33" />

### Associer un style à une couche

1. Aller dans la couche concernée (**Couches → [nom de la couche] → Publication**).
2. Dans la section **Cartography Settings** :
   - Choisir le **style par défaut**
   - Ajouter des **styles additionnels** si besoin (liste "Styles disponibles" → "Styles sélectionnés")
3. **Sauvegarder**.

<img width="602" height="291" alt="Image9" src="https://github.com/user-attachments/assets/1cd09bd5-fe05-405c-89b3-9873465ef8d2" />

### Vérifier le résultat

Retourner dans **Prévisualisation de la couche** pour voir le rendu avec la nouvelle symbologie appliquée.

<img width="602" height="308" alt="Image10" src="https://github.com/user-attachments/assets/10783304-d1dd-400d-941a-53869863c0b2" />

---

## 7. Workflow complet PostgreSQL/PostGIS → QGIS → GeoServer

```
PostgreSQL/PostGIS         QGIS                      GeoServer
──────────────────         ────                      ─────────
Créer la BD                Chercher les données
dans Alwaysdata      →     Afficher dans QGIS
                            Connecter QGIS
Importer les données  →    à la BD Alwaysdata
dans des schémas
en 2154                    Mettre en forme
                            les données dans QGIS
                            Créer les fichiers SLD  →  Créer un entrepôt
                                                        Publier les couches
                            Tester le résultat      ←  Créer et affecter
                                                        les styles aux couches
```

**Résumé des étapes :**

1. Créer la base de données PostgreSQL/PostGIS (ex. sur Alwaysdata).
2. Importer les données dans des schémas, projetées en **EPSG:2154**.
3. Dans QGIS : rechercher, afficher et connecter QGIS à la base de données.
4. Mettre en forme les données et créer les fichiers de style **SLD**.
5. Dans GeoServer : créer un entrepôt, publier les couches, créer et affecter les styles.
6. Tester le résultat final (prévisualisation / QGIS).

<img width="606" height="264" alt="Image11" src="https://github.com/user-attachments/assets/2b058fc4-c209-4200-9ea1-b75f6194beb9" />

---

## 8. Sécurité — Gérer les rôles et utilisateurs

GeoServer dispose d'un module de gestion des accès : **Sécurité → Utilisateurs, groupes et rôles**.

- **Services / Utilisateurs-Groupes / Rôles** : trois onglets pour gérer respectivement les services d'authentification, les comptes utilisateurs, et les rôles associés.
- Pour chaque utilisateur, on peut voir s'il est **activé** et s'il **possède des attributs** spécifiques.
- Actions possibles :
  - **Ajouter un nouvel utilisateur**
  - **Supprimer la sélection**
  - **Supprimer la sélection et les associations de rôles**

> 🔐 Cette section permet de contrôler qui peut **visualiser**, **modifier** ou **administrer** les espaces de travail, entrepôts et couches (droits granulaires par rôle).

<img width="602" height="292" alt="Image12" src="https://github.com/user-attachments/assets/1a3b07b7-639e-4276-a9ae-c66ae8b7f236" />

---

## 9. Ressources utiles

- [Documentation GeoServer officielle — Installation macOS](https://docs.geoserver.org/2.19.x/en/user/installation/osx_binary.html)
- [Support IDGEO — Module GeoServer 16](https://supports.idgeo.fr/malt/geoserver_2025/co/GEOSERVER_module_16.html)
- Serveur de formation : `http://formationgeomatique.fr:8094/geoserver/web/?0`

---

*Notes mises à jour le 09/09/2026.*
