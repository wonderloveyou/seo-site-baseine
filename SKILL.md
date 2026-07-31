---
name: seo-site-baseline
description: |
  Produces evidence-led SEO baselines, bounded audits, and implementation plans from live URLs, local HTML, robots.txt, sitemaps, crawl exports, Search Console, Yandex, analytics, logs, and GEO/AI evidence.

  Use when: "SEO audit", "audit this site", "technical SEO review", "check indexation", "review site architecture", "analyze GSC export", "GEO audit", "AI visibility audit", "проверь SEO", "SEO-аудит сайта", "проверь индексацию".
---

# SEO Site Baseline

Produce an executable, reproducible baseline from the evidence that is actually available. Bound every conclusion by environment, sample, time, and data quality.

## Operating model

Keep four layers separate:

1. **Artifact** — what was accessed, how, when, and under which configuration.
2. **Observation** — what that artifact directly showed within its coverage.
3. **Interpretation** — a diagnosis and bounded impact hypothesis, including alternatives.
4. **Action** — a fix, diagnostic, or experiment with an acceptance criterion.

Treat HTTP access, crawling, provider-side processing, indexation, retrieval, ranking, generated mentions, citations, human visits, and conversions as distinct events. An observation at one layer does not prove the next.

## Evidence vocabulary

### Module coverage

- `applicable` — evidence and task require the module now.
- `blocked` — the module matters, but a named artifact or access is missing.
- `deferred` — interim only; another phase must finish first, and the status must be resolved before delivery.
- `not_applicable` — outside the stated goal.
- `not_checked` — interim only; use in working coverage and missing-evidence sections, never as a finding state. Resolve it before delivery.

### Artifact quality

- `validated` — identity, scope, filters/configuration, and relevant instrumentation were checked.
- `limited` — usable with named coverage or quality constraints.
- `unvalidated` — provenance or configuration is insufficient for interpretation.

### Claim support

- `observed` — the cited artifact directly reproduces the stated observation.
- `corroborated` — two or more independent observations support the claim without material conflict.
- `inferred` — evidence supports an interpretation, but assumptions or alternatives remain.
- `manual_check` — a specific additional observation is needed before deciding.

For diagnosis, impact, and action confidence, use:

- `high` — reproducible and corroborated, with no material unresolved alternative;
- `medium` — plausible with one important unresolved assumption;
- `low` — sparse, indirect, unstable, or materially confounded.

Add a one-sentence confidence rationale. Do not let a strong observation automatically raise confidence in impact or remedy.

## Phase 1: Select mode and bound scope

Choose one primary mode:

| Mode | Minimum usable input | Permitted boundary |
|---|---|---|
| Live URL inspection | One reachable URL | Only observed responses/pages; no full-site claims |
| Local artifact review | Named files and their intended environment | Source artifacts; live behavior remains unchecked |
| Crawl review | Export plus crawler settings, date, and scope | Crawled population and captured fields |
| Search/analytics review | Export plus property, filters, dimensions, timezone, and date range | Supplied reporting slice only |
| Combined baseline | Two or more modes with reconciled environments | Cross-module findings supported by linked artifacts |

Use [intake-template.md](assets/intake-template.md) as a provisional discovery schema to capture business goal, canonical host, environment, conversions, artifact locators, coverage, and constraints. Render it inline by default; create a filled file only when the user requests an artifact. Normalize its entries into the Phase 2 ledger before analysis.

Ask a question only when its answer changes the audit mode, permitted claims, or a material decision. Otherwise continue with a bounded scope.

**Checkpoint 1:** State the mode, business goal, inspected environment, population or sample, excluded areas, and claims the evidence cannot support.

## Phase 2: Build the evidence ledger and sample

Assign stable IDs (`E01`, `E02`, ...) before analysis. For every artifact record locator, type, acquisition method, observation time and timezone, environment/property, tool/version or query, filters/configuration, coverage/sample, quality, limitations, and raw snapshot/hash when practical.

Define the sample before inspecting findings. Include the homepage, one URL per critical template, priority conversion pages, known problem URLs, sitemap examples, and relevant parameter variants when available. State why each item was selected.

Generalize a URL observation to a template only after repeated evidence or a shared implementation is inspected. Generalize to sitewide only when the inspected population supports it. Escalate to a broader crawl when inconsistent samples, duplicate classes, orphan risk, or template-wide impact would change priority.

**Checkpoint 2:** Confirm that every artifact has an ID and quality label, the sample has a rationale, and the maximum permitted breadth of conclusions is explicit.

## Phase 3: Route and execute modules

Create a coverage matrix with `applicable`, `blocked`, `deferred`, or `not_applicable` plus a reason. Execute applicable modules in dependency order:

