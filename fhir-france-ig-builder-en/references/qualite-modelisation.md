# Modelling quality: profiles, examples, documentation, build

SUSHI and the IG Publisher check that the IG is well formed, not that the modelling choices are right. The rules below cover the mistakes that pass the build.

## Profile rules

- Do not restate the base. No cardinality identical to the base resource or parent profile, no rule written twice in the same profile, no invariant copied from the base specification (it already applies). Restating choice types (`value[x]`, `definition[x]`) as slices can even make the validator reject valid instances.
- Binding strength says what is enforced. `required`: a closed list. `extensible`: use the ValueSet when a code fits, another code otherwise - the validator then cannot tell a typo from a legitimate new code. `preferred`: a recommendation. `example`: no recommendation. Always write the strength explicitly.
- Do not narrow a base ValueSet without a reason. Restricting the ValueSet bound in the base resource to a subset of its codes makes valid resources invalid, and forces a ValueSet change for every new use case.
- Do not bind a whole repeating coded element. A `required` or `extensible` binding on a `0..*` or `1..*` CodeableConcept applies to every repetition and forbids codes from other systems next to it. Slice the element (open slicing) and bind the slice.
- Constrain references to the national profile. Where a profile exists for the target (FR Core or an ANS guide), write `only Reference(FRCorePatientProfile)` rather than the base resource, and the same on sibling profiles that point at the same target. The validator then checks the target, and the publisher links to its page. Keep the base resource only where no profile exists.
- Reuse before extending. Before an extension, check that no existing element carries the data, then look for a standard extension: <https://hl7.org/fhir/extensions/> and <https://packages2.fhir.org/xig?type=ext>.
- One profile with additional bindings rather than several profiles when only the ValueSet changes with the context (`^binding.additional` with a `usage`).
- Do not give a generic name to a narrow profile. A profile `MyTask` written for a single workflow blocks a real base `Task` profile later, and its canonical is already in implementers' systems.
- Must Support only for what every system must handle, and the IG says what Must Support means for senders and receivers.

## Examples

Implementers copy the examples: a wrong example propagates.

- Every profile has at least one example (`Usage: #example`) that fills its mandatory and Must Support elements, and declares the profile in `meta.profile`.
- Read each example as a story: are the dates in order (prescription before dispensing, start before end)? Would a clinician recognise the situation? Do the sibling examples of a workflow follow the same patient through the same chain of events?
- No business identifier invented for a resource that does not get one in practice: the resource `id` already makes it unique.
- Codes and displays are real: check them on the terminology server rather than writing them from memory.

## Documentation

- The `Description` of each profile says what it is used for, phrased like its siblings.
- A profile has an intro and notes page (`StructureDefinition-<id>-intro.md` and `-notes.md`) when the formal views are not enough to use it: what the resource represents, what references it, a minimal valid example.
- A new identifier system has a `NamingSystem` explaining who issues it and what it identifies.
- A `ConceptMap` says whether its mapping is complete or will grow.
- When the IG creates its own artifact rather than reusing an existing one, the description says why.

## Build and QA hygiene

- Never edit the publisher scripts (`_genonce.sh`, `_genonce.bat`, `_updatePublisher.*`, `_build.*`): they are overwritten on update and the change is lost. For memory, set `JAVA_TOOL_OPTIONS` (e.g. `-Xmx8g`) instead.
- Suppressing a warning is not fixing it. Every line added to `input/ignoreWarnings.txt` has a reason. If the warning comes from a publisher bug that will be fixed, wait for the fix rather than disabling validation permanently - it never gets turned back on.
- `ignoreWarnings.txt` matches by `%`, not by regex: `%foo%` contains, `foo%` starts with, `%foo` ends with, `foo` exact.

## Verification

- [ ] No cardinality, rule or invariant that restates the base or the parent profile.
- [ ] Every binding has an explicit strength that matches what is enforced.
- [ ] No base ValueSet narrowed without a stated reason.
- [ ] No `required`/`extensible` binding on a whole repeating CodeableConcept.
- [ ] References point at the national profiles where they exist, consistently across sibling profiles.
- [ ] Every new extension was checked against existing elements and standard extensions.
- [ ] Every profile has a coherent example with `meta.profile`.
- [ ] Every new identifier system has a `NamingSystem`.
- [ ] No publisher script edited; every new `ignoreWarnings.txt` line has a reason.
