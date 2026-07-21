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


---

### Schéma

Voir le document schéma en dessous de docs qui détaille le fonctionnement de la veille.


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
- **Description**

> Important : l’IA n’analyse **pas** l’audio, ni le contenu vidéo lui-même. La classification repose sur les **champs textuels** disponibles.

### 1.2 Quels réseaux sociaux / plateformes ?

- **Instagram** (périmètre explicitement documenté à ce stade), dans un premier temps.


### 1.3 Combien de médias sont suivis ?

- **69 médias** différents sont suivis (périmètre opérationnel actuel), qui cummulent plus de 50 millions d'abonnés, la liste nominale des 69 médias est disponible sur le fichier 'liste de médias' avec la méthodologie de sélection.

#### Répartition par type de média (69)

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


### 1.4 Quelle volumétrie ?

- **~9000 publications Instagram / mois** (ordre de grandeur)


### 1.5 Limites du corpus (structurelles)

**Limites liées aux plateformes et aux champs :**
- Analyse limitée aux **champs textuels** fournis par la source (titre/description/extrait).
- Pas d’analyse des éléments non textuels (audio/vidéo/images), ce qui peut introduire :
  - des **faux négatifs** (jeunesses présentes mais non mentionnées),
  - des **ambiguïtés** (ironie, références culturelles, contexte absent du texte).

**Limites liées au périmètre médias :**
- Le corpus reflète **les 65 médias configurés**.
- Les résultats ne doivent pas être interprétés comme  “la totalité des médias français” au sens exhaustif.

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


### 2.4 Automatisation (état actuel)

- La collecte est aujourd’hui **semi-automatisée** (infrastructure RSS + outil), mais le déclenchement reste **manuel**.


---

## 3. Analyse IA

Cette section documente **la brique principale** : comment un LLM transforme des items collectés en labels + justifications, dans un format exploitable.

### 3.1 Modèles utilisés

Modèle mentionné dans la documentation projet :
- **OpenAI — “Mini 4o”**


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
- `N/A` si la publication n'est pas lié aux jeunesses


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
- `Récupéré le`
- `Publié à`
- `Source` (média)
- `Type de média`
- `Tittre`
- `URL`
- `Extrait`
- `Présence des jeunesses`
- `Présence des jeunesses Justification`
- `Thématique`
- `Thématique Justification`
- `Le récit`
- `Le récit Justification`


---

## 4. Export des données

### 4.1 Données exportées

Les exports contiennent :
- les **métadonnées** (source, dates, URL),
- les **entrées textuelles** (titre, extrait/description),
- les **sorties IA** (labels + justifications).

### 4.2 Format

Format : **Excel** 


### 4.3 Structure tabulaire

La structure est un tableau “1 ligne = 1 item analysé”, avec des colonnes stables (cf. §3.8).

---

## 5. Traitement des données

Le traitement consolide les exports IA pour produire un jeu de données “master” et des indicateurs. Un **Excel master** est utilisé comme point de centralisation.

### 5.1 Pipeline Excel / Power Query (attendu)

Étapes:
1. **Import** des exports IA (Excel) dans Power Query.
2. **Nettoyage** (types, champs vides, titres tronqués, dates).
3. **Normalisation** (noms médias, types de médias, formats de date, URLs).
4. **Dédoublonnage** (même URL, même titre+date, etc.).
5. **Historisation** (conserver les versions / cycles de collecte).
6. **Calcul d’indicateurs** (agrégations, ratios, séries temporelles).
7. **Sortie** vers table “finale” alimentant le dashboard.

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

### 6.2 Tableau final & dashboard (à date)

Éléments en interne :
- **Dashboard principal**
- **Tableau final**

Éléments partagé en externe:
- **Brief mensuel**
- **Baromètre visuel**

---

## 7. Validation et amélioration continue

