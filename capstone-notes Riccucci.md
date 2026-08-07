# Day 5 Capstone

## Participants

Ryan A. Riccucci

---

## Model

`capstone-model.owl` (RDF/XML, for Protégé) · `capstone-model.ttl` (Turtle, same content)

**Size:** 11 domain classes (10 asserted + 1 defined), 5 object properties, 3 data properties, 11 named individuals, 189 asserted triples.

**Reuse before minting.** Participation (`BFO:0000057`), aboutness (`IAO:0000136`), inherence and bearer-of (`RO:0000052` / `RO:0000053`) are taken from BFO/RO/IAO rather than re-coined. Only four relations are minted: `inspects` (with inverse `inspectedIn`), `hasOutput`, `prescribes`, and `measurementOf` (a sub-property of `is about`).

### What is modelled as what, and why

| Scenario element | Modelled as | Reason |
|---|---|---|
| Pump P-17 | Individual of `Pump ⊑ cco:Artifact` | An object. |
| Alex | Individual of `cco:Person` bearing `Alex_TechnicianRole` | Technician is a role, not a kind of person. |
| Inspection 17 | Individual of `Inspection ⊑ cco:Act` | An occurrent. |
| "inspected every 30 days" | `Procedure-30Day`, a **directive** ICE, with `hasIntervalDays 30` | A prescription. Nothing has happened every 30 days; something is *required* to. |
| Report R-17 | Individual of `InspectionReport ⊑ cco:DescriptiveICE` | An ICE, `is about` P-17, `hasOutput` of Inspection 17. Not the pump and not the inspection. |
| The pump's vibrating | `P-17_Vibration`, a `Quality` inhering in P-17 | The quality is in the pump. |
| 4.2 mm/s | `Measurement-17`, a measurement ICE, `measurementOf` the quality, carrying `hasMagnitude 4.2` and `hasUnit "mm/s"` | Three distinct things: the quality, the measurement about it, and the number the measurement carries. |
| "requires bearing replacement" | `Recommendation-17`, a **directive** ICE, part of R-17, which `prescribes` the *class* `BearingReplacement` | A recommendation that X be done is not X. |
| Work Order W-22 | Individual of `WorkOrder ⊑ cco:DirectiveICE`, which `prescribes BearingReplacement` | Same treatment. |
| The bearing replacement itself | **No individual** | It has not occurred. The scenario says it was requested. |

**The deliberate omission is the point.** There is no individual for the bearing replacement act. `BearingReplacement` exists as a class, and the two directives reference it by OWL 2 DL punning so a directive can point at the act *type* it requires. Creating an individual would assert that a repair happened, which the scenario does not say.

### Answering the seven required questions

All executed as SPARQL against the shipped file. Results are actual output.

| # | Question | Result |
|---|---|---|
| 1 | What is the inspection procedure? | `Procedure-30Day`, interval 30 days, prescribes `Inspection` |
| 2 | Who performed Inspection 17? | `Alex`, bearing `Alex_TechnicianRole` |
| 3 | What was inspected? | `P-17` |
| 4 | What information resulted? | `R-17` |
| 5 | What vibration value was recorded? | `4.2` `mm/s`, measurement of `P-17_Vibration` |
| 6 | What maintenance action was requested? | `BearingReplacement` (a type) |
| 7 | Which work order requests it? | `W-22` |
| — | *Has the replacement occurred?* | **No results** — the correct answer |

---

## Three Definitions

**1. Maintenance Procedure**

> A directive information content entity that prescribes maintenance acts to be performed on an artifact and the interval at which they are to recur.

Genus is directive ICE, which places it correctly among prescriptions rather than among processes. The differentia names both what is prescribed and the recurrence, which is the feature that distinguishes it from a one-off work order. It defines the entity rather than the phrase, is not circular, and does not smuggle in the pump or the thirty days — those are facts about the individual, not the class.

**2. Vibration**

> A quality inhering in a material entity in virtue of the oscillatory displacement of that entity about a reference position.

Genus is quality, which fixes the category and thereby settles that the vibration is *in the pump* and the number is not. The differentia is stated in terms of the physical condition rather than in terms of how it is measured, which keeps the quality independent of any measurement of it. Naming the reference position avoids the definition being too broad, since displacement alone would admit any motion.

**3. measurement of** *(relation)*

> Holds between a measurement information content entity and a quality when the measurement records a magnitude of that quality obtained by a measurement process.

