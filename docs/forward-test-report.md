# Forward-test report: seo-site-baseline

Дата: 2026-07-31

## Проверенные сценарии

| Сценарий | Артефакты | Ключевая проверка | Результат |
|---|---|---|---|
| Только публичный URL | `https://example.com` | Не изображать полный crawl/GSC; отделить назначение домена от неизвестных SEO-модулей | Passed после 2 итераций |
| robots + sitemap | тестовый shop fixture | Найти конфликт blocked URL в sitemap; не выдумать HTTP/canonical/indexation | Passed |
| Локальный HTML | `product-a.html` | Найти cross-product canonical и расхождение description/content; ограничиться source HTML | Passed |
| GSC export | 4 query-page rows | Посчитать агрегаты, разделить интенты, не создавать pricing page автоматически | Passed |
| Многостраничная локальная сборка | Velmi build + ops | Отличить шаблонные/единичные проблемы, plans/live evidence и не выдумать P0/GSC | Passed после 1 итерации |

## Найденные и исправленные проблемы skill

1. Severity inflation: отсутствие внутренних ссылок и возможный `/index.html` redirect были ошибочно повышены до P0 без live-доказательства.
2. Missing-evidence inflation: отсутствие optional robots.txt и непроверенные canonical/sitemap/performance сначала попадали в prioritized findings.

Исправления:

- P0 разрешён только при подтверждённом блокирующем или широком production-impact;
- `not_checked` и возможные, но неподтверждённые условия вынесены в missing evidence/next diagnostics без P0–P3;
- отсутствие optional robots.txt прямо исключено из ошибок, если нет подтверждённой потребности в crawl rules.

## Итоговые quality gates

- Evidence boundary: passed.
- Evidence-state labels: passed.
- Crawl/indexation/ranking/traffic/conversion separation: passed.
- Intent/SERP gate: passed.
- GEO causality guardrails: passed.
- No unsafe SEO myths/manipulation: passed.
- Actionable fixes and recheck criteria: passed.
- Official `quick_validate.py`: passed.
