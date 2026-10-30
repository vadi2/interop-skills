# Préparer une release d'IG FHIR (ANS)

Référence faisant autorité : <https://interop.esante.gouv.fr/ig/documentation/mod_bonnes_pratiques.html#release-dun-ig-fhir> — vérifier la version avant de trancher un cas ambigu. Documentation `publication-request.json` (HL7) : <https://confluence.hl7.org/spaces/FHIR/pages/144970227/IG+Publication+Request+Documentation>.

## Tableau de correspondance statut CI-SIS → champs de configuration

| Statut CI-SIS | `sushi-config.yaml` > `status` | `sushi-config.yaml` > `releaseLabel` | `publication-request.json` > `status` | `publication-request.json` > `mode` |
|---|---|---|---|---|
| draft | `draft` | `ci-build` | `ci-build` | N/A |
| public-comment | `draft` | `ballot` | `ballot` | `working` |
| for implementation | `active` | `trial-use` | `trial-use` | `milestone` |
| final-text | `active` | `final-text` | `final-text` | `milestone` |
| withdrawn/deprecated | `retired` | N/A | `withdrawn` ou `retired` | `withdrawal` |

De plus, en `public-comment` la `version` (sushi-config et publication-request) et le `path` (publication-request) doivent se terminer par `-ballot` ; en `for implementation`/`final-text`, aucun suffixe de version. Ne jamais déduire ces valeurs par analogie avec une release précédente — toujours revérifier contre ce tableau.

## Étapes, dans l'ordre

Lorsque l'utilisateur demande de préparer une release `X.Y.Z` :

### 1. Mettre à jour la branche avec `main`

```bash
git fetch origin main && git merge origin/main
```

### 2. Modifier `sushi-config.yaml`

- `version` : `X.Y.Z-ballot` → `X.Y.Z` (ou l'inverse selon le statut visé)
- `status` : maturity status de l'IG — voir le tableau ci-dessus
- `releaseLabel` : voir le tableau ci-dessus

### 3. Modifier `publication-request.json`

- `version` : cohérent avec `sushi-config.yaml`
- `path` : `.../X.Y.Z-ballot` → `.../X.Y.Z` (ou l'inverse)
- `status` : label de publication (différent du `status` de `sushi-config.yaml`, voir le tableau)
- `mode` : voir le tableau ci-dessus

### 4. Mettre à jour `input/pagecontent/change-log.md`

- Remplacer les placeholders par la version `X.Y.Z` et l'URL canonique.
- Lister les PRs mergées depuis la version précédente (`gh pr list --state merged`) au format `* Titre de la PR [#N](url_pr)`.
- Si c'est la première release FHIR de ce volet, ajouter une section « Versions antérieures (PDF) » listant les versions PDF antérieures (consulter le site ANS du volet concerné).
- Si la version modifie un artefact déjà publié d'une façon qui casse les implémenteurs (voir `changements-incompatibles.md`), ajouter une section dédiée **Changements incompatibles** qui dit ce qu'ils doivent changer, avec une table ancien -> nouveau pour les codes, canonicals ou systèmes d'identifiants.
- Vérifier l'absence de tout placeholder résiduel (`xxx`, `TODO`, `TBD`, `FIXME`, `<insérer`, `placeholder`, numéro d'issue générique).

### 5. Assigner la milestone `X.Y.Z` à toutes les PRs citées dans le change-log

```bash
# Récupérer le numéro de milestone
gh api repos/ansforge/[repo]/milestones
# Assigner
gh api repos/ansforge/[repo]/pulls/[N] --method PATCH -f milestone=[numéro]
```

### 6. Mettre à jour la description de la PR de release

Lister tous les fichiers modifiés (`git diff origin/main...HEAD --name-only`) et décrire chaque changement dans la PR avec le format `* [fichier] Description du changement`.

## Vérification croisée avant de considérer la release prête

- [ ] Aucun commit non intégré depuis `origin/main`.
- [ ] `sushi-config.yaml` et `publication-request.json` cohérents entre eux et avec le tableau de correspondance.
- [ ] `change-log.md` à jour, sans placeholder, avec au moins une PR listée.
- [ ] Chaque changement incompatible depuis la version précédente (comparer avec son tag) figure sous **Changements incompatibles**.
- [ ] Chaque PR citée dans le change-log a la milestone `X.Y.Z` assignée (`gh api .../pulls/N --jq '.milestone.title'`).
- [ ] Une PR de release est ouverte vers `main`.

Si un outil d'audit automatisé de ces étapes est disponible dans l'environnement, l'utiliser pour produire un rapport d'état plutôt que de tout revérifier manuellement poste par poste.
