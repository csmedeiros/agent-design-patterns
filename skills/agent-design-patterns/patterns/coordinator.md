# Coordinator pattern

**Google Cloud:** a central coordinator analyzes the request, decomposes it into subtasks, and
routes each to the appropriate specialist. Dispatch is model-driven (dynamic), unlike Parallel's
fixed logic.

**When to use:** the work splits into parts of *different kinds* that each need a different
specialist lens, and you don't know the split up front — a classify step decides the routes.

**Primitive:** one `agent()` that classifies/routes → `parallel()` fanning each routed item to
a specialist prompt keyed by its route. The routing decision is what makes it Coordinator, not
plain Parallel.

## Template

```js
export const meta = {
  name: 'coordinator',
  description: 'Classify each unit, route to the matching specialist',
  phases: [{ title: 'Route' }, { title: 'Specialists' }],
}

const SPECIALISTS = {                                       // route → lens
  backend:  f => `Review ${f} as a PHP/Laravel backend reviewer: services, queries, contracts.`,
  blade:    f => `Review ${f} as a Blade/UX reviewer: template correctness, i18n, a11y.`,
  react:    f => `Review ${f} as a React reviewer: state, effects, render correctness.`,
}

const routed = await agent(
  `Classify each of these files into one of [backend, blade, react] by its role:\n` +
  args.files.join('\n'),
  { phase: 'Route', schema: { type: 'object', properties: { routes: { type: 'array',
    items: { type: 'object', properties: { file:{type:'string'},
      route:{type:'string', enum:['backend','blade','react']} }, required:['file','route'] } } },
    required: ['routes'] } })

const reviews = (await parallel(routed.routes.map(r => () =>
  agent(SPECIALISTS[r.route](r.file),
    { label: `${r.route}:${r.file}`, phase: 'Specialists' })
    .then(text => ({ ...r, review: text }))
))).filter(Boolean)
return reviews
```
