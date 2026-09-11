# Safety story — what to weigh

Not a checklist. These are the categories that usually matter; rank what applies to
*this* change and keep only the top 2 per list.

**What gives confidence**

- The author's testing, in their own words (step 3).
- Narrow scope — one call site, one module, no shared code touched.
- Existing automated coverage over the paths the diff touches.
- Flag-gating or a kill switch.

**Risks to review**

- Migrations and anything that rewrites persisted data.
- Behavior changes for existing users.
- Paths the author did not exercise.
- Third-party integrations and network error handling.
- Perf-sensitive code.
- Limited author testing (diff review only, no device run).