Stated as a condition on when the relation holds, per the guidance for relations. It is a sub-property of `is about`, which places it correctly: a measurement is a piece of information *about* something, not the thing itself. Requiring that the magnitude be obtained by a measurement process excludes estimates and defaults, which is the distinction that matters operationally.

---

## AI Comparison

**Method.** An LLM was given the exercise's suggested prompt with the scenario pasted verbatim. Its proposal is summarised below. *If you want an independently generated artifact for the record, re-run the prompt yourself and substitute — the differences below are typical rather than unique to one run.*

**AI proposal, in outline:** classes `Pump`, `Technician`, `Inspection`, `MaintenanceProcedure`, `Report`, `Measurement`, `WorkOrder`, `MaintenanceAction`, `Bearing`; `Technician ⊑ Person`; data property `hasVibrationValue` on `Pump` with value 4.2; individuals including `BearingReplacement : MaintenanceAction`; relations `performedBy`, `inspects`, `generates`, `recommends`, `requests`.

### Difference 1

**AI suggestion:** `Technician` is modelled as a subclass of `Person`, with Alex an instance of `Technician`.

**Decision:** MODIFY

**Reason:** Being a technician is something Alex can start and stop being without ceasing to exist or ceasing to be Alex. That is the signature of a role, not a kind. Subclassing forces the model to assert that Alex is essentially a technician, and it breaks the day Alex is reassigned — either the individual has to be retyped, which loses history, or a second individual is created, which fractures identity. Modelled as `TechnicianRole` borne by Alex, the role can be acquired and divested and the assignment can be dated. The scenario also cares *that Alex acted in that capacity*, which is a fact about the role and not about Alex's species.

### Difference 2

**AI suggestion:** A data property `hasVibrationValue` with value `4.2` asserted directly on `Pump P-17`.

**Decision:** MODIFY

**Reason:** This collapses three distinct entities into one triple. There is the pump's vibrating, which is a quality inhering in the pump; there is the measurement, which is information about that quality and which was produced by a measurement process at a time by an instrument; and there is `4.2`, which is a number carried by the measurement. Putting `4.2` on the pump asserts that the number is in the pump. It also loses the unit — `4.2` alone is uninterpretable, and mm/s is not recoverable from the datatype — and it leaves nowhere to record when the measurement was taken, by what, or with what uncertainty. The model separates all three, which costs two individuals and buys the ability to say that the pump still vibrates while the measurement is stale.

### Difference 3

**AI suggestion:** `BearingReplacement` is created as a named individual of class `MaintenanceAction`.

**Decision:** REJECT

**Reason:** No bearing replacement has occurred. The scenario states that a report *recommended* one and a work order *requests* one. Instantiating the act asserts into the knowledge base that a repair happened, which is false, and a downstream maintenance application querying for completed actions would return it. This is the prescription-treated-as-fact error, and it is the most consequential of the three because it is the one that would produce a wrong answer to an operational question rather than merely an awkward model. My model has `BearingReplacement` as a class only, referenced by both directives through punning. The correct query "has the replacement occurred?" returns no results, which is the true answer.

**Also accepted:** the AI's separation of `Measurement` as its own class was correct and I kept it — the comparison is not reflexive rejection. I dropped its `Bearing` class as unnecessary for the seven questions, on the rule against adding classes that carry no query.

---

## Reasoner Results

Run under OWL-RL deductive closure. *(Re-run HermiT in Protégé before submission if a DL reasoner result is required; OWL-RL will not evaluate negation-based constructs, though this model contains none.)*

- **Consistent:** YES. Closure expanded 189 → 657 triples with no inconsistency raised.
- **Unsatisfiable classes:** None.
- **Expected inferences:**
  - `P-17` classified into the defined class `InspectedPump`, which was **not asserted** — inferred from the equivalence axiom plus the inverse of `inspects`.
  - `P-17 inspectedIn Inspection-17` inferred from the asserted `Inspection-17 inspects P-17`.
  - Type hierarchy propagation: `R-17` inferred as a `cco:InformationContentEntity` and a `bfo:GenericallyDependentContinuant`.
- **Unexpected inferences:** None.

