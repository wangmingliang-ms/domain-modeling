---
name: domain-modeling
description: Use when solving a product problem or designing, implementing, extending, repairing, or refactoring any feature, before choosing architecture or writing implementation code
license: MIT
metadata:
  author: wangmingliang-ms
  version: "0.1.0"
---

# Domain Modeling

## Overview

Code is an expression of a domain model. Starting from files, functions, or the current
implementation preserves accidental complexity and produces locally correct patches that conflict
globally.

**Core principle:** Re-derive the design from domain concepts before changing implementation.

**Following the existing design without first proving that it represents the domain is a violation
of this process.**

The current code reveals an implemented model, not necessarily the correct model. The user reveals
domain intent, not necessarily a complete model. Use both as evidence, make assumptions explicit,
and iterate with the user toward a better concept relationship model.

## The Iron Law

```text
NO FEATURE IMPLEMENTATION WITHOUT A DOMAIN MODEL AND REDESIGN FIRST
```

This applies to:

- new features;
- additions to existing features;
- behavior changes and refactoring;
- bugs that expose unclear state, ownership, lifecycle, or module boundaries.

“Redesign” means deriving the best design again from the domain. It does not require rewriting
working code. The result may validate the existing design, but that must be a conclusion, not an
assumption.

<HARD-GATE>
Do NOT write production code, propose a code-level patch, or create an implementation plan until
you have presented the domain model and the resulting design to the user and received approval.

Before approval, the only allowed actions are investigation, domain discovery, modeling, comparing
design alternatives, and presenting the design. Do not place “apply the smallest patch,” “add a
field,” “change the reducer,” or any other implementation action earlier in the same checklist and
then claim the gate still applies.

Approval applies to the final To-Be domain model, not merely to a feature description or a list of
files to edit.
</HARD-GATE>

## The Two-Model Discipline

Always distinguish:

1. **As-Is model:** concepts and relationships currently implied by code, documentation, persisted
   data, behavior, and the user's description.
2. **To-Be model:** the concept structure that should govern the feature after contradictions,
   duplicated facts, misplaced ownership, and missing concepts are resolved.

Never silently turn the As-Is model into the proposed design. Never present the To-Be model as
certain when it contains unconfirmed domain assumptions.

For every important claim, label its source:

- **Observed:** directly represented by code, schema, tests, or behavior.
- **User-stated:** supplied as domain intent by the user.
- **Inferred:** the Agent's current interpretation.
- **Proposed:** part of the candidate To-Be model.

## The Process

Complete these phases in order.

### 1. Gather domain evidence

- Explore current architecture, documentation, data flow, recent changes, and failures.
- Describe the user-visible behavior and business goal without implementation vocabulary.
- Ask the user for missing business meaning when code cannot establish it.
- Treat existing classes, tables, flags, and module boundaries as evidence, not truth.
- For a bug, use systematic root-cause investigation first. Repeated fixes across different
  components are evidence that the model or boundary may be wrong.

Do not equate a database schema, API payload, or class diagram with the domain model. These are
representations that may flatten, duplicate, or distort domain concepts.

### 2. Reconstruct the As-Is concept model

Build a shared vocabulary from:

- entities with identity;
- values and policies;
- commands, events, and outcomes;
- actors and external systems;
- lifecycles and time-dependent behavior.

For every concept, state what it means and what it is not. Resolve overloaded or synonymous terms
before designing types.

For each concept, answer:

- What contains, references, creates, or terminates it?
- What is its lifecycle and which transitions are legal?
- Who owns each mutable fact?
- What can it do, and what is it forbidden to do?
- Which facts are stored, and which must be derived?
- What invariants must always hold?

Prefer one owner for every mutable fact. Make illegal states unrepresentable. Do not use independent
flags for facts that are consequences of one underlying state or relationship.

Present a compact As-Is relationship diagram and an ownership/capability table. Mark uncertain
edges as inferred rather than inventing certainty.

### 3. Challenge the As-Is model

Do not ask only whether the current code is internally consistent. Ask whether its concepts are the
right concepts.

Look for:

- one domain concept split across several implementation objects;
- one implementation object combining unrelated domain concepts;
- stored flags that duplicate a lifecycle state or relationship;
- concepts named after UI, transport, database, or workflow accidents;
- relationships that exist only to coordinate callbacks;
- behavior with no clear owner;
- differences between the user's language and the code's language;
- business rules that cannot be stated without mentioning modules.

