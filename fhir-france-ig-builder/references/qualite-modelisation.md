# Qualité de modélisation : profils, exemples, documentation, build

SUSHI et l'IG Publisher vérifient que l'IG est bien formé, pas que les choix de modélisation sont les bons. Les règles ci-dessous couvrent les erreurs qui passent le build.

## Règles de profil

- Ne pas répéter la base. Pas de cardinalité identique à celle de la ressource de base ou du profil parent, pas de règle écrite deux fois dans le même profil, pas d'invariant recopié de la spécification de base (il s'applique déjà). Redéclarer des types à choix (`value[x]`, `definition[x]`) sous forme de slices peut même faire rejeter des instances valides par le validateur.
- La force de binding dit ce qui est imposé. `required` : une liste fermée. `extensible` : utiliser le ValueSet quand un code convient, un autre code sinon - le validateur ne distingue alors plus une faute de frappe d'un nouveau code légitime. `preferred` : une recommandation. `example` : aucune recommandation. Toujours écrire la force explicitement.
- Ne pas restreindre un ValueSet de base sans raison. Restreindre le ValueSet lié dans la ressource de base à un sous-ensemble de ses codes rend invalides des ressources valides, et impose de modifier le ValueSet à chaque nouveau cas d'usage.
- Ne pas lier un élément codé répétable en entier. Un binding `required` ou `extensible` sur un CodeableConcept `0..*` ou `1..*` s'applique à chaque répétition et interdit d'y ajouter des codes d'autres systèmes. Découper l'élément en slices (slicing ouvert) et lier la slice.
- Contraindre les références au profil national. Quand un profil existe pour la cible (FR Core ou un guide ANS), écrire `only Reference(FRCorePatientProfile)` plutôt que la ressource de base, et de la même façon sur les profils frères qui pointent vers la même cible. Le validateur vérifie alors la cible, et le publisher renvoie vers sa page. Ne garder la ressource de base que lorsqu'aucun profil n'existe.
- Réutiliser avant d'étendre. Avant une extension, vérifier qu'aucun élément existant ne porte la donnée, puis chercher une extension standard : <https://hl7.org/fhir/extensions/> et <https://packages2.fhir.org/xig?type=ext>.
- Un profil avec des bindings additionnels plutôt que plusieurs profils quand seul le ValueSet change selon le contexte (`^binding.additional` avec un `usage`).
- Ne pas donner un nom générique à un profil étroit. Un profil `MyTask` écrit pour un seul workflow bloque plus tard un vrai profil `Task` de base, et son canonical est déjà dans les systèmes des implémenteurs.
- Must Support uniquement pour ce que tous les systèmes doivent gérer, et l'IG précise ce que Must Support signifie pour les émetteurs et les récepteurs.

## Exemples

Les implémenteurs recopient les exemples : un exemple faux se propage.

- Chaque profil a au moins un exemple (`Usage: #example`) qui renseigne ses éléments obligatoires et Must Support, et déclare le profil dans `meta.profile`.
- Lire chaque exemple comme une histoire : les dates sont-elles dans l'ordre (prescription avant dispensation, début avant fin) ? Un clinicien reconnaîtrait-il la situation ? Les exemples frères d'un workflow suivent-ils le même patient dans la même suite d'événements ?
- Pas d'identifiant métier inventé pour une ressource qui n'en reçoit pas en pratique : l'`id` de la ressource assure déjà l'unicité.
- Les codes et displays sont réels : les vérifier sur le serveur de terminologies plutôt que les écrire de mémoire.

## Documentation

- La `Description` de chaque profil dit à quoi il sert, formulée comme celle de ses profils frères.
- Un profil a une page d'introduction et de notes (`StructureDefinition-<id>-intro.md` et `-notes.md`) quand les vues formelles ne suffisent pas pour l'utiliser : ce que représente la ressource, ce qui la référence, un exemple valide minimal.
- Un nouveau système d'identifiants a un `NamingSystem` qui explique qui l'attribue et ce qu'il identifie.
- Un `ConceptMap` indique si sa correspondance est complète ou appelée à s'étendre.
- Quand l'IG crée son propre artefact plutôt que d'en réutiliser un existant, la description dit pourquoi.

## Hygiène du build et de la QA

- Ne jamais modifier les scripts du publisher (`_genonce.sh`, `_genonce.bat`, `_updatePublisher.*`, `_build.*`) : ils sont écrasés à la mise à jour et la modification est perdue. Pour la mémoire, définir plutôt `JAVA_TOOL_OPTIONS` (ex. `-Xmx8g`).
- Masquer un warning ne le corrige pas. Chaque ligne ajoutée à `input/ignoreWarnings.txt` a une justification. Si le warning vient d'un bug du publisher qui sera corrigé, attendre la correction plutôt que désactiver la validation définitivement - elle n'est jamais réactivée.
- `ignoreWarnings.txt` compare avec `%`, pas avec des regex : `%foo%` contient, `foo%` commence par, `%foo` finit par, `foo` exact.

## Vérification

- [ ] Aucune cardinalité, règle ou invariant qui répète la base ou le profil parent.
- [ ] Chaque binding a une force explicite qui correspond à ce qui est imposé.
- [ ] Aucun ValueSet de base restreint sans raison indiquée.
- [ ] Aucun binding `required`/`extensible` sur un CodeableConcept répétable entier.
- [ ] Les références pointent vers les profils nationaux quand ils existent, de façon cohérente entre profils frères.
- [ ] Chaque nouvelle extension a été confrontée aux éléments existants et aux extensions standard.
- [ ] Chaque profil a un exemple cohérent avec `meta.profile`.
- [ ] Chaque nouveau système d'identifiants a un `NamingSystem`.
- [ ] Aucun script du publisher modifié ; chaque nouvelle ligne de `ignoreWarnings.txt` est justifiée.
