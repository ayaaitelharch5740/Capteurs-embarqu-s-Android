# Lab — Capteurs Android (Sensors)

## Objectif général

Développer progressivement une application Android permettant d'exploiter les capteurs embarqués d'un smartphone. L'application doit afficher les capteurs disponibles, lire leurs caractéristiques techniques, visualiser les mesures sous forme de graphes et exploiter les capteurs de mouvement pour proposer une reconnaissance simple d'activité.

---

## Compétences visées

À la fin du lab, il sera possible de :

- Comprendre le rôle de `SensorManager` dans Android
- Lister les capteurs disponibles dans un dispositif
- Lire les propriétés techniques d'un capteur
- Exploiter `SensorEventListener` pour recevoir des mesures en temps réel
- Afficher l'évolution d'un capteur sous forme de graphe
- Utiliser l'accéléromètre, le gyroscope, le magnétomètre, le capteur de proximité et le compteur de pas
- Mettre en place une logique simple de reconnaissance d'activité

> **Note :** L'Android Emulator inclut un ensemble de commandes de capteur virtuel permettant de tester des capteurs tels que l'accéléromètre, la température ambiante, le magnétomètre, la proximité, la lumière, etc.

---

## Prérequis

- Android Studio installé
- Projet de base Sensor téléchargé depuis le lien fourni par l'enseignant
- Menu latéral contenant les entrées : `Sensors`, `Température`, `Humidité`, `Proximité`, `Magnetic`

---

## Structure du projet

```
app/src/main/java/com/example/sensors/
│
├── MainActivity.java
│
├── fragments/
│   ├── SensorsListFragment.java       ← Liste de tous les capteurs
│   ├── SensorGraphFragment.java       ← Graphe générique (temp, humidité, proximité, magnétique)
│   ├── MotionSensorFragment.java      ← Accéléromètre, gravité, gyroscope
│   ├── StepCounterFragment.java       ← Compteur de pas
│   ├── CompassFragment.java           ← Boussole numérique
│   └── ActivityRecognitionFragment.java ← Reconnaissance d'activité
│
├── utils/
│   └── SensorFormatter.java           ← Formatage des infos d'un capteur
│
└── views/
    └── LineChartView.java             ← Graphe personnalisé sans bibliothèque externe
```

---

## Étapes de réalisation

### Partie 1 — Préparation du projet

**Étape 1 — Ouvrir le projet existant**

Ouvrir le code source de l'application Sensor avec Android Studio. Vérifier que le projet contient déjà un menu avec les entrées : `Sensors`, `Température`, `Humidité`, `Proximité`, `Magnetic`.

**Étape 2 — Organiser les fichiers**

Créer les packages suivants dans le projet :
- `fragments` — contient les écrans de l'application
- `utils` — contient les classes d'aide
- `views` — contient les composants graphiques personnalisés

---

### Partie 2 — Affichage des capteurs disponibles

**Objectif :** Afficher la liste complète des capteurs disponibles avec leurs informations techniques :
- Résolution
- Besoins en énergie (`getPower()` en mA)
- Maximum Range
- Int Type (`getType()`)
- Vitesse maximale d'acquisition (`getMinDelay()` en µs)

**Étape 1 — Créer `SensorFormatter.java`**

Créer dans `utils/` une classe utilitaire qui transforme un objet `Sensor` en texte lisible.

Méthodes Android à utiliser :
- `getName()` — nom du capteur
- `getVendor()` — fabricant
- `getVersion()` — version du capteur
- `getStringType()` — type textuel
- `getType()` — type entier (Int Type)
- `getResolution()` — résolution
- `getPower()` — consommation en milliampères
- `getMaximumRange()` — valeur maximale mesurable
- `getMinDelay()` — délai minimal entre deux acquisitions

**Étape 2 — Créer `SensorsListFragment.java`**

Ce fragment doit :
- Utiliser `SensorManager` pour accéder aux capteurs
- Appeler `getSensorList(Sensor.TYPE_ALL)` pour récupérer tous les capteurs
- Afficher chaque capteur dans un `TextView` via `SensorFormatter.format(sensor)`
- Utiliser un `ScrollView` pour la navigation
- Ajouter des séparateurs visuels entre chaque capteur

---

### Partie 3 — Création du graphe personnalisé

