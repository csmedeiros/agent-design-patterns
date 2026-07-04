# Loop pattern

**Google Cloud:** agents execute a sequence repeatedly, evaluating between cycles, until a
termination condition is met.

**When to use:** unknown number of iterations — retry-until-success, refine-until-good-enough,
or discover-until-dry (K consecutive empty rounds). A fixed fan-out would either stop short or
waste rounds.

**Primitive:** a JS `while` around `agent()`. Track a counter or a "dry" streak. Guard the
loop so it can't run forever (max rounds, or `budget.total && budget.remaining() > ...`).

## Template — loop until dry

```js
export const meta = {
  name: 'loop-until-dry',
  description: 'Repeat find→verify rounds until K consecutive rounds surface nothing new',
  phases: [{ title: 'Find' }, { title: 'Verify' }],
}

const seen = new Set(), confirmed = []
let dry = 0, round = 0
while (dry < 2 && round < 8) {                             // exit condition + hard cap
  round++
  const found = await agent(`Round ${round}: find issues in ${args.target} not already seen.`,
    { label: `find#${round}`, phase: 'Find', schema: { type: 'object',
      properties: { issues: { type: 'array', items: { type: 'object',
        properties: { key:{type:'string'}, desc:{type:'string'} }, required:['key','desc'] } } },
      required: ['issues'] } })
  const fresh = found.issues.filter(i => !seen.has(i.key))
  if (!fresh.length) { dry++; log(`round ${round}: dry (${dry}/2)`); continue }
  dry = 0; fresh.forEach(i => seen.add(i.key))             // dedup vs seen, NOT vs confirmed
  const judged = await parallel(fresh.map(i => () =>
    agent(`Is this real? ${i.desc}. Default to false if uncertain.`,
      { phase: 'Verify', schema: { type:'object', properties:{ real:{type:'boolean'} },
        required:['real'] } }).then(v => ({ i, real: v?.real }))))
  confirmed.push(...judged.filter(Boolean).filter(j => j.real).map(j => j.i))
}
return confirmed
```