La VeilleMedIA Jeunesses s'inscrit dans une démarche d'amélioration continue. Depuis le lancement du prototype, plusieurs campagnes de validation ont été menées afin d'évaluer la qualité des classifications produites par l'IA, d'identifier les erreurs récurrentes et d'améliorer progressivement les prompts ainsi que les catégories d'analyse.

Ces campagnes ne sont **pas réalisées de manière systématique** à chaque exécution de la veille. Elles interviennent lors des principales évolutions du prototype ou de la méthodologie, dans une logique d'itération continue.


### 7.1 Historique des campagnes de validation

À ce jour, **trois campagnes de validation** ont été réalisées.


**Décembre 2025 — Première revue exploratoire**

Un mois après le lancement du prototype, une première revue qualitative a été réalisée.

Cette première étape n'avait pas vocation à mesurer précisément les performances du système, mais à identifier les erreurs les plus fréquentes produites par l'IA.

Deux bénévoles ont parcouru un ensemble de classifications afin d'observer les principaux problèmes rencontrés, notamment :

- erreurs de détection des contenus liés aux jeunesses ;
- ambiguïtés dans certaines catégories ;
- classifications incohérentes ou peu pertinentes.

Les observations recueillies ont servi de base aux premières améliorations des prompts et de la logique de classification.

> **Remarque :** cette première campagne n'a pas fait l'objet d'un protocole formalisé ni d'une mesure quantitative comparable aux campagnes suivantes. Elle constitue une première étape exploratoire dans le développement du prototype.


**Mars 2026 — Première campagne de validation quantitative**

Une seconde campagne de validation a été menée selon une méthodologie plus structurée.

Un échantillon de publications a été annoté indépendamment par plusieurs annotateurs humains avant d'être comparé :

- entre les annotateurs eux-mêmes ;
- avec les classifications produites par l'IA.

Cette campagne a permis de :

- mesurer les performances du système sur un jeu de données défini ;
- identifier les principales sources d'erreur ;
- améliorer les prompts ;
- affiner certaines catégories d'analyse.

Les résultats obtenus ont conduit à une diminution significative du taux d'erreur observé.


**Juillet 2026 — Validation de confirmation**

Une troisième campagne de validation a été réalisée selon une méthodologie similaire afin d'évaluer les améliorations apportées depuis la campagne précédente.

Cette nouvelle revue a permis :

- de confirmer les progrès réalisés ;
- d'identifier de nouveaux cas limites ;
- de poursuivre l'amélioration des prompts et des classifications.


### 7.2 Méthodologie de validation

Les campagnes de validation reposent sur les étapes suivantes :

1. Constitution d'un échantillon de publications.
2. Annotation indépendante par plusieurs annotateurs.
3. Comparaison des annotations humaines.
4. Comparaison entre les annotations humaines et les résultats produits par l'IA.
5. Identification des erreurs récurrentes.
6. Amélioration des prompts, des catégories et de la logique de classification.

Cette démarche permet d'améliorer progressivement la robustesse du système tout en documentant les principales limites observées.


### 7.3 Résultats

Les résultats détaillés de chaque campagne de validation sont présentés ci-dessous.

Cette section a vocation à regrouper notamment :

- le taux d'erreur global observé ;
- les principaux types d'erreurs identifiés ;
- les évolutions du système après chaque campagne ;
- les améliorations obtenues entre deux validations.

**À compléter.**


### 7.4 Limites

Les campagnes de validation restent **ponctuelles** et ne constituent pas un contrôle qualité systématique de l'ensemble des classifications produites par la veille.

Par ailleurs, certaines dimensions de l'analyse — notamment l'identification des récits ou la catégorisation de certains contenus — comportent une part d'interprétation susceptible de conduire à des divergences entre annotateurs humains.

L'objectif de cette démarche n'est donc pas d'obtenir une classification parfaite, mais d'améliorer continuellement la cohérence, la transparence et la robustesse méthodologique du système.

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
