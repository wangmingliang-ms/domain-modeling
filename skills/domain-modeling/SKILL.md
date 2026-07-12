---
name: domain-modeling
description: Use for any product feature or behavior change, including apparently small additions to existing code, before architecture or implementation; also use when state has multiple writers, behavior depends on flags or callback order, fixes move bugs between paths, late events mutate completed work, or one behavior spans unrelated modules
license: MIT
metadata:
  author: wangmingliang-ms
  version: "0.1.2"
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

## The Story-to-Model Rule

Domain concepts come from domain stories, not directly from class names, tables, API payloads, or
task wording.

For existing software, follow this order:

```text
code and behavior evidence
  -> natural-language domain story
  -> user correction and confirmation
  -> domain concepts, capabilities, events, relationships, and invariants
  -> domain model
  -> implementation design
```

A ticket-style User Story can be useful input, but it is often too narrow. Expand it into a domain
story that explains the actors, intent, starting conditions, business activities and decisions,
state changes, outcomes, and important failure or interruption branches.

Do not mechanically turn every noun into a class. The story reveals which nouns have domain
meaning, which verbs are capabilities, which decisions are policies, which changes are events, and
which statements imply invariants.

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

### 1. Reconstruct and confirm the domain story

- Explore current architecture, documentation, data flow, recent changes, and failures.
- Trace representative behavior end to end, from an actor's intent or triggering event to the
  business outcome.
- Include the normal path plus material boundary, failure, interruption, and concurrency scenarios.
- Rewrite what the code does as a natural-language domain story without class, module, database,
  API, UI, transport, callback, or framework vocabulary.
- State the actors, intent, starting conditions, business actions and decisions, state changes,
  outcomes, and unresolved questions.
- Label each part of the story as observed, user-stated, or inferred.
- Present the story to the user and ask one focused correction question at a time.
- Revise until the user confirms that the story is an accurate enough account of the business.
- Ask the user for missing business meaning when code cannot establish it.
- Treat existing classes, tables, flags, and module boundaries as evidence, not truth.
- For a bug, use systematic root-cause investigation first. Repeated fixes across different
  components are evidence that the model or boundary may be wrong.

Do not equate a database schema, API payload, or class diagram with the domain model. These are
representations that may flatten, duplicate, or distort domain concepts.

Do not extract the concept model before the domain story is confirmed. If the user cannot confirm a
part, preserve it as an explicit inference or open question instead of silently treating it as fact.

### 2. Extract the As-Is concept model from the story

Build a shared vocabulary from:

- entities with identity;
- values and policies;
- commands, events, and outcomes;
- actors and external systems;
- lifecycles and time-dependent behavior.

Use the confirmed story as the extraction source:

- actors and business objects suggest candidate concepts;
- verbs and responsibilities suggest capabilities;
- decisions and calculations suggest policies;
- meaningful state changes suggest domain events and lifecycle transitions;
- “always,” “never,” and “only when” statements suggest invariants;
- possession, participation, and handoff language suggests relationships, ownership, and
  cardinality.

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

1. confirmed domain story and important scenarios;
2. problem and domain vocabulary;
3. evidence sources and unresolved assumptions;
4. As-Is concept relationship diagram;
5. As-Is problems and contradictions;
6. approved To-Be concept relationship diagram;
7. ownership and capability table;
8. lifecycle/state transitions;
9. invariants;
10. proposed architecture and data flow;
11. differences from the current design;
12. alternatives and trade-offs;
13. testing strategy based on invariants.

Scale the detail to the problem. A small change may need only a few paragraphs, but it never skips
the model. After approval, write the specification, then create the implementation plan.

## Architecture Warning

The following signals mean the implementation is exposing a modeling problem:

1. Fixing one path moves the same bug to another path.
2. Every fix adds another flag, token, generation, guard, or rollback branch.
3. The same business fact is stored or mutated in multiple modules.
4. A UI, transport, persistence, or delivery object owns a business lifecycle.
5. External I/O success or failure determines or rolls back domain truth.
6. Terminal or historical state can still be changed by late callbacks.
7. Nobody can answer in one sentence who owns a mutable fact.
8. Tests keep growing while the global invariants remain hard to state.
9. Controllers, reducers, and orchestration grow faster than the domain vocabulary.
10. A migration or compatibility path keeps two semantic writers active.
11. One behavior requires special branches across unrelated modules.
12. The design can only be explained with modules, callbacks, and execution order rather than
    domain concepts.

Use these mandatory checkpoints:

- **At the start of every feature:** perform a lightweight As-Is model check.
- **Before adding coordination state:** stop and re-evaluate ownership in Phase 3.
- **After a second cross-module regression:** perform a full To-Be redesign. Do not wait for a
  fourth fix.

Any structural signal above is enough to return to Phase 3. Do not pre-commit to a surgical fix.
First determine whether the current model can express the required invariants without more
coordination state.

## Common Rationalizations

| Excuse | Reality |
| --- | --- |
| “The user asked me to implement now.” | Urgency does not make an unknown model correct. |
| “It is a small field or endpoint.” | Small changes still encode ownership, lifecycle, and compatibility decisions. |
| “The architecture already exists.” | Existing code is evidence, not proof that the abstraction is right. |
| “The classes show us the domain model.” | Classes show the implemented interpretation; persistence and control flow may have distorted the domain. |
| “The code already tells the story.” | Code shows execution details. Translate it into business language and let the user correct its meaning. |
| “The nouns are obvious, so I can model them now.” | Concepts emerge from their role in a story; noun extraction alone creates accidental classes. |
| “I can infer what the user means.” | Label the inference and confirm it; do not turn uncertainty into architecture. |
| “One more flag or generation will make the race safe.” | Repeated coordination state usually means ownership or lifecycle is modeled at the wrong boundary. |
| “The external update failed, so domain state must roll back.” | Presentation and transport failures do not undo domain facts unless the domain explicitly says so. |
| “We need old and new writers active during migration.” | Two semantic writers create an overlap whose invariants cannot be guaranteed. Define one cutover boundary. |
| “The reducer and controller already handle every case.” | Sophisticated control flow may be compensating for the wrong concepts or aggregate boundary. |
| “A redesign is too expensive.” | Re-deriving the design may validate the current structure; it does not mandate a rewrite. |
| “I only need the smallest patch.” | The smallest local diff can create the largest global complexity. |
| “I identified the relevant modules.” | Module discovery is not domain discovery. |
| “Tests will catch mistakes.” | Tests cannot rescue a model that permits contradictory states. |

## Red Flags — Stop

- Searching for insertion points before naming the concepts
- Extracting concepts directly from class, table, or API names before telling the domain story
- Writing a domain story with implementation vocabulary
- Building a concept diagram before the user has corrected the inferred business narrative
- Producing only one diagram and calling it both current and proposed
- Presenting an inferred relationship as a confirmed domain fact
- Asking the user to approve a finished model without exploring uncertain concepts
- Proposing fields, flags, callbacks, or classes before assigning ownership
- Copying the existing architecture into the “design”
- Treating UI, API, transport, or database representations as the domain object by default
- Allowing presentation, persistence, timers, or callbacks to become additional semantic writers
- Letting late work mutate terminal or historical state
- Adding another identity token or ordering guard without revisiting the aggregate boundary
- Keeping parallel legacy and replacement writers alive for convenience
- Saying “surgical fix” after repeated cross-component regressions
- Producing an implementation plan without explicit invariants
- Using “pragmatic” to justify skipping redesign

**All of these mean: stop implementation and return to domain discovery.**
