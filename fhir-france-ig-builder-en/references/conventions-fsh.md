# FSH writing conventions

## FSH grammar

Reference documentation: <https://build.fhir.org/ig/HL7/fhir-shorthand> (FHIR Shorthand). Convention point not to forget: **FSH comments use `//`, never `#`**.

## Naming conventions by artifact type

Authoritative source: <https://interop.esante.gouv.fr/ig/documentation/mod_bonnes_pratiques.html#modèle-de-nommage-par-ressource-fhir> — check that the version consulted is the latest before settling an ambiguous case, and re-verify the cited section number.

| Artifact | FSH name | `Id` | Filename suffix |
|---|---|---|---|
| `Profile` | PascalCase | kebab-case | `...Profile.fsh` |
| `Extension` | PascalCase | kebab-case | `...Extension.fsh` |
| `Instance` (example, `Usage: #example`) | PascalCase | UUID for a patient/clinical resource (format `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`) — not required for a `CapabilityStatement` or a `SearchParameter` | `...Example.fsh` |
| `Instance` (`InstanceOf: CapabilityStatement`) | PascalCase | kebab-case | `...Instance.fsh` |
| `Instance` (`InstanceOf: SearchParameter`) | PascalCase; `Id` in `Resource-code` format; `* code` in camelCase | — | `...SearchParameter.fsh` |
| `Logical` | PascalCase | kebab-case | `...Logical.fsh` |

For an `Instance`, the `InstanceOf` and `Usage` (`#example`, `#definition`, or `#inline`) fields are required.

## Which profile to inherit from

Before creating a new `Profile`, check whether an existing profile (FR Core, an ANS référentiel guide...) already covers all or part of the need, and inherit from it rather than starting from the base FHIR resource. Catalogue of existing French IGs/profiles: see the `fhir-france-implementer-en` skill and its `references/catalogue-igs.md` file — not duplicated here.

## Aliases

All FSH aliases must be centralized in a **single `aliases.fsh` file** — do not define aliases anywhere else.

## Terminology bindings

- The main binding (extensible or required depending on the case) goes in the element's default binding.
- Additional bindings (`additional binding`) are reserved for alternative ValueSets — do not use them as the default binding.
- Choose the strength for what senders can actually provide: `required` only for a closed list that every sender can use. Tightening it after publication is a breaking change (`references/changements-incompatibles.md`).
- Before creating a local CodeSystem or code, search the international terminologies (SNOMED CT, LOINC...) and the existing French terminologies.
- A ValueSet or CodeSystem that no binding, ValueSet or ConceptMap uses is dead weight: bind it or remove it.
- Naming conventions for the terminologies themselves (`TRE_`, `JDV_`, `ASS_`) and reference servers (SMT, IG Terminologies): see `fhir-france-implementer-en/references/terminologies.md` — do not duplicate these conventions here, this skill only covers FSH writing.

## Consistency with `sushi-config.yaml`

- Canonical URLs used in FSH files must match the `canonical` value of `sushi-config.yaml`.
- Do not hand-write `^url` on an artifact: SUSHI derives it from `canonical` and the `Id`, and a hand-written one drifts when either changes.
- The `publisher` declared in `CapabilityStatement` resources must match the `publisher` of `sushi-config.yaml`.

## Build requirement

**After any change to a `.fsh` file or `sushi-config.yaml`, run `sushi .` and fix all errors before committing or pushing.** Never leave an unresolved SUSHI error in a commit.

## Verification

- [ ] No `#` comments in `.fsh` files (only `//`).
- [ ] Naming (FSH name, `Id`, filename suffix) conforms to the table above for every artifact.
- [ ] All aliases are in `aliases.fsh`, none elsewhere.
- [ ] No residual placeholder (`xxx`, `TODO`, `TBD`, `FIXME`, `à compléter`, `<insert`, `placeholder`).
- [ ] No hand-written `^url`.
- [ ] Every new ValueSet and CodeSystem is used by a binding, a ValueSet or a ConceptMap.
- [ ] Every change to an artifact already published was checked against `references/changements-incompatibles.md`.
- [ ] `sushi .` runs without error.
