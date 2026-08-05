# Generation Prompt — University Courses Ontology

*Day 3 Exercise, Step 2. This is the prompt submitted to the LLM to produce
`generated-ontology.owl`. It is preserved as issued; the model's raw output was
not edited before evaluation.*

---

## Prompt as issued to the model

> **Role / task.** You are an ontology engineer. Produce a small OWL ontology for
> the **university course catalog and offering** domain.
>
> **Purpose.** Support a registrar-facing system that has to answer catalog,
> scheduling, prerequisite, enrollment, and grading questions. The model is a
> conceptual layer, not a database schema — I care about ontological correctness,
> not storage efficiency.
>
> **Competency questions the ontology must be able to answer:**
> 1. What courses does a given department offer?
> 2. What are the prerequisites of a given course, including prerequisites of
>    prerequisites (the full transitive chain)?
> 3. Which offerings/sections of a course run in a given semester?
> 4. Who is teaching a given offering, and which students are enrolled in it?
> 5. What final grade did a given student receive in a given offering?
> 6. How many credit hours is a course worth?
> 7. Can a single person be both a student and an instructor at the same time
>    (e.g., a graduate teaching assistant)?
>
> **Serialization.** RDF/XML, loadable in Protégé, `.owl` extension.
>
> **Definitions.** Yes — every class needs a real definition (natural-language
> `rdfs:comment`), written in genus–differentia form (an *X* is a *Y* that *Zs*).
> No circular or metalinguistic ("a class that represents…") definitions.
>
> **Individuals.** Yes — include a handful of example individuals (at least one
> course, one semester, and one graduate TA who both teaches and takes courses)
> so the competency questions can be exercised and a reasoner has something to
> chew on.
>
> **Upper-ontology / constraints.** Align to **BFO 2020**, and use **CCO** /
> **IAO** patterns where they apply. In particular:
> - Distinguish *continuants* from *occurrents*.
> - Distinguish an **information content entity** (the catalog course, the
>   syllabus, the grade record) from the **material entities** and **processes**
>   it is about.
> - Model **student** and **instructor** as **roles** (realizable entities borne
>   by a person), not as rigid kinds of person.
> - Model **enrollment** and **course delivery** as **processes** (occurrents),
>   not as static continuants.
> - Give relations explicit domains, ranges, and logical characteristics
>   (transitivity, inverses) where warranted.
>
> Return only the RDF/XML.

---

## Notes for the evaluator (me)

**Intended representation (documented before inspecting the output, per the
evaluation framework):**

- `Course` = the catalog entity — an *information content entity* / plan
  specification (generically dependent continuant). It is *not* the act of
  teaching and *not* a section.
- `CourseOffering` = the concrete, scheduled delivery of a course in a term — an
  *occurrent* (process) that realizes/is prescribed by a `Course`.
- `Person` = a *material entity*. `StudentRole` / `InstructorRole` = *roles*
  (realizable entities) borne by a person. The same person can bear both.
- `EnrollmentProcess` = an *occurrent* in which a person (bearing a student role)
  participates.
- `Semester` = a *temporal region*.
- Grade = an *information content entity* (an assigned datum) that is *about* a
  student's performance in an offering — not a quality of the person, not a
  free-floating class of letters.

**Load-bearing distinction the whole exercise turns on:**
*Logical consistency is necessary but not sufficient for ontological adequacy.*
The generated model can be made consistent by deleting one axiom and still be
ontologically wrong. The revision has to fix the **kinds**, not just silence the
reasoner.