**Objectif :** Afficher l'évolution des mesures sous forme de courbe, sans bibliothèque externe.

**Créer `LineChartView.java`** dans le package `views/`

Cette vue personnalisée doit :
- Étendre `View`
- Maintenir une liste des dernières valeurs (max 80 points)
- Implémenter `addValue(float value)` qui ajoute une valeur et appelle `invalidate()`
- Dessiner les axes dans `onDraw(Canvas canvas)` via `Canvas`
- Relier les points avec un `Path` pour former une courbe continue
- Afficher les valeurs min/max en haut du graphe
- Normaliser les valeurs pour occuper toute la hauteur disponible

---

### Partie 4 — Température, humidité, proximité et champ magnétique

**Objectif :** Créer un fragment générique réutilisable pour ces quatre capteurs.

**Créer `SensorGraphFragment.java`**

Ce fragment générique doit :
- Recevoir via `newInstance(int sensorType, String title, String mode)` :
  - Le type de capteur Android
  - Le titre à afficher
  - Le mode de lecture (`"FIRST_VALUE"` ou `"MAGNITUDE"`)
- S'enregistrer auprès du `SensorManager` dans `onResume()`
- Se désenregistrer dans `onPause()` pour économiser la batterie
- Implémenter `onSensorChanged(SensorEvent event)` pour recevoir les mesures
- Extraire la valeur selon le mode :
  - `FIRST_VALUE` → `values[0]`
  - `MAGNITUDE` → `sqrt(x² + y² + z²)`
- Activer une simulation si le capteur est indisponible (émulateur)

**Relier les menus aux capteurs dans `MainActivity.java` :**

| Menu | Type Android | Mode |
|------|-------------|------|
| Température | `Sensor.TYPE_AMBIENT_TEMPERATURE` | `FIRST_VALUE` |
| Humidité | `Sensor.TYPE_RELATIVE_HUMIDITY` | `FIRST_VALUE` |
| Proximité | `Sensor.TYPE_PROXIMITY` | `FIRST_VALUE` |
| Magnetic | `Sensor.TYPE_MAGNETIC_FIELD` | `MAGNITUDE` |

---

### Partie 5 — Accéléromètre, gravité et gyroscope

**Objectif :** Mesurer le changement d'accélération (x, y, z), la gravité et le taux de rotation.

**Créer `MotionSensorFragment.java`**

Ce fragment doit :
- Recevoir le type de capteur et le titre via `newInstance()`
- Afficher les valeurs x, y, z séparément
- Calculer et afficher la norme : `sqrt(x² + y² + z²)`
- Envoyer la norme au graphe `LineChartView`

**Correspondance des axes dans `event.values` :**
- `event.values[0]` → axe x
- `event.values[1]` → axe y
- `event.values[2]` → axe z

**Ajouter les menus et fragments correspondants :**

| Menu | Type Android | Description |
|------|-------------|-------------|
| Accéléromètre | `Sensor.TYPE_ACCELEROMETER` | Accélération incluant la gravité |
| Gravité | `Sensor.TYPE_GRAVITY` | Composante gravitationnelle uniquement |
| Gyroscope | `Sensor.TYPE_GYROSCOPE` | Taux de rotation en rad/s |

---

### Partie 6 — Compteur de pas

**Objectif :** Mesurer les pas depuis le dernier redémarrage et économiser la batterie.

**Étape 1 — Ajouter la permission dans `AndroidManifest.xml` :**

```xml
<uses-permission android:name="android.permission.ACTIVITY_RECOGNITION" />
```

**Créer `StepCounterFragment.java`**

Ce fragment doit :
- Utiliser `Sensor.TYPE_STEP_COUNTER`
- Gérer la permission `ACTIVITY_RECOGNITION` (requise depuis Android Q)
- Utiliser `ActivityResultLauncher` pour demander la permission
- Mémoriser la première valeur reçue (`initialSteps`)
- Calculer les pas de session : `totalStepsSinceBoot - initialSteps`
- Se désenregistrer dans `onPause()` pour préserver la batterie

**Affichage attendu :**
- Pas depuis le dernier redémarrage
- Pas de la session

---

### Partie 7 — Boussole numérique

**Objectif :** Déterminer la position du téléphone par rapport au monde extérieur.

