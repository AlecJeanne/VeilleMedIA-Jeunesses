# Methodology

## Introduction

Cette page documente la **méthodologie technique** du projet *Veille IA Jeunesses* et décrit le **pipeline de bout en bout** — depuis la collecte des contenus jusqu’à la production d’un jeu de données exploitable (exports, traitements, indicateurs, visualisations).

Objectif : permettre à un lecteur extérieur de comprendre **ce qui est analysé**, **comment**, **avec quelles hypothèses**, et **où se situent les limites** — sans lire le code.

---

## Vue d’ensemble du pipeline

### Étapes principales

1. **Sources**  
   Définition du périmètre : liste des médias suivis, plateformes cibles, types de contenus et champs réellement analysés.

2. **Collecte**  
   Récupération des publications via **flux RSS** (via une plateforme externe), selon une cadence opérationnelle (actuellement manuelle).  
   La collecte est pilotée par une **configuration JSON** qui décrit les sources et les paramètres nécessaires au traitement.

3. **Analyse IA**  
   Pour chaque item collecté (ex. publication Instagram), un modèle LLM produit des **classifications** et **justifications** à partir des champs textuels disponibles (titre/description).  
   À ce stade, le pipeline réalise notamment :
   - Détection si la pu lication est “**lié aux jeunesses / non lié”**
   - Attribution d’une **thématique**
   - Attribution d’un **récit** (ex. Positif / Négatif / Neutre / N/A)
   - Génération d’**explications** (justifications / rationales) associées à chaque décision

4. **Export des données**  
   Production d’un export tabulaire (ex. fichier Excel) contenant les métadonnées source + les sorties IA (labels + justifications).

5. **Traitement des données**  
   Consolidation dans un fichier “master” et/ou un pipeline de traitement (Excel / Power Query) : nettoyage, normalisation, dédoublonnage, historisation, calcul d’indicateurs.

6. **Visualisation**  
   Alimentation d’un tableau final et d’un dashboard (baromètre, suivis temporels, répartitions, etc.).

7. **Validation**  
   Protocoles de contrôle qualité (revue humaine, campagnes d’annotation, métriques).  
   **À compléter** : la page projet mentionne cette brique, mais les métriques/protocoles détaillés ne sont pas renseignés dans les éléments actuellement consolidés.

8. **Limites et évolutions**  
   Limites de corpus, limites LLM, biais potentiels, et axes d’amélioration (techniques + validation).

---

### Schéma (Mermaid)

à compélter

---

### Données factuelles déjà établies (à date)

- Périmètre opérationnel : **~65 médias** suivis.
- Volumétrie indicative : **~9000 publications Instagram / mois** (ordre de grandeur).
- Données analysées côté IA : **texte** (notamment *titre* et *description*).
  - **Non analysé** : audio/vidéo (pas d’analyse du son ou de l’audio).
- Modèle utilisé : **OpenAI — “Mini 4o”**.
  

---


- ## 1. Sources

Cette section décrit **le corpus analysé** : plateformes, médias suivis, volumétrie, champs disponibles, et limites structurelles.

### 1.1 Quelles données sont analysées ?

Le pipeline analyse des **publications issues de médias**, collectées via des flux RSS et traitées ensuite par l’IA.

**Champs effectivement analysés par l’IA (contenu textuel) :**
- **Titre**
- **Description / extrait** (selon ce que fournit la source)

> Important : l’IA n’analyse **pas** l’audio, ni le contenu vidéo lui-même. La classification repose sur les **champs textuels** disponibles.

### 1.2 Quels réseaux sociaux / plateformes ?

- **Instagram** (périmètre explicitement documenté à ce stade)


### 1.3 Combien de médias sont suivis ?

- **65 médias** différents sont suivis (périmètre opérationnel actuel)

#### Répartition par type de média (65)

| Type de média | Nombre |
|---|---:|
| Pureplayer | 20 |
| PQN | 12 |
| Média Indépendant | 11 |
| TV | 9 |
| PQR | 8 |
| Radio | 5 |
| Magazine | 3 |
| Web | 1 |

> La liste nominale des 65 médias est disponible sur demande.

### 1.4 Quelle volumétrie ?

- **~9000 publications Instagram / mois** (ordre de grandeur)

