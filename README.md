# Scénario 7 : Détection des usagers vulnérables de la route et simulation du freinage d'urgence automatique

## 1. Présentation du projet

Ce projet s'inscrit dans le domaine de la vision par ordinateur appliquée à la conduite autonome. Il vise à détecter les usagers vulnérables de la route (*Vulnerable Road Users — VRU*) à partir d'images vidéo, puis à simuler le comportement d'un système de freinage d'urgence automatique (*Automatic Emergency Braking — AEB*).

La solution repose sur le modèle de détection d'objets YOLOv8, associé à une analyse géométrique de la trajectoire du véhicule et à une logique décisionnelle basée sur une machine à états.

Le système analyse les images successives d'une vidéo, identifie les objets pertinents, vérifie leur position par rapport à un corridor de trajectoire prédéfini et détermine l'état de sécurité correspondant.

L'objectif est de mettre en œuvre une approche expérimentale combinant détection d'objets, analyse spatiale et prise de décision dans un environnement simplifié de conduite autonome.

## 2. Objectifs

Les principaux objectifs du projet sont les suivants :

- Détecter les piétons et d'autres catégories d'objets à l'aide du modèle YOLOv8.
- Localiser les objets détectés dans les images à partir de leurs boîtes englobantes.
- Définir un corridor représentant approximativement la trajectoire du véhicule.
- Déterminer si les objets détectés se trouvent dans la zone de trajectoire étudiée.
- Simuler différents états de conduite en fonction des objets détectés.
- Mettre en œuvre une logique de gestion des avertissements et du freinage d'urgence.
- Visualiser les détections et l'état du système pendant le traitement vidéo.

## 3. Fonctionnalités principales

### 3.1. Détection des objets

Le système utilise YOLOv8, via la bibliothèque Ultralytics, pour détecter les objets présents dans chaque image.

Pour chaque détection, le modèle peut fournir :

- Les coordonnées de la boîte englobante.
- L'identifiant de la classe détectée.
- Le score de confiance associé à la prédiction.

Ces informations servent de base à l'analyse spatiale et à la prise de décision.

### 3.2. Analyse du corridor de trajectoire

Un corridor de trajectoire est représenté par un polygone trapézoïdal défini à partir des dimensions de l'image.

Cette zone représente une approximation de l'espace situé devant le véhicule. Elle permet de distinguer les objets susceptibles d'interférer avec la trajectoire étudiée des objets détectés en dehors de cette zone.

La fonction OpenCV `cv2.pointPolygonTest` peut être utilisée pour déterminer si un point appartient au polygone défini.

Dans cette approche, le point inférieur central de la boîte englobante est utilisé pour évaluer la position de l'objet par rapport au corridor.

### 3.3. Machine à états

Le système repose sur une logique décisionnelle organisée autour de trois états principaux.

- **CRUISING** : état de circulation normale lorsque les conditions configurées ne nécessitent ni avertissement ni freinage d'urgence.
- **WARNING** : état d'avertissement activé lorsqu'un objet répond aux conditions prévues par les règles de détection.
- **EMERGENCY_BRAKING** : état de simulation du freinage d'urgence activé lorsqu'un piéton détecté satisfait les conditions définies par le système.

La transition entre ces états dépend de la classification des objets, de leur position dans le corridor et des règles de gestion des temporisateurs.

Les conditions exactes de transition doivent correspondre à l'implémentation présente dans le notebook.

### 3.4. Visualisation des résultats

Le traitement peut inclure une visualisation des éléments suivants :

- Les boîtes englobantes des objets détectés.
- Les classes prédites par le modèle.
- Le corridor de trajectoire.
- L'état courant du système.
- Les informations de télémétrie disponibles.

Ces éléments permettent d'observer le comportement du système pendant l'analyse de la vidéo.

## 4. Architecture technique

Le pipeline de traitement est organisé en plusieurs étapes successives.

### Étape 1 : Acquisition des images

Le système récupère les images à partir d'une source vidéo compatible avec le traitement OpenCV.

### Étape 2 : Définition du corridor

Un polygone représentant la zone de trajectoire est défini à partir des dimensions de l'image.

### Étape 3 : Détection des objets

Le modèle YOLOv8 analyse l'image et retourne les détections disponibles, notamment les boîtes englobantes, les classes et les scores de confiance.

### Étape 4 : Évaluation spatiale

Pour chaque objet pertinent, le système détermine un point représentatif à partir de sa boîte englobante, puis vérifie sa position par rapport au corridor.

### Étape 5 : Prise de décision

Les règles métier déterminent si une détection doit déclencher un avertissement ou activer la simulation du freinage d'urgence.

### Étape 6 : Gestion des états

Le système met à jour son état en fonction des conditions détectées et des temporisateurs prévus dans l'implémentation.

### Étape 7 : Affichage des résultats

Les informations de détection et l'état du système sont utilisés pour produire les visualisations du pipeline.

## 5. Technologies utilisées