**Principe :**
- L'accéléromètre donne l'orientation du téléphone par rapport à la gravité
- Le magnétomètre donne la direction du champ magnétique terrestre

**Créer `CompassFragment.java`**

Ce fragment doit :
- Écouter deux capteurs simultanément : `TYPE_ACCELEROMETER` et `TYPE_MAGNETIC_FIELD`
- Maintenir deux tableaux : `gravityValues[3]` et `magneticValues[3]`
- Calculer la matrice de rotation via `SensorManager.getRotationMatrix()`
- Extraire l'orientation via `SensorManager.getOrientation()`
- Convertir l'azimut de radians en degrés
- Normaliser entre 0° et 360°
- Afficher le nom de la direction (Nord, Nord-Est, Est, Sud-Est, Sud, Sud-Ouest, Ouest, Nord-Ouest)

---

### Partie 8 — Reconnaissance simple d'activité

**Objectif :** Déterminer si l'utilisateur marche, saute, est assis ou debout à l'aide de l'accéléromètre.

**Principe pédagogique :**
- L'accéléromètre mesure gravité + mouvement réel
- Un filtre passe-bas permet d'estimer la gravité : `gravity[i] = ALPHA * gravity[i] + (1 - ALPHA) * raw[i]`
- En soustrayant la gravité, on obtient le mouvement linéaire
- Une fenêtre de 30 valeurs permet une classification stable

**Créer `ActivityRecognitionFragment.java`**

Ce fragment doit :
- Utiliser `SENSOR_DELAY_GAME` pour une acquisition rapide
- Appliquer le filtre passe-bas (`ALPHA = 0.8`)
- Calculer le mouvement linéaire (sans gravité)
- Maintenir une fenêtre glissante de 30 valeurs
- Calculer la moyenne, le maximum et l'écart-type
- Classifier l'activité selon les seuils :

| Condition | Activité détectée |
|-----------|------------------|
| `max > 10` | Saut |
| `écart-type > 1.2` | Marche |
| `abs(z) > 8` | Stable / téléphone à plat |
| `abs(y) > 7` ou `abs(x) > 7` | Assis ou debout |
| Sinon | Position stable |

---

### Partie 9 — Intégration finale dans MainActivity

Ajouter dans `MainActivity.java` la méthode commune d'ouverture des fragments :

```java
private void openFragment(Fragment fragment) {
    getSupportFragmentManager()
            .beginTransaction()
            .replace(R.id.fragment_container, fragment)
            .commit();
}
```

Relier tous les menus aux fragments correspondants dans le gestionnaire de menu.

---

## Tests et validation

| Test | Action | Résultat attendu |
|------|--------|-----------------|
| 1 — Liste capteurs | Ouvrir menu Sensors | Affichage de Name, Vendor, Type, Resolution, Power, etc. |
| 2 — Température | Ouvrir menu Température | Valeur et graphe évoluent |
| 3 — Humidité | Ouvrir menu Humidité | Valeur et graphe évoluent |
| 4 — Proximité | Approcher un objet | Valeur proche de 0 |
| 5 — Magnétique | Tourner le téléphone | Norme du champ varie |
| 6 — Accéléromètre | Poser à plat | Valeur proche de 9,81 m/s² |
| 7 — Gravité | Comparer avec accéléromètre | Uniquement la composante gravitationnelle |
| 8 — Gyroscope | Faire tourner le téléphone | Valeurs en rad/s |
| 9 — Pas | Marcher avec le téléphone | Compteur augmente |
| 10 — Boussole | Tourner le téléphone | Direction change |
| 11 — Activité | Tester marche, saut, stable | Activité correctement détectée |

---

## Synthèse pédagogique

Ce lab montre comment une application Android peut exploiter les capteurs embarqués d'un téléphone :

- La première partie introduit la découverte des capteurs disponibles
- La deuxième partie ajoute la visualisation graphique des mesures
- Les parties suivantes exploitent les capteurs de mouvement
- La dernière partie applique une logique de classification d'activité

> **Note :** La reconnaissance d'activité repose sur des seuils simples. Pour une application industrielle ou scientifique, il serait nécessaire de collecter des données réelles, d'annoter les activités, d'entraîner un modèle d'apprentissage automatique et d'évaluer sa précision.