### 1.5 Quels types de contenus sont analysés ?

- Publications Instagram de médias (au sens “posts”/publications, via les informations textuelles associées)
- Le pipeline est conçu pour fonctionner sur des items courts (titre/extrait) et produire :
  - un indicateur “lié aux jeunesses” (binaire),
  - une thématique,
  - un récit/tonalité,
  - des justifications.

### 1.6 Limites du corpus (structurelles)

**Limites liées aux plateformes et aux champs :**
- Analyse limitée aux **champs textuels** fournis par la source (titre/description/extrait).
- Pas d’analyse des éléments non textuels (audio/vidéo/images), ce qui peut introduire :
  - des **faux négatifs** (jeunesses présentes mais non mentionnées),
  - des **ambiguïtés** (ironie, références culturelles, contexte absent du texte).

**Limites liées au périmètre médias :**
- Le corpus reflète **uniquement les 65 médias configurés**.
- Les résultats ne doivent pas être interprétés comme “les médias français” au sens exhaustif.

**Limites liées à la collecte (RSS) :**
- La qualité/complétude dépend de la **disponibilité et du format** des flux RSS et de la plateforme de collecte.
- Les champs récupérés peuvent varier selon les sources (titres tronqués, descriptions manquantes, liens cassés, etc.).


---

## 2. Collecte

Cette section décrit **comment les publications sont récupérées** et injectées dans le pipeline d’analyse.

### 2.1 Fonctionnement général (RSS)

La collecte s’appuie sur des **flux RSS** :
- Les flux RSS sont **récupérés via une plateforme externe**.
- Cette plateforme est **connectée directement** à l’outil *MediaMonitor* (le système qui centralise les items à analyser).

> Le RSS sert ici de mécanisme standard pour “surveiller” des sources et récupérer des items récents de manière régulière.

### 2.2 Rôle du fichier JSON de configuration

La collecte et l’analyse sont pilotées par une **configuration JSON**.

Ce JSON contient (au minimum) :
- la **liste des sources** (médias) à suivre ;
- les **instructions nécessaires** au fonctionnement du pipeline côté outil (et, selon l’implémentation, des éléments liés aux prompts / paramètres d’analyse).



### 2.3 Fréquence de collecte

La collecte est **déclenchée manuellement** environ **tous les 4 jours**.

Paramètre opérationnel mentionné :
- récupération possible **jusqu’aux ~50 dernières publications par média** à chaque cycle.

Objectif de cette cadence :
- éviter les “trous” de collecte (ne pas louper de publications) tout en restant réaliste opérationnellement.



**À compléter :**
- critères d’inclusion (type de média, audience, orientation éditoriale, etc.) ;
- procédure de QA à l’ajout (checklist, tests de non-régression, validation manuelle).

### 2.5 Automatisation (état actuel)

- La collecte est aujourd’hui **semi-automatisée** (infrastructure RSS + outil), mais le déclenchement reste **manuel**.

**À compléter :**
- si une automatisation complète est prévue (cron/scheduler, triggers) ;
- gestion des erreurs (sources en échec, flux indisponible, retry) ;
- monitoring (logs, alerting).

---

## 3. Analyse IA

Cette section documente **la brique principale** : comment un LLM transforme des items collectés en labels + justifications, dans un format exploitable.

### 3.1 Modèles utilisés

Modèle mentionné dans la documentation projet :
- **OpenAI — “Mini 4o”**

**À compléter :**
- version exacte / identifiant modèle côté API ;
- paramètres d’inférence (temperature, max tokens, top_p, etc.) ;
- politique de mise à jour (quand/ pourquoi changer de modèle).

### 3.2 Architecture générale (logique de traitement)

Pour chaque item collecté, l’analyse suit un principe de **classification multi-étapes** :

1. **Déterminer si le contenu est “lié aux jeunesses”** (binaire).
2. **Attribuer une thématique** (catégorie unique).
3. **Attribuer un récit / tonalité** (4 modalités : Positif / Négatif / Neutre / N/A).
4. **Générer des justifications** (explanations) pour rendre les décisions auditables.

Entrées minimales (observées) :
- `Title`
- `Excerpt` / `Description`
- métadonnées source (nom média, type, dates, URL…)

