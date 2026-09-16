# Tutoriel : Cartographie web dynamique avec OpenLayers, PostGIS et PHP

> Ce tutoriel retrace pas à pas la construction d'une carte web interactive : depuis la préparation de l'hébergement jusqu'à la géolocalisation de l'utilisateur, en passant par l'affichage de données PostGIS, la symbologie, les contrôles et les infobulles.

## Sommaire

1. [Pré-requis et mise en place de l'hébergement](#1-pré-requis-et-mise-en-place-de-lhébergement)
2. [Éditer et publier ses fichiers avec Notepad++ et FTP](#2-éditer-et-publier-ses-fichiers-avec-notepad-et-ftp)
3. [Bases de JavaScript : variables et fonctions](#3-bases-de-javascript--variables-et-fonctions)
4. [jQuery : simplifier le JavaScript](#4-jquery--simplifier-le-javascript)
5. [Premiers pas avec OpenLayers](#5-premiers-pas-avec-openlayers)
6. [Afficher des données PostGIS via PHP](#6-afficher-des-données-postgis-via-php)
7. [Symbologie : couleurs graduées et catégorisées](#7-symbologie--couleurs-graduées-et-catégorisées)
8. [Les contrôles : échelle et sélecteur de couches](#8-les-contrôles--échelle-et-sélecteur-de-couches)
9. [Les interactions : sélectionner un objet](#9-les-interactions--sélectionner-un-objet)
10. [Les overlays : infobulle au clic](#10-les-overlays--infobulle-au-clic)
11. [La géolocalisation](#11-la-géolocalisation)
12. [Organiser son code : externaliser en fichier .js](#12-organiser-son-code--externaliser-en-fichier-js)
13. [Pièges rencontrés et comment les éviter](#13-pièges-rencontrés-et-comment-les-éviter)
14. [Structure finale du projet](#14-structure-finale-du-projet)

---

## 1. Pré-requis et mise en place de l'hébergement

Ce tutoriel utilise **AlwaysData** (alwaysdata.com), un hébergeur qui propose gratuitement :
- un espace web (dossier `www/`)
- une base de données PostgreSQL/PostGIS
- un accès FTP pour transférer ses fichiers
- l'exécution de scripts PHP côté serveur

### 1.1 Créer son compte et récupérer ses accès

1. Créez un compte sur [alwaysdata.com](https://www.alwaysdata.com/).
2. Dans l'espace d'administration, notez :
   - votre **nom de compte** (ex. `abdoulahat`)
   - votre **domaine par défaut** (ex. `abdoulahat.alwaysdata.net`)
   - vos **identifiants FTP** (souvent les mêmes que votre compte)
   - les **identifiants de connexion à la base PostgreSQL** (hôte, port, nom de la base, utilisateur, mot de passe) — disponibles dans la section "Bases de données" de l'administration.

### 1.2 Structure du dossier web

Sur AlwaysData, tout ce qui est déposé dans le dossier `www/` est directement accessible par le navigateur, à l'adresse :

```
http://VOTRE_COMPTE.alwaysdata.net/
```

Exemple pour ce tutoriel :

```
www/
├── images/
├── site_temporel/
│   ├── images/
│   └── ol/
│       ├── data/
│       ├── carte_web_cpgeom.html
│       ├── carte_commune_cpgeom.html
│       ├── geolocalisation.js
│       ├── ol-layerswitcher.css
│       ├── ol-layerswitcher.js
│       └── postgis_geojson_abdoulahat_alwaysdata.php
├── index.html
```

> ⚠️ **Règle d'or à retenir** : tous les chemins utilisés dans vos fichiers HTML/JS (`src="..."`, `url:'...'`) sont **relatifs à l'emplacement du fichier HTML qui les utilise**, pas à la racine du site. Si votre page est dans `site_temporel/ol/`, un chemin `images/regions.png` cherchera un dossier `images` **à l'intérieur de** `ol/`, pas ailleurs. Pour remonter d'un niveau, on utilise `../`.

---

## 2. Éditer et publier ses fichiers avec Notepad++ et FTP

### 2.1 Installer le plugin NppFTP

1. Téléchargez et installez [Notepad++](https://notepad-plus-plus.org/).
2. Ouvrez **Plugins → Gestionnaire de plugins**, cherchez **NppFTP**, installez-le.
3. Une fois installé, ouvrez le panneau **Plugins → NppFTP → Show NppFTP Window**.

### 2.2 Configurer la connexion au serveur

Dans le panneau NppFTP, cliquez sur l'icône d'engrenage (Settings) → **Profile settings** :

| Champ | Valeur |
|---|---|
| Hostname | `ftp.alwaysdata.com` (ou l'hôte FTP indiqué dans votre espace AlwaysData) |
| Port | `21` |
| Nom d'utilisateur | votre identifiant AlwaysData |
| Mot de passe | votre mot de passe |
| Protocole | FTP (ou FTPS si proposé) |

Une fois connecté, vous verrez l'arborescence de votre serveur (`/www/...`) dans le panneau de droite, et vous pouvez double-cliquer sur un fichier distant pour l'ouvrir et l'éditer directement dans Notepad++.

### 2.3 ⚠️ Le piège classique : les modifications qui ne s'enregistrent pas

**Symptôme :** vous modifiez un fichier, vous l'enregistrez (Ctrl+S), vous rechargez la page dans le navigateur... et rien n'a changé. Vous avez beau corriger le code, l'erreur persiste.

**Cause fréquente :** la session FTP dans Notepad++ s'est déconnectée silencieusement (perte de réseau, timeout du serveur...). Notepad++ continue d'afficher le fichier comme s'il était synchronisé, mais l'enregistrement local ne repart plus vers le serveur.

**Solution :**
1. Dans le panneau NppFTP, **déconnectez-vous** (bouton de déconnexion).
2. **Reconnectez-vous** au serveur.
3. Ré-ouvrez le fichier depuis le serveur (pas depuis un onglet déjà ouvert), refaites vos modifications si besoin, et enregistrez.
4. Vérifiez toujours, en cas de doute, la **date de dernière modification** du fichier affichée dans l'arborescence FTP.

**Bon réflexe de vérification :** testez votre page en **navigation privée** (Ctrl+Maj+N) après chaque upload important, pour être certain de ne pas regarder une version en cache.

---

## 3. Bases de JavaScript : variables et fonctions

### 3.1 Les variables

Une variable stocke une donnée qui peut changer pendant l'exécution du script.

```javascript
var chaine = "bonjour";
let couche = "regions";   // 'let' est la version moderne de 'var'
```

Règles de nommage :
- doit commencer par une lettre ou `_`
- peut contenir lettres, chiffres, `_`
- sensible à la casse (`Liste` ≠ `liste`)

Types de données en JavaScript : nombres, chaînes de caractères (string), booléens (`true`/`false`), `null`.

**Les tableaux** stockent plusieurs valeurs, indexées à partir de 0 :

```javascript
var mon_tab = ['regions', 'depts', 'communes'];
console.log(mon_tab[1]); // affiche "depts"
```

**Les tableaux associatifs** (objets) utilisent des clés personnalisées :

```javascript
var tab_depts = {"nom_dept": "AIN", "code_dept": "01", "pop": 220000};
console.log(tab_depts.code_dept); // affiche "01"
```

### 3.2 Les fonctions

Une fonction regroupe des instructions réutilisables :

```javascript
function choix_liste(param_choix){
    document.getElementById('info').innerHTML = param_choix;
}
```

On l'appelle ainsi :
```javascript
choix_liste('regions');
```

Les fonctions doivent être **déclarées avant d'être appelées** — d'où l'habitude de les écrire dans le `<HEAD>` de la page, avant que le `<BODY>` ne s'exécute.

### 3.3 Exemple complet : lier une liste déroulante à un texte affiché

```html
<HTML>
 <HEAD>
   <META charset="utf-8" />
   <SCRIPT>
   function choix_liste(param_choix){
       document.getElementById('info').innerHTML = param_choix;
   }
   </SCRIPT>
 </HEAD>
 <BODY>
   <H1>Le javascript</H1>
   <FORM name="form">
    <SELECT id="liste" name="liste" onchange="choix_liste(this.value);">
      <OPTION value=0>----choisir dans la liste-----</OPTION>
      <OPTION value="regions">Les régions</OPTION>
      <OPTION value="departements">Les départements</OPTION>
      <OPTION value="communes">Les communes</OPTION>
    </SELECT>
   </FORM>
   <DIV id="info"></DIV>
 </BODY>
</HTML>
```

`this.value` transmet à la fonction la valeur actuellement sélectionnée dans la liste (technique dite du **passage de paramètre**).

### 3.4 Changer une image selon le choix, avec un `switch`

```javascript
function test_fonction(){
    let couche = document.getElementById('liste').value;

    switch (couche){
        case '0':
            document.getElementById('info').innerHTML = '';
            document.getElementById('carte').src = '';
            break;
        case 'communes':
            document.getElementById('info').innerHTML = 'couche non disponible';
            document.getElementById('carte').src = '';
            break;
        default:
            let url_image = 'images/' + couche + '.png';
            document.getElementById('info').innerHTML = '';
            document.getElementById('carte').src = url_image;
    }
}
```

> **Erreurs classiques à surveiller** : `.scr` au lieu de `.src`, `getElementByID` au lieu de `getElementById` (respecter la casse !), guillemets non fermés, oubli du `.` avant l'extension `png`, `breack` au lieu de `break`.

---

## 4. jQuery : simplifier le JavaScript

jQuery est une bibliothèque qui simplifie l'écriture du JavaScript, notamment pour manipuler le DOM (les éléments de la page) et gérer les événements.

### 4.1 Charger jQuery

**Option A — Fichier téléchargé en local** (recommandé pour un usage hors-ligne ou pédagogique) :

1. Téléchargez le fichier sur [jquery.com/download](https://jquery.com/download/) (version compressée "production").
2. Placez le fichier dans un dossier `scripts/` de votre projet.
3. Chargez-le dans le `<HEAD>` :
```html
<script src="scripts/jquery-4.0.0.min.js"></script>
```

**Option B — CDN** (plus simple, nécessite une connexion internet) :
```html
<script src="https://code.jquery.com/jquery-4.0.0.min.js"></script>
```

> ⚠️ Si vous ajoutez les attributs `integrity` et `crossorigin` (vérification de sécurité du fichier CDN), la valeur de `integrity` doit être **copiée exactement** depuis jquery.com. Une valeur inventée ou un texte de remplacement fera échouer le chargement de jQuery **silencieusement** — symptôme typique : rien ne se passe quand on interagit avec la page, sans message d'erreur visible dans la console.

### 4.2 Syntaxe de base

| JavaScript pur | jQuery |
|---|---|
| `document.getElementById('liste').value` | `$('#liste').val()` |
| `document.getElementById('info').innerHTML = x` | `$('#info').html(x)` |
| élément avec `onchange="..."` dans le HTML | `$('#liste').on('change', function(){...})` |

### 4.3 Exemple : réagir au changement de sélection

```javascript
$(document).ready(function(){
    $('#liste').on('change', function(){
        let couche = $(this).val();

        switch (couche){
            case '0':
                $('#info').html('');
                $('#carte').attr('src', '');
                break;
            case 'communes':
                $('#info').html('couche non disponible');
                $('#carte').attr('src', '');
                break;
            default:
                let url_image = 'images/' + couche + '.png';
                $('#info').html('');
                $('#carte').attr('src', url_image);
        }
    });
});
```

> ⚠️ **Point essentiel** : ce code doit être enveloppé dans `$(document).ready(function(){ ... });`. Sans cela, si le script est placé dans le `<HEAD>`, il s'exécute **avant** que le `<SELECT id="liste">` existe dans la page — `$('#liste')` ne trouve alors rien, et aucun événement n'est attaché. `$(document).ready()` attend que toute la page soit chargée avant d'exécuter le code qu'il contient.

### 4.4 Attacher un événement à un bouton

```html
<BUTTON id="b_ok">ok avec jquery</BUTTON>
<DIV id="info"></DIV>

<SCRIPT>
$('#b_ok').click(function(){
    console.log("click");
    for (i = 1; i < 10; i++){
        $('#info').append(i + "<BR/>");
    }
});
</SCRIPT>
```

---

## 5. Premiers pas avec OpenLayers

[OpenLayers](https://openlayers.org/) est une bibliothèque JavaScript pour créer des cartes web interactives.

### 5.1 Charger la bibliothèque

```html
<script src="https://cdn.jsdelivr.net/npm/ol@v10.10.0/dist/ol.js"></script>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/ol@v10.10.0/ol.css">
```

### 5.2 Créer une carte vide

`ol.Map` est l'objet central : il s'insère dans une `<DIV>` cible et contiendra ensuite couches, vue, contrôles et interactions.

```html
<HTML>
<HEAD>
   <META charset="utf-8" />
   <script src="https://cdn.jsdelivr.net/npm/ol@v10.10.0/dist/ol.js"></script>
   <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/ol@v10.10.0/ol.css">
 </HEAD>
 <STYLE>
 #carte{
    width:100%;
    height:80%;
    background-color:#eceae5;
 }
 </STYLE>
<BODY>
<DIV id="carte"></DIV>
<SCRIPT>
    var map = new ol.Map({
        target: 'carte'
    });
</SCRIPT>
</BODY>
</HTML>
```

À ce stade, vous verrez seulement le cadre gris et les boutons de zoom (`+`/`-`) : **aucun fond de carte n'est encore chargé**, car aucune couche ni vue n'a été définie.

### 5.3 Ajouter une vue (`ol.View`)

La vue définit le point central et le niveau de zoom initial.

```javascript
map.setView(new ol.View({
    center: [263652, 5919266],  // coordonnées en projection Web Mercator (EPSG:3857)
    zoom: 6
}));
```

> ⚠️ Par défaut, OpenLayers utilise la projection **EPSG:3857** (Web Mercator) pour les coordonnées. Si vos coordonnées viennent d'un système différent (ex. Lambert 93 / EPSG:2154), il faut soit les convertir au préalable, soit préciser la projection dans la vue.

### 5.4 Ajouter un fond de carte (`ol.source` + `ol.layer`)

Le principe général d'OpenLayers : une **source** (d'où viennent les données) est encapsulée dans une **couche** (comment on les affiche), puis la couche est ajoutée à la carte.

```javascript
// SOURCE
var source_osm = new ol.source.OSM();
// LAYER
var couche_osm = new ol.layer.Tile({
    source: source_osm
});
// MAP (les couches peuvent être passées directement ici...)
var map = new ol.Map({
    target: 'carte',
    layers: [couche_osm]
});
// ...ou ajoutées après coup
// map.addLayer(couche_osm);
```

### 5.5 Fichier complet à cette étape

```html
<HTML>
<HEAD>
   <META charset="utf-8" />
   <script src="https://cdn.jsdelivr.net/npm/ol@v10.10.0/dist/ol.js"></script>
   <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/ol@v10.10.0/ol.css">
 </HEAD>
 <STYLE>
 #carte{
    width:100%;
    height:80%;
    background-color:#eceae5;
 }
 </STYLE>
<BODY>
<DIV id="carte"></DIV>
<SCRIPT>
    //SOURCE
    var source_osm = new ol.source.OSM();
    //LAYER
    var couche_osm = new ol.layer.Tile({
        source: source_osm
    });
    //MAP
    var map = new ol.Map({
        target: 'carte',
        layers: [couche_osm]
    });
    //VIEW
    map.setView(new ol.View({
        center: [263652, 5919266],
        zoom: 6
    }));
</SCRIPT>
</BODY>
</HTML>
```

---

## 6. Afficher des données PostGIS via PHP

Le navigateur ne peut pas interroger directement une base PostgreSQL/PostGIS : il faut un intermédiaire côté serveur. C'est le rôle du fichier **PHP**.

### 6.1 Principe général

```
Navigateur (OpenLayers)  --requête HTTP-->  Fichier PHP  --requête SQL-->  Base PostGIS
Navigateur (OpenLayers)  <--réponse GeoJSON--  Fichier PHP  <--résultat--  Base PostGIS
```

Le fichier PHP (fourni par le formateur dans ce cours, nommé ici `postgis_geojson_abdoulahat_alwaysdata.php`) :
1. reçoit un paramètre dans l'URL, par exemple `?geotable=batiments`,
2. se connecte à la base PostgreSQL avec les identifiants du compte AlwaysData,
3. exécute une requête SQL sur la table demandée (avec une protection contre l'injection SQL — le nom de table doit être validé côté serveur),
4. convertit le résultat en GeoJSON (souvent via la fonction PostGIS `ST_AsGeoJSON`),
5. renvoie ce GeoJSON en réponse HTTP.

> Ce fichier PHP contient les **identifiants de connexion à la base de données** (hôte, utilisateur, mot de passe). Il doit rester **côté serveur** uniquement — ne jamais le rendre public tel quel sur un dépôt GitHub sans en retirer les identifiants (voir section 13.4).

### 6.2 Contenu réel du fichier PHP

Voici le script complet utilisé dans ce projet (`postgis_geojson_abdoulahat_alwaysdata.php`). Il est générique : il accepte plusieurs paramètres dans l'URL, pas seulement `geotable`.

| Paramètre GET | Rôle | Obligatoire | Valeur par défaut |
|---|---|---|---|
| `geotable` | nom de la table ou vue PostGIS à interroger | **oui** | — |
| `geomfield` | nom de la colonne géométrie | non | `geom` |
| `srid` | SRID de sortie du GeoJSON | non | `3857` |
| `fields` | colonnes à retourner | non | `*` (toutes) |
| `parameters` | clause `WHERE` additionnelle | non | vide |
| `orderby` | colonne de tri | non | vide |
| `sort` | sens du tri (`ASC`/`DESC`) | non | `ASC` |
| `limit` | nombre max de résultats | non | vide (pas de limite) |
| `offset` | décalage pour la pagination | non | vide |

Exemple d'URL avec plusieurs paramètres :
```
postgis_geojson_abdoulahat_alwaysdata.php?geotable=batiments&fields=id,sup_m2&orderby=sup_m2&sort=DESC&limit=50
```

**Étapes internes du script :**

1. Récupère et valide les paramètres GET (avec des valeurs par défaut si absents).
2. Se connecte à PostgreSQL avec `pg_connect(...)`.
3. Construit dynamiquement une requête SQL, avec `st_asgeojson(st_transform(geom, srid))` pour convertir chaque géométrie PostGIS en GeoJSON, dans le SRID demandé.
4. Exécute la requête (`pg_query`) et parcourt chaque ligne du résultat (`pg_fetch_assoc`).
5. Construit manuellement une structure `FeatureCollection` GeoJSON, en assemblant pour chaque ligne un objet `Feature` avec sa géométrie (`geometry`) et ses attributs (`properties`).
6. Renvoie l'ensemble en `echo`, ce que le navigateur reçoit comme réponse à l'appel `fetch`/`ol.source.Vector`.

```php
<?php
/**
 * PostGIS to GeoJSON
 * Interroge une table ou vue PostGIS et retourne le résultat au format GeoJSON,
 * directement exploitable par OpenLayers, Leaflet, etc.
 *
 * Paramètres GET :
 * - geotable    (obligatoire) nom de la table/vue PostGIS
 * - geomfield   nom de la colonne géométrie (défaut : geom)
 * - srid        SRID de sortie du GeoJSON (défaut : 3857)
 * - fields      colonnes à retourner (défaut : toutes, "*")
 * - parameters  clause WHERE additionnelle
 * - orderby     colonne de tri
 * - sort        sens du tri (ASC/DESC, défaut : ASC)
 * - limit       nombre maximum de résultats
 * - offset      décalage (pagination)
 */

function escapeJsonString($value) {
    // Liste des caractères à échapper selon json.org (\b backspace, \f formfeed, etc.)
    $escapers     = array("\\", "/", "\"", "\n", "\r", "\t", "\x08", "\x0c");
    $replacements = array("\\\\", "\\/", "\\\"", "\\n", "\\r", "\\t", "\\f", "\\b");
    if ($value != null) {
        return str_replace($escapers, $replacements, $value);
    }
}

// --- Récupération et valeurs par défaut des paramètres GET ---

if (empty($_GET['geotable'])) {
    echo "missing required parameter: <i>geotable</i>";
    exit;
} else {
    $geotable = $_GET['geotable'];
}

$geomfield = empty($_GET['geomfield']) ? 'geom' : $_GET['geomfield'];
$srid      = empty($_GET['srid'])      ? '3857' : $_GET['srid'];
$fields    = empty($_GET['fields'])    ? '*'    : $_GET['fields'];
$parameters= empty($_GET['parameters'])? ''     : $_GET['parameters'];
$orderby   = empty($_GET['orderby'])   ? ''     : $_GET['orderby'];
$sort      = empty($_GET['sort'])      ? 'ASC'  : $_GET['sort'];
$limit     = empty($_GET['limit'])     ? ''     : $_GET['limit'];
$offset    = empty($_GET['offset'])    ? ''     : $_GET['offset'];

// --- Connexion à la base PostgreSQL ---
// ⚠️ Les identifiants sont chargés depuis config.php, qui n'est PAS versionné sur GitHub
// (voir section 13.6). Ne jamais écrire les identifiants en clair dans ce fichier.
require_once('config.php');

$conn = pg_connect(
    "dbname='$db_name' user='$db_user' password='$db_pass' host='$db_host' port='$db_port'"
);
if (!$conn) {
    echo "Not connected : " . pg_error();
    exit;
}

// --- Construction de la requête SQL ---
$sql  = "SELECT " . pg_escape_string($conn, $fields);
$sql .= ", st_asgeojson(st_transform(" . $geomfield . ",$srid)) AS geojson FROM ";
$sql .= pg_escape_string($conn, $geotable);
$sql .= " WHERE geom is not null";

if (strlen(trim($parameters)) > 0) {
    $sql .= " AND " . $parameters;
}
if (strlen(trim($orderby)) > 0) {
    $sql .= " ORDER BY " . pg_escape_string($conn, $orderby) . " " . $sort;
}
if (strlen(trim($limit)) > 0) {
    $sql .= " LIMIT " . pg_escape_string($conn, $limit);
}
if (strlen(trim($offset)) > 0) {
    $sql .= " OFFSET " . pg_escape_string($conn, $offset);
}

// --- Exécution ---
$rs = pg_query($conn, $sql);
if (!$rs) {
    echo "An SQL error occured.\n";
    exit;
}

// --- Construction manuelle du GeoJSON ---
$output    = '';
$rowOutput = '';

while ($row = pg_fetch_assoc($rs)) {
    $rowOutput  = (strlen($rowOutput) > 0 ? ',' : '');
    $rowOutput .= '{"type": "Feature","geometry":' . $row['geojson'] . ',"properties": {';
    $props = '';
    $id    = '';
    foreach ($row as $key => $val) {
        if ($key != "geojson") {
            $props .= (strlen($props) > 0 ? ',' : '') . '"' . $key . '":"' . escapeJsonString($val) . '"';
        }
        if ($key == "id") {
            $id .= ',"id":"' . escapeJsonString($val) . '"';
        }
    }
    $rowOutput .= $props . '}';
    $rowOutput .= $id;
    $rowOutput .= '}';
    $output    .= $rowOutput;
}

$output = '{"type": "FeatureCollection", "crs": { "type": "name", "properties": { "name": "urn:ogc:def:crs:EPSG::' . $srid . '" } },"features": [' . $output . ']}';
echo $output;

pg_free_result($rs);
pg_close($conn);
?>
```

**Fichier `config.php` correspondant** (à créer à côté, et à ajouter au `.gitignore`) :

```php
<?php
// config.php — NE PAS VERSIONNER CE FICHIER SUR GITHUB
$db_name = 'abdoulahat_alwaysdata';
$db_user = 'abdoulahat';
$db_pass = 'VOTRE_MOT_DE_PASSE';
$db_host = 'postgresql-abdoulahat.alwaysdata.net';
$db_port = '5432';
?>
```

> ⚠️ **Sécurité — points de vigilance sur ce script** :
> - Les paramètres `fields`, `parameters` et `orderby` sont insérés dans la requête SQL avec `pg_escape_string`, qui échappe les caractères spéciaux mais **ne protège pas contre l'injection de mots-clés SQL** (par exemple via `parameters=1=1; DROP TABLE ...` ou l'ajout de sous-requêtes). Dans un contexte de production, il est préférable de valider `geotable`, `fields` et `orderby` contre une **liste blanche** de valeurs autorisées, et de passer `parameters` sous une forme structurée plutôt qu'en SQL brut.
> - Le mot de passe ne doit **jamais** figurer en clair dans un fichier versionné sur GitHub — voir la section 13.6 pour la méthode avec `config.php` + `.gitignore`.

### 6.3 Appeler ce PHP depuis OpenLayers

```javascript
//SOURCE DES BATIMENTS (PostGIS via PHP)
var donnees_batiments = new ol.source.Vector({
    url: 'postgis_geojson_abdoulahat_alwaysdata.php?geotable=batiments',
    format: new ol.format.GeoJSON()
});

//LAYER DES BATIMENTS
var couche_batiments = new ol.layer.Vector({
    source: donnees_batiments
});

map.addLayer(couche_batiments);
```

> Le chemin `'postgis_geojson_abdoulahat_alwaysdata.php?geotable=batiments'` est **relatif à l'emplacement du fichier HTML**. S'il est dans le même dossier que le PHP, ce chemin simple suffit. S'il est dans un sous-dossier, il faudra `../nom_du_fichier.php`.

### 6.4 Zoomer automatiquement sur l'étendue des données chargées

Le chargement des données est **asynchrone** (il prend un peu de temps). L'événement `featuresloadend` se déclenche une fois que toutes les entités sont arrivées :

```javascript
donnees_batiments.on('featuresloadend', function(){
    console.log('Nombre objets :', donnees_batiments.getFeatures().length);
    map.getView().fit(donnees_batiments.getExtent(), {
        padding: [20, 20, 20, 20],
        maxZoom: 14
    });
});
```

- `getExtent()` calcule le rectangle englobant de toutes les entités.
- `fit()` ajuste le centre et le zoom de la vue pour cadrer cette étendue.
- `padding` ajoute une marge en pixels autour des données.
- `maxZoom` évite un zoom excessif si les données sont très concentrées.

Il est utile d'écouter aussi les erreurs de chargement :

```javascript
donnees_batiments.on('featuresloaderror', function(){
    console.error('Erreur de chargement de la couche batiments');
});
```

---

## 7. Symbologie : couleurs graduées et catégorisées

Au lieu d'un style fixe, on peut passer une **fonction** à la propriété `style` d'une couche vectorielle : elle est appelée pour chaque entité et doit retourner un `ol.style.Style`.

### 7.1 Symbologie graduée (dégradé selon une valeur numérique)

Exemple : colorer les bâtiments selon leur surface (`sup_m2`), du plus clair (petit) au plus foncé (grand) :

```javascript
//PALETTE GRADUEE SELON sup_m2
function couleur_selon_surface(sup_m2){
    var valeur = parseFloat(sup_m2) || 0;
    if (valeur < 500)        return '#ffffb2';
    else if (valeur < 1000)  return '#fed976';
    else if (valeur < 5000)  return '#feb24c';
    else if (valeur < 20000) return '#fd8d3c';
    else if (valeur < 50000) return '#f03b20';
    else                     return '#bd0026';
}

function style_batiment(feature){
    var sup_m2 = feature.get('sup_m2');
    var couleur = couleur_selon_surface(sup_m2);

    return new ol.style.Style({
        stroke: new ol.style.Stroke({color: '#555555', width: 1}),
        fill: new ol.style.Fill({color: couleur})
    });
}

var couche_batiments = new ol.layer.Vector({
    source: donnees_batiments,
    style: style_batiment
});
```

**Version plus avancée : classes calculées automatiquement (quantiles)**, pour que la répartition des couleurs s'adapte aux données réellement chargées plutôt qu'à des seuils fixes :

```javascript
var classes_sup_ha = [];
var couleurs = ['#ffffcc', '#a1dab4', '#41b6c4', '#2c7fb8', '#253494'];

function classe_sup_ha(valeur){
    if (classes_sup_ha.length === 0) return 0;
    if (valeur <= classes_sup_ha[0]) return 0;
    if (valeur <= classes_sup_ha[1]) return 1;
    if (valeur <= classes_sup_ha[2]) return 2;
    if (valeur <= classes_sup_ha[3]) return 3;
    return 4;
}

function style_epci(feature){
    var superficie = parseFloat(feature.get('sup_ha')) || 0;
    var classe = classe_sup_ha(superficie);
    return new ol.style.Style({
        stroke: new ol.style.Stroke({color: '#333333', width: 1.5}),
        fill: new ol.style.Fill({color: couleurs[classe] + '80'})
    });
}

// Une fois les données chargées, on calcule les seuils (quintiles) :
donnees_epci.on('featuresloadend', function(){
    var superficies = [];
    donnees_epci.getFeatures().forEach(function(feature){
        var valeur = parseFloat(feature.get('sup_ha'));
        if (!isNaN(valeur)) superficies.push(valeur);
    });
    superficies.sort(function(a, b){ return a - b; });

    if (superficies.length > 0){
        classes_sup_ha = [
            superficies[Math.floor(superficies.length * 0.20)],
            superficies[Math.floor(superficies.length * 0.40)],
            superficies[Math.floor(superficies.length * 0.60)],
            superficies[Math.floor(superficies.length * 0.80)]
        ];
    }
    couche_epci.changed(); // force le réaffichage avec les nouveaux seuils
});
```

### 7.2 Symbologie catégorisée (une couleur par valeur de texte)

Deux approches possibles :

**a) Dictionnaire fixe** (on connaît à l'avance les valeurs possibles) :

```javascript
var couleurs_par_section = {
    'A':  '#e6194B',
    'ZA': '#3cb44b',
    'ZB': '#4363d8',
    'ZC': '#f58231',
    'ZD': '#911eb4',
    'ZE': '#42d4f4'
};
var couleur_defaut = '#808080';

function style_parcelle(feature){
    var section = feature.get('section');
    var couleur = couleurs_par_section[section] || couleur_defaut;

    return new ol.style.Style({
        stroke: new ol.style.Stroke({color: couleur, width: 2}),
        fill: new ol.style.Fill({color: couleur + '55'})
    });
}
```

**b) Attribution dynamique** (on ne connaît pas à l'avance toutes les valeurs — une couleur est piochée dans une palette au fur et à mesure qu'une nouvelle valeur est rencontrée) :

```javascript
var palette_couleurs = ['#e6194B', '#3cb44b', '#4363d8', '#f58231', '#911eb4'];
var couleurs_attribuees = {};
var compteur_couleur = 0;

function style_categorise(feature){
    var valeur = feature.get('section');

    if (!couleurs_attribuees.hasOwnProperty(valeur)){
        couleurs_attribuees[valeur] = palette_couleurs[compteur_couleur % palette_couleurs.length];
        compteur_couleur++;
    }
    var couleur = couleurs_attribuees[valeur];

    return new ol.style.Style({
        stroke: new ol.style.Stroke({color: couleur, width: 2}),
        fill: new ol.style.Fill({color: couleur + '55'})
    });
}
```

---

## 8. Les contrôles : échelle et sélecteur de couches

Un **contrôle** est un élément d'interface fixe (bouton, panneau) qui ne bouge pas quand on déplace la carte — à la différence d'une couche ou d'un overlay.

### 8.1 Les contrôles par défaut

`ol.Map` inclut par défaut : `ol.control.Zoom`, `ol.control.Rotate`, `ol.control.Attribution`. On peut les désactiver :

```javascript
map = new ol.Map({
    controls: ol.control.defaults({
        attribution: false,
        rotate: false,
        zoom: false
    }),
    target: 'carte'
});
```

### 8.2 La barre d'échelle

```javascript
map.addControl(new ol.control.ScaleLine({
    minWidth: 64,
    units: 'metric'
}));
```

### 8.3 Le sélecteur de couches (LayerSwitcher)

`ol-layerswitcher` est une bibliothèque externe (pas incluse dans OpenLayers) qui ajoute un panneau permettant d'afficher/masquer chaque couche.

**Fichiers nécessaires** : `ol-layerswitcher.js` et `ol-layerswitcher.css`, à charger dans le `<HEAD>` — soit depuis un CDN, soit depuis des fichiers téléchargés et déposés à côté de votre page.

```html
<script src="ol-layerswitcher.js"></script>
<link rel="stylesheet" href="ol-layerswitcher.css">
```

Chaque couche destinée à apparaître dans le sélecteur doit recevoir une propriété `title`. Les fonds de carte (un seul actif à la fois) reçoivent en plus `type: 'base'` :

```javascript
var couche_osm = new ol.layer.Tile({
    source: new ol.source.OSM(),
    title: 'Fond OSM',
    type: 'base'
});

var couche_esri = new ol.layer.Tile({
    source: new ol.source.XYZ({
        url: 'https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}',
        attributions: '© Esri'
    }),
    title: 'Fond Esri',
    type: 'base'
});

var couche_batiments = new ol.layer.Vector({
    source: donnees_batiments,
    style: style_batiment,
    title: 'Les bâtiments de la commune'
});

var map = new ol.Map({
    target: 'carte',
    layers: [couche_osm, couche_esri, couche_batiments]
});

var layerSwitcher = new ol.control.LayerSwitcher({});
map.addControl(layerSwitcher);
```

---

## 9. Les interactions : sélectionner un objet

Une **interaction** est un comportement déclenché par un événement utilisateur (clic, molette, clavier...), sans forcément passer par un élément HTML visible — contrairement à un contrôle.

### 9.1 Les interactions par défaut

Le comportement standard (déplacer la carte au cliqué-glissé, zoomer à la molette, etc.) est fourni par `ol.interaction.defaults()`. Pour ajouter une interaction **en plus** de celles-ci, sans les supprimer :

```javascript
map = new ol.Map({
    interactions: ol.interaction.defaults().extend([new ol.interaction.Select()]),
    target: 'carte'
});
```

### 9.2 `ol.interaction.Select`

Permet de sélectionner un objet au clic, avec un style dédié pour l'objet sélectionné.

```javascript
var selectInteraction = new ol.interaction.Select({
    layers: [couche_batiments], // limite la sélection à cette couche
    style: new ol.style.Style({
        fill: new ol.style.Fill({color: 'yellow'}),
        stroke: new ol.style.Stroke({color: 'red', width: 2})
    })
});
map.addInteraction(selectInteraction);
```

### 9.3 Récupérer les informations de l'objet sélectionné

```javascript
selectInteraction.on('select', function(evt){
    var objets_selectionnes = selectInteraction.getFeatures();
    var texte = '';
    objets_selectionnes.forEach(function(feature){
        texte += 'Surface : ' + feature.get('sup_m2') + ' m²';
    });
    document.getElementById('info').innerHTML = texte || 'Cliquez sur un objet pour voir ses informations';
});
```

### 9.4 Afficher tous les attributs dynamiquement

Plutôt que de citer chaque attribut un par un, on peut parcourir toutes les propriétés d'une entité :

```javascript
map.on('singleclick', function(evt){
    var feature = map.forEachFeatureAtPixel(evt.pixel, function(feature){
        return feature;
    }, {
        layerFilter: function(layer){ return layer === couche_epci; }
    });

    if (feature){
        var proprietes = feature.getProperties();
        var texte = '<b>Informations</b><br><br>';
        for (var nom in proprietes){
            if (nom === 'geom' || nom === 'geometry') continue; // on ignore la géométrie
            texte += '<b>' + nom + ' :</b> ' + proprietes[nom] + '<br>';
        }
        document.getElementById('info').innerHTML = texte;
    }
});
```

---

## 10. Les overlays : infobulle au clic

Un **overlay** est une surcouche positionnée en coordonnées cartographiques : contrairement à un contrôle, il **se déplace** avec la carte, car il est ancré à un point géographique précis.

### 10.1 Principe

1. Une `<DIV>` sert de support visuel pour l'infobulle.
2. On la transforme en `ol.Overlay`, rattaché à la carte.
3. Au clic, on cherche un objet à l'endroit cliqué (`forEachFeatureAtPixel`).
4. Si un objet est trouvé : on remplit la `<DIV>` et on positionne l'overlay sur les coordonnées du clic.
5. Sinon : on masque l'overlay (`setPosition(undefined)`).

### 10.2 Exemple complet

```html
<DIV id="carte"></DIV>
<DIV id="info">Cliquez sur une entité pour voir ses informations</DIV>

<STYLE>
#info{
    position: absolute;
    bottom: 20px;
    left: 20px;
    z-index: 1000;
    background: white;
    padding: 10px 14px;
    box-shadow: 1px 1px 12px #555;
    border-radius: 4px;
    max-width: 300px;
}
</STYLE>

<SCRIPT>
var info = document.getElementById('info');
var popup = new ol.Overlay({
    element: info,
    offset: [0, -15],
    positioning: 'bottom-center',
    stopEvent: false
});
map.addOverlay(popup);

map.on('singleclick', function(evt){
    var feature = map.forEachFeatureAtPixel(evt.pixel, function(feature){
        return feature;
    }, {
        layerFilter: function(layer){ return layer === couche_batiments; }
    });

    if (feature){
        info.innerHTML = 'Surface : ' + feature.get('sup_m2') + ' m²';
        popup.setPosition(evt.coordinate);
    } else {
        popup.setPosition(undefined);
        info.innerHTML = 'Cliquez sur une entité pour voir ses informations';
    }
});
</SCRIPT>
```

> **Astuce** : on peut combiner `ol.interaction.Select` (pour le surlignage visuel de l'objet) et l'overlay (pour l'infobulle positionnée), comme dans l'exemple final de la section 14.

---

## 11. La géolocalisation

`ol.Geolocation` utilise l'API de géolocalisation du navigateur (HTML5) pour positionner la carte sur l'utilisateur.

> ⚠️ **Pré-requis technique important** : la géolocalisation HTML5 nécessite une page servie en **HTTPS** (ou en local via `localhost`) dans la plupart des navigateurs modernes. Une page en `http://` simple peut voir la fonctionnalité bloquée par le navigateur.

### 11.1 Mise en place de base

```javascript
var geolocation = new ol.Geolocation({
    projection: map.getView().getProjection(),
    tracking: false // ne démarre pas automatiquement
});

// Recentre la carte dès qu'une position est obtenue, puis arrête le suivi
geolocation.on('change', function(){
    map.getView().setCenter(geolocation.getPosition());
    map.getView().setZoom(15);
    geolocation.setTracking(false);
});
```

### 11.2 Afficher la position et le cercle de précision

```javascript
// Point représentant la position de l'utilisateur
var point_position = new ol.Feature();
point_position.setStyle(new ol.style.Style({
    image: new ol.style.Circle({
        radius: 6,
        fill: new ol.style.Fill({color: '#3399CC'}),
        stroke: new ol.style.Stroke({color: '#ffffff', width: 2})
    })
}));

geolocation.on('change:position', function(){
    var coordinates = geolocation.getPosition();
    if (coordinates){
        point_position.setGeometry(new ol.geom.Point(coordinates));
    }
});

// Cercle de précision (zone d'incertitude du GPS)
var precisionzone = new ol.Feature();
geolocation.on('change:accuracyGeometry', function(){
    precisionzone.setGeometry(geolocation.getAccuracyGeometry());
});

// Couche dédiée à l'affichage de ces deux entités
var couche_position = new ol.layer.Vector({
    source: new ol.source.Vector({
        features: [precisionzone, point_position]
    }),
    title: 'Ma position'
});
map.addLayer(couche_position);
```

### 11.3 Activer le suivi via un bouton

```html
<button id="bouton_geoloc">📍 Me géolocaliser</button>
```

```javascript
document.getElementById('bouton_geoloc').addEventListener('click', function(){
    geolocation.setTracking(true);
});
```

### 11.4 Gérer les erreurs

```javascript
geolocation.on('error', function(error){
    console.error('Erreur de géolocalisation :', error.message);
    info.innerHTML = 'Erreur de géolocalisation : ' + error.message;
});
```

Causes typiques d'erreur : utilisateur qui refuse la permission, page non servie en HTTPS, navigateur/appareil sans GPS.

---

## 12. Organiser son code : externaliser en fichier .js

Une fois le script principal chargé de fonctionnalités, il devient utile de **découper le code en fichiers séparés**, réutilisables d'une page à l'autre.

### 12.1 Exemple : `geolocalisation.js`

```javascript
// geolocalisation.js
// Encapsule toute la logique de géolocalisation dans une fonction réutilisable.

function activer_geolocalisation(map, info){

    var geolocation = new ol.Geolocation({
        projection: map.getView().getProjection(),
        tracking: false
    });

    var point_position = new ol.Feature();
    point_position.setStyle(new ol.style.Style({
        image: new ol.style.Circle({
            radius: 6,
            fill: new ol.style.Fill({color: '#3399CC'}),
            stroke: new ol.style.Stroke({color: '#ffffff', width: 2})
        })
    }));

    var precisionzone = new ol.Feature();

    var couche_position = new ol.layer.Vector({
        source: new ol.source.Vector({
            features: [precisionzone, point_position]
        }),
        title: 'Ma position'
    });
    map.addLayer(couche_position);

    geolocation.on('change:accuracyGeometry', function(){
        precisionzone.setGeometry(geolocation.getAccuracyGeometry());
    });

    geolocation.on('change:position', function(){
        var coordinates = geolocation.getPosition();
        if (coordinates){
            point_position.setGeometry(new ol.geom.Point(coordinates));
        }
    });

    geolocation.on('change', function(){
        map.getView().setCenter(geolocation.getPosition());
        map.getView().setZoom(15);
        geolocation.setTracking(false);
    });

    geolocation.on('error', function(error){
        console.error('Erreur de géolocalisation :', error.message);
        if (info){
            info.innerHTML = 'Erreur de géolocalisation : ' + error.message;
        }
    });

    return geolocation;
}
```

### 12.2 Utilisation dans la page HTML

```html
<script src="https://cdn.jsdelivr.net/npm/ol@v10.10.0/dist/ol.js"></script>
<script src="geolocalisation.js"></script>
```

```javascript
// Après la création de la carte :
var geolocation = activer_geolocalisation(map, info);

document.getElementById('bouton_geoloc').addEventListener('click', function(){
    geolocation.setTracking(true);
});
```

**Avantage** : ce fichier peut être copié tel quel dans n'importe quelle autre page contenant une carte OpenLayers, sans dupliquer le code.

---

## 13. Pièges rencontrés et comment les éviter

Cette section recense les erreurs effectivement rencontrées pendant la réalisation de ce projet — elles valent la peine d'être connues à l'avance.

### 13.1 Chemins relatifs incorrects

**Symptôme** : image cassée, fichier JS non chargé (`$ is not defined`, `ol is not defined`...).
**Cause** : chemin relatif écrit sans tenir compte de l'emplacement réel du fichier HTML qui l'utilise.
**Solution** : toujours vérifier, dans l'explorateur FTP, le dossier exact où se trouve le fichier HTML, et compter les niveaux à remonter (`../`) pour atteindre la ressource visée.

### 13.2 Bouton submit qui recharge la page

**Symptôme** : erreur `Unsafe attempt to load URL file:///... 'file:' URLs are treated as unique security origins.`
**Cause** : un `<INPUT type="submit">` à l'intérieur d'un `<FORM>` déclenche un rechargement de page, ce qui est bloqué sur un fichier local (`file://`).
**Solution** : utiliser `type="button"` au lieu de `type="submit"` si le bouton ne doit pas soumettre de formulaire.

### 13.3 jQuery non chargé à cause d'un `integrity` invalide

**Symptôme** : rien ne se passe à l'interaction, sans erreur visible si on ne regarde pas la console.
**Cause** : l'attribut `integrity` (vérification de hash SRI) contient une valeur incorrecte ou un texte de remplacement — le navigateur bloque alors le script entièrement.
**Solution** : soit copier exactement la vraie valeur `integrity` fournie par jquery.com, soit retirer les attributs `integrity`/`crossorigin` si vous n'en avez pas l'usage.

### 13.4 Script exécuté avant que l'élément HTML n'existe

**Symptôme** : `$('#liste')` (ou `document.getElementById('liste')`) ne trouve rien, aucun événement ne se déclenche.
**Cause** : le script, placé dans le `<HEAD>`, s'exécute avant que le `<BODY>` ne soit chargé.
**Solution** : envelopper le code dans `$(document).ready(function(){ ... });`, ou déplacer le `<SCRIPT>` juste avant `</BODY>`.

### 13.5 Modifications FTP qui ne remontent pas

Voir section 2.3 — reconnecter la session FTP dans Notepad++, vérifier la date de modification du fichier côté serveur, tester en navigation privée.

### 13.6 Sécuriser le fichier PHP avant publication sur GitHub

Le fichier `postgis_geojson_*.php` contient des **identifiants de connexion à la base de données**. Avant de le publier sur un dépôt public :
- Sortez les identifiants dans un fichier de configuration **non versionné** (ajouté à `.gitignore`), par exemple `config.php` :

```php
<?php
// config.php — NE PAS VERSIONNER CE FICHIER
$db_host = 'votre_hote';
$db_name = 'votre_base';
$db_user = 'votre_utilisateur';
$db_pass = 'votre_mot_de_passe';
```

```php
<?php
// postgis_geojson.php
require_once('config.php');
// ... utilise $db_host, $db_name, $db_user, $db_pass pour se connecter
```

- Ajoutez `config.php` à votre `.gitignore` :

```
config.php
```

- Vérifiez également que le nom de table reçu en paramètre (`?geotable=...`) est **validé** côté PHP contre une liste blanche de tables autorisées, pour éviter toute injection SQL.

---

## 14. Structure finale du projet

```
site_temporel/
├── images/
│   └── (images statiques éventuelles)
├── ol/
│   ├── carte_web_cpgeom.html          # page principale de la carte
│   ├── carte_commune_cpgeom.html      # variante avec géolocalisation
│   ├── geolocalisation.js             # module de géolocalisation réutilisable
│   ├── ol-layerswitcher.js            # sélecteur de couches (bibliothèque externe)
│   ├── ol-layerswitcher.css
│   ├── postgis_geojson_....php        # pont PHP entre OpenLayers et PostGIS
│   ├── config.php                     # identifiants de connexion (NON versionné)
│   └── data/
```

### Exemple de page complète, combinant toutes les briques du tutoriel

```html
<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Carte EPCI</title>
<script src="https://cdn.jsdelivr.net/npm/ol@v10.10.0/dist/ol.js"></script>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/ol@v10.10.0/ol.css">
<script src="ol-layerswitcher.js"></script>
<link rel="stylesheet" href="ol-layerswitcher.css">
<script src="geolocalisation.js"></script>
<style>
html,body{ margin:0; padding:0; width:100%; height:100%; }
#carte{ width:100%; height:100%; background-color:#eceae5; }
#info{
    position:absolute; bottom:20px; left:20px; z-index:1000;
    background:white; padding:10px 14px; box-shadow:1px 1px 12px #555;
    border-radius:4px; font-family:Arial,sans-serif; font-size:14px;
    max-width:300px;
}
#bouton_geoloc{
    position:absolute; top:20px; left:20px; z-index:1000;
    background:white; padding:8px 14px; box-shadow:1px 1px 12px #555;
    border-radius:4px; border:none; cursor:pointer;
}
</style>
</head>
<body>
<div id="carte"></div>
<div id="info">Chargement...</div>
<button id="bouton_geoloc">📍 Me géolocaliser</button>
<script>
// --- Fonds de carte ---
var couche_osm = new ol.layer.Tile({
    source: new ol.source.OSM(), title:'Fond OSM', type:'base'
});
var couche_esri = new ol.layer.Tile({
    source: new ol.source.XYZ({
        url:'https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}',
        attributions:'© Esri'
    }),
    title:'Fond Esri', type:'base'
});

// --- Données PostGIS ---
var donnees_epci = new ol.source.Vector({
    url: 'postgis_geojson_abdoulahat_alwaysdata.php?geotable=epci_chalons',
    format: new ol.format.GeoJSON({
        dataProjection:'EPSG:4326',
        featureProjection:'EPSG:3857'
    })
});

function style_epci(feature){
    return new ol.style.Style({
        stroke: new ol.style.Stroke({color:'#333333', width:1.5}),
        fill: new ol.style.Fill({color:'#41b6c480'})
    });
}

var couche_epci = new ol.layer.Vector({
    source: donnees_epci, style: style_epci,
    title:'EPCI', name:'epci_chalons'
});

// --- Carte ---
var map = new ol.Map({
    target: 'carte',
    layers: [couche_osm, couche_esri, couche_epci],
    view: new ol.View({ center:[263652,5919266], zoom:6 })
});

// --- Zoom auto sur les données ---
donnees_epci.on('featuresloadend', function(){
    map.getView().fit(donnees_epci.getExtent(), {padding:[50,50,50,50], maxZoom:14});
    document.getElementById('info').innerHTML = 'Cliquez sur une entité';
});

// --- Contrôles ---
map.addControl(new ol.control.ScaleLine({minWidth:64, units:'metric'}));
map.addControl(new ol.control.LayerSwitcher({}));

// --- Sélection + infobulle ---
var info = document.getElementById('info');
var popup = new ol.Overlay({element:info, offset:[0,-15], positioning:'bottom-center', stopEvent:false});
map.addOverlay(popup);

map.on('singleclick', function(evt){
    var feature = map.forEachFeatureAtPixel(evt.pixel, function(f){ return f; }, {
        layerFilter: function(l){ return l === couche_epci; }
    });
    if (feature){
        info.innerHTML = 'Nom : ' + feature.get('nom_dep');
        popup.setPosition(evt.coordinate);
    } else {
        popup.setPosition(undefined);
    }
});

// --- Géolocalisation (module externe) ---
var geolocation = activer_geolocalisation(map, info);
document.getElementById('bouton_geoloc').addEventListener('click', function(){
    geolocation.setTracking(true);
});
</script>
</body>
</html>
```

---

## Ressources

- [Documentation officielle OpenLayers](https://openlayers.org/en/latest/apidoc/)
- [ol-layerswitcher (GitHub)](https://github.com/walkermatt/ol-layerswitcher)
- [jQuery](https://jquery.com/)
- [PostGIS](https://postgis.net/)
- [AlwaysData](https://www.alwaysdata.com/)
- [MDN — API Geolocation](https://developer.mozilla.org/fr/docs/Web/API/Geolocation_API)

---

*Tutoriel rédigé à partir d'un projet pédagogique réel, incluant les erreurs rencontrées et leurs résolutions.*
