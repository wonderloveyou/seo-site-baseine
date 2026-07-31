# Search and conversion analytics

## Measurement gate

Validate in this order:

1. Confirm property, environment, host filters, search type, timezone, consent constraints, and export dimensions.
2. Reproduce priority events in debug or realtime tools; check naming, deduplication, and known loss.
3. Record date ranges, comparison logic, segments, denominators, and instrumentation changes.
4. Label the dataset `validated`, `limited`, or `unvalidated` before interpretation.

Separate discovery/crawl, indexation, impressions and position, clicks and CTR, landing engagement, conversion events, qualified pipeline, and revenue.

## Diagnostic sequence

- For impressions with weak CTR, inspect query intent, snippet accuracy, position distribution, device/locale, and SERP features before rewriting.
- For positions around 8–20, inspect intent fit, decision-relevant subtopics, internal-link evidence, competing result types, accuracy, originality, and usefulness.
- For absent visibility or indexation, inspect platform coverage evidence, access, status, canonical, index directive, sitemap, discovery, and page value as separate signals.
- For irrelevant queries, compare page purpose, headings, copy, anchors, and competing page roles.
- For traffic without outcomes, validate tracking first; then inspect traffic intent, offer, CTA, and friction.

## Change measurement

Treat a simple pre/post comparison as descriptive. Record concurrent releases, campaigns, tracking changes, demand shifts, seasonality, algorithm changes, and regression-to-the-mean risk.

When the decision warrants stronger causal language, prefer a defensible design such as phased rollout, matched untreated pages, interrupted time series with an adequate pre-period, or difference-in-differences with stated assumptions. Report effect size, denominator, uncertainty, and residual confounding.

Derive observation windows and sample requirements from update latency, decision horizon, seasonality, traffic volume, variance, and acceptable uncertainty. Present numeric periods as study-specific choices rather than universal minimums.

Use priority, dependency, and business value to choose the design; a baseline audit can recommend an experiment without pretending one already exists.

## Module output

Return property/configuration evidence, validation status, date/segment definition, metric table with denominators, observed patterns, alternative explanations, proposed tests, primary outcome, recheck window, and known confounders.
