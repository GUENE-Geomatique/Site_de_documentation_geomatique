# Automatisation du traitement FME avec un script Batch et planification
---
https://docs.qgis.org/3.44/fr/docs/user_manual/processing/modeler.html

## 1. Création du script Batch (.bat)

Afin d’automatiser l’exécution du traitement FME, un script Batch a été créé.  
Ce script permet de lancer directement le workspace FME sans intervention manuelle.

Le script utilise deux variables :

- une variable pour définir le chemin de l’exécutable FME (`fme.exe`)  
- une variable pour définir le chemin du workspace (`.fmw`)  

***Contenu du fichier `arbre_stats.bat`***

```bat
@echo off

REM Définition des variables
set FME_EXE="C:\Program Files\FME\fme.exe"
set WORKSPACE="C:\Users\aguene\Downloads\FME_projet\EXERCICES\exo_arbre\work\arbres.fmw"

REM Exécution du traitement FME
%FME_EXE% %WORKSPACE%

pause
```

Ce script a été enregistré dans le dossier de travail du projet :
C:\Users\aguene\Downloads\FME_projet\EXERCICES\exo_arbre\work

Son exécution permet de lancer automatiquement le traitement FME et de générer les fichiers de sortie attendus.

<img width="1206" height="442" alt="1-pour executer le projet fme par script bash egalement possible automatique telle heure" src="https://github.com/user-attachments/assets/bf3316d1-4ff7-4013-b5a7-6ea45aa55082" />


## 2. Planification automatique avec le Planificateur de tâches Windows

Afin d’automatiser complètement le traitement, le script Batch a été planifié avec le Planificateur de tâches Windows.

### Paramètres de la tâche :

***Nom de la tâche :*** FME_arbre_hebdo

***Fréquence :*** Hebdomadaire

***Jour :*** (au choix, par exemple lundi)

***Heure :*** 13h00

***Action :*** Lancer un programme

***Programme exécuté :*** arbre_stats.bat


### Fonctionnement
Chaque semaine à 13h00 :

le script .bat est exécuté automatiquement
le logiciel FME est lancé
le workspace arbres.fmw est exécuté
les résultats sont générés sans intervention utilisateur


### Points importants
Pour que l’automatisation fonctionne correctement, il faut vérifier :

que le chemin vers fme.exe est correct
que le fichier .fmw existe au bon emplacement
que le PC est allumé à l’heure prévue
que la tâche est autorisée à s’exécuter même en arrière-plan

<img width="1480" height="570" alt="2-demarrer l&#39;execution automatique avec planificateur de tache de windows" src="https://github.com/user-attachments/assets/6992a277-5b6c-4614-ac73-f686c6fecf0d" />


### Conclusion
Cette mise en place permet :

✅ d’automatiser complètement le traitement FME

✅ de garantir une exécution régulière sans intervention manuelle

✅ de simuler un cas réel utilisé en bureau d’étude ou en entreprise

Cette solution correspond à une pratique professionnelle courante en géomatique et en gestion de données automatisées.

## 3. Exemples de projets FME

<img width="1918" height="1040" alt="arbre_secteur" src="https://github.com/user-attachments/assets/a39729ce-53d8-46c3-b6ed-feaef0ba5fa5" />

<img width="1918" height="1036" alt="projet fme" src="https://github.com/user-attachments/assets/b3d771c5-af4d-45d6-8130-fd5de9215824" />

<img width="1919" height="1033" alt="population impactée_projet" src="https://github.com/user-attachments/assets/3c8c6fce-831c-4dff-a9f4-ca4a8502df3b" />




