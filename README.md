
# Scénario 7 : Détection des usagers vulnérables de la route et simulation du freinage d'urgence automatique

## 1. Présentation du projet

Ce projet s'inscrit dans le domaine de la vision par ordinateur appliquée à la conduite autonome. Il vise à détecter les usagers vulnérables de la route (*Vulnerable Road Users — VRU*) à partir d'une vidéo, puis à simuler le comportement d'un système de freinage d'urgence automatique (*Automatic Emergency Braking — AEB*).

La solution repose sur le modèle de détection d'objets YOLOv8n (version nano), associé à une analyse géométrique du corridor de trajectoire du véhicule et à une logique décisionnelle fondée sur des temporisateurs (implémentation d'une machine à états simplifiée).

Le système analyse les images successives d'une vidéo, identifie les objets pertinents, vérifie leur position par rapport à un corridor de trajectoire prédéfini et met à jour l'état du système affiché à l'écran.

L'objectif est de mettre en œuvre une approche expérimentale combinant détection d'objets, analyse spatiale et prise de décision dans un environnement simplifié de conduite autonome.

## 2. Objectifs

Les principaux objectifs du projet sont les suivants :

- Détecter les piétons et d'autres catégories d'objets à l'aide du modèle YOLOv8n.
- Localiser les objets détectés dans les images à partir de leurs boîtes englobantes.
- Définir un corridor trapézoïdal représentant approximativement la trajectoire du véhicule.
- Déterminer si les objets surveillés se trouvent dans la zone de trajectoire étudiée.
- Déclencher la simulation du freinage d'urgence lorsqu'un piéton est détecté dans le corridor.
- Déclencher un avertissement lorsqu'une balle (*Sports ball*) est détectée dans le corridor.
- Visualiser les détections, le corridor et l'état du système pendant le traitement vidéo.

## 3. Fonctionnalités principales

### 3.1. Détection des objets

Le système utilise YOLOv8n, via la bibliothèque Ultralytics, pour détecter les objets présents dans chaque image.

Pour chaque détection, le modèle fournit :

- Les coordonnées de la boîte englobante (`box.xyxy`).
- L'identifiant de la classe détectée (`box.cls`).
- Le score de confiance associé à la prédiction (`box.conf`).

Dans cette implémentation, seuls les identifiants de classe et les coordonnées des boîtes sont exploités par la logique décisionnelle.

### 3.2. Analyse du corridor de trajectoire

Un corridor de trajectoire est représenté par un polygone trapézoïdal défini à partir des dimensions de l'image (largeur `L`, hauteur `H`) :

```text
(L * 0.12, H * 0.95), (L * 0.42, H * 0.55), (L * 0.58, H * 0.55), (L * 0.88, H * 0.95)
```

Cette zone représente une approximation de l'espace situé devant le véhicule. Elle permet de distinguer les objets susceptibles d'interférer avec la trajectoire étudiée des objets détectés en dehors de cette zone.