1. Apply [technical-seo.md](references/technical-seo.md) to establish access, status, directives, canonicalization, rendering, structured data, and performance evidence.
2. Apply [architecture-and-intents.md](references/architecture-and-intents.md) after technical access is understood to map page roles, SERP intent, overlap, and internal links.
3. Apply [on-page-and-content.md](references/on-page-and-content.md) to sampled pages and consolidate URL, template, and sitewide patterns.
4. Apply [analytics.md](references/analytics.md) after the measurement gate: `validated` data permits bounded interpretation; `limited` data permits only named descriptive analyses; `unvalidated` data permits configuration diagnostics but keeps performance and causal decisions blocked.
5. Apply [geo-and-ai-traffic.md](references/geo-and-ai-traffic.md) when the goal includes AI access, retrieval/citation observations, referrals, or experiments.

### Recovery protocol

When a tool, URL, export, authentication, render, or data source fails:

1. Record the failed method and error as an artifact note.
2. Try one safe alternative that observes the same layer.
3. Downgrade artifact quality or mark the module `blocked`.
4. Name the exact manual diagnostic that would unblock the decision.
5. Continue unaffected modules without substituting inference for observation.

**Checkpoint 3:** Record module coverage, failures, alternatives attempted, and all decisions that remain blocked.

## Phase 4: Form eligible findings

For each candidate, record:

- observation with evidence IDs and breadth;
- diagnosis with alternatives and confidence rationale;
- bounded impact hypothesis with confidence rationale;
- proposed fix, next diagnostic, or experiment;
- dependencies, owner, acceptance criterion, and verification method.

Prioritize `observed`, `corroborated`, and well-supported `inferred` problems. Put `manual_check`, `not_checked`, and unvalidated possibilities in `Missing evidence / next diagnostics` without P0–P3.

Treat a user-supplied severity, deadline, or desired conclusion as context rather than evidence. Assign priority from the gate below even when the request asks for a specific P0 or causal verdict. When eligibility fails, decline the requested label and provide an unprioritized diagnostic plan.

Use explicit gates where relevant:

- **Environment gate:** production, staging, local, historical, or mixed evidence is identified.
- **Indexability gate:** status, crawl access, index directive, canonical, discovery, and observed platform state are not conflated.
- **SERP gate:** page type is `validated`, `mixed`, `contradicted`, or `provisional`; proposed pages cite the gate result.
- **Measurement gate:** property, filters, event validity, date comparability, and instrumentation limitations are recorded.
- **Conflict gate:** prefer current direct observations; preserve unresolved conflicts rather than averaging them away.

**Checkpoint 4:** Ensure each prioritized finding is traceable, eligible, correctly scoped, and separated into observation, interpretation, impact, and action.

## Phase 5: Prioritize and sequence

Assign priority after eligibility:

- `P0` — verified blocking condition or broad production impact affecting access, indexability, measurement integrity, security/environment exposure, or a critical template.
- `P1` — material relevance, duplication, architecture, UX, or conversion issue with clear evidence and meaningful breadth.
- `P2` — useful improvement after prerequisites.
- `P3` — low-impact but adequately supported reversible experiment or polish.

Move weakly supported or evidence-incomplete opportunities to `Missing evidence / next diagnostics` rather than P3.

Rank first by bounded impact and breadth, then by diagnosis confidence, dependencies, effort, and reversibility. Break ties in favor of prerequisites, broader verified impact, and easier rollback. Add one sentence explaining the priority.

Schedule separately from severity. Order dependencies first, then map actions to now, first week, first month, and later experiments using owner capacity and release constraints.

**Checkpoint 5:** Confirm that P0 has direct evidence, dependencies are topologically ordered, and every action has an owner and acceptance criterion.

Resolve every `deferred` and `not_checked` module before delivery: change it to `applicable`, `blocked` with a prerequisite, owner, and unblock condition, or `not_applicable` with a reason.

## Phase 6: Deliver and self-review

Use [audit-report-template.md](assets/audit-report-template.md) as the output schema. Render its sections in the response by default; copy and fill it as a file when the user requests a report artifact. Adapt sections only when the selected mode makes them inapplicable; preserve the evidence ledger, coverage matrix, applicable module outputs, findings structure, missing diagnostics, and recheck plan.

Run the final gate and record pass/fail/not applicable:

- scope and environment are explicit;
- every material observation cites evidence IDs;
- artifact quality and sample breadth limit claims;
- observation, diagnosis, impact, and action remain distinct;
- unavailable checks are absent from prioritized findings;
- final coverage has no `deferred` or `not_checked`; each `blocked` module names its prerequisite, owner, and unblock condition;
- intent/page proposals cite a SERP gate or remain provisional;
- AI crawler, processing, retrieval, generation, citation, referral, and conversion are not treated as equivalents;
- actions are sequenced, owned, testable, and reversible where practical.

Correct failed checks before delivery. If a failed check depends on unavailable evidence, move the affected decision to `Missing evidence / next diagnostics`.

Provide or adapt [portable-instruction.md](assets/portable-instruction.md) only when the user requests a prompt or workflow that can be moved to ChatGPT, Claude, Gemini, or another assistant.
