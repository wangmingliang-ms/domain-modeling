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

## What it enforces

- Existing code is evidence of a model, not proof that the model is correct.
- As-Is and To-Be concept models remain distinct.
- Important claims are labeled as observed, user-stated, inferred, or proposed.
- The user and Agent iteratively refine the concept relationship model.
- Implementation waits until the To-Be model and its invariants are approved.

## License

MIT
