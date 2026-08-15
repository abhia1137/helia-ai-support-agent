# Helia agent actions — team standard

**Owner:** Agent Platform. Payments and Accounts both build to this page.  
**Exceptions:** Platform + the risk owner (Finance for money, Legal for PII/comms).  
**If you cannot meet it, the action does not ship.** We will not “standardise later.”

---

### 1. The model proposes. The gateway acts.

- Agent code calls **registered tools**, never Helia APIs.
- `customer_id`, `charge_id`, amounts, and account fields are **bound from `CaseContext`** (loaded for this ticket). Model output is not the source of those values.
- Every mutation has an `idempotency_key`. Retrying a mutation without a new key is a bug.
- Mutation preconditions (like "no refund exists on this charge") are re-checked **at execute time**, not only at planning — a human may act on same case from console in parallel.

---

### 2. What needs a human

Enforced in the **gateway**. A prompt that says “ask a human” is not enough.

| Action | Human required? |
|--------|-----------------|
| Look up account (this ticket’s customer) | No |
| Escalate to human | No |
| Draft a reply on the case | No |
| Send a freeform reply | **Yes** |
| Send an allowlisted template after a successful action in this run | No |
| Update account | **Yes** |
| Change plan | **Yes** |
| Refund ≤ $50, full amount of last captured charge, all policy predicates pass | No — approver = `policy:<version>` |
| Any other refund (partial amount, older charge, above ceiling) | **Yes** — approver = `human:<id>`, senior role above supervisor limit |
| Answer a question covered by approved KB articles | Draft-first (human reviews); auto-send only after the eval bar (rollout Phase 5) |
| Answer a question KB does not cover | Escalate — model memory is not a source |
| Send a holding / acknowledgement template (no claim) | No |

Changing a row is a **policy version bump**, not a prompt edit. Finance owns the dollar figure.

---

### 3. What each tool is allowed to do

- **One** Helia capability per tool. No `doWhatever(json)`.
- Register: side effect, max amount/fields, bind list, approval class, compensating action (or `irreversible`), PII class.
- **Refund:** `charge_id` + bound amount only. No `amount` argument from the model. Auto path refunds the full last captured charge only, within Finance's refund window; partial or older charge goes to human.
- **Reply send modes:** `draft` (human sends), `send_template(id)` (after a successful action; gateway fills placeholders), `send_kb_grounded` (question answer — **draft-first**, earns auto-send only after the claim-grounding eval bar; may only contain links/values from the returned KB articles), `send_freeform` (always human), and **acknowledgement templates** (allowed with no prior action because they make no claim). No `send_raw`.
- **Lookup:** this ticket’s customer only. **KB lookup:** read-only, approved articles only.
- **Plan / account:** payload is a diff against `CaseContext`.
- **Approval token:** minted and signed by the **gateway** only (key in KMS; console just relays the human click). Single-use, 30-min TTL, generic shape `{case_id, customer_id, tool, params_hash, policy_version, approver}`. Kill-switch and all preconditions are re-checked at execute time, so a stale token will not fire.

---

### 4. What must be logged (every run)

`run_id`, `case_id`, model id, prompt hash, proposal, bound parameters, policy decision + version, **approver** (and on a reject: `denied` + reason), tool request/response hashes, account snapshot before/after, outcome, kill-switch flags.

This log is the Finance/Legal record of who approved **or denied** what. Retention per Legal. No PAN/SSN in model traces.

---

### 5. What has to pass before it ships

- [ ] Tool registered in the gateway (no private HTTP client)
- [ ] Policy row + approval class
- [ ] Contract tests
- [ ] Golden evals: happy path, prompt injection, wrong customer, double refund, partial failure (action vs reply), and for any auto-send answer: hallucinated citation + claim-does-not-match-article
- [ ] Inbound PII redaction runs before the prompt is built (not just before logging)
- [ ] Shadow on historic tickets
- [ ] Dashboard: success, overturn, dollars moved, cost/case
- [ ] Kill switch `agent.tool.<name>`
- [ ] Canary ≤ 5% of eligible traffic
- [ ] Named owner (Payments or Accounts) on-call for that tool

---

### 6. Loop, cost, console

- Max **6** steps, **$0.03**, **25s** auto path. Mutations: no retry. If unsure: escalate.
- Ship onto the **existing case**. Do not launch a second support UI. Humans can always act with the agent off.

---

### 7. Two teams, one plane

You own your tool’s evals and on-call. You do not own a private agent, a private policy, or a private audit trail. Platform reviews new tools. The kill switch is operational, not a design doc.
