# OntoJIT as related work for ESKA

## Why OntoJIT matters

OntoJIT, developed by Selena Baset and Kilian Stoffel, is important related work for ESKA because it tackles the ontology-to-execution boundary directly.

Its central move is to translate OWL ontologies into an **active C# representation** so ontology constructs can live inside a conventional programming/runtime environment rather than only as passive data consumed through a separate ontology API.

A simplified translated path is:

```text
OWL ontology
        ↓ OntoJIT translation
C# classes + explicit OWL meta-properties
        ↓ CLR compilation
active runtime representation
        ↓
reflection / LINQ / inference support / procedural extensions
```

This makes OntoJIT a direct historical precedent for ontology-to-executable-code projection.

## Motivation and advantage

The main motivation is to reduce the impedance mismatch between ontology engineering and mainstream application development.

OntoJIT attempts to let developers exploit familiar language/runtime mechanisms such as:

- C# classes and inheritance;
- reflection;
- lambda expressions and LINQ;
- dynamic compilation;
- procedural methods attached to the active ontology representation.

This can make semantic knowledge easier to integrate into ordinary software than an architecture in which applications always cross into a separate ontology/reasoner subsystem.

## C# alone is not enough

OntoJIT explicitly recognizes that ordinary object-oriented language constructs cannot represent all OWL DL semantics.

Where a plausible native counterpart exists, it can use C# structures. For semantics such as equivalence, disjointness, restrictions, `owl:sameAs`, and related OWL constructs, it introduces an explicit **meta-properties layer**.

The important architectural consequence is that the generated C# representation is not, by itself, equivalent to saying that ordinary CLR semantics implement OWL.

A more accurate deployment interpretation is:

```text
OWL source ontology
        ↓
Generated C# representation
(classes + semantic meta-properties)
        ↓
semantic execution support
(native CLR behavior + meta-property interpretation
 + reflection / LINQ / query and inference procedures)
        ↓
Application / consumer
```

The semantic execution layer has therefore not disappeared; it has been integrated into the conventional programming environment. It is also not one single opaque reasoner. In the published prototype, semantic behavior is distributed across native C# inheritance/type information, explicit meta-properties, reflection, developer-authored LINQ/query procedures, and optional classification/procedural mechanisms.

## Second-pass preservation and responsibility analysis

The first-order lesson — projection plus runtime semantics both matter — can be made more precise by asking **how each semantic concern survives the transformation**.

| Semantic concern | Generated representation | What makes it executable | Preservation / limitation |
| --- | --- | --- | --- |
| OWL class and straightforward subclass structure | C# classes and native inheritance | CLR type information and reflection can expose hierarchy traversal | partly preserved by native projection, but this does not generalize to all OWL subsumption semantics |
| equivalence, intersection, union, complement, disjointness, restrictions, property characteristics, `owl:sameAs` and related axioms | explicit static or instance meta-properties | query/inference procedures must interpret the meta-properties | meaning remains inspectable in the projection, but ordinary CLR execution does not supply the OWL semantics |
| `rdfs:range` and similar constructs where native typing would strengthen semantics | kept as semantic metadata rather than ordinary type enforcement | semantic query/inference logic | deliberate avoidance of a superficially convenient mapping that would conflict with open-world semantics |
| multiple parent classes | one native C# inheritance path plus `subClassOf` meta-property information | runtime traversal must combine native hierarchy and metadata | prototype-specific approximation; the 2016 paper explicitly reports an inconsistency in the workaround |
| blank nodes | deterministically named private generated classes | reflection/query logic can reach the generated handles | structural identity is made operationally inspectable, but it is a transformed representation rather than source authority |
| import closure / unsupported source constructs | recursively generated namespaces for supported imports | parser + runtime over the generated closure | unsupported OWL Full constructs in imported ontologies could be skipped in the published prototype, with an acknowledged impact on reasoning soundness |
| OWL 2 constructs beyond the tested profile | not established by the demonstrated parser | not established | the 2016 paper reports SHOIN(D) support and says SROIQ(D) extensions such as general concept inclusions had not been tested |
| inferred semantics materialized by `Classify()` | generated code may be modified and dynamically recompiled | procedural classification + dynamic compilation | the new runtime state is **derived semantic state**, not a new authoritative assertion source |

This exposes a useful distinction for ESKA:

```text
semantic carrier
    what source meaning is represented in the projection

semantic interpreter
    what procedure gives that representation operational meaning

runtime policy
    which query-world assumptions or application rules are selected

result state
    what was inferred, materialized, or returned
```

Calling all four simply “the runtime” hides important authority boundaries.

## Authority at runtime

For the translated OntoJIT path, the **source OWL remains the authority for asserted ontology semantics**. The generated C# assembly is a derived executable carrier.

