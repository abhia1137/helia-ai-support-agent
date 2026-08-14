# Prompting log

AI was used as a drafting partner. Architecture choices, HITL line, and trade-offs are mine; I treated model output as a proposal and edited it down to the brief (critique + short design + one-page standard, no code).

## What I asked for

- Critique the teammate draft against the constraints (speed/cost, refund compliance, existing console, two teams).
- Separate **policy** (what may happen) from **enforcement** (gateway vs prompt).
- Bind money and identity from Helia, not from the model.
- Keep the existing console; do not design a second product.
- Cut length until it fit “a few lines of critique, 2–3 pages, one-page standard.”

## What I rejected from AI drafts

- Multi-agent Payments vs Accounts runtimes (two bills, two audit stories, one customer).
- Vendor bake-offs and wireframes (explicitly out of scope).
- Retry/backoff as the core reliability story (dangerous on refunds).
- “Human in the prompt” without a gateway 403.
- Auto-send of freeform replies.
- A code PoC — the assignment says design only.

## Where a human stays in the loop on this submission

- Dollar threshold ($50) and cost/latency budgets are stated as assumptions, not as discovered truth.
- Compensating actions vs “refunds cannot be undone” — I kept refunds irreversible and put the complexity before the call.
- Final wording and what to give up (day-one fleet launch, per-team snowflake agents, autonomous plan changes).
