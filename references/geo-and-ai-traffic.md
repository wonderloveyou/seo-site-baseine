# GEO and AI visibility

Treat GEO as an evidence and distribution layer. Use this causal map:

`crawler access → provider-side processing/indexing → retrieval or parametric generation → mention or recommendation → citation/link → human referral → measured conversion`

Each arrow is a separate hypothesis. Site-side evidence normally cannot establish training-data inclusion, provider-side indexing, parametric memory, or the cause of a generated answer.

## Evidence classes

### Crawler access

Record evidence ID, timestamp, URL, status, bytes, frequency, user-agent, and identification method. User-agent alone is spoofable and supports only `inferred` provider activity. Use validated IP/rDNS or provider-published ranges when available and state their limits.

### Retrieval, mentions, and citations

Record the exact product, model/version when exposed, account state, locale, prompt, conversation context, answer, linked and unlinked sources, and observation time. A generated mention does not reveal whether retrieval, training, parametric memory, or another mechanism produced it.

### Human referrals

Define referral-source rules, landing URLs, UTMs when available, session/event deduplication, consent constraints, and referrer loss. Report known-attributed AI referrals as a lower-bound subset rather than total AI influence.

### Business outcomes

Use a metric hierarchy: access diagnostics, retrieval/citation observations, known-attributed visits, engaged visits, qualified conversions, and revenue. Upstream signals diagnose a pathway; they are not substitutes for downstream outcomes.

## Prompt-monitoring protocol

Treat synthetic monitoring as an experiment:

1. Version and freeze a representative prompt panel stratified by user intent.
2. Record exact prompts and conversation state; use clean sessions where practical.
3. Repeat each prompt/product/locale cell and preserve raw outputs.
4. Code mention, recommendation, linked citation, unlinked attribution, factual support, and absence separately.
5. Report counts with denominators, variation, and known routing/personalization limits.
6. Re-run a stable control panel on every observation date.

Predeclare primary and secondary outcomes, observation window, minimum useful sample, guardrails, and reporting/stopping rules. Prompt checks alone do not prove durable visibility.

Derive sample size and repetitions from the decision, prompt population, observed variability, cost, and acceptable uncertainty. Present any numeric panel size as a study-specific design choice, not a universal minimum.

## Content and access review

Check crawl/render access, explicit current attributable facts, unambiguous entities and dates, useful answer blocks, schema matching visible content, and original cases/research. Frame changes as interventions on a named node in the causal map rather than promises of citation or revenue.

## Module output

Return the causal node observed, evidence IDs, identification/sampling method, denominator, artifact quality, finding limits, alternative mechanisms, proposed intervention, primary outcome, and recheck design.
