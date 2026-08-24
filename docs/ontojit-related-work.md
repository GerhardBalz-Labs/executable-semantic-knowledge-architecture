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
OntoJIT semantic runtime / inference support
        ↓
Application / consumer
```

The semantic runtime has therefore not disappeared; it has been integrated into the conventional programming environment.

## Open World versus Closed World

OntoJIT's published Pizza experiment is particularly useful for ESKA because it shows that preserving source constructs structurally does not automatically guarantee equivalent runtime semantics.

In the demonstrated set-relative-complement query, OntoJIT used Closed World behavior while OWL DL reasoning uses the Open World Assumption. That produced different answer sets from the same source ontology.

This should **not** be read as evidence that compiling OWL into C# inherently forces a Closed World interpretation.

An OntoJIT-like semantic runtime could in principle preserve Open World / OWL-compatible entailment semantics. The demonstrated divergence is better understood as a property of the chosen runtime/query semantics.

The key lesson is therefore:

> **Semantic continuity depends not only on whether source constructs survive a projection, but also on the semantics of the runtime that interprets that projection.**

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
semantic runtime / inference engine
        ↓
Execution
        ↓
Result / verification / provenance
```

This mapping is specific to the OntoJIT translated path. ESKA does **not** require an `ImplementationProjection` in every execution path; source-owned semantic artifacts may participate directly in execution.

The closest relationship is:

- OWL source ontology → authoritative semantic source / Semantic Model;
- generated C# + meta-properties → Implementation Projection;
- OntoJIT inference/runtime support → execution mechanism interpreting the projection;
- query/classification result → Result;
- comparison with independent OWL reasoners → verification evidence.

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

## Sources

Primary sources:

- Baset, S.; Stoffel, K. **OntoJIT: Parsing Native OWL DL into Executable Ontologies in an Object Oriented Paradigm**. DOI: https://doi.org/10.1007/978-3-319-54627-8_1
- Baset, S.; Stoffel, K. **OntoJIT: Exploiting CLR Compiler Support for Performing Entailment Reasoning over Executable Ontologies**. DOI: https://doi.org/10.18178/ijke.2018.4.1.093
- Baset, S.; Stoffel, K. **Procedural extensions for executable ontologies in conventional software development**. DOI: https://doi.org/10.1142/9789813273238_0110
- Baset, S.; Stoffel, K. **Object-Oriented Modeling with Ontologies Around: A Survey of Existing Approaches**. DOI: https://doi.org/10.1142/S0218194018400284
- Baset, S. **Towards the Democratization of Ontological Modeling through a New Pervasive Means of Representation**. DOI: https://doi.org/10.35662/unine-thesis-2787

This note positions OntoJIT as related work only. It does not claim equivalence, endorsement, or architectural lineage beyond what the cited material supports.
