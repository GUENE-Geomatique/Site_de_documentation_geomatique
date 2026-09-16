# Tutoriel : Éditer des objets vecteurs avec OpenLayers (Draw, Modify, Translate, suppression) et les enregistrer dans PostGIS

> Ce tutoriel fait suite à **Page web avec OpenLayer**. Il couvre les interactions d'édition d'OpenLayers, qui permettent à l'utilisateur de dessiner, modifier, déplacer ou supprimer des objets directement sur la carte — jusqu'à l'enregistrement d'un point dessiné dans une base PostGIS via PHP.

## Sommaire

1. [Introduction](#1-introduction)
2. [Dessiner un objet vecteur : `ol.interaction.Draw`](#2-dessiner-un-objet-vecteur--olinteractiondraw)
3. [Modifier un objet vecteur : `ol.interaction.Modify`](#3-modifier-un-objet-vecteur--olinteractionmodify)
4. [Déplacer un objet vecteur : `ol.interaction.Translate`](#4-déplacer-un-objet-vecteur--olinteractiontranslate)
5. [Supprimer un objet vecteur](#5-supprimer-un-objet-vecteur)
6. [Exemple combiné : dessiner, sélectionner, supprimer une ligne](#6-exemple-combiné--dessiner-sélectionner-supprimer-une-ligne)
7. [Enregistrer un point dessiné dans PostGIS](#7-enregistrer-un-point-dessiné-dans-postgis)
8. [⚠️ Sécuriser l'insertion en base (injection SQL)](#8-️-sécuriser-linsertion-en-base-injection-sql)
9. [Mise en pratique : projet `dae_chalons`](#9-mise-en-pratique--projet-dae_chalons)
10. [Aller plus loin](#10-aller-plus-loin)

---

## 1. Introduction

Jusqu'ici, les cartes réalisées étaient **en lecture seule** : on affichait des données existantes, on cliquait dessus pour les consulter. OpenLayers propose aussi des **interactions d'édition**, qui permettent à l'utilisateur d'agir sur les géométries affichées :

| Interaction | Rôle |
|---|---|
| `ol.interaction.Draw` | dessiner un nouvel objet (point, ligne, polygone, cercle...) |
| `ol.interaction.Modify` | modifier la forme d'un objet déjà sélectionné |
| `ol.interaction.Translate` | déplacer un objet sélectionné |
| suppression | retirer un objet de sa source, après sélection |

Ces interactions travaillent sur une **source vectorielle locale** (`ol.source.Vector`), en mémoire dans le navigateur. Pour que les modifications soient conservées, il faut ensuite les envoyer vers le serveur (typiquement via un script PHP qui les insère en base PostGIS) — ce dernier point est traité en section 7.

---

## 2. Dessiner un objet vecteur : `ol.interaction.Draw`

`ol.interaction.Draw` permet de cliquer sur la carte pour tracer un point, une ligne, un polygone ou un cercle. Le double-clic termine le tracé (pour les lignes/polygones).

### 2.1 Options principales

| Option | Rôle | Défaut |
|---|---|---|
| `source` | source vectorielle où seront stockés les objets dessinés | — |
| `type` | `'Point'`, `'LineString'`, `'Polygon'`, `'MultiPoint'`, `'MultiLineString'`, `'MultiPolygon'`, `'Circle'` | requis |
| `style` | style des objets en cours de dessin | style par défaut |
| `condition` | fonction qui détermine si le dessin est actif | toujours actif |
| `freehand` | dessin à main levée | `false` |
| `clickTolerance` | tolérance de clic en pixels | 6 |
| `snapTolerance` | tolérance d'accrochage | 12 |
| `minPoints` / `maxPoints` | nombre de points min/max | 3 pour polygone, 2 pour ligne |
| `finishCondition` | fonction qui décide quand arrêter le dessin | — |
| `geometryFunction` | fonction personnalisée pour construire la géométrie | — |

### 2.2 Événements

- `drawstart` : déclenché au démarrage d'un tracé.
- `drawend` : déclenché à la fin d'un tracé — c'est l'événement à utiliser pour **récupérer la géométrie dessinée** et, par exemple, l'envoyer vers le serveur.

### 2.3 Activer/désactiver le dessin avec une case à cocher

Une pratique courante : lier l'interaction `Draw` à une case à cocher, pour que l'utilisateur active le mode dessin uniquement quand il le souhaite (sinon, chaque clic sur la carte déclencherait un tracé, ce qui gênerait la navigation normale).

```javascript
var source_ligne = new ol.source.Vector();

var ligne = new ol.layer.Vector({
    source: source_ligne
});
map.addLayer(ligne);

var draw = new ol.interaction.Draw({
    source: source_ligne,
    type: 'LineString',
    condition: function(evt){
        return document.getElementById('dessin').checked;
    }
});
map.addInteraction(draw);
```

Dans le cours original, cette condition utilise jQuery (`$('#dessin').is(':checked')`) — l'équivalent en JavaScript pur est `document.getElementById('dessin').checked`, utilisé ci-dessus.

**Alternative plus explicite** : ajouter/retirer complètement l'interaction plutôt que de la conditionner en permanence :

```javascript
document.getElementById('dessin').addEventListener('change', function(){
    if (this.checked){
        map.addInteraction(draw);
    } else {
        map.removeInteraction(draw);
    }
});
```

> Les deux approches sont valables. La première (`condition`) garde l'interaction toujours "présente" mais l'active/désactive en interne. La seconde (`addInteraction`/`removeInteraction`) ajoute/retire complètement l'interaction de la carte — plus explicite, recommandée quand plusieurs interactions doivent cohabiter sans interférer entre elles (voir section 6).

---

## 3. Modifier un objet vecteur : `ol.interaction.Modify`

`ol.interaction.Modify` permet de déplacer les sommets d'un objet déjà présent sur la carte. **L'objet doit d'abord être sélectionné** (via `ol.interaction.Select`) pour être modifiable.

### 3.1 Options principales

| Option | Rôle | Défaut |
|---|---|---|
| `features` | **requis** — collection des objets modifiables (typiquement `select.getFeatures()`) | — |
| `condition` | condition pour déclencher la modification | clic principal (glisser) |
| `deleteCondition` | condition pour supprimer un point du tracé en cours de modification | clic simple sans touche modificatrice |
| `pixelTolerance` | distance d'accrochage en pixels | 10 |
| `style` | style pendant la modification | style par défaut |

### 3.2 Événements

- `modifystart`
- `modifyend`

### 3.3 Exemple

```javascript
var donnees_eclairage = new ol.source.Vector({
    url: '../data/geojson/ep_depart_ht.geojson',
    format: new ol.format.GeoJSON()
});

var couche_eclairage = new ol.layer.Vector({
    source: donnees_eclairage
});
map.addLayer(couche_eclairage);

// 1. Sélectionner un objet
var select = new ol.interaction.Select();

// 2. Modifier l'objet sélectionné
var modifier = new ol.interaction.Modify({
    features: select.getFeatures()
});

// Activation via une case à cocher
document.getElementById('modif').addEventListener('change', function(){
    if (this.checked){
        map.addInteraction(select);
        map.addInteraction(modifier);
    } else {
        map.removeInteraction(select);
        map.removeInteraction(modifier);
    }
});
```

> **Principe clé** : `Modify` ne fonctionne que sur les entités contenues dans la collection passée à `features`. En pratique, on lie presque toujours `Modify` à une interaction `Select`, en lui passant `select.getFeatures()` — ainsi, seul l'objet actuellement sélectionné devient modifiable.

---

## 4. Déplacer un objet vecteur : `ol.interaction.Translate`

`ol.interaction.Translate` permet de déplacer un objet entier (translation), plutôt que de modifier sa forme sommet par sommet.

### 4.1 Options principales

| Option | Rôle | Défaut |
|---|---|---|
| `features` | objets déplaçables (sinon, tous les objets visibles le sont) | tous |
| `layers` | couches dont les objets sont déplaçables | toutes les couches visibles |
| `hitTolerance` | tolérance d'accrochage en pixels | 0 |

### 4.2 Événements

- `translatestart`
- `translating` (déclenché en continu pendant le déplacement)
- `translateend`

### 4.3 Exemple

```javascript
var select = new ol.interaction.Select();
map.addInteraction(select);

var deplace = new ol.interaction.Translate({
    features: select.getFeatures()
});
map.addInteraction(deplace);

deplace.on('translating', function(evt){
    var coord = evt.coordinate;
    document.getElementById('info').innerHTML =
        'Déplacement : ' + ol.coordinate.format(coord, '{x},{y}', 2);
});
```

`ol.coordinate.format(coord, template, decimales)` formate des coordonnées pour l'affichage, en remplaçant `{x}` et `{y}` dans le gabarit fourni.

---

## 5. Supprimer un objet vecteur

Il n'existe pas d'interaction `Delete` dédiée dans OpenLayers. La suppression se fait en deux temps :

1. **Sélectionner** l'objet (`ol.interaction.Select`).
2. **Le retirer de sa source** avec une méthode de `ol.source.Vector` :
   - `source.removeFeature(feature)` : supprime un objet précis.
   - `source.clear(opt_fast)` : supprime tous les objets de la source.

### 5.1 Exemple

```javascript
var selectInteraction = new ol.interaction.Select({
    condition: ol.events.condition.singleClick,
    layers: [ligne],
    hitTolerance: 5
});

var deleteFeature = function(event){
    var feature = event.selected[0]; // le premier objet nouvellement sélectionné
    source_ligne.removeFeature(feature);
    selectInteraction.getFeatures().clear(); // vide la sélection après suppression
};

selectInteraction.on('select', deleteFeature);
map.addInteraction(selectInteraction);
```

> **Astuce** : en écoutant l'événement `select` d'une interaction `Select` dédiée au mode "suppression", chaque objet cliqué est immédiatement supprimé — l'utilisateur n'a qu'à cliquer sur ce qu'il veut effacer.

---

## 6. Exemple combiné : dessiner, sélectionner, supprimer une ligne

Cet exemple illustre comment faire cohabiter plusieurs modes d'édition **exclusifs entre eux**, pilotés par des boutons radio (un seul actif à la fois) : dessiner, supprimer, ou sortir du mode édition.

```javascript
var source_ligne = new ol.source.Vector();

var ligne = new ol.layer.Vector({
    source: source_ligne,
    style: new ol.style.Style({
        stroke: new ol.style.Stroke({width: 2})
    })
});
map.addLayer(ligne);

// --- Interaction de dessin ---
var draw = new ol.interaction.Draw({
    source: source_ligne,
    type: 'LineString',
    style: new ol.style.Style({
        stroke: new ol.style.Stroke({width: 3})
    })
});

// --- Interaction de sélection (pour la suppression) ---
var selectInteraction = new ol.interaction.Select({
    condition: ol.events.condition.singleClick,
    layers: [ligne],
    hitTolerance: 5
});

// --- Bouton "dessiner" ---
document.getElementById('dessin').addEventListener('change', function(){
    if (this.checked){
        map.addInteraction(draw);
    } else {
        map.removeInteraction(draw);
    }
});

// --- Fonction de suppression ---
var deleteFeature = function(event){
    var feature = event.selected[0];
    source_ligne.removeFeature(feature);
    selectInteraction.getFeatures().clear();
};

// --- Bouton "supprimer" ---
document.getElementById('suppr').addEventListener('change', function(){
    if (this.checked){
        map.addInteraction(selectInteraction);
        selectInteraction.on('select', deleteFeature);
    } else {
        map.removeInteraction(selectInteraction);
    }
});

// --- Bouton "sortir du mode édition" ---
document.getElementById('sortir').addEventListener('change', function(){
    if (this.checked){
        map.removeInteraction(draw);
        map.removeInteraction(selectInteraction);
    }
});
```

```html
<div id="panneau_edition">
    <label><input type="radio" name="control_dessin" id="dessin"/> Dessiner une ligne</label><br/>
    <label><input type="radio" name="control_dessin" id="suppr"/> Supprimer une ligne</label><br/>
    <label><input type="radio" name="control_dessin" id="sortir" checked/> Sortir du mode édition</label>
</div>
```

> Utiliser des **boutons radio** (`type="radio"`, même `name`) plutôt que des cases à cocher indépendantes garantit qu'un seul mode est actif à la fois — évite les interférences entre `draw` et `selectInteraction` qui, actives simultanément, pourraient se gêner mutuellement au clic.

---

## 7. Enregistrer un point dessiné dans PostGIS

Dessiner un objet dans OpenLayers ne le rend visible **que dans le navigateur de l'utilisateur**, et seulement le temps de la session — rien n'est conservé côté serveur tant qu'on ne l'enregistre pas explicitement en base.

### 7.1 Principe général

```
Utilisateur dessine un point
        ↓
OpenLayers capture la géométrie (drawend)
        ↓
JavaScript envoie les coordonnées au serveur (fetch / AJAX)
        ↓
PHP reçoit les coordonnées et exécute un INSERT dans PostGIS
        ↓
Réponse (succès/erreur) renvoyée au navigateur
        ↓
JavaScript met à jour l'affichage (recharge la couche, message de confirmation...)
```

### 7.2 Récupérer la géométrie dessinée

```javascript
draw.on('drawend', function(evt){
    var geom = evt.feature.getGeometry();
    var coord = geom.getCoordinates(); // pour un point : [x, y]
    console.log(coord);
});
```

### 7.3 Envoyer la géométrie au serveur

**Avec `fetch` (JavaScript pur, sans dépendance)** :

```javascript
fetch('ajout_point.php?x=' + coord[0] + '&y=' + coord[1])
    .then(function(reponse){ return reponse.json(); })
    .then(function(resultat){
        if (resultat.success){
            console.log('Point enregistré');
        } else {
            console.error('Erreur :', resultat.error);
        }
    });
```

**Avec jQuery (`$.ajax`)**, comme dans l'exemple du cours :

```javascript
$.ajax({
    url: 'ajout_point.php?x=' + coord[0] + '&y=' + coord[1]
}).done(function(data){
    $('#zone_text').append(data);
});
```

Les deux approches sont équivalentes. `fetch` est disponible nativement dans tous les navigateurs modernes, sans nécessiter de charger jQuery.

---

## 8. ⚠️ Sécuriser l'insertion en base (injection SQL)

### 8.1 La version du cours — à ne PAS reproduire telle quelle

Le fichier PHP fourni dans le cours original construit la requête SQL **entièrement côté JavaScript**, puis l'envoie telle quelle au serveur, qui l'exécute sans aucune vérification :

```javascript
// ⚠️ Extrait du cours original — NE PAS REPRODUIRE
draw.on('drawend', function(evt){
    var req = 'INSERT INTO t_poi(geom) values ';
    var formatwkt = new ol.format.WKT();
    var objwkt = formatwkt.writeFeatures([new ol.Feature({geometry: evt.feature.getGeometry()})]);
    req += '(st_geomfromtext(\'' + objwkt + '\',3857));';

    $.ajax({ url: 'ajout_point.php?req=' + req });
});
```

```php
<?php
// ⚠️ Extrait du cours original — NE PAS REPRODUIRE
$connexion = pg_connect("host=localhost user=postgres password=postgres dbname=bd_test_postgis port=5434");
$req = $_GET['req'];
echo $req;
$result_insert_reg = pg_query($connexion, $req);
?>
```

**Pourquoi c'est dangereux** : le paramètre `req` reçu par le PHP est exécuté **tel quel** comme requête SQL, sans aucun filtrage. N'importe qui connaissant l'URL du script pourrait y glisser n'importe quelle commande SQL — y compris une suppression de table (`DROP TABLE ...`), une modification de données existantes, ou une extraction d'informations sensibles d'autres tables de la base. C'est une faille d'**injection SQL** classique et sérieuse : un formulaire censé n'ajouter qu'un point donne en réalité un accès en écriture libre à toute la base.

### 8.2 La version sécurisée utilisée dans ce tutoriel

**Principe** : le navigateur n'envoie **jamais** de SQL — seulement des données brutes (ici, des coordonnées numériques `x` et `y`). C'est le PHP, côté serveur, qui construit la requête, avec des **paramètres liés** (`pg_query_params`), qui empêchent toute injection quelle que soit la valeur envoyée.

```javascript
// Côté JavaScript : on envoie uniquement des coordonnées, jamais du SQL
draw.on('drawend', function(evt){
    var geom = evt.feature.getGeometry();
    var coord = geom.getCoordinates();

    fetch('ajout_point.php?x=' + coord[0] + '&y=' + coord[1])
        .then(function(reponse){ return reponse.json(); })
        .then(function(resultat){
            if (resultat.success){
                console.log('Point ajouté');
            } else {
                console.error('Erreur :', resultat.error);
            }
        });
});
```

```php
<?php
// ajout_point.php — version sécurisée
require_once('config.php'); // identifiants non versionnés (voir tutoriel principal, section 13.6)

header('Content-Type: application/json');

$conn = pg_connect("dbname='$db_name' user='$db_user' password='$db_pass' host='$db_host' port='$db_port'");
if (!$conn) {
    echo json_encode(['success' => false, 'error' => 'Connexion échouée : ' . pg_last_error()]);
    exit;
}

// On attend uniquement des nombres — jamais de SQL brut
$x = isset($_GET['x']) ? floatval($_GET['x']) : null;
$y = isset($_GET['y']) ? floatval($_GET['y']) : null;

if ($x === null || $y === null){
    echo json_encode(['success' => false, 'error' => 'Coordonnées manquantes']);
    exit;
}

// pg_query_params : les valeurs ($1, $2) sont liées séparément de la requête,
// PostgreSQL ne les interprète jamais comme du code SQL.
$sql = "INSERT INTO dae_chalons(geom) VALUES (ST_SetSRID(ST_MakePoint($1, $2), 3857))";
$result = pg_query_params($conn, $sql, array($x, $y));

if ($result){
    echo json_encode(['success' => true, 'x' => $x, 'y' => $y]);
} else {
    echo json_encode(['success' => false, 'error' => pg_last_error($conn)]);
}

pg_close($conn);
?>
```

### 8.3 Pourquoi cette version est sûre

- `floatval($_GET['x'])` force la valeur reçue à être interprétée comme un **nombre** — toute tentative d'injecter du texte ou du SQL est automatiquement convertie en `0` ou rejetée, jamais exécutée comme code.
- `pg_query_params($conn, $sql, array($x, $y))` sépare strictement la **structure** de la requête (le texte SQL, fixe) des **valeurs** (`$x`, `$y`), qui sont transmises à part au serveur PostgreSQL. Le moteur SQL ne peut donc jamais confondre une valeur avec une instruction.
- Le nom de la table (`dae_chalons`) est **écrit en dur** dans le script PHP, pas reçu depuis le client — impossible de le détourner pour cibler une autre table.

> **Règle générale à retenir** : dès qu'une requête SQL doit inclure une donnée fournie par l'utilisateur (formulaire, URL, clic sur la carte...), cette donnée doit être transmise comme **paramètre lié** (`pg_query_params` en PHP/PostgreSQL, l'équivalent existe dans tous les langages et SGBD), jamais concaténée directement dans le texte de la requête.

---

## 9. Mise en pratique : projet `dae_chalons`

Dans ce projet, la table PostGIS `dae_chalons` (type point) a été créée pour recevoir des points ajoutés interactivement depuis la carte.

### 9.1 Fichiers du projet

```
site_temporel/ol/
├── dessin_dae_chalons.html              # page de dessin (carte + outil de dessin)
├── ajout_point.php                      # insertion sécurisée d'un point
├── postgis_geojson_abdoulahat_alwaysdata.php   # lecture des points existants
└── config.php                           # identifiants base (non versionné)
```

Ce fichier a été volontairement créé **indépendant** des autres pages de carte du projet (pas de dépendance avec `carte_commune_cpgeom.html`), pour rester simple à tester et à faire évoluer séparément.

### 9.2 Fonctionnement de `dessin_dae_chalons.html`

1. **Au chargement** : affiche un fond OSM, charge les points déjà présents dans `dae_chalons` (via `postgis_geojson_....php?geotable=dae_chalons`), et zoome automatiquement sur leur étendue.
2. **Une case à cocher** active/désactive l'outil de dessin de point.
3. **Au clic sur la carte** (case cochée) : un point apparaît immédiatement dans une couche temporaire (retour visuel instantané), puis ses coordonnées sont envoyées à `ajout_point.php`.
4. **Si l'insertion réussit** : la couche des points existants est rechargée (`donnees_dae.refresh()`) pour afficher le point officiellement enregistré, et le point temporaire est retiré.
5. **Si erreur** : un message s'affiche, sans modifier la base.

### 9.3 Code complet — `dessin_dae_chalons.html`

```html
<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Ajout de points - DAE Châlons</title>
<script src="https://cdn.jsdelivr.net/npm/ol@v10.10.0/dist/ol.js"></script>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/ol@v10.10.0/ol.css">
<style>
html,body{ margin:0; padding:0; width:100%; height:100%; }
#carte{ width:100%; height:100%; background-color:#eceae5; }
#info{
    position:absolute; bottom:20px; left:20px; z-index:1000;
    background:white; padding:10px 14px; box-shadow:1px 1px 12px #555;
    border-radius:4px; font-family:Arial,sans-serif; font-size:14px; max-width:300px;
}
#panneau_dessin{
    position:absolute; top:20px; left:20px; z-index:1000;
    background:white; padding:10px 14px; box-shadow:1px 1px 12px #555;
    border-radius:4px; font-family:Arial,sans-serif; font-size:14px;
}
</style>
</head>
<body>
<div id="carte"></div>
<div id="info">Cochez la case pour ajouter un point</div>
<div id="panneau_dessin">
    <label><input type="checkbox" id="dessin_dae"/> Ajouter un point (dae_chalons)</label>
</div>
<script>
var couche_osm = new ol.layer.Tile({
    source: new ol.source.OSM(),
    title: 'Fond OSM'
});

var donnees_dae = new ol.source.Vector({
    url: 'postgis_geojson_abdoulahat_alwaysdata.php?geotable=dae_chalons',
    format: new ol.format.GeoJSON()
});

var style_point_dae = new ol.style.Style({
    image: new ol.style.Circle({
        radius: 7,
        fill: new ol.style.Fill({color: '#ff5500'}),
        stroke: new ol.style.Stroke({color: 'black', width: 1})
    })
});

var couche_dae = new ol.layer.Vector({
    source: donnees_dae,
    style: style_point_dae,
    title: 'Points DAE (existants)'
});

var source_point_dessin = new ol.source.Vector();
var couche_point_dessin = new ol.layer.Vector({
    source: source_point_dessin,
    style: style_point_dae,
    title: 'Points en cours de dessin'
});

var map = new ol.Map({
    target: 'carte',
    layers: [couche_osm, couche_dae, couche_point_dessin],
    view: new ol.View({
        center: [263652, 5919266],
        zoom: 12
    })
});

donnees_dae.on('featuresloadend', function(){
    var features = donnees_dae.getFeatures();
    console.log('Nombre de points dae_chalons :', features.length);
    if (features.length > 0){
        map.getView().fit(donnees_dae.getExtent(), {
            padding: [50, 50, 50, 50],
            maxZoom: 16
        });
    }
});

var infoDiv = document.getElementById('info');
var checkbox_dessin = document.getElementById('dessin_dae');

var draw_dae = new ol.interaction.Draw({
    source: source_point_dessin,
    type: 'Point',
    style: style_point_dae,
    condition: function(evt){
        return checkbox_dessin.checked;
    }
});
map.addInteraction(draw_dae);

draw_dae.on('drawend', function(evt){
    var geom = evt.feature.getGeometry();
    var coord = geom.getCoordinates();

    infoDiv.innerHTML = 'Enregistrement en cours...';

    fetch('ajout_point.php?x=' + coord[0] + '&y=' + coord[1])
        .then(function(reponse){ return reponse.json(); })
        .then(function(resultat){
            if (resultat.success){
                infoDiv.innerHTML = 'Point ajouté avec succès dans dae_chalons';
                donnees_dae.clear();
                donnees_dae.refresh();
                source_point_dessin.clear();
            } else {
                infoDiv.innerHTML = 'Erreur lors de l\'ajout : ' + resultat.error;
            }
        })
        .catch(function(){
            infoDiv.innerHTML = 'Erreur de communication avec le serveur';
        });
});
</script>
</body>
</html>
```

### 9.4 Code complet — `ajout_point.php`

```php
<?php
// ajout_point.php — insertion sécurisée d'un point dans dae_chalons
require_once('config.php');

header('Content-Type: application/json');

$conn = pg_connect("dbname='$db_name' user='$db_user' password='$db_pass' host='$db_host' port='$db_port'");
if (!$conn) {
    echo json_encode(['success' => false, 'error' => 'Connexion échouée : ' . pg_last_error()]);
    exit;
}

$x = isset($_GET['x']) ? floatval($_GET['x']) : null;
$y = isset($_GET['y']) ? floatval($_GET['y']) : null;

if ($x === null || $y === null){
    echo json_encode(['success' => false, 'error' => 'Coordonnées manquantes']);
    exit;
}

$sql = "INSERT INTO dae_chalons(geom) VALUES (ST_SetSRID(ST_MakePoint($1, $2), 3857))";
$result = pg_query_params($conn, $sql, array($x, $y));

if ($result){
    echo json_encode(['success' => true, 'x' => $x, 'y' => $y]);
} else {
    echo json_encode(['success' => false, 'error' => pg_last_error($conn)]);
}

pg_close($conn);
?>
```

<img width="1919" height="1029" alt="1" src="https://github.com/user-attachments/assets/e620351a-ef48-4992-b882-69d7e9e4f42c" />

<img width="1919" height="1029" alt="2" src="https://github.com/user-attachments/assets/8ef47366-3a3a-429d-a8a5-1c64be2478fb" />

---

## 10. Aller plus loin


### 10.1 Découvrir la vraie structure de la table `dae_chalons`

En inspectant la table en base, deux différences importantes apparaissent par rapport aux hypothèses de départ (section 9) :

| Hypothèse initiale | Réalité en base |
|---|---|
| clé primaire `id` | clé primaire **`gid`** (SERIAL) |
| colonne géométrie `geom` | colonne géométrie **`the_geom`** |
| pas d'attributs métier | de nombreux champs (`c_nom`, `c_adr_voie`, `c_com_nom`, `c_expt_rais`, `c_acc`, `c_disp_j`, `c_disp_h`...) — schéma proche du Registre National des DAE |

Avant d'écrire la moindre requête SQL sur une table existante, il faut donc toujours vérifier sa structure réelle (via un client PostGIS type pgAdmin/phpPgAdmin, ou `\d dae_chalons` en ligne de commande `psql`), plutôt que de supposer une convention de nommage.

Il faut aussi vérifier le SRID réellement stocké dans `the_geom` :

```sql
SELECT Find_SRID('public', 'dae_chalons', 'the_geom');
```

Dans ce projet, la carte et les scripts PHP utilisent **EPSG:3857** de bout en bout (cohérent avec le résultat obtenu).

---

### 10.2 Corriger le script de lecture GeoJSON

Le script `postgis_geojson_abdoulahat_alwaysdata.php` (basé sur le script fourni par le professeur) accepte bien un paramètre `geomfield` en entrée, mais contenait un bug : la clause `WHERE` était codée en dur sur `geom`, ignorant ce paramètre.

```php
// AVANT — bug : ignore le paramètre $geomfield
$sql .= " WHERE geom is not null";
```

```php
// APRÈS — corrigé
$sql .= " WHERE " . $geomfield . " is not null";
```

**Conséquence du bug** : sur une table où la colonne s'appelle `the_geom` (et non `geom`), la requête SQL échouait systématiquement (`An SQL error occured.`), donc **aucune feature n'était renvoyée** → la couche restait vide et la carte ne zoomait jamais sur l'étendue des points (puisque `featuresloadend` recevait 0 features).

Un second ajustement était nécessaire pour que l'identifiant de chaque feature GeoJSON corresponde à la vraie clé primaire, afin que `feature.getId()` fonctionne côté OpenLayers (indispensable pour cibler un point à modifier ou supprimer) :

```php
// AVANT
if ($key == "id") {
    $id .= ',"id":"' . escapeJsonString($val) . '"';
}
```

```php
// APRÈS
if ($key == "gid") {
    $id .= ',"id":"' . escapeJsonString($val) . '"';
}
```

**Appel corrigé, avec le bon nom de champ géométrie :**
```
postgis_geojson_abdoulahat_alwaysdata.php?geotable=dae_chalons&geomfield=the_geom
```

---

### 10.3 Ajouter un formulaire d'attributs avant l'enregistrement

Plutôt que d'enregistrer un point dès la fin du tracé (`drawend`), on intercale une petite fenêtre modale demandant de saisir quelques attributs utiles de la table : `c_nom`, `c_adr_voie`, `c_com_nom`. L'enregistrement en base n'a lieu qu'après validation du formulaire.

**Principe :**
1. `drawend` ne déclenche plus l'appel réseau directement — il stocke les coordonnées du point dans une variable temporaire (`coord_en_attente`) et affiche la modale.
2. Le bouton **Enregistrer** de la modale lit les champs saisis, construit l'URL avec `encodeURIComponent()` sur chaque valeur, puis appelle `ajout_point.php`.
3. Le bouton **Annuler** referme la modale et vide la couche de dessin temporaire, sans rien enregistrer.

```javascript
var overlay_modale = document.getElementById('overlay_modale');
var champ_nom = document.getElementById('champ_nom');
var champ_adr_voie = document.getElementById('champ_adr_voie');
var champ_com_nom = document.getElementById('champ_com_nom');
var coord_en_attente = null;

draw_dae.on('drawend', function(evt){
    var geom = evt.feature.getGeometry();
    coord_en_attente = geom.getCoordinates(); // [x, y] en EPSG:3857
    champ_nom.value = '';
    champ_adr_voie.value = '';
    champ_com_nom.value = '';
    overlay_modale.style.display = 'flex';
});

document.getElementById('btn_annuler_modale').addEventListener('click', function(){
    overlay_modale.style.display = 'none';
    source_point_dessin.clear();
    coord_en_attente = null;
});

document.getElementById('btn_valider_modale').addEventListener('click', function(){
    if (!coord_en_attente) return;

    var nom = champ_nom.value.trim();
    var adr_voie = champ_adr_voie.value.trim();
    var com_nom = champ_com_nom.value.trim();

    overlay_modale.style.display = 'none';

    var url = 'ajout_point.php?x=' + coord_en_attente[0] +
              '&y=' + coord_en_attente[1] +
              '&c_nom=' + encodeURIComponent(nom) +
              '&c_adr_voie=' + encodeURIComponent(adr_voie) +
              '&c_com_nom=' + encodeURIComponent(com_nom);

    fetch(url)
        .then(function(r){ return r.json(); })
        .then(function(resultat){
            if (resultat.success){
                donnees_dae.clear();
                donnees_dae.refresh();
                source_point_dessin.clear();
            }
            coord_en_attente = null;
        });
});
```

**Côté PHP**, `ajout_point.php` reçoit ces trois champs en plus de `x`/`y`, et les insère via des **paramètres liés** (`pg_query_params`), donc sans risque d'injection SQL, même si l'utilisateur saisit des caractères spéciaux :

```php
<?php
// ajout_point.php — insertion sécurisée d'un point dans dae_chalons
require_once('config.php');

header('Content-Type: application/json');

$conn = pg_connect("dbname='$db_name' user='$db_user' password='$db_pass' host='$db_host' port='$db_port'");
if (!$conn) {
    echo json_encode(['success' => false, 'error' => 'Connexion échouée : ' . pg_last_error()]);
    exit;
}

$x = isset($_GET['x']) ? floatval($_GET['x']) : null;
$y = isset($_GET['y']) ? floatval($_GET['y']) : null;
$c_nom = isset($_GET['c_nom']) ? trim($_GET['c_nom']) : null;
$c_adr_voie = isset($_GET['c_adr_voie']) ? trim($_GET['c_adr_voie']) : null;
$c_com_nom = isset($_GET['c_com_nom']) ? trim($_GET['c_com_nom']) : null;

if ($x === null || $y === null){
    echo json_encode(['success' => false, 'error' => 'Coordonnées manquantes']);
    exit;
}

$sql = "INSERT INTO dae_chalons(the_geom, c_nom, c_adr_voie, c_com_nom)
        VALUES (ST_SetSRID(ST_MakePoint($1, $2), 3857), $3, $4, $5)
        RETURNING gid";
$result = pg_query_params($conn, $sql, array($x, $y, $c_nom, $c_adr_voie, $c_com_nom));

if ($result){
    $row = pg_fetch_assoc($result);
    echo json_encode(['success' => true, 'gid' => $row['gid'], 'x' => $x, 'y' => $y]);
} else {
    echo json_encode(['success' => false, 'error' => pg_last_error($conn)]);
}

pg_close($conn);
?>
```

---

### 10.4 Modifier (déplacer) un point existant

Comme en section 3 du tutoriel principal, on combine `ol.interaction.Select` (pour choisir le point) et `ol.interaction.Modify` (pour le déplacer), en les limitant à la couche `couche_dae` et en les conditionnant à une case à cocher dédiée :

```javascript
var select_dae = new ol.interaction.Select({
    layers: [couche_dae],
    condition: function(evt){
        return checkbox_modifier.checked && ol.events.condition.click(evt);
    }
});
map.addInteraction(select_dae);

var modify_dae = new ol.interaction.Modify({
    features: select_dae.getFeatures(),
    condition: function(evt){
        return checkbox_modifier.checked;
    }
});
map.addInteraction(modify_dae);

modify_dae.on('modifyend', function(evt){
    var feature = evt.features.item(0);
    var gid = feature.getId();
    var coord = feature.getGeometry().getCoordinates();

    fetch('update_point.php?gid=' + gid + '&x=' + coord[0] + '&y=' + coord[1])
        .then(function(r){ return r.json(); });
});
```

> Ce mécanisme ne fonctionne que si `feature.getId()` renvoie bien le `gid` — d'où la correction apportée en 10.2 sur le script de lecture GeoJSON.

**Script PHP associé**, `update_point.php`, symétrique à `ajout_point.php` mais avec une clause `WHERE` sur la clé primaire :

```php
<?php
// update_point.php — déplacement sécurisé d'un point dans dae_chalons
require_once('config.php');

header('Content-Type: application/json');

$conn = pg_connect("dbname='$db_name' user='$db_user' password='$db_pass' host='$db_host' port='$db_port'");
if (!$conn) {
    echo json_encode(['success' => false, 'error' => 'Connexion échouée : ' . pg_last_error()]);
    exit;
}

$gid = isset($_GET['gid']) ? intval($_GET['gid']) : null;
$x = isset($_GET['x']) ? floatval($_GET['x']) : null;
$y = isset($_GET['y']) ? floatval($_GET['y']) : null;

if ($gid === null || $x === null || $y === null){
    echo json_encode(['success' => false, 'error' => 'Paramètres manquants']);
    exit;
}

$sql = "UPDATE dae_chalons SET the_geom = ST_SetSRID(ST_MakePoint($1, $2), 3857) WHERE gid = $3";
$result = pg_query_params($conn, $sql, array($x, $y, $gid));

if ($result){
    echo json_encode(['success' => true, 'gid' => $gid]);
} else {
    echo json_encode(['success' => false, 'error' => pg_last_error($conn)]);
}

pg_close($conn);
?>
```

---

### 10.5 Supprimer un point existant

Un bouton "Supprimer le point sélectionné" apparaît uniquement quand `select_dae` contient une entité. Il demande confirmation (`confirm()`) avant d'appeler `delete_point.php`, puis retire localement l'entité de la couche pour un retour visuel immédiat :

```javascript
document.getElementById('btn_supprimer').addEventListener('click', function(){
    var features = select_dae.getFeatures();
    if (features.getLength() === 0) return;

    var feature = features.item(0);
    var gid = feature.getId();

    if (!confirm('Supprimer ce point (gid ' + gid + ') ?')) return;

    fetch('delete_point.php?gid=' + gid)
        .then(function(r){ return r.json(); })
        .then(function(resultat){
            if (resultat.success){
                donnees_dae.removeFeature(feature);
                select_dae.getFeatures().clear();
            }
        });
});
```

```php
<?php
// delete_point.php — suppression sécurisée d'un point dans dae_chalons
require_once('config.php');

header('Content-Type: application/json');

$conn = pg_connect("dbname='$db_name' user='$db_user' password='$db_pass' host='$db_host' port='$db_port'");
if (!$conn) {
    echo json_encode(['success' => false, 'error' => 'Connexion échouée : ' . pg_last_error()]);
    exit;
}

$gid = isset($_GET['gid']) ? intval($_GET['gid']) : null;

if ($gid === null){
    echo json_encode(['success' => false, 'error' => 'Identifiant manquant']);
    exit;
}

$sql = "DELETE FROM dae_chalons WHERE gid = $1";
$result = pg_query_params($conn, $sql, array($gid));

if ($result){
    echo json_encode(['success' => true, 'gid' => $gid]);
} else {
    echo json_encode(['success' => false, 'error' => pg_last_error($conn)]);
}

pg_close($conn);
?>
```

> **Point de vigilance** : les deux modes (dessin / modification) doivent s'exclure mutuellement, sinon les interactions `Draw` et `Modify` entrent en conflit sur les mêmes clics. Dans `dessin_dae_chalons.html`, chaque case à cocher décoche automatiquement l'autre lors de son activation.

---

### 10.6 Fichiers finaux du projet

```
site_temporel/ol/
├── dessin_dae_chalons.html                     # carte + dessin + formulaire + modification + suppression
├── ajout_point.php                             # INSERT sécurisé (géométrie + attributs)
├── update_point.php                            # UPDATE sécurisé (déplacement)
├── delete_point.php                            # DELETE sécurisé
├── postgis_geojson_abdoulahat_alwaysdata.php   # lecture des points existants (corrigé : WHERE $geomfield, gid→id)
└── config.php                                  # identifiants base (non versionné)
```

**Récapitulatif des correctifs et ajouts de cette section :**

| Élément | Avant | Après |
|---|---|---|
| Clé primaire utilisée dans le JS/PHP | `id` | `gid` |
| Colonne géométrie utilisée dans le JS/PHP | `geom` | `the_geom` |
| Clause `WHERE` du script de lecture | codée en dur sur `geom` | dynamique sur `$geomfield` |
| Mapping de l'id GeoJSON | `if ($key == "id")` | `if ($key == "gid")` |
| Attributs enregistrés à l'ajout | aucun (géométrie seule) | `c_nom`, `c_adr_voie`, `c_com_nom` |
| Modification d'un point existant | non implémentée | `Select` + `Modify` + `update_point.php` |
| Suppression d'un point existant | non implémentée | bouton de suppression + `delete_point.php` |

---

*Cette section prolonge le tutoriel principal en s'appuyant sur une table PostGIS réelle (`dae_chalons`), illustrant l'importance de vérifier la structure exacte d'une table existante avant d'écrire du code qui la manipule.*

---------------------------------------------------------------------------------


**Quelques pistes pour prolonger cette base (certains déjà faits) :**

- **Formulaire d'attributs** : après le `drawend`, ouvrir une petite fenêtre demandant de saisir des attributs (nom, catégorie...) avant l'envoi au PHP, plutôt que d'enregistrer uniquement la géométrie.
- **Modifier/déplacer/supprimer les points existants** : combiner les interactions `Select` + `Modify`/`Translate` sur la couche `couche_dae`, avec des appels PHP `UPDATE`/`DELETE` (toujours avec `pg_query_params`, jamais de SQL construit côté client).
- **Undo/Redo** : conserver un historique des actions pour permettre d'annuler un ajout avant confirmation définitive.
- **Feedback visuel** : changer temporairement la couleur du point pendant l'envoi AJAX (ex. orange = en attente, vert = confirmé, rouge = erreur), pour un retour plus clair que le seul texte dans `#info`.
- **Authentification** : si l'ajout de points doit être réservé à certains utilisateurs, ajouter une vérification de session/API key dans `ajout_point.php` avant d'exécuter l'insertion.

---

*Ce tutoriel fait suite à **Page web avec OpenLayer**, qui couvre la mise en place de l'hébergement, les bases JavaScript/jQuery, l'affichage de données PostGIS, la symbologie, les contrôles, les interactions de sélection et la géolocalisation.*
