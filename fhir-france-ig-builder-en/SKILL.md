---
name: fhir-france-ig-builder-en
description: "Guides authors/editors of French FHIR Implementation Guides (IG) through writing and publishing an IG — starting a new IG from the right skeleton repo (HL7 or ANS), FSH/SUSHI conventions (naming, aliases, bindings), Git/GitHub workflow (branch + PR, ANS template), release preparation (CI-SIS status ↔ sushi-config.yaml/publication-request.json mapping, change-log, milestones). Does not cover consuming already-published FHIR resources (which FHIR version to use, which IGs exist, terminologies, CI-SIS governance) — see the `fhir-france-implementer-en` skill for that. Use this skill proactively whenever a user writes or publishes a French FHIR IG — creates a new FSH profile/extension/example, configures a sushi-config.yaml, prepares a release, or opens a PR on an ansforge/Interop-Sante repo."
---

# Writing and publishing a French FHIR IG

> **Content last updated: 2026-10-30**
> This landscape evolves (new versions of the skeleton repos, publisher/SUSHI updates, ANS best practices). Systematically check whether newer versions of the points below exist before answering with certainty.

**Synchronized French twin**: `fhir-france-ig-builder/SKILL.md` is the reference version. Any content update here must be mirrored there (and vice versa).

## Purpose of this skill

This skill is aimed at **authors and editors of French FHIR Implementation Guides** — FSH/SUSHI writing, project structuring, Git/GitHub workflow, release preparation — as opposed to developers who consume or produce FHIR resources conformant to an already-published IG (for that, see the `fhir-france-implementer-en` skill). It helps find **the right convention or step at the right time** rather than letting the user reinvent a project structure or release process already established by the ANS.

**Out of scope**: which FHIR version to use, which IGs already exist, where to find French terminologies, how CI-SIS is governed — see the `fhir-france-implementer-en` skill and its `references/` files.

## 1. Starting a new IG

Always start from a skeleton repo rather than building a project structure by hand. Full details: `references/demarrage-ig.md`.

- Skeleton repo: **`FHIR/sample-ig`** for an IG outside the ANS/Interop-Santé ecosystem, **`ansforge/IG-modele`** for an ANS/ansforge IG, **`Interop-Sante/hl7.ig.fr.sample`** for an Interop-Santé IG (`ig.ini`, `sushi-config.yaml`, generation scripts).
- Template (`ig.ini` > `template`): `fhir.base.template` by default (neutral, no logo); `hl7.fhir.template` only if the official HL7/FHIR look is explicitly wanted. Do not impose an organization-specific style by default.
- Menus are declared in `sushi-config.yaml` (`menu:` key), never via a `menu.xml`.
- `.md` files in `input/pagecontent/` must start at heading level `###` — levels `#` and `##` are generated automatically by the publisher.

## 2. FSH writing conventions

Full details, naming conventions by artifact type, and authoritative source: `references/conventions-fsh.md`.

- FSH comments: `//`, never `#`.
- Before creating a new `Profile`, check whether an existing profile (FR Core, an ANS référentiel guide...) already covers the need and inherit from it: see the `fhir-france-implementer-en` skill and its `references/catalogue-igs.md` file (not duplicated here).
- All aliases in a single `aliases.fsh` — do not define any elsewhere.
- Before creating a local CodeSystem or code, search the international terminologies (SNOMED CT, LOINC...) and the existing French terminologies: a local code later replaced by a standard one is a breaking change for every sender.
- Default terminology binding: the main binding (extensible/required); reserve additional bindings for alternative ValueSets.
- `TRE_`/`JDV_`/`ASS_` terminology naming conventions: see `fhir-france-implementer-en/references/terminologies.md` (not duplicated here).
- **After any change to a `.fsh` file or `sushi-config.yaml`, run `sushi .` and fix all errors before committing or pushing.**

Modelling mistakes that pass the build - restating the base, binding strength, references, examples, documentation, `ignoreWarnings.txt`: `references/qualite-modelisation.md`.

- Constrain references to the FR Core or ANS profile of the target where one exists, not to the base resource.
- Every profile has an example that a clinician would recognise, with `meta.profile`.
- Never edit the publisher scripts (`_genonce.*`, `_updatePublisher.*`), and justify every line added to `ignoreWarnings.txt`.

## 3. Git/GitHub workflow

Full details and the exact ANS PR template: `references/workflow-git-github.md`.

- Never edit directly on `main`: create a dedicated branch + a PR for each change, unless the user explicitly says otherwise.
- Always confirm the target branch and exact repository with the user before starting any Git operation (branching, push, PR).
- Before creating a PR on an ANS repo, read the target repo's `.github/pull_request_template.md` and prioritize it if it differs from the standard template reproduced in `references/workflow-git-github.md`.
- After a push, wait for the GitHub Pages deployment to finish before reading `qa.html`.

## 4. Preparing a release

CI-SIS status ↔ configuration field mapping table, and detailed steps: `references/preparation-release.md`.

- The `status` of `sushi-config.yaml`, its `releaseLabel`, and the `status`/`mode` of `publication-request.json` must match the targeted CI-SIS status (draft, public-comment, for implementation, final-text, withdrawn/deprecated) — never infer these values by analogy, always check the mapping table.
- `change-log.md` must list the PRs merged since the previous version, with no residual placeholder.
- Every PR cited in the change-log must have the version's milestone assigned.
- If an automated audit tool for these steps is available in the environment, use it to verify the work rather than re-checking everything manually.

## 5. Changing an artifact that is already published

Kinds of breaking change, the non-breaking alternative to look for first, and transition rules: `references/changements-incompatibles.md`.

- SUSHI and the IG Publisher QA stay green on most breaking changes: check them at authoring and review time.
- Compare against the latest published tag, not `main`. An artifact in no published version yet can still change freely.
- Never reuse a published code with a new meaning or change its display: add a new code and retire the old one.
- A breaking change that survives gets a dedicated **Breaking changes** entry in `change-log.md`, with an old-to-new table.

## 6. Reference sources and documentation

Links to FHIR IG Publisher documentation, FSH grammar, tool source code (HAPI FHIR, IG Publisher, SUSHI), and reference GitHub organizations: `references/sources-documentation.md`.

## Feedback on specs

If a convention or step described here causes a problem (ambiguity, implementation question, improvement suggestion), feedback is welcome via the **GitHub issues of the relevant repo** (e.g. `github.com/ansforge/<repo>/issues`) or via a PR on this repository (`github.com/ansforge/interop-skills`) to evolve this skill itself.
