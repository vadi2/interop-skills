# Preparing a FHIR IG release (ANS)

Authoritative reference: <https://interop.esante.gouv.fr/ig/documentation/mod_bonnes_pratiques.html#release-dun-ig-fhir> — check the version before settling an ambiguous case. `publication-request.json` documentation (HL7): <https://confluence.hl7.org/spaces/FHIR/pages/144970227/IG+Publication+Request+Documentation>.

## CI-SIS status → configuration fields mapping table

| CI-SIS Status | `sushi-config.yaml` > `status` | `sushi-config.yaml` > `releaseLabel` | `publication-request.json` > `status` | `publication-request.json` > `mode` |
|---|---|---|---|---|
| draft | `draft` | `ci-build` | `ci-build` | N/A |
| public-comment | `draft` | `ballot` | `ballot` | `working` |
| for implementation | `active` | `trial-use` | `trial-use` | `milestone` |
| final-text | `active` | `final-text` | `final-text` | `milestone` |
| withdrawn/deprecated | `retired` | N/A | `withdrawn` or `retired` | `withdrawal` |

In addition, in `public-comment` the `version` (sushi-config and publication-request) and the `path` (publication-request) must end with `-ballot`; in `for implementation`/`final-text`, no version suffix. Never infer these values by analogy with a previous release — always re-check against this table.

## Steps, in order

When the user asks to prepare a release `X.Y.Z`:

### 1. Update the branch with `main`

```bash
git fetch origin main && git merge origin/main
```

### 2. Modify `sushi-config.yaml`

- `version`: `X.Y.Z-ballot` → `X.Y.Z` (or the reverse depending on the targeted status)
- `status`: the IG's maturity status — see the table above
- `releaseLabel`: see the table above

### 3. Modify `publication-request.json`

- `version`: consistent with `sushi-config.yaml`
- `path`: `.../X.Y.Z-ballot` → `.../X.Y.Z` (or the reverse)
- `status`: publication label (different from `sushi-config.yaml`'s `status`, see the table)
- `mode`: see the table above

### 4. Update `input/pagecontent/change-log.md`

- Replace placeholders with the `X.Y.Z` version and the canonical URL.
- List the PRs merged since the previous version (`gh pr list --state merged`) in the format `* Titre de la PR [#N](url_pr)`.
- If this is the first FHIR release of this volet, add a "Versions antérieures (PDF)" section listing earlier PDF versions (check the ANS site for the relevant volet).
- If the version changes an artifact already published in a way that breaks implementers (see `changements-incompatibles.md`), add a dedicated **Breaking changes** section saying what they must change, with an old-to-new table for codes, canonicals or identifier systems.
- Check for any residual placeholder (`xxx`, `TODO`, `TBD`, `FIXME`, `<insert`, `placeholder`, generic issue number).

### 5. Assign the `X.Y.Z` milestone to every PR cited in the change-log

```bash
# Get the milestone number
gh api repos/ansforge/[repo]/milestones
# Assign it
gh api repos/ansforge/[repo]/pulls/[N] --method PATCH -f milestone=[number]
```

### 6. Update the release PR description

List all modified files (`git diff origin/main...HEAD --name-only`) and describe each change in the PR using the format `* [fichier] Description du changement`.

## Cross-check before considering the release ready

- [ ] No unmerged commits since `origin/main`.
- [ ] `sushi-config.yaml` and `publication-request.json` are consistent with each other and with the mapping table.
- [ ] `change-log.md` is up to date, no placeholder, with at least one PR listed.
- [ ] Every breaking change since the previous version (compare with its tag) is listed under **Breaking changes**.
- [ ] Every PR cited in the change-log has the `X.Y.Z` milestone assigned (`gh api .../pulls/N --jq '.milestone.title'`).
- [ ] A release PR is open toward `main`.

If an automated audit tool for these steps is available in the environment, use it to produce a status report rather than re-checking everything manually item by item.