Runtime/query code can legitimately own a different concern — for example an application-specific query policy — but that does not make it authority for the meaning of the source OWL axioms. If a runtime procedure intentionally adopts semantics different from OWL entailment, that choice must be explicit and independently justified by the application/specification concern it serves.

This becomes especially important when OntoJIT materializes inferred information by modifying and recompiling runtime code:

```text
asserted OWL semantics
        ↓ projection
executable representation
        ↓ inference / classification
materialized inferred runtime state
```

The last state is a **result or derived cache of inference**. Treating it as a new semantic source of truth would invert the authority chain unless a separate governance decision explicitly promotes it.

## Open World versus Closed World

OntoJIT's published Pizza experiment is particularly useful for ESKA because it shows that preserving source constructs structurally does not automatically guarantee equivalent runtime semantics.

In the demonstrated set-relative-complement query, OntoJIT used Closed World behavior while OWL DL reasoning uses the Open World Assumption. That produced different answer sets from the same source ontology.

This should **not** be read as evidence that compiling OWL into C# inherently forces a Closed World interpretation.

The earlier parser work actually provides the opposite kind of evidence: it avoids some apparently natural C# mappings when native compiler/type semantics would conflict with OWL's open-world interpretation. The later Pizza divergence occurs in the **chosen query procedure**: the relative-complement algorithm treats the relevant sets as complete.

An OntoJIT-like semantic execution layer could in principle preserve Open World / OWL-compatible entailment semantics. Conversely, Closed World or locally closed behavior can be a legitimate application/runtime policy when the specification explicitly owns that concern.

The precise ESKA rule is therefore:

> **A different world assumption is not automatically an error, but it is a semantic change if the runtime is still claiming to answer the authoritative OWL question under OWL semantics.**

The key lesson remains:

> **Semantic continuity depends not only on whether source constructs survive a projection, but also on the semantics of the runtime that interprets that projection.**

## Verification is not provenance

The 2018 OntoJIT experiments provide useful **bounded verification evidence**: selected terminological queries over Gene Ontology and Pizza were compared with HermiT/FaCT++, with most demonstrated results matching apart from the documented Open World cases. The paper explicitly limits the evaluated scope to terminological entailment and does not include assertional queries.

That is not the same concern as provenance.

The reviewed OntoJIT papers do not describe a first-class lineage contract that records, for a particular runtime answer:

```text
exact source ontology/version
        ↓
source axiom(s)
        ↓
transformation rule / generated construct
        ↓
query or inference procedure + world assumption
        ↓
runtime/materialized state
        ↓
result
```

This distinction becomes more important, not less, when inferred information is dynamically materialized into executable code. ESKA's provenance responsibility is therefore not satisfied merely by having reproducible generated code or empirical agreement with an independent reasoner.

## Relationship to ESKA

For the translated OntoJIT path, an approximate ESKA-oriented decomposition is:

```text
Authoritative OWL semantics
        ↓
Semantic Model
        ↓
Implementation Projection
generated C# + semantic meta-properties
        ↓
semantic execution support
(native runtime + query/inference procedures)
        ↓
Execution
        ↓
Result
        ↓
Verification / provenance as distinct evidence concerns
```

This mapping is specific to the OntoJIT translated path. ESKA does **not** require an `ImplementationProjection` in every execution path; source-owned semantic artifacts may participate directly in execution.

The closest relationship is:

- OWL source ontology → authoritative semantic source / Semantic Model;
- generated C# + meta-properties → Implementation Projection;
- native CLR mechanisms plus OntoJIT/developer query and inference procedures → execution mechanism interpreting the projection;
- query/classification output or materialized inferred code → Result / derived runtime state;
- comparison with independent OWL reasoners → bounded verification evidence;
- exact result lineage to source semantics and transformation/runtime decisions → an ESKA provenance responsibility not demonstrated as a first-class OntoJIT mechanism in the reviewed papers.

## Falsification against newer SKE evidence

The later SKE/AIUP and SFBE experiments add an important guardrail that was not available to the original OntoJIT work:

> **The ability to translate or execute semantics does not by itself prove that a separate semantic layer has independent value.**

