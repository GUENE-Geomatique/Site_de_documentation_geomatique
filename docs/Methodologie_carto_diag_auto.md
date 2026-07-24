
# Méthodologie d’automatisation de cartes thématiques

<img width="2500" height="1474" alt="image" src="https://github.com/user-attachments/assets/7b97e1df-297a-41c7-96bb-6fbdf795d66e" />

---
[🔗  Documentation QGIS](https://docs.qgis.org/3.40/en/docs/about/preamble.html)  

## Introduction

La simplification de ce travail commence par l'application d'un ***filtre sur les couches*** afin de ne conserver que les territoires qui nous intéressent. Cela permet d’éviter d’être perturbé par l’affichage de nombreuses données, comme c’est le cas pour les communes. 
Il est important de noter que les couches utilisées pour l’automatisation proviennent de **scripts SQL** exécutés directement sur la base de données, et non de couches stockées de manière permanente dans la base. 
Cependant, certaines exceptions existent, notamment pour les **rasters** et les **couches d’étiquettes**, qui peuvent être enregistrées séparément pour des raisons de performance et de gestion des données. 

## 1\. Automatisation de la couche de couverture

Certaines couches doivent être paramétrées afin de permettre l’automatisation de l’atlas. 
La **couche de génération de l’atlas** constitue la base du projet. Elle définit les différentes entités à cartographier et sert de référence pour la production des cartes. Grâce à une symbologie basée sur un ensemble de règles, chaque EPCI (Établissement Public de Coopération Intercommunale) peut être représenté individuellement tout en conservant une mise en page homogène. 

⚠️ **Attention :** 

Il est essentiel de choisir un champ unique pour identifier chaque entité et éviter les doublons. Par exemple, on peut utiliser le champ ***lib*** **(libellé)** ou ***code\_epci*** **(code EPCI)**. 

<img width="1386" height="908" alt="image" src="https://github.com/user-attachments/assets/2b1fcfe7-f535-498d-86fe-ccfdbb8175c8" />

**Figure 1 :** Configuration de la couche principale de l'atlas 

**Utilisation des expressions QGIS** 

● Les **expressions QGIS** sont souvent utilisées pour filtrer ou ajuster l'affichage des éléments en fonction de l'entité active dans l'atlas. 

● Certaines permettent d'afficher uniquement les éléments correspondant à la page en cours lors de la génération des cartes. 
Cependant, la génération effective de l’atlas s’effectue uniquement dans la mise en page QGIS, où l’ensemble des paramètres définis prend effet. 

## 2\. Paramétrage des autres couches

### 2.1 . Les couches de points

### 2.1.1. Avec @atlas\_pagename

Si l’on souhaite afficher des couches de points (par exemple : gares TGV, arrêts bus), il est nécessaire de paramétrer leur symbologie à l’aide d’un **ensemble de règles**. Cela permet de restreindre leur affichage à l’emprise de l’entité de l’atlas en cours d’affichage. 
Le processus de paramétrage est similaire à celui de la couche de génération de l’atlas (partie 1), mais appliqué spécifiquement à chaque couche concernée.

<img width="1385" height="906" alt="image" src="https://github.com/user-attachments/assets/5b8efa09-bbb9-43be-a321-c16b80a6d332" />

**Figure 2 :** Configuration des couches à automatiser 

### 2.1.2. Avec intersects($geometry, @atlas\_geometry)

L'expression intersects($geometry, @atlas\_geometry) ou un autre prédicat spatial tel que within, disjoint, etc. permet de contrôler l’affichage des couches de points, de lignes et de surfaces en fonction de la géométrie de l’entité de l’atlas actif. 
Cet outil est particulièrement utile lorsqu'on souhaite filtrer spatialement les entités, indépendamment de leurs attributs. Il peut être appliqué dans des symbologies catégorisées ou graduées, ce qui permet d'affiner l'affichage des éléments selon leur relation géographique avec l’entité active. 
Cependant, pour que ce filtrage fonctionne dans le cadre de symbologies catégorisées ou graduées, il est nécessaire de passer à une symbologie basée sur un **ensemble de règles** après avoir effectué la catégorisation ou la graduation. Cela permet d’appliquer des critères spatiaux tout en conservant une gestion de la symbologie dynamique et détaillée. 
La **Figure 3** illustre le cas où seuls les types d’**aérodromes** de l’EPCI actif sont affichés. 

En revanche, la **Figure 4** montre un cas où **toutes les catégories surfaciques** sont masquées en dehors de l’entité active de l’atlas. 

<img width="1390" height="554" alt="image" src="https://github.com/user-attachments/assets/00a730ec-55f2-4285-b913-3fbadf882240" />

**Figure 3 :** Exemple des couches de points catégorisées 

<img width="1382" height="458" alt="image" src="https://github.com/user-attachments/assets/e64f4c22-f9f4-45ec-a0a9-43088524e9b1" />

**Figure 4 :** Exemple des couches surfaciques classées par catégorie 

Il est également possible d'utiliser ces expressions pour gérer l'affichage des cercles proportionnels et des étiquettes en fonction de l'atlas. Ces réglages s'effectuent dans la section "***Rendu***" des diagrammes pour les cercles proportionnels, et dans la section "Étiquettes" pour les étiquettes. 
**La figure 5** illustre un exemple de positionnement des cercles proportionnels à l'intérieur de l'emprise définie par l'atlas. 

<img width="1296" height="447" alt="image" src="https://github.com/user-attachments/assets/e7b07f74-81d4-4bbd-8c21-98112d701b6a" />

**Figure 5 :** Affichage des cercles proportionnels (type camembert) 

**Différence entre** intersects($geometry, @atlas\_geometry) et @atlas\_pagename 

**\- @atlas\_pagename** 

**Principe** : Il s'agit d'une variable qui stocke la valeur du champ défini comme "nom" de l'atlas (souvent un identifiant ou un libellé). 

**Utilisation** : Permet de filtrer des couches en se basant sur une **correspondance attributaire** entre les données et la page active de l'atlas. 

**Exemple** :  

Si la couche de génération de l’atlas contient un champ nom\_epci, on peut filtrer une couche de gares en utilisant : ***"nom\_epci" \= @atlas\_pagename*** 

**\- intersects($geometry, @atlas\_geometry)** 

**Principe** : Vérifie si la **géométrie** d'une entité intersecte la géométrie de l’entité active dans l’atlas. 

**Utilisation** : Permet de filtrer les entités **géographiquement**, sans avoir besoin d’un champ commun. 

**Exemple** :

Si l’atlas est basé sur une couche d’EPCI et qu'on veut afficher uniquement les gares situées à l’intérieur de l’EPCI actif : ***intersects($geometry, @atlas\_geometry) ou within($geometry, @atlas\_geometry)*** 

| Critère  | @atlas\_pagename | intersects($geometry,  @atlas\_geometry) |
| ----- | ----- | ----- |
| Basé sur  | Valeur attributaire  | Relation spatiale |
| Nécessite un champ commun  | Oui  | Non |
| Fonctionne avec | Données ayant un  identifiant partagé | Toutes les données  spatiales |
| Performance  | Très rapide  | Plus gourmand en calcul |
| Exemple d’usage  | Filtrer par nom d’epci | Filtrer par intersection géométrique |

**Figure 6 :** Tableau de comparaison synthétique 

## 3\. Visualisation des cartes

### 3.1. Carte à plat

Les cartes à plat sont plus faciles à gérer et leurs ajustements posent moins de soucis. Lorsqu'il s'agit de symbologies graduées ou catégorisées,  
l'affichage des éléments dans l’atlas actif est géré de la même manière que décrite dans la section **2.1.2**. 

### 3.2. Carte avec cercles proportionnels (camemberts) 

La gestion de ce type de carte peut être complexe (***cf partie problématique***), car elle nécessite le réglage de plusieurs paramètres, notamment la taille et la position des cercles. La variabilité des tailles est directement liée aux données (valeurs minimales et maximales). 
Cependant, il est possible d’ajuster la taille globale des cercles afin d’optimiser leur affichage et de mieux les adapter aux entités spatiales représentées (figure 7). 

<img width="1382" height="324" alt="image" src="https://github.com/user-attachments/assets/2c633425-a5d2-468a-ac1f-206e061b1f73" />

**Figure 7 :** Échelle de visualisation des diagrammes en camembert 

## 4\. Mise en page et génération de l’atlas

La première étape consiste à réaliser une **mise en page classique**, comme pour une carte sans atlas, en intégrant tous les **éléments cartographiques nécessaires** selon la demande reçue. 
Une fois tous ces éléments définis dans la mise en page, on peut passer à la **génération de l’atlas** en cliquant sur le bouton **paramètres de l’atlas** dans la barre d’outils. <img width="130" height="39" alt="image" src="https://github.com/user-attachments/assets/8b2befb1-6267-4c47-bfb7-4e24d22941e8" />

Cela ouvre une fenêtre dans laquelle il faut renseigner la **couche de couverture**, définie plus haut (**partie 1**), qui contient les différentes entités à cartographier (EPCI). Cette couche sert de référence pour l’automatisation de l’atlas et permet de générer les cartes de manière dynamique. 

<img width="382" height="466" alt="image" src="https://github.com/user-attachments/assets/22a0a353-e595-4f03-a762-6c1e719334de" />

**Figure 8 :** Paramétrage et génération de l'atlas 

Le champ **« lib »** est choisi pour identifier et individualiser les entités de l’atlas. Il permet également d’afficher automatiquement le libellé en haut de chaque page, facilitant ainsi la lecture et la compréhension des cartes. <img width="98" height="29" alt="image" src="https://github.com/user-attachments/assets/b9eeec15-ae36-4be4-83d6-1b4f5f5adfb4" />
Néanmoins, il est aussi possible de choisir le **code des EPCI** comme identifiant unique. Avec ces paramètres définis, nous disposons de toutes les informations nécessaires pour lancer l’atlas. 

## 5\. Lancement de l’atlas

L’atlas se lance à l’aide du bouton **aperçu de l’atlas,** disponible dans la barre d’outils.  <img width="41" height="32" alt="image" src="https://github.com/user-attachments/assets/6709c7d0-0eea-437c-b11b-242d9dcb268f" />

Toutefois, au premier affichage, la carte peut sembler **fixe**, même si l’atlas est activé. Cela est dû au fait que l’emprise cartographique n’est pas encore contrôlée par l’atlas. 
Pour afficher correctement toutes les entités, il est nécessaire d’activer cette option dans les **propriétés de l’emprise de la carte**. Il suffit alors de cocher l’option permettant de **lier l’emprise à l’atlas.** Une fois cette étape réalisée, les cartes de l’atlas s'afficheront dynamiquement, et il sera possible de naviguer entre les entités à l’aide des **boutons de défilement**. <img width="141" height="21" alt="image" src="https://github.com/user-attachments/assets/9e6c1133-5285-47c4-8154-94f22f0c8bb3" />

<img width="716" height="46" alt="image" src="https://github.com/user-attachments/assets/bf0ff271-3093-4375-aa59-f9c958274db2" />

**Figure 9 :** Affichage des cartes générées automatiquement de l'atlas 

**\- Titre** 

Le **titre de la carte** peut être automatisé en fonction de l’atlas, à condition que cette information soit disponible dans un champ de la table de **génération de l’atlas** (voir Partie 1). 

Par exemple, si nous souhaitons faire apparaître le nom de l’EPCI dans le titre de chaque carte, nous pouvons récupérer cette information depuis la couche de l’atlas en utilisant le champ **lib**. Ainsi, le titre reste constant, mais le nom du territoire varie automatiquement en fonction de l’atlas actif. 

<img width="1637" height="351" alt="image" src="https://github.com/user-attachments/assets/822dfa5b-67bf-4fe2-b344-245e5d1dd986" />

**Figure 10 :** Automatisation du titre de la carte en fonction de l'atlas actif 

**\- Légende** 

L’automatisation de la **légende** semble relativement simple et ne nécessite pas d'expressions QGIS complexes. Il suffit de cocher les cases **optionnelles** pour restreindre l’apparence des éléments en fonction des besoins cartographiques. 
Il est **idéalement recommandé** de désactiver l'option **« mise à jour automatique »** pour pouvoir ajuster manuellement les éléments de la légende (par exemple, regroupement, ajout, retrait, renommage, etc.). 
De plus, il est aussi possible de restreindre l’affichage des éléments de la légende à **l’emprise de la carte**, ou encore de l'afficher uniquement pour l'**entité active** de l'atlas (figure 11).  

<img width="517" height="410" alt="image" src="https://github.com/user-attachments/assets/405d10bc-17da-436e-8667-6dc1578fde91" />

**Figure 11 :** Automatisation de l’affichage des légendes 

**\- Échelle** 

L’automatisation de l’**échelle** s’avère moins complexe, notamment grâce à l’option d’**ajustement automatique** en fonction de la largeur du segment. 
Cependant, il est essentiel de **contrôler la taille de la barre d’échelle** afin d’éviter qu’elle ne **dépasse sur la carte**, ce qui pourrait nuire à la lisibilité. 
La **figure 12** illustre un exemple d’**échelle dynamique**, configurée avec une longueur de **20 mm** et une seule **subdivision**. 

<img width="472" height="172" alt="image" src="https://github.com/user-attachments/assets/5db5ae68-53a2-4f7b-805e-3ac4f6be0dc4" />


**Figure 12 :** Automatisation de l'échelle de la carte en fonction de l'atlas actif 

**\- Date de la carte** 

Étant donné que cette carte sera mise à jour en fonction des demandes des EPCI, il est important d’automatiser l’affichage de la date de réalisation. Cela permet de gagner du temps et d’éviter d’éventuels oublis. 

<img width="368" height="269" alt="image" src="https://github.com/user-attachments/assets/828248a5-d294-4bcd-b140-8d7f2170e426" />

**Figure 13 :** Adaptation des échelles cartographiques 

**\- Logo des EPCI** 

Afficher automatiquement le logo correspondant à l’EPCI actif sur chaque page de l’atlas dans QGIS, en fonction du nom ou du code de l’EPCI. 

**Données utilisées** 

● **Couche de couverture de l’atlas** : atlas\_EPCI, avec la colonne code (ou lib) correspondant aux EPCI. 

● **Fichier Excel de correspondance** : test.xlsx, contenant trois colonnes : 

- code : identifiant de l’EPCI (correspondant à la couche atlas), 

- lib : nom de l’EPCI (facultatif si on utilise le code), 

- chemin : chemin absolu du logo image correspondant (format PNG, JPG...). 

**Principe de la méthode** 

1\. **Lier dynamiquement le logo** au paramètre actif de l’atlas (EPCI).

2\. Utiliser une **jointure virtuelle** dans QGIS entre la couche de couverture (atlas\_EPCI) et le fichier Excel (test.xlsx) via le champ code. 

3\. Ajouter une **image dynamique** dans la mise en page de l’atlas, en utilisant une **expression QGIS** pour lire le chemin d’image stocké dans le fichier Excel. 

**Étapes de mise en œuvre** 

**1\.** **Charger le fichier Excel dans QGIS** 

● Menu : *Layer \> Add Layer \> Add Layer from Excel* (test.xlsx). 

● Vérifie que les champs code, lib et chemin sont bien reconnus.

**2\.** **Joindre le fichier Excel à la couche atlas** 

● Dans les **propriétés de la couche atlas\_EPCI**, onglet *Jointures* : 

- Couche cible : test.xlsx,  

- Champ commun : code, 

- Active la jointure. 

**3\.** ️ **Ajouter le logo dans la mise en page** 

**\-** Ouvre la mise en page de ton atlas. 

**\-** Insère une **image** (Ajouter une image). 

**\-** Dans l’onglet **propriétés de l’image** : 

- Coche : **"Charger l’image depuis une expression"**. 

- Mets cette expression qui correspond au nom de la colonne jointe dans la couche de couverture de l’atlas : 
"test-Feuil1\_chemin" 

<img width="388" height="414" alt="image" src="https://github.com/user-attachments/assets/9ff42072-c254-4668-9274-1417fc16206c" />

<img width="378" height="160" alt="image" src="https://github.com/user-attachments/assets/342101ab-500b-45e2-b6a8-c9561d35fbbe" />

**Figure 14 :** Affichage dynamique du logo EPCI dans l’atlas à partir d’un fichier Excel 

## 6\. Problématique liée à l’automatisation (atlas) 

Tout d'abord, il est **idéal** d’afficher les étiquettes uniquement pour les communes de l’EPCI actif **dan**s l’atlas. Pour cela, il est nécessaire de définir un **étiquetage basé sur des règles**, en appliquant le filtre suivant : ***within ($geometry, @atlas\_geometry).***  

<img width="1268" height="245" alt="image" src="https://github.com/user-attachments/assets/26942f4a-c059-4b87-915f-5cd9b8d2abd6" />

**Figure 15 :** Filtrage des étiquettes selon l'atlas actif 

Cependant, les étiquettes s'affichent souvent **dans des positions non optimales**, compromettant la lisibilité de la carte, même après ajustement classique des paramètres d’étiquetage (figure 14). 

<img width="925" height="241" alt="image" src="https://github.com/user-attachments/assets/d8c6ad46-dc9d-4f12-9baa-2bec644bf2cc" />

**Figure 16 :** La position des étiquettes sur la carte 

### 6.1. Méthode pour un positionnement optimal des étiquettes

Afin d’optimiser la position des étiquettes, il serait nécessaire de **créer une couche de points** associée aux entités (ici, chaque commune dans un EPCI). 

**\- Création de la couche de points** 

Cette couche de points doit contenir : 

**\-** Un **champ d’identifiant unique (insee\_com)**, permettant une **jointure attributaire** avec la couche d’étiquettes. 

**\-** Deux **champs de coordonnées** :  

- coordonnées\_X (longitude)

- coordonnées\_Y (latitude) 
Ces valeurs sont obtenues grâce aux expressions suivantes dans la **calculatrice de champs** : 

***$x*** pour la coordonnée X 

***$y*** pour la coordonnée Y 

⚠️ **Attention** : Avant d’effectuer ces calculs, il est indispensable de **digitaliser les points**, de renseigner le champ **insee\_com** et de **positionner manuellement** chaque point à l’emplacement souhaité pour l’étiquette. 

**\- Jointure attributaire avec la couche d’étiquettes** 

Une fois la couche de points créée, une **jointure attributaire** est réalisée afin d’intégrer les champs coordonnées\_X et coordonnées\_Y dans la couche d’étiquettes. 
Grâce à ces coordonnées, les étiquettes des communes pourront être **positionnées précisément** à l’endroit des points définis. 

**\- Paramétrage de l’étiquetage** 

Sur la couche d’étiquettes, il est désormais possible de **définir le positionnement** en fonction des coordonnées des points obtenus via la jointure.  

**Astuce** : 

Il est important de cocher l’option **« Stocker les données dans le projet »** pour éviter des modifications non souhaitées si ces mêmes couches sont utilisées dans d’autres projets. 

<img width="810" height="320" alt="image" src="https://github.com/user-attachments/assets/214dab45-9f96-446b-a1db-5fd76cd9be15" />

**Figure 17 :** Optimisation de la position des étiquettes sur la carte 

Ensuite, pour que l’étiquette de chaque commune se positionne correctement en fonction des coordonnées du point associé, il est nécessaire de **lier les champs de coordonnées** aux champs correspondants dans la couche d'étiquette (voir figure 15). Cela permettra d’ajuster automatiquement la position de chaque étiquette selon les points définis. 

### 6.2. Méthode pour un positionnement optimal des cercles proportionnels

Le même processus utilisé pour le réglage des **étiquettes** s’applique également aux **cercles proportionnels**. 
Toutefois, dans ce cas, les ajustements de **positionnement** doivent être effectués directement dans les paramétrages **des diagrammes** (voir figure 16). Il est essentiel de définir correctement les coordonnées des cercles pour garantir une représentation claire et lisible sur la carte.  

<img width="998" height="635" alt="image" src="https://github.com/user-attachments/assets/83d7e51a-3d5c-4f30-814d-01c9a080630c" />

**Figure 18 :** Optimisation de la position des cercles proportionnels sur la carte 

## 7\. Avantages et mise à jour cartographique

Le fait que les données soient directement liées à la **base de données** permet une **mise à jour automatique** des cartes en fonction des nouvelles données disponibles. Ainsi, cette cartographie automatisée constitue un **socle dynamique et pérenne** pour l’actualisation des cartes, garantissant une cohérence et une fiabilité des informations pour toutes les entités concernées. 
Cependant, cette automatisation présente certaines **limites**, notamment pour la gestion des **étiquettes**, des **cartes graduées** et des **cercles proportionnels**, qui nécessitent parfois des ajustements manuels pour un affichage optimal.  

## 8 \. Les limites

La **pertinence cartographique** de certaines thématiques nécessite parfois de **dupliquer le projet QGIS** afin d’apporter des **modifications spécifiques**. Cela permet d’**optimiser la visualisation** tout en se concentrant sur un **territoire précis** (EPCI) parmi l’ensemble des entités cartographiées dans l’atlas. 
**\- Étiquettes** 
La gestion des **étiquettes** est particulièrement **complexe**, notamment dans le cas des **cartes avec cercles proportionnels**. Un problème récurrent est le **chevauchement des étiquettes**, surtout lorsque les entités concernées (communes, quartiers) sont de **petite taille**. 
De plus, l’utilisation d’une **couche spécifique pour positionner les étiquettes** peut s’avérer **fastidieuse**, en particulier lorsque l’atlas couvre un **grand nombre d’entités.** 

**\- Cartes à symbologie graduée** 

L’**échelle de répartition des classes** peut ne pas être adaptée à toutes les cartes de l’atlas, notamment en raison de **grands écarts entre les valeurs maximales et minimales**. Cela est souvent dû à la présence de **grandes villes**, comme **Perpignan** dans notre cas, qui influencent fortement la classification. 
Ainsi, il est **nécessaire d’ajuster les classes** en fonction du territoire étudié. Par exemple, pour l’**EPCI de Pyrénées Cerdagne**, il peut être pertinent de **revoir la classification** afin qu’elle soit plus représentative des données locales. 
Pour cela, il est recommandé **d’appliquer un filtre** en amont sur toutes les couches concernées. Cette méthode est **particulièrement efficace**, car elle **limite l’affichage des données** tout en **préservant l’ensemble des informations dans la même couche**. 
**\>** 

<img width="1002" height="524" alt="image" src="https://github.com/user-attachments/assets/08e9fc41-d8f1-4109-b194-bad957fb97df" />

<img width="1002" height="524" alt="image" src="https://github.com/user-attachments/assets/f4a8d4e1-cc13-4554-9359-1fb1da1f2048" />

**Figure 19 :** Filtrage des données selon le territoire d'intérêt 

**\- Cercles proportionnels** 

Comme mentionné précédemment, **l’échelle d’affichage des cercles proportionnels** dépend directement des **valeurs minimales et maximales** du champ représenté. Cependant, la présence de **grandes villes** peut entraîner des **valeurs extrêmes**, ce qui fausse l’échelle et réduit fortement la **taille des cercles** dans les territoires moins peuplés (population). 
Pour **corriger ce déséquilibre**, il est recommandé de **dupliquer le projet QGIS** et d’appliquer un **filtre sur le territoire d’intérêt**. Cela permet d’**ajuster les échelles de visualisation** spécifiquement en fonction des valeurs propres à ce territoire, sans être influencé par les données des autres zones couvertes par l’atlas.  
Il est également obligatoire de revoir la légende des cercles proportionnels en cas de modification des valeurs extrêmes, afin de mettre à jour leur libellé et d'assurer une représentation fidèle des données. 

## 9 \. Cas spécifique: atlas des flux pendulaire

Dans le cas des flux pendulaires, il est nécessaire de créer une nouvelle colonne à partir du champ "code" de la couche "Flux entrants". Voici les étapes à suivre dans QGIS : 

**Étapes dans QGIS** 

**Créer une nouvelle colonne `code_racine` (facultatif mais recommandé)** Dans la couche principale de ton atlas : 
- Ouvre la calculatrice de champ. 
- Créer un nouveau champ appelé code\_racine (type texte). ○ Utilise l'expression suivante pour extraire la partie avant le tiret du champ code : 
regexp\_substr("code", '^\[^-\]+') 

<img width="1287" height="244" alt="image" src="https://github.com/user-attachments/assets/87fc0d84-9554-4832-9941-585b17462b4c" />

**Figure 20 :** Création de la colonne "code\_racine" pour regrouper les flux pendulaires 

**On pouvait aussi directement utiliser la colonne code\_epcilt.** 

**Activer l'atlas sur cette couche** 

Avant d'activer l'atlas, il est nécessaire de créer une couche regroupée à partir de la couche "Flux entrants" en utilisant la colonne `code_racine` afin d'éviter la répétition des occurrences. Ensuite, applique le filtre de l'atlas sur cette couche.  

<img width="1243" height="487" alt="image" src="https://github.com/user-attachments/assets/7aa4448f-1c73-4b5f-9ad2-26e315262bee" />

**Figure 21 :** Activation de l’atlas à partir d’une couche regroupée par EPCI 

● Dans l'onglet Atlas, cocher Générer un atlas. 

● Sélectionne ta couche de couverture (couche regroupée) comme couche principale.

● Dans Champ d’identifiant de page, choisis le champ code\_racin. 

**Filtrer les autres couches (dans les propriétés de chaque couche)** 

Pour appliquer un filtre basé sur code\_racine dans les autres couches (Flux\_entrants, Flux\_sortants et Flux\_internes) : 

● Va dans les Propriétés de chaque couche, dans l'onglet Filtre ou Rendu conditionnel. 

● Utilise l'expression suivante pour que chaque entité de la couche corresponde à l'atlas généré : 

regexp\_substr("code", '^\[^-\]+') \= @atlas\_pagename 

**\- Flèche du déplacement pendulaire** 

La taille des flèches est proportionnelle aux valeurs de la colonne concernée. La couche vectorielle est composée de lignes représentant les déplacements entre les points de départ (source) et d’arrivée (destination). 

**NB :** 

Les lignes peuvent être créées avec ***RStudio*** ou ***Python*** à partir d’une couche polygonale des communes contenant tous les champs nécessaires (flux, code\_insee, code\_insee\_destination, etc.). Dans ce cas, l’algorithme génère les lignes en reliant les centroïdes des communes, en utilisant code\_insee comme commune source et code\_insee\_destination comme commune de destination du flux. 

<img width="1456" height="612" alt="image" src="https://github.com/user-attachments/assets/31e29172-2e69-4352-af0e-02a1b2880173" />

**Figure 22 :** Représentation des flux pendulaires par flèches proportionnelles 

Nous avons besoin d’une **couche de couverture spécifique pour l’atlas**, créée à partir d’un **regroupement par la colonne code\_epci** de la couche de lignes représentant les déplacements. Ce regroupement est nécessaire pour **éliminer les doublons** et ainsi **individualiser chaque EPCI** dans l’atlas. 
Cette couche de couverture, utilisée uniquement pour définir les territoires (EPCI), est **stockée localement** sur l’ordinateur et **n’est pas générée automatiquement**. 
En revanche, la couche contenant les **données de déplacement**, issue d’un fichier **SQL, Excel ou CSV**, reste **automatisée** : elle est **liée par jointure** à la couche de couverture via la colonne code\_commune. 

<img width="831" height="289" alt="image" src="https://github.com/user-attachments/assets/367de7ab-10eb-428e-86a6-7fc28508801c" />

**Figure 23 :** Création d’une couche de couverture pour individualiser les EPCI dans l’atlas

Comme pour les autres atlas, **un filtrage est appliqué sur cette couche pour n’afficher que l’entité active**, c’est-à-dire l’EPCI en cours de génération dans l’atlas. 

<img width="454" height="86" alt="image" src="https://github.com/user-attachments/assets/ba20586e-29bb-4ed4-9fb0-df05d7fe9e76" />

**Figure 24 :** Filtrage de l’entité active lors de la génération de l’atlas 

On peut également appliquer une symbologie avancée en faisant **varier la taille des symboles** grâce à un symbole gradué.  

<img width="1296" height="465" alt="image" src="https://github.com/user-attachments/assets/19031368-c42c-4377-8a18-97dee1400d9b" />

**Figure 25 :** Application d’une symbologie avancée avec symboles gradués 

Il est possible d’attribuer une **taille de flèche** spécifique à chaque classe. 

<img width="1425" height="509" alt="image" src="https://github.com/user-attachments/assets/04a75ec8-0ca4-4fd4-abf9-bf6583f174d3" />

**Figure 26 :** Attribution de tailles de flèches spécifiques par classe 

N’oublie pas ensuite de basculer vers une symbologie par ensemble de règles afin d’ajuster précisément l’atlas. 

<img width="1315" height="207" alt="image" src="https://github.com/user-attachments/assets/d81e5776-c4c3-4b5a-a320-dc763a6c1768" />

**Figure 27 :** Transition vers une symbologie par ensemble de règles pour un ajustement précis 

Le paramétrage de la taille des flèches ci-dessous ne sera plus nécessaire. 

<img width="1456" height="612" alt="image" src="https://github.com/user-attachments/assets/659cf235-250f-41cc-a9c4-fa1b2b623a30" />

**Figure 28 :** Suppression du paramétrage manuel de la taille des flèches 

Une fois sur la mise en page, la taille des flèches dans la légende ne correspond pas toujours à celle affichée sur la carte. Il est donc nécessaire d’ajuster manuellement la taille de chaque flèche dans la légende en personnalisant les valeurs de l’expression ci-dessous (même valeur lors de la symbolise).

<img width="554" height="301" alt="image" src="https://github.com/user-attachments/assets/80ba2343-1258-457b-a18f-c0acb3d23ddd" />

**Figure 29 :** Ajustement manuel de la taille des flèches dans la légende 

Veillez à utiliser les mêmes valeurs que celles définies dans la symbologie pour chaque flèche. 

<img width="564" height="498" alt="image" src="https://github.com/user-attachments/assets/804b515a-b2a2-4609-ac6c-6051160c4ac3" />

**Figure 30 :** Uniformisation des tailles des flèches entre symbole et légende 

**\- Flèche du déplacement pendulaire** 

La taille des flèches est proportionnelle aux valeurs de la colonne concernée. La couche vectorielle est composée de lignes représentant les déplacements entre les points de départ (source) et d’arrivée (destination).  

**NB :** Les lignes peuvent être créées avec ***RStudio*** ou ***Python*** à partir d’une couche polygonale des communes contenant tous les champs nécessaires (flux, code\_insee, code\_insee\_destination, etc.). Dans ce cas, l’algorithme génère les lignes en reliant les centroïdes des communes, en utilisant code\_insee comme commune source et code\_insee\_destination comme commune de destination du flux. 

<img width="1456" height="612" alt="image" src="https://github.com/user-attachments/assets/c2dfa26e-552d-406a-9e46-f180c4a98eb4" />

**Figure 31 :** Représentation des déplacements pendulaires par flèches proportionnelles 

Nous avons besoin d’une **couche de couverture spécifique pour l’atlas**, créée à partir d’un **regroupement par la colonne code\_epci** de la couche de lignes représentant les déplacements. Ce regroupement est nécessaire pour **éliminer les doublons** et ainsi **individualiser chaque EPCI** dans l’atlas. 
Cette couche de couverture, utilisée uniquement pour définir les territoires (EPCI), est **stockée localement** sur l’ordinateur et **n’est pas générée automatiquement**. 
En revanche, la couche contenant les **données de déplacement**, issue d’un fichier **SQL, Excel ou CSV**, reste **automatisée** : elle est **liée par jointure** à la couche de couverture via la colonne code\_commune. 

<img width="831" height="289" alt="image" src="https://github.com/user-attachments/assets/bed54071-3148-4240-a5da-a4f466b3bcf6" />

**Figure 32 :** Création et gestion d’une couche de couverture spécifique pour l’atlas des EPCI 

Comme pour les autres atlas, **un filtrage est appliqué sur cette couche pour n’afficher que l’entité active**, c’est-à-dire l’EPCI en cours de génération dans l’atlas. 

<img width="454" height="86" alt="image" src="https://github.com/user-attachments/assets/8a7c66eb-710c-4667-b4f9-dda092f747e4" />

**Figure 33 :** Filtrage de la couche pour afficher l’entité active dans l’atlas 

On peut également appliquer une symbologie avancée en faisant **varier la taille des symboles** grâce à un symbole gradué.  

<img width="1296" height="465" alt="image" src="https://github.com/user-attachments/assets/848a03e9-c526-4f46-8578-aa73c2f5ea79" />

**Figure 34 :** Application d’une symbologie graduée pour varier la taille des symboles 

Il est possible d’attribuer une **taille de flèche** spécifique à chaque classe. 

<img width="1425" height="509" alt="image" src="https://github.com/user-attachments/assets/41000c25-fdab-404e-8f63-450a56612044" />

**Figure 35 :** Attribution de tailles de flèches spécifiques par classe 

N’oublie pas ensuite de basculer vers une symbologie par ensemble de règles afin d’ajuster précisément l’atlas. 

<img width="1315" height="207" alt="image" src="https://github.com/user-attachments/assets/c54e49d5-db68-4ea7-b4dc-5d6f7895a7da" />

**Figure 36 :** Passage à une symbologie par règles pour un ajustement précis de l’atlas 

Le paramétrage de la taille des flèches ci-dessous ne sera plus nécessaire. 

<img width="1456" height="612" alt="image" src="https://github.com/user-attachments/assets/d3329ecb-0c7b-424e-b168-4cb27345c3d4" />

**Figure 37 :** Fin du paramétrage manuel de la taille des flèches 

Une fois sur la mise en page, la taille des flèches dans la légende ne correspond pas toujours à celle affichée sur la carte. Il est donc nécessaire d’ajuster manuellement la taille de chaque flèche dans la légende en personnalisant les valeurs de l’expression ci-dessous (même valeur lors de la symbolise). 

<img width="554" height="301" alt="image" src="https://github.com/user-attachments/assets/7fa74740-4761-4e28-b6d8-a516e1a0ac85" />

**Figure 38 :** Ajustement manuel des tailles des flèches dans la légende  

Veillez à utiliser les mêmes valeurs que celles définies dans la symbologie pour chaque flèche. 

<img width="564" height="498" alt="image" src="https://github.com/user-attachments/assets/6651e25b-cf44-45bc-8f36-fdf16ba204b6" />

**Figure 39 :** Correspondance des tailles des flèches entre symbole et légende 

## Workflow de génération des flux domicile–travail pour le pays de Châlons‑en‑Champagne
Il faut savoir que la couches de l'insee nommée est enregistrée déja dans la base de donnée: "flux_domicile_travail_2022"
Cette étape décrit les étapes et les requêtes SQL permettant de :
1. Extraire les flux sortants et entrants à partir du référentiel INSEE 2022 présent dans la base (flux_domicile_travail_2022).
2. Générer les couches géométriques linéaires (flows lines) grâce aux centroides des communes.
3. Utiliser ensuite ces flux dans QGIS pour produire des analyses par communes puis par EPCI.

### 1. Pré-requis

Les données suivantes doivent déjà être présentes dans la base PostgreSQL/PostGIS :
- bdtopo.flux_domicile_travail_2022 : flux INSEE domicile ↔ travail
- bdtopo.commune_adminexpr : communes avec géométries et codes INSEE
Les opérations suivantes créent plusieurs tables permettant de travailler sur les flux du pays de Châlons‑en‑Champagne.
La liste des codes INSEE considérés correspond aux communes du périmètre concerné.

### 2. Extraction des flux sortants (domicile → ailleurs)

```sql
CREATE TABLE bdtopo.domicile_travail_ou_pays_vers_ailleurs AS
SELECT *
FROM bdtopo.flux_domicile_travail_2022 AS f
WHERE f.codgeo IN (
'51003','51023','51031','51078','51087','51097','51099','51106','51108','51117',
'51146','51147','51148','51149','51150','51160','51161','51168','51178','51179',
'51193','51197','51203','51208','51212','51227','51231','51242','51244','51259',
'51260','51278','51285','51301','51303','51307','51312','51317','51319','51326',
'51339','51354','51357','51371','51372','51377','51388','51389','51409','51415',
'51436','51438','51453','51476','51482','51483','51485','51486','51490','51491',
'51501','51502','51504','51506','51509','51512','51515','51525','51538','51544',
'51545','51546','51547','51548','51553','51555','51556','51559','51566','51572',
'51574','51587','51594','51595','51616','51617','51634','51648','51656'
);
```

### 3. Extraction des flux entrants (ailleurs → travail dans le pays)

```sql
CREATE TABLE bdtopo.travail_domicile_ou_ailleurs_vers_pays AS
SELECT *
FROM bdtopo.flux_domicile_travail_2022 AS f
WHERE f.dclt IN (
'51003','51023','51031','51078','51087','51097','51099','51106','51108','51117',
'51146','51147','51148','51149','51150','51160','51161','51168','51178','51179',
'51193','51197','51203','51208','51212','51227','51231','51242','51244','51259',
'51260','51278','51285','51301','51303','51307','51312','51317','51319','51326',
'51339','51354','51357','51371','51372','51377','51388','51389','51409','51415',
'51436','51438','51453','51476','51482','51483','51485','51486','51490','51491',
'51501','51502','51504','51506','51509','51512','51515','51525','51538','51544',
'51545','51546','51547','51548','51553','51555','51556','51559','51566','51572',
'51574','51587','51594','51595','51616','51617','51634','51648','51656'
);
```

### 4. Création des flux entrants sous forme de lignes (géométrie)

```sql
CREATE TABLE bdtopo.travail_domicile_ou_ailleurs_vers_pays_geom AS
SELECT
    td.*,
    ST_MakeLine(
        ST_Centroid(cdom.geom),   -- géométrie domicile
        ST_Centroid(ctrav.geom)   -- géométrie travail
    ) AS geom
FROM bdtopo.travail_domicile_ou_ailleurs_vers_pays td
LEFT JOIN bdtopo.commune_adminexpr cdom
    ON cdom.code_insee = td.codgeo
LEFT JOIN bdtopo.commune_adminexpr ctrav
    ON ctrav.code_insee = td.dclt
WHERE td.dclt IN (
'51003','51023','51031','51078','51087','51097','51099','51106','51108','51117',
'51146','51147','51148','51149','51150','51160','51161','51168','51178','51179',
'51193','51197','51203','51208','51212','51227','51231','51242','51244','51259',
'51260','51278','51285','51301','51303','51307','51312','51317','51319','51326',
'51339','51354','51357','51371','51372','51377','51388','51389','51409','51415',
'51436','51438','51453','51476','51482','51483','51485','51486','51490','51491',
'51501','51502','51504','51506','51509','51512','51515','51525','51538','51544',
'51545','51546','51547','51548','51553','51555','51556','51559','51566','51572',
'51574','51587','51594','51595','51616','51617','51634','51648','51656'
);
```

### 5. Création des flux sortants sous forme de lignes (géométrie)

```sql
CREATE TABLE bdtopo.domicile_travail_ou_pays_vers_ailleurs_geom AS
SELECT
    dt.*,
    ST_MakeLine(
        ST_Centroid(cdom.geom),   -- centroid domicile
        ST_Centroid(ctrav.geom)   -- centroid travail
    ) AS geom
FROM bdtopo.domicile_travail_ou_pays_vers_ailleurs dt
LEFT JOIN bdtopo.commune_adminexpr cdom
    ON cdom.code_insee = dt.codgeo
LEFT JOIN bdtopo.commune_adminexpr ctrav
    ON ctrav.code_insee = dt.dclt
WHERE dt.codgeo IN (
'51003','51023','51031','51078','51087','51097','51099','51106','51108','51117',
'51146','51147','51148','51149','51150','51160','51161','51168','51178','51179',
'51193','51197','51203','51208','51212','51227','51231','51242','51244','51259',
'51260','51278','51285','51301','51303','51307','51312','51317','51319','51326',
'51339','51354','51357','51371','51372','51377','51388','51389','51409','51415',
'51436','51438','51453','51476','51482','51483','51485','51486','51490','51491',
'51501','51502','51504','51506','51509','51512','51515','51525','51538','51544',
'51545','51546','51547','51548','51553','51555','51556','51559','51566','51572',
'51574','51587','51594','51595','51616','51617','51634','51648','51656'
);
```

### 6. Analyse par EPCI dans QGIS

Une fois les couches générées :

**1. Charger dans QGIS :**

Les couches de flux (entrants / sortants)

La couche des communes

La couche des EPCI

**2. Pour obtenir les flux par EPCI :**

Sélectionner l’EPCI cible

Sélectionner toutes les communes correspondant à cet EPCI

Filtrer les flux pour ne conserver que ceux dont la commune origine/destination appartient à l’EPCI

Dissoudre ou sommer les flux selon les besoins

**3. Exporter les résultats en :**

Shapefile

GeoPackage

CSV

Couche de flux EPCI → EPCI

### 7. Et si nous voulons une cartes avec cercles proportionnels incluant les 3 types de flux

```sql
WITH pays_chalons AS (
  SELECT unnest(ARRAY[
    '51504','51326','51409','51312','51538','51595','51483','51372','51099','51634',
    '51656','51227','51149','51244','51319','51587','51193','51108','51453','51301',
    '51357','51178','51031','51146','51242','51486','51212','51389','51307','51388',
    '51285','51003','51231','51078','51377','51168','51555','51509','51617','51023',
    '51150','51566','51278','51147','51117','51545','51161','51476','51512','51556',
    '51203','51087','51160','51303','51594','51525','51106','51506','51354','51574',
    '51616','51502','51148','51436','51339','51648','51482','51097','51415','51371',
    '51490','51572','51438','51179','51515','51548','51208','51260','51259','51559',
    '51547','51544','51553','51317','51491','51546','51501','51197','51485'
  ]) AS code_insee
),

entrants AS (
  SELECT
    s.dclt AS code_insee,
    SUM(s.nbflux_c22_actocc15p)::int AS entrants
  FROM bdtopo.flux_domicile_travail_2022 s
  WHERE s.dclt IN (SELECT code_insee FROM pays_chalons)
  GROUP BY s.dclt
),

sortants AS (
  SELECT
    s.codgeo AS code_insee,
    SUM(s.nbflux_c22_actocc15p)::int AS sortants
  FROM bdtopo.flux_domicile_travail_2022 s
  WHERE s.codgeo IN (SELECT code_insee FROM pays_chalons)
  GROUP BY s.codgeo
),

internes AS (
  SELECT
    s.codgeo AS code_insee,
    SUM(s.nbflux_c22_actocc15p)::int AS internes
  FROM bdtopo.flux_domicile_travail_2022 s
  WHERE s.codgeo IN (SELECT code_insee FROM pays_chalons)
    AND s.dclt   IN (SELECT code_insee FROM pays_chalons)
  GROUP BY s.codgeo
)

SELECT
  pc.code_insee,

  COALESCE(e.entrants, 0) AS flux_entrants,
  COALESCE(s.sortants, 0) AS flux_sortants,
  COALESCE(i.internes, 0) AS flux_internes,

  /* Total des 3 flux */
  ( COALESCE(e.entrants, 0)
  + COALESCE(s.sortants, 0)
  + COALESCE(i.internes, 0)
  ) AS total_flux,

  /* Pourcentages formatés */
  CASE
    WHEN (COALESCE(e.entrants, 0)
        + COALESCE(s.sortants, 0)
        + COALESCE(i.internes, 0)) = 0 THEN '0%'
    ELSE round(
      100.0 * COALESCE(e.entrants, 0)
      / (COALESCE(e.entrants, 0)
       + COALESCE(s.sortants, 0)
       + COALESCE(i.internes, 0))
    )::int || '%'
  END AS pct_entrants,

  CASE
    WHEN (COALESCE(e.entrants, 0)
        + COALESCE(s.sortants, 0)
        + COALESCE(i.internes, 0)) = 0 THEN '0%'
    ELSE round(
      100.0 * COALESCE(s.sortants, 0)
      / (COALESCE(e.entrants, 0)
       + COALESCE(s.sortants, 0)
       + COALESCE(i.internes, 0))
    )::int || '%'
  END AS pct_sortants,

  CASE
    WHEN (COALESCE(e.entrants, 0)
        + COALESCE(s.sortants, 0)
        + COALESCE(i.internes, 0)) = 0 THEN '0%'
    ELSE round(
      100.0 * COALESCE(i.internes, 0)
      / (COALESCE(e.entrants, 0)
       + COALESCE(s.sortants, 0)
       + COALESCE(i.internes, 0))
    )::int || '%'
  END AS pct_internes

FROM pays_chalons pc
LEFT JOIN entrants e ON pc.code_insee = e.code_insee
LEFT JOIN sortants s ON pc.code_insee = s.code_insee
LEFT JOIN internes i ON pc.code_insee = i.code_insee
--where pc.code_insee like '51108'
ORDER BY pc.code_insee;
```

**Limitation au périmètre du pays de Châlons-en-Champagne (si besoin)**

On aura un résultat complet avec toutes les communes de la France sans filtre avec les communes du Pays. Donc si vous voulez travailler qu'avec les communes du pays de Châlons-en-Champagne, il faut toujours appliquer un filtre d'affichage sur QGIS avec ces communes.

### Exemples de cartes flux

<img width="2338" height="1653" alt="M5_flux_domicile_travail_plus_15ans" src="https://github.com/user-attachments/assets/ecb8de43-7a0c-43e1-a12e-dc88cc4a662f" />

<img width="1653" height="2338" alt="bis_M5_flux_domicile_travail_plus_15ans" src="https://github.com/user-attachments/assets/7ac42670-8517-4eec-b8f7-3e4525c8cc86" />

<img width="2480" height="1748" alt="E2_DEPLACEMENTS_PENDULAIRES_FC" src="https://github.com/user-attachments/assets/f9bbc491-00d4-479b-97c4-8cb86e8c356f" />

### Exemples d'autres cartes produites lors des missions d'automatisation sur QGIS

<img width="2338" height="1653" alt="M1_accessibilite_generale_en_voiture_depuis_les_principales_infrastructures_de_transports" src="https://github.com/user-attachments/assets/3dd90abd-86e6-4c84-9af5-a12993cf3628" />

<img width="2338" height="1653" alt="M2_frequentation_gares" src="https://github.com/user-attachments/assets/fcc712c3-1e05-48b4-a91b-cf146e73844d" />

<img width="2338" height="1653" alt="M3_comptage_routie" src="https://github.com/user-attachments/assets/7c02c2b9-fe25-4c5a-bbf9-e928cff10376" />

<img width="1653" height="2338" alt="E1_evolution_des_emplois_nombre_structure_prive_public_taux_chomage" src="https://github.com/user-attachments/assets/c907bfad-3c26-4b8e-968a-5537c7053ea6" />

<img width="1653" height="2338" alt="E1_evolution_des_emplois_nombre_structure_prive_public_taux d&#39;evolution emploi" src="https://github.com/user-attachments/assets/43132727-0e06-4fae-87b0-7964c25b1c0b" />

<img width="2480" height="1748" alt="E10_INDICATEUR_DE_CONCENTRATION_EMPLOI_FC" src="https://github.com/user-attachments/assets/4298836b-b58d-4f10-b9e4-bee28ff65fa0" />

<img width="1653" height="2338" alt="E1_Nombre et part des effectifs salariés en 2024" src="https://github.com/user-attachments/assets/ee914335-9b99-4b04-95b6-2b9c3d95f6ae" />

<img width="1653" height="2338" alt="M4_part modale" src="https://github.com/user-attachments/assets/10c95eb6-f2f2-40e6-adc9-586c6c74f9b0" />

<img width="1653" height="2338" alt="bis_M5_base_cc_emploi_pop_active_2022" src="https://github.com/user-attachments/assets/0fe83e6e-7eee-4508-b0ac-b43c4fb0d630" />

<img width="2480" height="1748" alt="EQ2_NIVEAUX_DE_CENTRES_EQUIPEMENTS_ET_REPARTITION_PAR_DOMAINES_FC" src="https://github.com/user-attachments/assets/f877bc1d-3637-4d7d-9524-cda925400c9e" />

<img width="2338" height="1653" alt="D2_Population_2022_juillet2025" src="https://github.com/user-attachments/assets/c910c6cb-4356-47d9-9aaa-ec7845a4d35c" />

<img width="2480" height="1748" alt="D4_REVENU_MEDIAN_MENSUEL_DES_MENAGES_FC" src="https://github.com/user-attachments/assets/510ba2bf-7eae-46db-b772-ae59914f91b3" />

<img width="2338" height="1653" alt="D5_Indice_jeunesse_veillesse_2022_juillet2025" src="https://github.com/user-attachments/assets/e0cb0a50-dd02-4118-b3a7-a63e35cd3362" />

<img width="2480" height="1748" alt="EQ6_offre_equip_scolaire" src="https://github.com/user-attachments/assets/ea64d8c6-155e-4a7c-80dd-f32e2fd7c8f8" />

<img width="2338" height="1653" alt="H2_Logements_autorises_par_type_2014-2024_juillet2025" src="https://github.com/user-attachments/assets/d061f023-4f4e-4007-9c37-db92eaea94a5" />

<img width="2480" height="1748" alt="M1_accessibilite_generale_FC" src="https://github.com/user-attachments/assets/c8f1ddcf-d5ca-4910-8e6d-106d88f228c9" />

<img width="2480" height="1748" alt="M2_reseau_routier_trafic" src="https://github.com/user-attachments/assets/340a2d45-2281-40d5-b193-dd5a3b98763b" />

<img width="2480" height="1748" alt="M3_transport_en_commun" src="https://github.com/user-attachments/assets/ab377a68-9bd7-40d4-b6f4-19d2cd629af9" />

<img width="2480" height="1748" alt="P1_situation_generale" src="https://github.com/user-attachments/assets/c9d22ca2-282c-4e81-8bb0-c43d9a3611f9" />

<img width="1653" height="2338" alt="P1_evolution_habitants_sur_20_ans" src="https://github.com/user-attachments/assets/fabb5d6c-19cb-475a-898d-f37c80ef52c9" />

<img width="1653" height="2338" alt="P9_densite_population" src="https://github.com/user-attachments/assets/1f27a47f-22f5-48d4-9e5f-9addf882e0ef" />

### Zoom sur deux atlas cartographiques 

<img width="1653" height="2338" alt="image" src="https://github.com/user-attachments/assets/4be5a620-3f74-40e0-acf3-341e3df5c557" />
<img width="1653" height="2338" alt="image" src="https://github.com/user-attachments/assets/cfb7dc77-6fc5-49f6-a1d9-db059b4fdae0" />
<img width="1653" height="2338" alt="image" src="https://github.com/user-attachments/assets/91ffec75-5e7e-46c8-a608-470835d1a017" />
<img width="2048" height="1448" alt="image" src="https://github.com/user-attachments/assets/280ddc67-d06c-4e09-938e-9166571fbb3d" />
<img width="2048" height="1448" alt="image" src="https://github.com/user-attachments/assets/77edb7b6-c3c2-41a7-9868-44ad973590b9" />
<img width="2048" height="1448" alt="image" src="https://github.com/user-attachments/assets/3d6cfe38-d63d-4509-99a7-01ed918a0f9a" />

## Conclusion

L’automatisation des atlas cartographiques dans QGIS permet de simplifier et d’optimiser la production de cartes en s’appuyant sur des règles de symbologie et des expressions dynamiques. Grâce à l’intégration de données directement issues d’une base de données SQL et à l’utilisation de filtres spatiaux et attributaires, il est possible de générer des cartes précises et adaptées à chaque entité étudiée. 

Toutefois, malgré ces avancées, certains défis persistent, notamment la gestion des étiquettes et des éléments graphiques complexes comme les cercles proportionnels. Ces aspects nécessitent encore des ajustements manuels pour garantir une lisibilité optimale des cartes produites. 

En définitive, cette méthodologie d’automatisation constitue un gain de temps considérable et améliore la cohérence des représentations cartographiques, tout en permettant une mise à jour continue des données. Cependant, une réflexion complémentaire sur l’optimisation des paramètres d’affichage reste essentielle pour perfectionner le rendu final et s’adapter aux spécificités de chaque territoire.