Show the user the questionable assumptions and contradictions. A clear As-Is model is not approval
of that model.

### 4. Co-design the To-Be concept model

Ignore the current implementation structure temporarily and propose the simplest model that
expresses the concepts and invariants.

Consider 2-3 alternatives when ownership or boundaries are non-obvious. Compare them by:

- size of the representable state space;
- number of writers and coordination paths;
- clarity of ownership and dependency direction;
- behavior under concurrency, failure, and late events;
- ease of tracing and testing.

Choose data structures and relationships first. Then choose classes, algebraic data types, state
machines, services, or pure functions. Do not create classes merely because nouns exist.

Work with the user rather than presenting a finished model for rubber-stamping:

1. Ask one focused domain question at a time.
2. Prefer questions about meaning, ownership, lifecycle, and cardinality over code preferences.
3. Update the concept relationship diagram after each answer that changes the model.
4. Call out what changed from As-Is and why.
5. Repeat until the user and Agent agree on the vocabulary, relationships, capabilities, and
   invariants.

The output of this phase is an explicitly approved To-Be concept relationship diagram.

### 5. Map the approved model back to the codebase

Compare the derived model with the existing implementation:

- **Keep** structures that already express the model.
- **Change** structures with misplaced responsibility or duplicated state.
- **Remove** flags, branches, adapters, and controllers made unnecessary by the model.
- **Isolate** persistence, network, UI, and other effects from domain truth.
- Define migration and compatibility boundaries explicitly.

Prefer reducing concepts and branches over adding another orchestration layer.

### 6. Present the complete design and obtain approval

Present:

1. problem and domain vocabulary;
2. evidence sources and unresolved assumptions;
3. As-Is concept relationship diagram;
4. As-Is problems and contradictions;
5. approved To-Be concept relationship diagram;
6. ownership and capability table;
7. lifecycle/state transitions;
8. invariants;
9. proposed architecture and data flow;
10. differences from the current design;
11. alternatives and trade-offs;
12. testing strategy based on invariants.

Scale the detail to the problem. A small change may need only a few paragraphs, but it never skips
the model. After approval, write the specification, then create the implementation plan.

## Architecture Warning

STOP and redesign when any of these appear:

- three or more fixes reveal new problems in different components;
- the same business fact is stored or mutated in multiple places;
- correctness depends on flag combinations or callback ordering;
- multiple components believe they own the same object or lifecycle;
- external I/O failure rolls back or corrupts domain truth;
- adding one behavior requires branches across many unrelated modules;
- tests enumerate implementation paths but cannot state global invariants.

Do not add Fix #4 to an architecture that has not been reconsidered.
When this warning triggers, do not pre-commit to a surgical fix. First determine whether the existing
architecture can express the required invariants without additional coordination state.

## Common Rationalizations

| Excuse | Reality |
| --- | --- |
| “The user asked me to implement now.” | Urgency does not make an unknown model correct. |
| “It is a small field or endpoint.” | Small changes still encode ownership, lifecycle, and compatibility decisions. |
| “The architecture already exists.” | Existing code is evidence, not proof that the abstraction is right. |
| “The classes show us the domain model.” | Classes show the implemented interpretation; persistence and control flow may have distorted the domain. |
| “I can infer what the user means.” | Label the inference and confirm it; do not turn uncertainty into architecture. |
| “A redesign is too expensive.” | Re-deriving the design may validate the current structure; it does not mandate a rewrite. |
| “I only need the smallest patch.” | The smallest local diff can create the largest global complexity. |
| “I identified the relevant modules.” | Module discovery is not domain discovery. |
| “Tests will catch mistakes.” | Tests cannot rescue a model that permits contradictory states. |

## Red Flags — Stop

- Searching for insertion points before naming the concepts
- Producing only one diagram and calling it both current and proposed
- Presenting an inferred relationship as a confirmed domain fact
- Asking the user to approve a finished model without exploring uncertain concepts
- Proposing fields, flags, callbacks, or classes before assigning ownership
- Copying the existing architecture into the “design”
- Treating UI, API, transport, or database representations as the domain object by default
- Saying “surgical fix” after repeated cross-component regressions
- Producing an implementation plan without explicit invariants
- Using “pragmatic” to justify skipping redesign

**All of these mean: stop implementation and return to domain discovery.**
