# Hierarchical task decomposition pattern

**Google Cloud:** a root agent decomposes a complex/ambiguous task into subtasks, delegating to
subagents that may decompose further, across multiple levels, until subtasks are directly
executable.

**When to use:** the task is too big/ambiguous to hand to one agent, and you don't know the
unit list up front — a planning agent must produce it first. Differs from Coordinator: here the
root *plans and decomposes*, it doesn't just route pre-existing parts.

**Primitive:** a plan `agent()` that emits the unit list → `pipeline()` over the units, where
each unit's own chain can decompose again (a stage that itself splits work). One nesting level
via `pipeline` stages is usually enough; go deeper only if a unit is itself large.

## Template

```js
export const meta = {
  name: 'hierarchical-decompose',
  description: 'Root plans units; each unit runs its own implement→test sub-chain',
  phases: [{ title: 'Plan' }, { title: 'Implement' }, { title: 'Test' }],
}

const plan = await agent(
  `Decompose "${args.goal}" into independent, directly-executable units. ` +
  `For each: a title and a one-line scope.`,
  { phase: 'Plan', schema: { type: 'object', properties: { units: { type: 'array',
    items: { type: 'object', properties: { title:{type:'string'}, scope:{type:'string'} },
      required:['title','scope'] } } }, required: ['units'] } })

const done = await pipeline(
  plan.units,
  u => agent(`Implement unit "${u.title}": ${u.scope}. Return the change plan.`,
             { label: `impl:${u.title}`, phase: 'Implement' }),
  (impl, u) => agent(`Write the test that proves unit "${u.title}" works. Plan:\n${impl}`,
             { label: `test:${u.title}`, phase: 'Test' })
             .then(test => ({ unit: u.title, impl, test })),
)
return done.filter(Boolean)
```