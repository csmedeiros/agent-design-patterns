# Parallel pattern

**Google Cloud:** independent subagents run concurrently; their outputs are synthesized into
one consolidated response. Orchestration is predefined logic, not a model.

**When to use:** subtasks are independent (no shared state, no order dependency) and you want
to cut wall-clock time. Common for reviewing many files/dimensions at once, or fanning a
search across angles.

**Primitive:** `parallel(thunks)` — a barrier: awaits all, each failed thunk resolves to
`null` (so `.filter(Boolean)`). Follow with a synthesis stage that needs *all* results.

## Template

```js
export const meta = {
  name: 'parallel-fanout',
  description: 'Review independent units concurrently, then synthesize',
  phases: [{ title: 'Review' }, { title: 'Synthesize' }],
}

const units = args.files                                   // independent items
const results = (await parallel(units.map(f => () =>
  agent(`Review ${f} in isolation. Report defects and risks as findings.`,
        { label: `review:${f}`, phase: 'Review', schema: { type: 'object',
          properties: { findings: { type: 'array', items: { type: 'object',
            properties: { file:{type:'string'}, summary:{type:'string'} },
            required:['file','summary'] } } }, required: ['findings'] } })
))).filter(Boolean)                                        // drop dead thunks

const all = results.flatMap(r => r.findings)               // barrier is justified: need ALL
const summary = await agent(
  `Synthesize these ${all.length} findings into one consolidated review, deduped:\n` +
  JSON.stringify(all), { phase: 'Synthesize' })
return { findings: all, summary }
```