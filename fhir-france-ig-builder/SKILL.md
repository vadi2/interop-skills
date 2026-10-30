---
name: fhir-france-ig-builder
description: "Guide les auteurs/éditeurs d'Implementation Guide (IG) FHIR français dans la rédaction et la publication d'un IG — démarrer un nouvel IG à partir du repo modèle ANS, conventions FSH/SUSHI (nommage, alias, bindings), workflow Git/GitHub (branche + PR, template ANS), préparation d'une release (correspondance statut CI-SIS ↔ sushi-config.yaml/publication-request.json, change-log, milestones). Ne couvre pas la consommation de ressources FHIR déjà publiées (quelle version FHIR utiliser, quels IGs existent, terminologies, gouvernance CI-SIS) — voir le skill `fhir-france-implementer` pour ça. Utilise ce skill proactivement dès qu'un utilisateur rédige ou publie un IG FHIR français — crée un nouveau profil/extension/exemple FSH, configure un sushi-config.yaml, prépare une release, ou ouvre une PR sur un repo ansforge/Interop-Sante."
---

# Rédiger et publier un IG FHIR français

> **Dernière mise à jour du contenu : 2026-10-30**
> Ce paysage évolue (nouvelles versions du repo modèle, évolutions du publisher, de SUSHI, des bonnes pratiques ANS). Vérifie systématiquement s'il existe des versions plus récentes des points ci-dessous avant de répondre avec certitude.

**Doublon anglais synchronisé** : `fhir-france-ig-builder-en/SKILL.md` en est la traduction. Toute mise à jour de contenu ici doit être répercutée là-bas (et inversement).

## Objectif de ce skill

Ce skill s'adresse aux **auteurs et éditeurs d'Implementation Guides FHIR français** — rédaction FSH/SUSHI, structuration du projet, workflow Git/GitHub, préparation d'une release — par opposition aux développeurs qui consomment ou produisent des ressources FHIR conformes à un IG déjà publié (pour ça, voir le skill `fhir-france-implementer`). Il aide à retrouver **la bonne convention ou la bonne étape au bon moment** plutôt que de laisser l'utilisateur réinventer une structure de projet ou un process de release déjà établi par l'ANS.

**Hors périmètre** : quelle version FHIR utiliser, quels IGs existent déjà, où trouver les terminologies françaises, comment est gouverné le CI-SIS — voir le skill `fhir-france-implementer` et ses fichiers `references/`.

## 1. Démarrer un nouvel IG

Toujours partir d'un repo squelette plutôt que de construire une structure de projet à la main. Détails complets : `references/demarrage-ig.md`.

- Squelette de base : **`FHIR/sample-ig`** pour un IG hors écosystème ANS/Interop-Santé, **`ansforge/IG-modele`** pour un IG ANS/ansforge, **`Interop-Sante/hl7.ig.fr.sample`** pour un IG Interop-Santé (`ig.ini`, `sushi-config.yaml`, scripts de génération).
- Template (`ig.ini` > `template`) : `fhir.base.template` par défaut (neutre, sans logo) ; `hl7.fhir.template` seulement si l'apparence officielle HL7/FHIR est explicitement souhaitée. Ne pas imposer de style propre à un organisme par défaut.
- Les menus se déclarent dans `sushi-config.yaml` (clé `menu:`), jamais via un `menu.xml`.
- Les fichiers `.md` de `input/pagecontent/` doivent commencer au niveau `###` — les niveaux `#` et `##` sont générés automatiquement par le publisher.

## 2. Conventions d'écriture FSH

Détails complets, conventions de nommage par type d'artefact et source faisant autorité : `references/conventions-fsh.md`.

