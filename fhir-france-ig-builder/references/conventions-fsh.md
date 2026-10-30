# Conventions d'écriture FSH

## Grammaire FSH

Documentation de référence : <https://build.fhir.org/ig/HL7/fhir-shorthand> (FHIR Shorthand). Point de convention à ne pas oublier : les **commentaires FSH utilisent `//`, jamais `#`**.

## Conventions de nommage par type d'artefact

Source faisant autorité : <https://interop.esante.gouv.fr/ig/documentation/mod_bonnes_pratiques.html#modèle-de-nommage-par-ressource-fhir> — vérifier que la version consultée est la dernière avant de trancher un cas ambigu, et revérifier le numéro de section cité.

| Artefact | Nom FSH | `Id` | Suffixe de nom de fichier |
|---|---|---|---|
| `Profile` | PascalCase | kebab-case | `...Profile.fsh` |
| `Extension` | PascalCase | kebab-case | `...Extension.fsh` |
| `Instance` (exemple, `Usage: #example`) | PascalCase | UUID pour une ressource patient/clinique (format `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`) — pas exigé pour un `CapabilityStatement` ou un `SearchParameter` | `...Example.fsh` |
| `Instance` (`InstanceOf: CapabilityStatement`) | PascalCase | kebab-case | `...Instance.fsh` |
| `Instance` (`InstanceOf: SearchParameter`) | PascalCase ; `Id` au format `Ressource-code` ; `* code` en camelCase | — | `...SearchParameter.fsh` |
| `Logical` | PascalCase | kebab-case | `...Logical.fsh` |

Pour un `Instance`, les champs `InstanceOf` et `Usage` (`#example`, `#definition` ou `#inline`) sont obligatoires.

## Quel profil hériter

Avant de créer un nouveau `Profile`, vérifier si un profil existant (FR Core, un guide référentiel ANS...) couvre déjà tout ou partie du besoin, et en hériter plutôt que de repartir de la ressource FHIR de base. Catalogue des IGs/profils français existants : voir le skill `fhir-france-implementer` et son fichier `references/catalogue-igs.md` — pas dupliqué ici.

## Alias

Tous les alias FSH doivent être centralisés dans un **unique fichier `aliases.fsh`** — ne pas définir d'alias dans un autre fichier.

## Bindings de terminologie

- Le binding principal (extensible ou required selon le cas) va dans le binding par défaut de l'élément.
- Les bindings additionnels (`additional binding`) sont réservés aux ValueSets alternatifs — ne pas les utiliser comme binding par défaut.
- Choisir la force selon ce que les émetteurs peuvent réellement fournir : `required` uniquement pour une liste fermée que tous les émetteurs peuvent utiliser. La renforcer après publication est un changement incompatible (`references/changements-incompatibles.md`).
- Avant de créer un CodeSystem ou un code local, chercher dans les terminologies internationales (SNOMED CT, LOINC...) et les terminologies françaises existantes.
- Un ValueSet ou un CodeSystem utilisé par aucun binding, ValueSet ou ConceptMap est inutile : le lier ou le supprimer.
- Conventions de nommage des terminologies elles-mêmes (`TRE_`, `JDV_`, `ASS_`) et serveurs de référence (SMT, IG Terminologies) : voir `fhir-france-implementer/references/terminologies.md` — ne pas dupliquer ces conventions ici, ce skill ne couvre que la rédaction FSH.

## Cohérence avec `sushi-config.yaml`

- Les URLs canoniques utilisées dans les fichiers FSH doivent correspondre à la valeur `canonical` de `sushi-config.yaml`.
- Ne pas écrire `^url` à la main sur un artefact : SUSHI le déduit de `canonical` et de l'`Id`, et une valeur écrite à la main diverge dès que l'un des deux change.
- Le `publisher` déclaré dans les `CapabilityStatement` doit correspondre au `publisher` de `sushi-config.yaml`.

## Obligation de build

**Après toute modification d'un fichier `.fsh` ou de `sushi-config.yaml`, lancer `sushi .` et corriger toutes les erreurs avant de committer ou de pousser.** Ne jamais laisser une erreur SUSHI non résolue dans un commit.

## Vérification

- [ ] Aucun commentaire `#` dans les fichiers `.fsh` (uniquement `//`).
- [ ] Nommage (nom FSH, `Id`, suffixe de fichier) conforme au tableau ci-dessus pour chaque artefact.
- [ ] Tous les alias sont dans `aliases.fsh`, aucun ailleurs.
- [ ] Pas de placeholder résiduel (`xxx`, `TODO`, `TBD`, `FIXME`, `à compléter`, `<insérer`, `placeholder`).
- [ ] Aucun `^url` écrit à la main.
- [ ] Chaque nouveau ValueSet et CodeSystem est utilisé par un binding, un ValueSet ou un ConceptMap.
- [ ] Chaque modification d'un artefact déjà publié a été vérifiée avec `references/changements-incompatibles.md`.
- [ ] `sushi .` s'exécute sans erreur.
