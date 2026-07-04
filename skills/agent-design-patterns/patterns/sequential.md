# Sequential pattern

**Google Cloud:** specialized agents in a fixed linear order; each agent's output is the next
agent's input. Orchestration is code, not a model.

**When to use:** highly structured process with a fixed order — a dependency chain where stage
N genuinely needs stage N−1's result. If the stages are independent, use Parallel instead.

**Primitive:** `pipeline(items, stage1, stage2, ...)`. For a single linear run, pass one item.
No barrier between stages — but with one item that's moot; the value is the ordered handoff.

## Template

```js
export const meta = {
  name: 'sequential-chain',
  description: 'Run ordered stages where each feeds the next',
  phases: [{ title: 'Analyze' }, { title: 'Fix' }, { title: 'Verify' }],
}

const [out] = await pipeline(
  [args.target],                                          // one item = one linear run
  t  => agent(`Analyze ${t}. Return the root cause and the exact file:line to change.`,
              { phase: 'Analyze', schema: { type: 'object',
                properties: { cause: {type:'string', maxLength: 600}, site: {type:'string', maxLength: 120} },
                required: ['cause','site'] } }),
  a  => agent(`Given root cause "${a.cause}" at ${a.site}, produce the minimal patch.`,
              { phase: 'Fix', schema: { type: 'object',
                properties: { patch: {type:'string', maxLength: 2000} }, required: ['patch'] } }),
  f  => agent(`Verify this patch resolves the issue and introduces no regression:\n${f.patch}`,
              { phase: 'Verify', schema: { type: 'object',
                properties: { ok: {type:'boolean'}, note: {type:'string'} }, required: ['ok'] } }),
)
return out
```