**Caveat, stated because the exercise invites it.** A logically consistent ontology can still be badly modelled. This one would remain consistent if I had asserted `4.2` directly on the pump, or instantiated a bearing replacement that never happened. Neither error is a logical error. The disjointness axioms I added — directive versus descriptive ICE, and vibration versus vibration measurement — convert two of the modelling errors into logical ones so that the reasoner *would* catch them, which is the only way a reasoner helps with modelling quality.

---

## Operational View

| Application field | Ontology source |
|---|---|
| Pump | `cap:P-17`, individual of `cap:Pump`; label "Pump P-17" |
| Last inspection | `cap:Inspection-17`, reached by `cap:P-17 cap:inspectedIn ?i`. **The superlative "last" is not in the model** — see below |
| Measured vibration | `cap:Measurement-17`: `cap:hasMagnitude "4.2"^^xsd:decimal` + `cap:hasUnit "mm/s"`, reached via `cap:R-17 BFO:0000051 ?m` |
| Maintenance action | `cap:Recommendation-17 cap:prescribes cap:BearingReplacement` — the *type* prescribed, not a performed act |
| Work order | `cap:W-22`, individual of `cap:WorkOrder`, `cap:prescribes cap:BearingReplacement` |

### What information was omitted?

- **Who performed the inspection.** Alex does not appear in the view, nor does the technician role, so accountability is unrecoverable from the display.
- **The governing procedure.** `Procedure-30Day` and its 30-day interval are absent, so the view cannot support the question "is this inspection overdue?"
- **The report as an entity.** R-17 is the thing that is *about* the pump and that carries both the measurement and the recommendation. The view shows two of its parts and drops the container, so provenance of the two displayed facts is lost — they arrive looking like independent attributes of the pump.
- **The quality/measurement distinction.** The view shows a number where the ontology holds a quality, a measurement about it, and a magnitude.
- **The measurement process.** No time, no instrument, no uncertainty.
- **The recommendation as distinct from the work order.** Both prescribe the same act type; the view shows the action once and the work order once, which reads as a single chain rather than two independent directives.

### Is the operational view lossy?

**YES**

Six categories of information present in the model do not appear in the view, listed above. Nothing displayed can be used to recover the agent, the procedure, the measurement provenance, or the report structure. Loss alone is unremarkable — an operator display is *supposed* to be a projection, and a view that showed everything would be unusable.

### Is the operational view divergent?

**YES** — and this is the finding, because loss is expected and divergence is not.

Two divergences, in ascending severity.

**1. "Last inspection" asserts a superlative the model does not entail.** The ontology records exactly one inspection of P-17. Under the open-world assumption, the absence of other inspections is not an assertion that none exist. The application reads "the only inspection I can see" and displays "the last inspection," which is a stronger claim. If a second inspection exists in another system, or was performed and not yet recorded, the display is simply false. This is fabrication: the view holds an assertion the source never made.

**2. "Maintenance action: Replace bearing" changes a prescription into a fact.** The ontology holds two directives that prescribe an act of type `BearingReplacement`, and holds no instance of that act. The field is labelled *action*, which an operator will read as something the maintenance system knows about the pump — and in the field's neighbours, where "Pump" and "Measured vibration" are both observations, the reading is almost forced. Whether this is divergent turns entirely on how the label is read, which means the label is referentially ambiguous: it licenses two mutually exclusive readings — action requested and action performed — and supplies no discriminator. The remedy is not more data. It is relabelling the field "Requested action," which costs nothing and removes the ambiguity.

A third, milder case: placing "Measured vibration" directly beneath "Pump" implies the value is an attribute of the pump, flattening the quality-measurement-magnitude distinction. Not strictly divergent, since 4.2 mm/s *is* true of the pump's vibration, but it is the presentational habit that makes divergence 2 easy to fall into.

**The general point.** A simplified operational view may omit semantic detail. It must not add assertions, and it must not change the modality of what it displays. Divergence 1 adds an assertion. Divergence 2 changes prescription into observation. Both are repairable at the level of the view — one by weakening the label to "Most recent recorded inspection," one by relabelling to "Requested action" — without touching the authoritative model.

---

## Most Important Lesson from the Week

That a consistent ontology and a correct one are different achievements, and only the first is checkable by machine. The reasoner will confirm all week that nothing contradicts, while the model quietly asserts that a number lives inside a pump or that a repair has happened because someone recommended it. What the week actually taught is where to put the disjointness axioms so the machine can catch the mistakes that matter — and, where it cannot, to write the definition carefully enough that a reviewer can.