- Commentaires FSH : `//`, jamais `#`.
- Avant de créer un nouveau `Profile`, vérifier si un profil existant (FR Core, guide référentiel ANS...) couvre déjà le besoin et en hériter : voir le skill `fhir-france-implementer` et son fichier `references/catalogue-igs.md` (pas dupliqué ici).
- Tous les alias dans un unique `aliases.fsh` — ne pas en définir ailleurs.
- Avant de créer un CodeSystem ou un code local, chercher dans les terminologies internationales (SNOMED CT, LOINC...) et les terminologies françaises existantes : un code local remplacé ensuite par un code standard est un changement incompatible pour tous les émetteurs.
- Binding de terminologie par défaut : le binding principal (extensible/required) ; réserver les bindings additionnels aux ValueSets alternatifs.
- Conventions `TRE_`/`JDV_`/`ASS_` pour les terminologies : voir `fhir-france-implementer/references/terminologies.md` (pas dupliqué ici).
- **Après toute modification d'un `.fsh` ou de `sushi-config.yaml`, lancer `sushi .` et corriger toutes les erreurs avant de committer ou pousser.**

Erreurs de modélisation qui passent le build - répétition de la base, force de binding, références, exemples, documentation, `ignoreWarnings.txt` : `references/qualite-modelisation.md`.

- Contraindre les références au profil FR Core ou ANS de la cible quand il existe, pas à la ressource de base.
- Chaque profil a un exemple qu'un clinicien reconnaîtrait, avec `meta.profile`.
- Ne jamais modifier les scripts du publisher (`_genonce.*`, `_updatePublisher.*`), et justifier chaque ligne ajoutée à `ignoreWarnings.txt`.

## 3. Workflow Git/GitHub

Détails complets et template de PR ANS exact : `references/workflow-git-github.md`.

- Ne jamais éditer directement sur `main` : créer une branche dédiée + une PR pour chaque changement, sauf instruction contraire explicite.
- Toujours confirmer avec l'utilisateur la branche cible et le dépôt exact avant toute opération Git (branching, push, PR).
- Avant de créer une PR sur un repo ANS, lire `.github/pull_request_template.md` du repo cible et lui donner la priorité s'il diffère du template standard reproduit dans `references/workflow-git-github.md`.
- Après un push, attendre la fin du déploiement GitHub Pages avant de lire `qa.html`.

## 4. Préparer une release

Tableau de correspondance statut CI-SIS ↔ champs de configuration, et étapes détaillées : `references/preparation-release.md`.

- Le `status` de `sushi-config.yaml`, son `releaseLabel`, et le `status`/`mode` de `publication-request.json` doivent correspondre au statut CI-SIS visé (draft, public-comment, for implementation, final-text, withdrawn/deprecated) — ne jamais déduire ces valeurs par analogie, toujours vérifier le tableau de correspondance.
- Le `change-log.md` doit lister les PRs mergées depuis la version précédente, sans placeholder résiduel.
- Chaque PR citée dans le change-log doit avoir la milestone de la version assignée.
- Si un outil d'audit automatisé de ces étapes est disponible dans l'environnement, l'utiliser pour vérifier le travail plutôt que de tout revérifier manuellement.

## 5. Modifier un artefact déjà publié

Types de changements incompatibles, alternative compatible à chercher d'abord, et règles de transition : `references/changements-incompatibles.md`.

- SUSHI et la QA de l'IG Publisher restent au vert sur la plupart des changements incompatibles : les vérifier à la rédaction et en revue.
- Comparer avec le dernier tag publié, pas avec `main`. Un artefact absent de toute version publiée peut encore évoluer librement.
- Ne jamais réutiliser un code publié avec un nouveau sens ni modifier son display : ajouter un nouveau code et retirer l'ancien.
- Un changement incompatible maintenu a une entrée dédiée "Changements incompatibles" dans `change-log.md`, avec une table ancien -> nouveau.

## 6. Sources et documentation de référence

Liens vers la documentation FHIR IG Publisher, la grammaire FSH, le code source des outils (HAPI FHIR, IG Publisher, SUSHI) et les organisations GitHub de référence : `references/sources-documentation.md`.

## Retours sur les specs

Si une convention ou une étape décrite ici pose problème (ambiguïté, question d'implémentation, suggestion d'amélioration), les retours sont les bienvenus via les **issues GitHub du repo concerné** (ex. `github.com/ansforge/<repo>/issues`) ou via une PR sur ce dépôt (`github.com/ansforge/interop-skills`) pour faire évoluer ce skill lui-même.
