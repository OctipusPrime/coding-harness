Whenever a new conversation is created and you see no history, read `devlog.md` to get up to speed.

When doing manual testing with viewing of the UI and using the browser related harness, USE SUBAGENTS instructing them to test this feature. 

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## Where To Look

- `devlog.md`: current product shape, architecture history, and recent project direction.
- `README.md`: setup, env, scripts, route notes, and the fastest current-state overview.
- `contribution-guide.md`: repo conventions, package boundaries, canvas UI direction, and testing guidance.
- `docs/deployment.md`: web/API deployment commands, required env, CORS, and prompt asset notes.
- `apps/web`: Vite/React SPA, routes, pages, components, hooks, browser Supabase client, and web tests.
- `apps/api`: Hono/Mastra API, route groups, auth/validation middleware, server env loading, server OpenAI/Supabase adapters, tester access, and API tests.
- `apps/api/src/mastra/README.md`: Mastra agents, tools, prompt registry, runtime wrappers, and request-context rules.
- `packages/domain`: shared pure domain contracts and algorithms, including entity constants, world/session-prep types, layout logic, and DOT parsing.
- `apps/api/src/llm`: API-owned provider/model selection, completion wrappers, and Realtime session creation.
- `apps/api/src/mastra`: agents, prompts, tools, workflows, LLM-facing parsers, chunking, and AI pipelines.
- `packages/data`: shared database and persistence contract types.
- `supabase/migrations`: database schema and RLS changes.
- `e2e`: Playwright flows and browser/auth fixtures.
- `scripts`: local operator/debug helpers and standalone pipeline runners.
- `docs/plans` and `docs/issues`: planning and historical notes; check status headers before treating them as current instructions.