La fonction OpenCV `cv2.pointPolygonTest` est utilisée pour déterminer si un point appartient au polygone (retour `>= 0` : point à l'intérieur ou sur le bord).

Dans cette approche, le point inférieur central de la boîte englobante `(x1 + x2) / 2, y2` est utilisé pour évaluer la position de l'objet par rapport au corridor.

### 3.3. Logique décisionnelle (machine à états par temporisateurs)

Le système repose sur deux temporisateurs, `brake_timer` et `warning_timer`, décrémentés à chaque image, associés à une durée de maintien de `COOLDOWN_FRAMES = 12` images. Trois états en résultent :

- **CRUISING** : les deux temporisateurs sont à zéro. État de circulation normale, aucune menace dans le corridor.
- **WARNING** : `warning_timer > 0`. Activé lorsqu'une balle (*Sports ball*) est détectée dans le corridor **et** qu'aucun freinage n'est en cours (`brake_timer == 0`).
- **EMERGENCY_BRAKING** : `brake_timer > 0`. Activé dès qu'un piéton est détecté dans le corridor.

Règles de transition exactes appliquées à chaque image :

1. Si un piéton est détecté dans le corridor : `brake_timer = 12` et `warning_timer = 0` (le freinage prime sur l'avertissement).
2. Sinon, si une balle est détectée dans le corridor et que `brake_timer == 0` : `warning_timer = 12`.
3. Sinon : les deux temporisateurs sont décrémentés de 1 ; lorsque les deux atteignent zéro, le système revient à l'état **CRUISING**.

L'état est priorisé dans l'ordre : piéton > balle > absence de menace.

### 3.4. Visualisation des résultats

Le traitement produit, image par image, une visualisation comprenant :

- Le corridor de trajectoire : remplissage cyan semi-transparent (opacité ≈ 12 %) et contour cyan.
- Les objets surveillés situés **hors** du corridor : boîte englobante verte fine.
- Le piéton détecté **dans** le corridor : boîte englobante rouge épaisse et point rouge sur le point inférieur central.
- La balle détectée **dans** le corridor : boîte englobante orange.
- Un bandeau d'état en bas de l'image : rouge (freinage d'urgence), orange (avertissement) ou gris foncé avec texte vert (circulation normale).

Les images sont converties de BGR (OpenCV) vers RGB puis affichées directement dans le notebook via Matplotlib, avec rafraîchissement de la cellule à chaque image (`clear_output`).

Cette implémentation n'affiche ni les étiquettes de classes, ni les scores de confiance, ni de télémétrie numérique.

## 4. Architecture technique

Le pipeline de traitement est organisé en plusieurs étapes successives.

### Étape 1 : Acquisition des images

Lecture de la vidéo `source/test_driving_video.mp4` via `cv2.VideoCapture`, puis extraction des dimensions de l'image (`CAP_PROP_FRAME_WIDTH`, `CAP_PROP_FRAME_HEIGHT`).

### Étape 2 : Définition du corridor

Un polygone trapézoïdal représentant la zone de trajectoire est calculé à partir des dimensions de l'image.

### Étape 3 : Détection des objets

Le modèle YOLOv8n analyse l'image et retourne les détections disponibles, notamment les boîtes englobantes et les classes.

### Étape 4 : Évaluation spatiale

Pour chaque objet surveillé, le point inférieur central de sa boîte englobante est comparé au corridor à l'aide de `cv2.pointPolygonTest`.

### Étape 5 : Prise de décision

Des indicateurs `pedestrian_in_lane` et `ball_in_lane` sont positionnés selon les détections dans le corridor.

### Étape 6 : Gestion des temporisateurs

Les variables `brake_timer` et `warning_timer` sont mises à jour selon les règles de transition définies en 3.3.

### Étape 7 : Affichage des résultats

Les dessins OpenCV (corridor, boîtes, bandeau d'état) sont appliqués sur l'image, puis le rendu est affiché en ligne dans le notebook via Matplotlib.

## 5. Technologies utilisées

| Technologie | Utilisation |
|---|---|
| Python | Langage principal du projet |
| Jupyter Notebook | Développement et exécution interactive du pipeline |
| OpenCV | Lecture vidéo, traitement des images et analyse géométrique (`pointPolygonTest`, `fillPoly`, `addWeighted`) |
| NumPy | Manipulation des données numériques (polygone, boîtes) |
| Ultralytics YOLOv8n | Détection d'objets |
| Matplotlib | Rendu des images dans le notebook |
| IPython.display | Rafraîchissement de l'affichage inline (`clear_output`) |
| Machine à états (temporisateurs) | Gestion de la logique décisionnelle |

## 6. Classes d'objets surveillées

Le projet exploite les classes suivantes du jeu de données COCO prises en charge par le modèle YOLOv8n.

| Identifiant COCO | Classe | Rôle dans l'implémentation |
|---|---|---|
| 0 | Person | Déclenche la simulation du freinage d'urgence si présent dans le corridor |
| 32 | Sports ball | Déclenche l'avertissement si présent dans le corridor (hors freinage actif) |
| 1 | Bicycle | Surveillée (affichée en vert hors corridor), sans effet décisionnel |
| 3 | Motorcycle | Surveillée (affichée en vert hors corridor), sans effet décisionnel |
| 15 | Cat | Surveillée (affichée en vert hors corridor), sans effet décisionnel |
| 16 | Dog | Surveillée (affichée en vert hors corridor), sans effet décisionnel |
| 17 | Horse | Surveillée (affichée en vert hors corridor), sans effet décisionnel |
| 18 | Sheep | Surveillée (affichée en vert hors corridor), sans effet décisionnel |
| 21 | Bear | Surveillée (affichée en vert hors corridor), sans effet décisionnel |
| 22 | Zebra | Surveillée (affichée en vert hors corridor), sans effet décisionnel |

Seules les classes *Person* et *Sports ball* déclenchent des actions de sécurité. Les autres classes listées sont détectées et dessinées lorsqu'elles se trouvent hors du corridor, mais ne modifient pas l'état du système. Les classes COCO non listées (voitures, camions, etc.) sont ignorées par la logique décisionnelle.

Les catégories COCO présentées ici constituent une sélection de classes détectables et non une définition exhaustive des usagers vulnérables de la route.

## 7. Prérequis

Pour exécuter le projet, les éléments suivants sont nécessaires :

- Python 3.8 ou une version ultérieure, sous réserve de compatibilité avec les versions installées des bibliothèques.
- pip pour l'installation des dépendances.
- Jupyter Notebook ou JupyterLab.
- Les bibliothèques Python indiquées dans le fichier `requirements.txt`.
- Les poids du modèle `yolov8n.pt` : téléchargés automatiquement par Ultralytics au premier lancement (connexion Internet requise), ou à placer manuellement dans le répertoire de travail.
- Une vidéo de démonstration disponible à l'emplacement `source/test_driving_video.mp4`, ou l'adaptation du chemin dans le code.

Les ressources nécessaires doivent être disponibles aux chemins attendus par le notebook.

## 8. Installation et exécution

### 8.1. Récupération du projet

Cloner le dépôt Git :

```bash
git clone <URL_DU_DEPOT>
```

Accéder au dossier du projet :

```bash
cd scenario_7_vru_detection
```

Remplacer `<URL_DU_DEPOT>` par l'adresse réelle du dépôt GitHub.

### 8.2. Installation des dépendances

Installer les dépendances définies dans le fichier `requirements.txt` :

```bash
pip install -r requirements.txt
```

Si nécessaire, vérifier que Jupyter est installé :

```bash
pip install notebook
```

### 8.3. Lancement du notebook

Démarrer Jupyter Notebook :

```bash
jupyter notebook
```

Dans l'interface qui s'ouvre, sélectionner le fichier :

`scenario_7_vru_detection.ipynb`

Il est également possible d'utiliser JupyterLab :

```bash
jupyter lab
```

### 8.4. Exécution du pipeline

Ouvrir le notebook et exécuter les cellules dans l'ordre prévu.

Avant l'exécution, vérifier les éléments suivants :

- La disponibilité des poids `yolov8n.pt` (téléchargement automatique au premier lancement sinon).
- La présence de la vidéo à l'emplacement `source/test_driving_video.mp4` (ou l'adaptation du chemin dans le code).
- L'installation correcte des dépendances.
- La compatibilité de l'environnement Python avec les bibliothèques utilisées.

Le traitement s'exécute jusqu'à la fin de la vidéo ou jusqu'à l'interruption manuelle du noyau (bouton *Stop* / *Kernel → Interrupt*). Les ressources vidéo sont libérées à la fin de l'exécution.

Le déroulement exact dépend de l'organisation et du contenu du notebook.

## 9. Structure du projet

La structure actuelle du dépôt est la suivante :

```text
scenario_7_vru_detection/
├── assets/
├── models/
├── source/
│   └── test_driving_video.mp4
├── README.md
├── requirements.txt
└── scenario_7_vru_detection.ipynb
```

Description des éléments :

- `assets/` : dossier contenant les ressources du projet.
- `models/` : dossier destiné aux fichiers de modèles et aux poids utilisés par le système ; le fichier `yolov8n.pt` est téléchargé automatiquement par Ultralytics dans le répertoire de travail au premier lancement.
- `source/` : dossier contenant la vidéo de démonstration utilisée par le notebook.
- `README.md` : documentation du projet.
- `requirements.txt` : liste des dépendances Python.
- `scenario_7_vru_detection.ipynb` : notebook Jupyter contenant le code, les étapes de traitement et les visualisations du projet.

Le contenu détaillé des dossiers `assets/`, `models/` et `source/` dépend des fichiers présents dans le dépôt.

## 10. Limites et perspectives d'amélioration

### 10.1. Estimation de la trajectoire

Le corridor trapézoïdal constitue une approximation géométrique simplifiée, fixée par des proportions constantes de l'image. Il ne tient pas nécessairement compte de la calibration de la caméra, de la courbure de la route, de l'angle de braquage ou des mouvements du véhicule.

Une amélioration possible consisterait à intégrer la détection des voies, la calibration de la caméra et une estimation plus précise de la trajectoire.

### 10.2. Gestion des temporisateurs

La logique repose sur un nombre d'images (`COOLDOWN_FRAMES = 12`) et dépend donc de la fréquence d'images effective de la vidéo.

L'utilisation de temps écoulé réel permettrait de rendre la gestion des temporisateurs moins dépendante du nombre d'images traitées par seconde.

### 10.3. Analyse spatiale

Le point inférieur central d'une boîte englobante constitue une approximation de la position d'un objet. Cette méthode ne représente pas entièrement sa forme, son mouvement ou sa trajectoire future.

L'intégration d'un suivi multi-objets et d'une estimation du mouvement pourrait améliorer l'analyse des risques.

### 10.4. Prédiction des collisions

Le système repose sur des règles géométriques et décisionnelles simplifiées. Une extension possible consisterait à estimer le temps avant collision, ou *Time-to-Collision* (TTC), à partir de la distance et de la vitesse relative des objets.

Cette évolution nécessiterait des informations de mouvement et de distance suffisamment fiables.

### 10.5. Évaluation des performances

Des tests complémentaires permettraient d'évaluer le comportement du système dans différentes conditions : éclairage variable, occultations, objets éloignés et scènes complexes.

Les métriques envisageables comprennent la précision, le rappel, le taux de fausses détections, le taux d'objets manqués et le temps moyen de traitement d'une image.

### 10.6. Optimisation

Selon les résultats obtenus, plusieurs pistes pourraient être étudiées :

- L'accélération matérielle de l'inférence.
- L'optimisation du traitement vidéo (rendu Matplotlib image par image moins performant qu'une sortie vidéo native).
- L'export d'une vidéo annotée en plus de l'affichage inline.
- L'amélioration du suivi temporel des objets.
- L'adaptation du corridor à la trajectoire estimée du véhicule.
- L'intégration de méthodes de prédiction des risques.

### 10.7. Considérations de sécurité

Ce projet est un prototype expérimental de vision par ordinateur et de simulation de freinage d'urgence. Il ne constitue pas un système de freinage certifié et ne doit pas être utilisé pour commander directement un véhicule réel.

Une application réelle nécessiterait notamment une validation approfondie, des mécanismes de sécurité, une estimation fiable des distances et des vitesses, ainsi que le respect des normes automobiles applicables.

## 11. Conclusion

Ce projet illustre l'intégration de la détection d'objets, de l'analyse géométrique et d'une logique décisionnelle fondée sur des temporisateurs dans un scénario simplifié de conduite autonome.

L'association de YOLOv8n, d'un corridor de trajectoire et de règles de gestion des états permet d'étudier comment les informations visuelles peuvent contribuer à une première détection des situations potentiellement dangereuses.

Cette architecture fournit une base expérimentale pour approfondir des sujets tels que le suivi des objets, la prédiction des collisions et l'amélioration des systèmes de perception pour la conduite autonome.

## 12. Auteur

**Mohammed Boujaada**

Étudiant en Master Intelligence Artificielle et Technologies Émergentes  
Faculté des Sciences Appliquées de Nador (FSAN), Maroc

- LinkedIn : https://www.linkedin.com/in/mohammed-boujaada-229a3333a/
- GitHub : https://github.com/mohammed-boujaada
