# SEO baseline report

## 1. Executive summary

- Business goal:
- Audit mode and environment:
- Population/sample inspected:
- Excluded or blocked areas:
- Highest-confidence constraints and opportunities:

## 2. Evidence ledger

| ID | Artifact/locator | Method and tool/version | Observed at/timezone | Environment/property | Filters/config | Coverage/sample | Quality | Limits/raw snapshot |
|---|---|---|---|---|---|---|---|---|

Quality: `validated`, `limited`, or `unvalidated`.

## 3. Module coverage

| Module | Status | Evidence IDs | Reason / prerequisite / owner / unblock condition |
|---|---|---|---|

Final status: `applicable`, `blocked`, or `not_applicable`. Resolve interim `deferred` and `not_checked` before delivery. Every `blocked` row must name its prerequisite, owner, and unblock condition.

## 4. Findings

Repeat this card for every eligible finding. An observation may be only `observed` or `corroborated`. Diagnosis, impact, and action support may be `observed`, `corroborated`, or `inferred`, but each must name its evidence, governing rule, or assumptions. Put `manual_check` and unvalidated possibilities in section 6 without P0–P3.

### F01 — [P0–P3] Finding title

- **Observation:** [bounded statement] — support: [label]; evidence: [IDs]; breadth: [URL/template/site/sample].
- **Diagnosis:** [best explanation and alternatives] — support: [label]; basis: [evidence IDs, rule, assumptions]; confidence: [high/medium/low] because [rationale].
- **Impact:** [bounded business/search consequence] — support: [label]; basis: [evidence IDs, model, assumptions]; confidence: [high/medium/low] because [rationale].
- **Action:** [fix, diagnostic, or experiment] — support: [label]; basis: [evidence IDs, decision rule, assumptions]; confidence: [high/medium/low] because [rationale].
- **Priority rationale:** [verified breadth × bounded impact × confidence; explain why this priority].
- **Dependency / owner:**
- **Acceptance criterion:**
- **Verification / rollback:**

## 4A. Conditional module evidence

Include only applicable module outputs. Preserve full tables here or link their evidence artifact.

- **Technical:** sampled URL table with status, directives, canonical, discovery, source/render, schema, performance provenance, and breadth.
- **Architecture:** page-intent matrix and SERP gate records.
- **On-page:** sampled-page worksheets, repeated patterns, and roll-up breadth.
- **Analytics:** validation status, segments, denominators, metric table, alternatives, and proposed tests.
- **GEO/AI:** causal node, sampling/identity method, denominators, limits, intervention, outcome, and recheck design.

## 5. Sequenced plan

Priority expresses impact; horizon expresses dependency, effort, capacity, and release constraints.

### Now

- [ ] Action — owner — dependency — acceptance criterion

### First week

- [ ] Action — owner — dependency — acceptance criterion

### First month

- [ ] Action — owner — dependency — acceptance criterion

### Later experiments

- [ ] Experiment — owner — hypothesis — primary outcome — stopping/recheck rule

## 6. Missing evidence and next diagnostics

Repeat this card for every blocked or `manual_check` candidate. Do not assign P0–P3.

### M01 — Question or blocked decision

- **Current observation:** [what is known] — evidence: [IDs]; breadth: [scope/sample].
- **Candidate diagnosis / alternatives:** [explicitly provisional].
- **Current confidence:** [high/medium/low] because [rationale].
- **Exact missing observation:**
- **Collection method and required scope:**
- **Dependency / owner:**
- **Decision unlocked:** [what can be concluded or prioritized after collection].
- **Verification:** [how sufficiency and validity will be checked].

## 7. Measurement and recheck

- Baseline period, segments, denominators:
- Primary and secondary outcomes:
- Concurrent changes and confounders:
- Recheck trigger/date:
- Raw evidence location:

## 8. Final gate

| Check | Pass / fail / N/A | Evidence or remediation |
|---|---|---|

- Scope/environment explicit
- Material observations cite evidence IDs
- Breadth matches sample
- Observation, diagnosis, impact, and action remain separate
- Unknowns excluded from prioritized findings
- SERP and measurement gates recorded where applicable
- AI causal layers kept distinct where applicable
- Dependencies, owners, acceptance criteria, and verification complete
- No unresolved `deferred` or `not_checked`; every `blocked` module has prerequisite, owner, and unblock condition