Sorties attendues (observées) :
- labels (jeunesses / thématique / récit)
- justifications associées à chaque label
- format tabulaire (voir §3.8)

> Point clé de transparence : le pipeline privilégie une logique “décision + justification” plutôt qu’un simple label.

### 3.3 Prompts (principes)

Les prompts sont écrits pour :
- **contraindre le format** (réponses parmi un ensemble de valeurs autorisées),
- expliciter des **règles décisionnelles**,
- lister des **erreurs fréquentes** et des garde-fous,
- demander une décision **binaire/unique** (éviter les sorties ambiguës).

**À compléter :**
- organisation des prompts (fichiers, versioning, templates) ;
- stratégie de “prompt chaining” (un prompt par tâche vs prompt unique) ;
- mécanismes de fallback (ré-essais, re-prompting).

### 3.4 Classification “Jeunesses” (binaire)

**Objectif :**
Déterminer si un titre concerne explicitement les jeunesses.

**Sorties autorisées :**
- `Titre lié aux jeunesses`
- `Titre non lié aux jeunesses`

**Définition opérationnelle (résumé) :**
Un titre est considéré comme “lié aux jeunesses” s’il traite d’un sujet impliquant directement des personnes de **moins de 30 ans**, avec jeunesse **centrale** (enfants, ados, étudiants, jeunes adultes, mineurs), y compris lorsqu’un âge est explicitement mentionné et pertinent.

**Règles clés (garde-fous) :**
- Ne pas classer “lié” si :
  - une personne jeune est mentionnée mais **la jeunesse n’est pas le sujet** (ex. célébrité),
  - le sujet est populaire auprès des jeunes sans mention explicite,
  - le mot “jeunesse” est cité marginalement hors-sujet.

**Exemples donnés :**
- Lié : “Une étudiante de 19 ans lance un média féministe”
- Non lié : “Une actrice de 23 ans dévoile son nouveau film”

### 3.5 Classification “Thématiques”

**Objectif :**
Classer le titre dans **une seule catégorie thématique**, basée sur le **sujet principal**, et non sur les personnes mentionnées.

**État de la documentation disponible :**
- la section “Thématique” existe mais la **liste complète des catégories** n’est pas consolidée dans l’extrait disponible (“Santé…” + note indiquant que la liste est conservée ailleurs).

**À compléter (indispensable pour GitHub) :**
- la liste exhaustive des thématiques (libellés exacts),
- définitions de chaque thématique,
- exemples limites (titres ambigus),
- règle de priorité en cas de multi-thèmes.

### 3.6 Classification “Récits / tonalité”

**Objectif :**
Déterminer le sentiment / perspective exprimée à l’égard des jeunes dans le titre.

**Sorties autorisées :**
- `Positif`
- `Négatif`
- `Neutre`
- `N/A`

**À compléter :**
- définitions opérationnelles (comment distinguer Neutre vs N/A),
- règles sur les titres factuels, ironiques, ou très courts.

### 3.7 Justifications générées

Pour chaque décision, le pipeline produit une **justification textuelle**, afin de :
- rendre la classification **audit-able**,
- faciliter la revue humaine,
- permettre l’amélioration continue des prompts et des règles.

**Exemple (jeunesses) :**
> “Le titre mentionne explicitement des ‘lycéens’, qui sont des jeunes (…)”

**Exemple (thématique) :**
> “Le sujet principal est la remise de diplômes au lycée (…) — donc Éducation.”

**Exemple (récit) :**
> “Le titre décrit factuellement un événement (…) sans jugement (…) — Neutre.”

### 3.8 Format de sortie (colonnes observées)

Le format d’export inclut (au minimum) :

- `Nom de la source`
- `Retrieved At`
- `Publié à`
- `Source` (média)
- `Type de média`
- `Tittre`
- `URL`
- `Excerpt`
- `Présence des jeunesses`
- `Présence des jeunesses Justification`
- `Thématique`
- `Thématique Justification`
- `Le récit`
- `Le récit Justification`


> Remarque : les colonnes “Tonalité émotionnelle” apparaissent dans le tableau d’exemple mais ne sont pas documentées ailleurs. À clarifier : est-ce une dimension effectivement produite, ou un champ prévu/non rempli ?

