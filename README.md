# Domain Modeling

A design-first Agent Skill that requires domain concepts, relationships, ownership, capabilities,
lifecycles, and invariants to be understood before implementation begins.

The Skill reconstructs the model implied by existing code and user knowledge, challenges that model,
and guides the Agent and user toward an explicitly approved To-Be concept model.

## Install

Install with the Skills CLI:

```bash
npx skills add wangmingliang-ms/domain-modeling --skill domain-modeling
```

Install with GitHub CLI:

```bash
gh skill install wangmingliang-ms/domain-modeling domain-modeling
```

## When it activates

Agent Skills use progressive disclosure:

1. The Agent initially sees only Skill metadata, primarily `name` and `description`.
2. The Agent loads the full `SKILL.md` only after the description matches the current task.
3. Supporting resources are loaded later only when the Skill asks for them.

The README is for people; it is not the reliable activation mechanism. Trigger conditions therefore
belong in the `description` field. This Skill is designed to activate before every product feature
or behavior change, including apparently small additions to existing code. It also activates when
architecture symptoms appear, including:

- the same state has multiple writers;
- correctness depends on flags or callback order;
- fixes move bugs between paths;
- late events mutate completed work;
- a behavior requires branches across unrelated modules.

## What it enforces

- Existing code is evidence of a model, not proof that the model is correct.
- Existing behavior is first translated into a natural-language domain story and corrected by the
  user before concepts are extracted.
- Concepts, capabilities, events, relationships, and invariants are derived from the confirmed
  domain story rather than copied from classes or schemas.
- As-Is and To-Be concept models remain distinct.
- Important claims are labeled as observed, user-stated, inferred, or proposed.
- The user and Agent iteratively refine the concept relationship model.
- Implementation waits until the To-Be model and its invariants are approved.

## License

MIT
