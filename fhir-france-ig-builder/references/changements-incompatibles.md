# Modifier un artefact déjà publié

Dès qu'un profil, une extension, un ValueSet, un CodeSystem ou un code a figuré dans une version publiée de l'IG, des implémenteurs s'appuient dessus : instances en production, requêtes, IGs dépendants qui en héritent ou y font référence. La plupart des changements incompatibles ne sont détectés ni par SUSHI ni par la QA de l'IG Publisher - les deux restent au vert - ils doivent donc être repérés à la rédaction ou en revue.

## Le changement est-il incompatible ?

Comparer avec la dernière version publiée, pas avec `main`. Un artefact présent uniquement sur `main`, dans aucune version publiée, peut encore évoluer librement.

```bash
tag=$(git tag --sort=-v:refname | head -1)
git show "$tag:<chemin>"            # l'artefact tel que publié
git diff "$tag"...HEAD -- <chemin>  # ce que la branche change par rapport à lui
```

Un changement est incompatible si une instance, une requête ou un IG dépendant, valide avec la version publiée, cesse de l'être ou change de sens. Un assouplissement peut aussi casser : les récepteurs doivent désormais accepter des valeurs qu'ils n'avaient jamais à traiter.

## Cas fréquents, et l'alternative compatible

| Changement | Pourquoi il casse | Question à se poser d'abord |
|---|---|---|
| Profil, extension, ValueSet ou CodeSystem renommé, fusionné, scindé ou supprimé (`Id`, canonical) | Tous les `meta.profile`, `Coding.system`, bindings et IGs dépendants qui pointent vers l'ancien canonical ne se résolvent plus | Le canonical peut-il rester, en ne changeant que le titre et la description ? Le périmètre (un cas d'usage, une équipe) a-t-il été inscrit dans l'`Id`, imposant un renommage plus tard ? |
| Code existant réutilisé avec un nouveau sens, ou renuméroté | Les données stockées changent de sens sans bruit | Ajouter un nouveau code et retirer l'ancien (`status`/`inactive`) |
| Display d'un code publié modifié | Le validateur signale l'ancien display comme incorrect, les émetteurs qui l'ont recopié échouent | Est-ce une vraie erreur ? Corriger via une designation, ou un nouveau code, plutôt que réécrire le display |
| Code local remplacé ensuite par un code international (SNOMED CT, LOINC...) | Tous les émetteurs changent de système et de code | À l'ajout du code : chercher d'abord dans les terminologies internationales |
| Force de binding renforcée, ou binding vers un autre ValueSet | Des instances en production deviennent invalides | Un binding additionnel, ou un ValueSet plus large, suffirait-il ? |
| Cardinalité ou Must Support renforcé | Les émetteurs qui ne renseignaient pas l'élément échouent | Tous les systèmes qui envoient cette ressource ont-ils réellement la donnée ? |
| Cible de référence restreinte (`Reference(Patient)` -> `Reference(MonProfilPatient)`) | Les instances qui pointent vers d'autres profils échouent | L'annoncer une version à l'avance |
| Élément déplacé, type changé, ou contenu déplacé vers une autre ressource | Les mappings et les requêtes cassent | La donnée est-elle sur la bonne ressource dès le départ ? |
| Chaîne de version, `canonical` ou `id` du package modifié | Toutes les références `system|version` et tous les packages dépendants cassent | La version suit-elle SemVer dès la première publication ? |

## Règles de transition quand le changement est inévitable

1. Chercher d'abord une version compatible : garder le canonical et changer le titre, ajouter un code plutôt qu'en redéfinir un, ajouter un binding plutôt que renforcer le binding principal.
2. Garder résolvable ce qui a été publié : retirer les codes et artefacts (`status = retired`, `inactive`) plutôt que les supprimer.
3. Accepter le remplaçant avant de supprimer l'original (le ValueSet contient les nouveaux codes à côté des anciens pendant au moins une version).
4. Annoncer une restriction une version à l'avance quand elle peut attendre.
5. Le change-log comporte une entrée dédiée "Changements incompatibles" qui dit ce que les implémenteurs doivent changer, avec une table ancien -> nouveau pour les codes, canonicals ou systèmes d'identifiants (un ConceptMap si la correspondance est volumineuse).
6. Si d'autres IGs dépendent de celui-ci, y rechercher l'artefact et les mettre à jour dans le même cycle de release.

## Vérification

Partir des lignes que la branche supprime ou réécrit depuis le dernier tag publié - la plupart des changements incompatibles s'y trouvent :

```bash
git diff "$tag"...HEAD -- input/fsh sushi-config.yaml | grep '^-[^-]'
```

Par rapport au dernier tag publié, la branche :

- [ ] Ne renomme ni ne supprime aucun `Id`, canonical ou `id` de package, et ne modifie `version` que pour l'incrément prévu.
- [ ] Ne supprime, ne renumérote ni ne réutilise avec un nouveau sens aucun code publié ; un code remplacé est retiré (`status`/`inactive`), pas supprimé.
- [ ] Ne modifie le display d'aucun code publié.
- [ ] Ne renforce aucune force de binding et ne fait pointer aucun binding publié vers un autre ValueSet.
- [ ] N'augmente aucune cardinalité minimale, ne réduit aucune maximale, n'ajoute aucun Must Support sur un élément publié.
- [ ] Ne restreint aucune cible de référence.
- [ ] Ne déplace aucun élément et ne change aucun type de données.
- [ ] Signale dans `change-log.md` tout assouplissement que les récepteurs doivent désormais gérer (nouveaux codes, cardinalité ou binding élargis).

Pour chaque changement incompatible maintenu :

- [ ] La PR indique quelle alternative compatible a été envisagée et pourquoi elle ne convient pas.
- [ ] Le remplaçant est accepté pendant au moins une version avant la suppression de l'original, ou la PR explique pourquoi ce n'est pas possible.
- [ ] `change-log.md` a une entrée "Changements incompatibles" qui dit ce que les implémenteurs doivent changer, avec une table ancien -> nouveau.
- [ ] Les IGs dépendants ont été parcourus pour y chercher l'artefact, et ceux qui l'utilisent sont mis à jour dans le même cycle de release.
