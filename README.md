# Helia Support Agent — design (AI Principal Engineer)

**Author:** Abhi Arora  
**Role in this exercise:** Principal Engineer — design how the team builds this safely. Not a product build.

This repo is the screening submission. No application code: the brief asks for a critique, a short design, and a one-page team standard.

| Read | What it is |
|------|------------|
| [DESIGN.md](./DESIGN.md) | Critique of the draft, then architecture, agent loop, human-in-the-loop, failure/rollback, guardrails, rollout |
| [TEAM_STANDARD.md](./TEAM_STANDARD.md) | The one page Payments and Accounts both build to |
| [PROMPTING_LOG.md](./PROMPTING_LOG.md) | How AI was used on this submission |

**Thesis:** the model proposes; a shared gateway acts. Refund amounts and account identifiers come from Helia, not from the model. Human approval is enforced in that gateway, on the existing support console, not in a prompt. One platform, two teams, a kill switch.

**Assumptions** (used wherever the brief is silent) are listed at the top of `DESIGN.md`.