The [AIUP Task Manager falsification experiment](https://github.com/GerhardBalz/semantic-knowledge-engineering/issues/45) is a negative case. Task structure, status and membership behavior are already adequately owned by the application specification, implementation constraints and tests. Adding RDF/OWL machinery can restate those concerns, but the current case does not demonstrate a separate semantic authority or reasoning need.

The [SFBE semantic-value falsification experiment](https://github.com/GerhardBalz/semantic-federated-biomedical-evidence/blob/main/docs/semantic-value-falsification-experiment.md) provides mixed and positive cases. For example, an externally governed HPO hierarchy changes which phenotype evidence may validly match; removing that authority changes correct execution. Other concerns — such as false-precision guards or source protocol execution — remain adequately owned by specification/application/tests without extra ontology machinery.

Applied back to OntoJIT, the resulting test is concern-specific:

```text
Is the OWL meaning itself an independently justified authority
or reasoning oracle for this concern?
        │
        ├─ no
        │    → ontology-to-code machinery may only duplicate
        │      specification / application / test authority
        │
        └─ yes
             → an executable projection can add real value,
               but preservation, runtime assumptions and lineage
               must remain explicit
```

OntoJIT is therefore best understood as an **execution technique for semantic authority that is already justified**, not as evidence that every application should introduce ontology authority.

## Prototype-specific versus general lessons

Several OntoJIT findings should not be generalized into laws of ontology-to-execution architecture.

Prototype-specific evidence includes:

- C# single-inheritance constraints and the particular multiple-inheritance workaround;
- the demonstrated parser/profile boundary and unsupported import cases;
- LINQ as the query mechanism;
- the specific Pizza relative-complement query adopting Closed World behavior;
- the evaluated focus on terminological rather than assertional entailment;
- dynamic recompilation as the mechanism for materializing inferred semantics.

The more general lessons are architectural:

1. a target representation can preserve some semantic information natively and other information only as explicit metadata;
2. semantic metadata is not executable meaning until an interpreter/query/inference procedure gives it operational semantics;
3. target-language behavior can silently strengthen, weaken or otherwise change source semantics;
4. derived runtime state should not silently become semantic authority;
5. verification of selected answers is distinct from provenance of why an answer is justified;
6. semantic machinery should be retained only for concerns where semantic authority or inference adds independent value.

These lessons remain relevant to AI-generated and agentic software: generated code is not authoritative merely because it was generated from an authoritative semantic source. The derivation, omitted or transformed semantics, runtime assumptions, and evidence for resulting behavior remain separate responsibilities.

## Key difference from ESKA

OntoJIT optimizes primarily for bringing ontology semantics into a mainstream programming environment.

ESKA addresses a broader architectural question:

> how do relevant semantics, authority, assumptions, execution meaning, results, verification, and provenance remain explicit across different execution mechanisms and representations?

OntoJIT is therefore complementary prior art rather than the same architecture.

A useful distinction is:

```text
OntoJIT
semantic model
    ↓
bring semantics into executable C# + runtime support

ESKA
semantic model
    ↓
zero or more justified projections
    ↓
heterogeneous execution
    ↓
result / verification / provenance
```

## ESKA lesson

OntoJIT strengthens the current SKE/ESKA principle:

> **Semantics must remain explicit through execution.**

It demonstrates why the semantic-continuity contract has to include more than structural model-to-code traceability.

At minimum, the following need to remain inspectable when a semantic model is projected into another execution environment:

```text
semantic constructs
+ semantic assumptions
+ projection rules
+ runtime / inference semantics
+ result lineage
```

The projection itself is not inherently unsafe. A semantic break occurs when the execution environment silently interprets the projected representation under assumptions incompatible with the authoritative source semantics.

The newer falsification evidence adds one further condition:

> **First justify that semantic authority adds independent value for the concern; only then ask how to preserve it through projection and execution.**

## Sources

Primary OntoJIT sources:

- Baset, S.; Stoffel, K. **OntoJIT: Parsing Native OWL DL into Executable Ontologies in an Object Oriented Paradigm**. DOI: https://doi.org/10.1007/978-3-319-54627-8_1
- Baset, S.; Stoffel, K. **OntoJIT: Exploiting CLR Compiler Support for Performing Entailment Reasoning over Executable Ontologies**. DOI: https://doi.org/10.18178/ijke.2018.4.1.093
- Baset, S.; Stoffel, K. **Procedural extensions for executable ontologies in conventional software development**. DOI: https://doi.org/10.1142/9789813273238_0110
- Baset, S.; Stoffel, K. **Object-Oriented Modeling with Ontologies Around: A Survey of Existing Approaches**. DOI: https://doi.org/10.1142/S0218194018400284
- Baset, S. **Towards the Democratization of Ontological Modeling through a New Pervasive Means of Representation**. DOI: https://doi.org/10.35662/unine-thesis-2787

Current SKE comparison evidence:

- [AIUP Task Manager semantic falsification experiment](https://github.com/GerhardBalz/semantic-knowledge-engineering/issues/45)
- [AIUP related-work note](https://github.com/GerhardBalz/semantic-knowledge-engineering/blob/main/landscape/related-work/ai-unified-process.md)
- [SFBE semantic-value falsification experiment](https://github.com/GerhardBalz/semantic-federated-biomedical-evidence/blob/main/docs/semantic-value-falsification-experiment.md)

This note positions OntoJIT as related work only. It does not claim equivalence, endorsement, or architectural lineage beyond what the cited material supports.