| Technologie | Utilisation |
|---|---|
| Python | Langage principal du projet |
| Jupyter Notebook | Développement et exécution interactive du pipeline |
| OpenCV | Traitement des images et analyse géométrique |
| NumPy | Manipulation des données numériques |
| Ultralytics YOLOv8 | Détection d'objets |
| Machine à états | Gestion de la logique décisionnelle |

## 6. Classes d'objets surveillées

Le projet peut exploiter certaines classes du jeu de données COCO prises en charge par le modèle YOLOv8 utilisé.

| Identifiant COCO | Classe | Utilisation possible |
|---|---|---|
| 0 | Person | Détection des piétons |
| 32 | Sports ball | Détection d'une balle |
| 1 | Bicycle | Détection des vélos |
| 3 | Motorcycle | Détection des motos |
| 15 | Cat | Détection d'un animal |
| 16 | Dog | Détection d'un animal |
| 17 | Horse | Détection d'un animal |
| 18 | Sheep | Détection d'un animal |
| 21 | Bear | Détection d'un animal |
| 22 | Zebra | Détection d'un animal |

La présence d'une classe dans ce tableau ne signifie pas nécessairement qu'elle déclenche une action de sécurité. Ce comportement dépend des classes effectivement surveillées et des règles implémentées dans le notebook.

Les catégories COCO présentées ici constituent une sélection de classes détectables et non une définition exhaustive des usagers vulnérables de la route.

## 7. Prérequis

Pour exécuter le projet, les éléments suivants sont nécessaires :

- Python 3.8 ou une version ultérieure, sous réserve de compatibilité avec les versions installées des bibliothèques.
- pip pour l'installation des dépendances.
- Jupyter Notebook ou JupyterLab.
- Les bibliothèques Python indiquées dans le fichier `requirements.txt`.
- Les poids du modèle YOLOv8 requis par le notebook.
- Une vidéo de démonstration ou une autre source d'images compatible avec le code.

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

- La disponibilité des poids YOLOv8.
- Les chemins des fichiers utilisés dans le code.
- La présence de la vidéo d'entrée, si elle est nécessaire.
- L'installation correcte des dépendances.
- La compatibilité de l'environnement Python avec les bibliothèques utilisées.

Le déroulement exact dépend de l'organisation et du contenu du notebook.

## 9. Structure du projet

La structure actuelle du dépôt est la suivante :

```text
scenario_7_vru_detection/
├── assets/
├── models/
├── README.md
├── requirements.txt
└── scenario_7_vru_detection.ipynb
```

Description des éléments :

- `assets/` : dossier contenant les ressources du projet.
- `models/` : dossier destiné aux fichiers de modèles et aux poids utilisés par le système, selon son contenu réel.
- `README.md` : documentation du projet.
- `requirements.txt` : liste des dépendances Python.
- `scenario_7_vru_detection.ipynb` : notebook Jupyter contenant le code, les étapes de traitement et les visualisations du projet.

Le contenu détaillé des dossiers `assets/` et `models/` dépend des fichiers présents dans le dépôt.

## 10. Limites et perspectives d'amélioration

### 10.1. Estimation de la trajectoire

Le corridor trapézoïdal constitue une approximation géométrique simplifiée. Il ne tient pas nécessairement compte de la calibration de la caméra, de la courbure de la route, de l'angle de braquage ou des mouvements du véhicule.

Une amélioration possible consisterait à intégrer la détection des voies, la calibration de la caméra et une estimation plus précise de la trajectoire.

### 10.2. Gestion des temporisateurs

Une logique fondée sur le nombre d'images traitées dépend de la fréquence d'images effective.

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
- L'optimisation du traitement vidéo.
- L'amélioration du suivi temporel des objets.
- L'adaptation du corridor à la trajectoire estimée du véhicule.
- L'intégration de méthodes de prédiction des risques.

### 10.7. Considérations de sécurité

Ce projet est un prototype expérimental de vision par ordinateur et de simulation de freinage d'urgence. Il ne constitue pas un système de freinage certifié et ne doit pas être utilisé pour commander directement un véhicule réel.

Une application réelle nécessiterait notamment une validation approfondie, des mécanismes de sécurité, une estimation fiable des distances et des vitesses, ainsi que le respect des normes automobiles applicables.

## 11. Conclusion

Ce projet illustre l'intégration de la détection d'objets, de l'analyse géométrique et d'une logique décisionnelle basée sur une machine à états dans un scénario simplifié de conduite autonome.

L'association de YOLOv8, d'un corridor de trajectoire et de règles de gestion des états permet d'étudier comment les informations visuelles peuvent contribuer à une première détection des situations potentiellement dangereuses.

Cette architecture fournit une base expérimentale pour approfondir des sujets tels que le suivi des objets, la prédiction des collisions et l'amélioration des systèmes de perception pour la conduite autonome.

## 12. Auteur

**Mohammed Boujaada**

Étudiant en Master Intelligence Artificielle et Technologies Émergentes  
Faculté des Sciences Appliquées de Nador (FSAN), Maroc

- LinkedIn : https://www.linkedin.com/in/mohammed-boujaada-229a3333a/
- GitHub : https://github.com/mohammed-boujaada