---

## 4. Export des données

### 4.1 Données exportées

Les exports contiennent :
- les **métadonnées** (source, dates, URL),
- les **entrées textuelles** (titre, extrait/description),
- les **sorties IA** (labels + justifications).

### 4.2 Format

Format observé :
- **Excel** (

**À compléter :**
- convention de nommage (date, source, version),
- encodage / séparateurs (si CSV en alternative),
- dossier/endpoint de sortie (Drive/SharePoint/GitHub Releases).

### 4.3 Structure tabulaire

La structure est un tableau “1 ligne = 1 item analysé”, avec des colonnes stables (cf. §3.8).

---

## 5. Traitement des données

Le traitement consolide les exports IA pour produire un jeu de données “master” et des indicateurs. Un **Excel master (SharePoint)** est utilisé comme point de centralisation.

### 5.1 Pipeline Excel / Power Query (attendu)

Étapes typiques (à documenter précisément une fois la logique confirmée) :
1. **Import** des exports IA (Excel) dans Power Query.
2. **Nettoyage** (types, champs vides, titres tronqués, dates).
3. **Normalisation** (noms médias, types de médias, formats de date, URLs).
4. **Dédoublonnage** (même URL, même titre+date, etc.).
5. **Historisation** (conserver les versions / cycles de collecte).
6. **Calcul d’indicateurs** (agrégations, ratios, séries temporelles).
7. **Sortie** vers table “finale” alimentant le dashboard.

**À compléter (indispensable) :**
- clés de dédoublonnage retenues,
- règles d’historisation (append-only vs overwrite),
- gestion des corrections manuelles,
- contrôles qualité (lignes rejetées, anomalies).

---

## 6. Visualisation

### 6.1 Indicateurs produits

**À compléter :** la page projet mentionne des dashboards/baromètre mais ne liste pas les indicateurs.
Exemples d’indicateurs typiquement attendus (à confirmer, ne pas considérer comme implémentés) :
- part des contenus “liés aux jeunesses”
- répartition thématique
- répartition des récits (positif/négatif/neutre/N/A)
- évolution temporelle (semaine/mois)
- comparaisons par type de média / média

### 6.2 Tableau final & dashboard

Éléments mentionnés :
- **Dashboard principal : à compléter**
- **Baromètre visuel : à compléter**

**À compléter :**
- outil de dashboard (Excel, Power BI, autre),
- vues principales,
- définitions des filtres (période, média, type, thématique),
- usages cibles (plaidoyer, sensibilisation, suivi interne).

---

## 7. Validation

La validation vise à mesurer et améliorer la qualité des classifications IA.

### 7.1 Protocoles existants

État dans la documentation disponible :
- section “Contrôle qualité (revue humaine, métriques)” présente mais **non renseignée**.
- mention d’un besoin de définir : échantillons, métriques (precision/recall/F1), matrice de confusion, faux positifs/négatifs.


### 7.2 Campagnes d’annotation humaine

**À compléter :**
- qui annote (rôles),
- guide d’annotation,
- process de désaccord (double annotation, arbitrage),
- stockage des labels “gold”.

### 7.3 Résultats & améliorations successives

**À compléter :**
- résultats quantitatifs (si disponibles),
- principaux cas d’erreur,
- itérations sur prompts/règles.

---

## 8. Limites et évolutions

### 8.1 Limites actuelles

**Limites corpus / collecte :**
- périmètre limité aux médias configurés ;
- dépendance aux flux RSS et à la qualité des champs fournis ;
- déclenchement manuel (latence, variabilité de cadence).

**Limites LLM :**
- dépendance au contexte textuel (titre/extrait) ;
- ambiguïtés, implicites, ironie, titres “clickbait” ;
- risque d’incohérence inter-cycles si prompts/modèles changent.

**Limites méthodologiques :**
- classification “récit” délicate sur des titres courts ;
- thématiques non documentées publiquement à ce stade (liste manquante).

### 8.2 Biais possibles

- biais de sélection des médias (liste non exhaustive) ;
- biais éditoriaux des sources (ligne, format, fréquence de publication) ;
- biais linguistiques/culturels (références, argot, implicites) ;
- biais d’exposition (certaines thématiques plus “titres-friendly”).
