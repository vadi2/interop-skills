# Changing an artifact that is already published

Once a profile, extension, ValueSet, CodeSystem or code has been in a published version of the IG, implementers have built on it: instances in production, queries, dependent IGs that inherit from it or pin it. Most breaking changes are not caught by SUSHI or the IG Publisher QA - both stay green - so they have to be caught at authoring or review time.

## Is it breaking?

Compare against the **latest published version**, not against `main`. An artifact that is only on `main` and in no published version yet can still change freely.

```bash
tag=$(git tag --sort=-v:refname | head -1)
git show "$tag:<path>"            # the artifact as published
git diff "$tag"...HEAD -- <path>  # what the branch changes relative to it
```

A change is breaking if an instance, query or dependent IG that was valid against the published version stops being valid or changes meaning. Loosening can also break: receivers must now accept values they never had to handle.

## Common kinds, and the non-breaking alternative

| Change | Why it breaks | Ask first |
|---|---|---|
| Profile, extension, ValueSet or CodeSystem renamed, merged, split or removed (`Id`, canonical) | Every `meta.profile`, `Coding.system`, binding and dependent IG pointing at the old canonical stops resolving | Can the canonical stay and only the title and description change? Was the scope (a use case, a team) written into the `Id`, forcing a rename later? |
| Existing code reused with a new meaning, or renumbered | Stored data silently changes meaning | Add a new code and retire the old one (`status`/`inactive`) instead |
| Display of a published code changed | The validator flags the old display as wrong, so senders that copied it start failing | Is it a real error? Fix through a designation, or a new code, rather than rewriting the display |
| Local code later replaced by an international one (SNOMED CT, LOINC...) | Every sender switches system and code | When the code is **added**: search the international terminologies first |
| Binding strength tightened, or bound to a different ValueSet | Production instances become invalid | Would an additional binding, or a wider ValueSet, do? |
| Cardinality or Must Support tightened | Senders that never filled the element start failing | Does every system that sends this resource actually have the data? |
| Reference target narrowed (`Reference(Patient)` -> `Reference(MyPatientProfile)`) | Instances pointing at other profiles fail | Announce it a version ahead |
| Element moved, datatype changed, or content moved to another resource | Mappings and queries break | Is the data on the right resource from the start? |
| Version string, `canonical` or package `id` changed | Every `system|version` pin and every dependent package breaks | Is the version SemVer from the first publication? |

## Transition rules when the change has to happen

1. Look for a non-breaking version first: keep the canonical and change the title, add a code rather than redefine one, add a binding rather than tighten the main one.
2. Keep what was published resolvable: retire codes and artifacts (`status = retired`, `inactive`) rather than delete them.
3. Accept the replacement before removing the original (the ValueSet contains the new codes for at least one version alongside the old ones).
4. Announce narrowing a version ahead when it can wait.
5. The change-log gets a dedicated **Breaking changes** entry saying what implementers must change, with an old-to-new table for codes, canonicals or identifier systems (a ConceptMap if the mapping is large).
6. If other IGs depend on this one, check them for the artifact, and update them in the same release cycle.

## Verification

- [ ] Every changed artifact was compared against the latest published tag, not `main`.
- [ ] No code reused with a new meaning, no display of a published code changed without a new code.
- [ ] Every breaking change has a non-breaking alternative ruled out, and a **Breaking changes** entry in `change-log.md`.
