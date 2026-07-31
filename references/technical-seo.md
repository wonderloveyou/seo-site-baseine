# Technical SEO and indexation

## Technical additions to the Phase 2 sample

Include representative status/redirect variants, sitemap examples, known failures, parameter classes, and JavaScript-dependent templates when relevant. Record the selection rationale and evidence IDs.

Escalate from sampling to a broader crawl when:

- sampled URLs from one template disagree;
- duplicate, parameter, pagination, orphan, or redirect classes may be broad;
- a shared component can affect access, metadata, links, or rendering;
- breadth would change priority or implementation ownership.

## Inspection sequence

1. Resolve the intended canonical protocol and host; capture requested URL, final URL, status, and redirect chain.
2. Check representative HTTP responses and headers, including `X-Robots-Tag`.
3. Inspect robots.txt as crawl guidance. A missing file normally means crawling is unrestricted; report it only when explicit crawl rules or sitemap discovery are operationally needed.
4. Inspect page-level robots directives.
5. Compare canonical targets with redirects, sitemaps, internal links, hreflang where relevant, and intended indexability.
6. Check that submitted sitemap URLs are useful, canonical, indexable 200 responses; treat submission as discovery evidence, not proof of indexation.
7. Inspect duplicate classes, parameters, internal search, filters, staging/test URLs, soft 404s, and redirect chains.
8. Compare source HTML and rendered DOM when JavaScript controls content, navigation, metadata, canonicals, or structured data.
9. Validate structured data against visible content and current platform requirements.
10. Use field Core Web Vitals to describe user experience; use laboratory tests to diagnose bottlenecks.

## Interpretation boundaries

- A robots block can coexist with an indexed URL because crawl access and index presence are different observations.
- A canonical is a consolidation signal, not a guaranteed selection or a substitute for redirects and index directives.
- A noindex directive must be observable by a crawler before it can be processed.
- A 200 response can still represent a soft 404.
- JavaScript is a test condition, not proof of an indexation defect.
- Sitemap file organization follows operational needs and protocol limits, not an arbitrary page-count threshold.
- Performance findings cite field or lab provenance; a perfect score is not an acceptance criterion.
- Structured-data eligibility is not evidence of a rich result or AI citation.

## Module output

Return a sampled URL table containing evidence IDs, template, requested/final URL, status/chain, crawl directive, index directive, canonical, sitemap presence, discovery link, source/render result, schema, performance provenance, observation time, and permitted breadth.
