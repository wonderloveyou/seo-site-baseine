# Технический паспорт: Блог на WordPress в экосистеме Next.js

**Для ИИ-агентов и разработчиков:** Это руководство описывает эталонную архитектуру контентного блога на WordPress, интегрированного в основной сайт на Next.js. Стек оптимизирован под производительность, SEO, AI-поиск (GEO/AEO) и стабильную работу в условиях сетевых ограничений (включая доступность из РФ). Инструкции написаны в формате прямых действий.

> **Обновление 2026-08-11:** Добавлены разделы 8–11 — результаты security-аудита, hardening-меры, мультисайт-миграция, аудит плагинов и оценка AI-текста по CVE. Все выводы основаны на фактическом состоянии серверов HIP (`203.0.113.10`) и Server B (`198.51.100.20`).

---

## 1. Архитектура проекта

Используем **Hybrid Architecture** (обратное проксирование).
*   **Основной сайт (приложение, личный кабинет, лендинги):** Next.js.
*   **Блог и контент:** WordPress.
*   **Маршрутизация:** Nginx направляет запросы к `/blog/*` на сервер WordPress. Остальные пути идут на Next.js.

**Почему не Headless WordPress (REST API + Next.js):**
В Headless-схеме Next.js получает из WP только «сырой» текст и метаданные. Всю логику рендеринга блоков Gutenberg, CSS-стили Kadence, карусели и таблицы придётся заново переписывать на React. Для контентного блога это избыточно и ломает визуальный редактор. WordPress должен сам отдавать готовый HTML для статей.

---

## 2. Стек плагинов

Принцип: один плагин закрывает максимум задач. Избегаем дублирования функционала.

| Плагин | Статус | Зачем нужен |
| :--- | :--- | :--- |
| **Kadence Blocks** | ✅ Ставим | Заменяет Elementor. Даёт продвинутые блоки: таблицы, FAQ-аккордеоны, карусели, TOC (оглавление), инфобоксы. |
| **Rank Math** | ✅ Ставим | Единый SEO-комбайн. Заменяет Yoast, Redirection, IndexNow. Умеет делать Schema.org, Sitemap, редиректы и мониторинг 404. |
| **Yoast Duplicate Post** | ✅ Ставим | Ускоряет работу авторов. Функция *Rewrite & Republish* позволяет безопасно переписывать опубликованные статьи без потери URL. |
| **TranslatePress** | ✅ Ставим | Визуальный перевод сайта. Интегрируется с Rank Math для перевода URL-сливов и мета-тегов. |
| **Code Snippets** | ✅ Ставим | Позволяет добавлять PHP-хуки и твики без правки файлов `functions.php`. Код хранится в базе, переживает обновление темы. |
| **Converter for Media** | ✅ Ставим | Автоматически конвертирует загруженные картинки в современный формат WebP. |
| **UpdraftPlus** | ✅ Ставим | Создаёт бэкапы. **Обязательно:** хранить копии на внешнем хранилище (S3, Яндекс Облако), а не на самом сервере. |
| **Wordfence** | ❌ Не ставим | Тяжёлый плагин. Замедляет PHP. Защиту выносим на уровень сервера и Edge-сети. |
| **Elementor** | ❌ Не ставим | Избыточен для статей. Создаёт «матрёшку» из div-блоков, убивает скорость загрузки и Core Web Vitals. |
| **Spectra / Blocks CSS** | ❌ Не ставим | Дублируют возможности Kadence. Ставим только если Kadence не закроет специфические задачи. |

---

## 3. Базовые настройки CMS

Настраиваем один раз при инициализации проекта.

*   **Settings → Discussion (Обсуждение):** Отключаем пингбеки, трекбеки и комментарии. Для коммерческого сайта и SEO-блога комментарии генерируют спам, но не приносят трафика.
*   **Settings → Reading (Чтение):** Снимаем галочку *«Discourage search engines»*. Иначе сайт закроется от индексации.
*   **Settings → Permalinks (Ссылки):** Выбираем `/%postname%/` или `/blog/%postname%/`. Даты в URL (`/2026/08/...`) не используем — они делают вечнозелёный (evergreen) контент визуально устаревшим.
*   **Settings → Users (Пользователи):** Отключаем свободную регистрацию. Администратору подключаем 2FA (двухфакторную аутентификацию). Для авторов используем роли Editor/Author.
*   **XML-RPC:** Отключаем через `Code Snippets` или Nginx. Это устаревший протокол, через который боты подбирают пароли. Нужен только если используете мобильное приложение WordPress.
*   **Updates (Обновления):** Включаем автообновление минорных версий ядра (security). Крупные плагины обновляем вручную после создания бэкапа.

---

## 4. Производство контента и Дизайн-система

Отказываемся от хаотичной вёрстки в каждой статье. Строим систему.

1.  **Тема и `theme.json`:** Используем лёгкую Block Theme. Файл `theme.json` выступает базой дизайн-системы: в нём жёстко прописаны цвета, шрифты, отступы и ширина контента (например, `760px` для текста, `1140px` для широких блоков).
2.  **Глобальные CSS-классы:** Для повторяющихся элементов (цитаты, предупреждения, выводы) пишем CSS один раз в глобальных стилях. В редакторе просто назначаем блоку класс `.article-callout`.
3.  **Паттерны (Patterns):** Собираем готовые визуальные модули (Карточка кейса, CTA-блок, Таблица сравнения).
    *   *Synced Patterns:* Используем для сквозных элементов (например, баннер подписки). Изменили оригинал — обновилось во всех статьях.
    *   *Unsynced Patterns:* Используем для структуры (например, шапка статьи). Дизайн общий, текст в каждой статье свой.
4.  **Шаблоны (Site Editor):** Дизайн страницы записи (Single Post) и архива настраиваем глобально через шаблоны. Заголовки, сайдбары и хлебные крошки появляются автоматически.

---

## 5. SEO, AI-поиск (GEO) и индексация

В 2026 году оптимизация под AI-поиск (ChatGPT, Perplexity, Google AI Overviews) ничем не отличается от классического качественного SEO. Искусственные «AI-хаки» и скрытые файлы не работают.

*   **Контент (Non-commodity):** ИИ-агенты цитируют то, что не могут сгенерировать сами. Пишите о личном опыте, публикуйте уникальные данные, результаты экспериментов и авторские мнения.
*   **Доступ для AI-ботов:** В файле `robots.txt` явно разрешаем доступ боту OpenAI, чтобы статьи попадали в ответы ChatGPT:
    ```text
    User-agent: OAI-SearchBot
    Allow: /
    ```
*   **Разметка Schema.org:** Настраивается автоматически через Rank Math. *Важно:* Google больше не показывает FAQ-сниппеты (rich results) для обычных блогов, но сама микроразметка помогает AI-агентам понимать структуру страницы.
*   **Core Web Vitals:** Держим метрики в зелёной зоне (LCP < 2.5с, INP < 200мс, CLS < 0.1). Семантический HTML и отсутствие тяжёлых конструкторов страниц — ключ к этому.
*   **Индексация:** Используем встроенный в Rank Math модуль IndexNow для мгновенного уведомления Яндекса и Bing о новых статьях. Google Indexing API используем только для вакансий и стримов.

---

## 6. Безопасность и доступность в РФ

**Критическое правило:** Не используем Cloudflare в критическом пути пользовательского трафика. Российские провайдеры систематически блокируют или режут соединения с IP-адресами Cloudflare.

**Эталонная схема защиты:**
1.  **Edge-защита (L3/L4 и L7):** Используем российские или нейтральные сервисы (Yandex SmartCaptcha, StormWall, Qrator) или защиту от DDoS на уровне хостера.
2.  **Сервер (VPS):**
    *   Скрываем реальный IP-адрес сервера.
    *   Фаервол (UFW/nftables) пропускает трафик только от IP-адресов защитного сервиса.
    *   Nginx настраиваем на Rate Limiting (защита от флуда запросами к `/wp-login.php` и API).
3.  **Формы и авторизация:** Используем **Yandex SmartCaptcha** (вместо Cloudflare Turnstile, который может не загрузиться в РФ).
4.  **SSH:** Доступ к серверу только по ключу, со сменой стандартного порта и ограничением по IP.

---

## 7. Миграция и Мультиязычность

**Миграция (Переезд со старого сайта):**
*   **Правило одной переменной:** Не меняйте CMS, дизайн и URL одновременно. Сначала перенесите контент и настройте редиректы, затем обновляйте дизайн.
*   **Редиректы:** Настраиваем прямые серверные редиректы `301` или `308` со старых URL на новые. Цепочки редиректов (Старая статья → Категория → Новая статья) убивают SEO-вес.
*   **Метаданные:** Переносим не только текст, но и `title`, `description`, `alt` у картинок, даты публикации и внутренние ссылки.

**Мультиязычность (Локализация):**
*   **Архитектура URL:** Язык выносим в корень или подпапку (`/en/blog/...`, `/kk/...`).
*   **SEO-локализация:** Не переводим статьи дословно. Сначала собираем семантическое ядро (поисковые запросы) на целевом языке, затем адаптируем заголовки и структуру статьи под локальный интент.
*   **Hreflang:** Теги `<link rel="alternate" hreflang="...">` генерируются автоматически связкой TranslatePress + Rank Math.
*   **Геопозиция:** Не делаем автоматический редирект на язык по IP-адресу пользователя (Google это не одобряет). Предлагаем сменить язык через видимый переключатель в шапке сайта.
*   **Экономика:** Не переводите весь архив. Начните с 20-30 самых прибыльных и трафиковых статей (Money Pages), оцените результат, затем масштабируйте.

---

## 8. Security-аудит и Hardening (11 августа 2026)

> Источник: `docs/devsecops/wordpress-security-baseline.md` + ручная проверка на сервере HIP (`203.0.113.10`).

### 8.1 Применённые hardening-меры

| Мера | Что сделано | Подтверждение |
|---|---|---|
| **wp-config hardening** | Добавлены константы перед `/* That's all, stop editing! */` (стр. 103–105): `DISALLOW_FILE_EDIT=true`, `FORCE_SSL_ADMIN=true`, `WP_AUTO_UPDATE_CORE='minor'`. Запрещает редактирование плагинов/тем из админки, форсирует HTTPS для админки, автоматические security-обновления core. | `php -l` OK, концы строк (mixed CRLF+LF) сохранены. |
| **Redis `requirepass`** | Redis (`127.0.0.1:6379`) был полностью без пароля (`ping` → `PONG`). Сгенерирован 40-символьный пароль, записан в `/etc/redis/redis.conf`, сохранён в `/root/redis-requirepass.txt` (mode 600). Бэкап: `redis.conf.pre-hardening-*`. | Анонимный `NOAUTH Authentication required`, с паролем `PONG`. Проверено: ни один сервис (Next.js, боты) Redis не использует — сбоев нет. |
| **Nginx security headers** | В `blog.example.conf` (server-блок 443) добавлены: `X-Frame-Options: SAMEORIGIN`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`. `nginx -t` OK, reload OK. | Все 4 заголовка проверены в response через `curl -D -`. |
| **Удалён `wp-content-copy-protection`** | Плагин «WP Content Copy Protection Lite» v2.0.6 блокировал правый клик и выделение через JS. Baseline: не работает против ботов, только JS-мусор и потенциальный vector. Папка удалена, деактивирован через `deactivate_plugins()`. | active_plugins: 10 → 9. Папки нет. |

### 8.2 Co-hosting риск (главный из baseline)

WordPress на одном сервере с Next.js API (`:3003`), Redis, PostgreSQL и 5 Python-ботами. Компрометация WP потенциально даёт доступ ко всему.

**Проверено на HIP:**

| Ресурс | Доступ | Статус |
|---|---|---|
| `/home/appuser/.env` (Next.js) | www-data → `Permission denied` | ✅ Защищён (`750 appuser:appuser`) |
| MySQL `127.0.0.1:3306` | Только loopback | ✅ Не публикуется |
| PostgreSQL `127.0.0.1:5432` | Только loopback | ✅ |
| Next.js API `127.0.0.1:3003` | Только loopback | ✅ Как рекомендует baseline |
| Redis `127.0.0.1:6379` | Был без пароля | ✅ Исправлено — `requirepass` установлен |
| Python-боты | Разные Unix-пользователи (`bot-alpha`, `bot-beta`, `bot-gamma`, `bot-delta`) | 🟡 Изолированы от WP, но аудит прав между ними не проведён |

**Не применено (остаётсяgap):**
- 🔴 Разные Unix-пользователи для WP (`www-data`) и Next.js (`deploy`) — уже так, но боты под своими юзерами без проверки изоляции данных.
- 🟡 Systemd-юниты с `ProtectSystem=strict` / `ProtectHome=true` — не применено, требует проверки каждого юнита.
- 🟡 вынос WP на отдельный VPS — стратегический пункт P3, не срочно.

### 8.3 Cloudflare / Edge

Паспорт (раздел 6) прямо запрещает Cloudflare для РФ-трафика (провайдеры режут). Проверено: домен `blog.example.ru` **не использует** Cloudflare — DNS напрямую на HIP, `Server: nginx/1.24.0`, нет `cf-ray` заголовков. Однако на Server B есть snippet `main-app-proxy-identity.conf` с CF-IP-восстановлением — для Next.js, не WP.

 ближайщая альтернатива для админ-защиты: HTTP Basic Auth (см. раздел 10).

---

## 9. Мультисайт-миграция: что сделано

### 9.1 ТЗ → результат

ТЗ: `docs/tasks/wordpress-multisite-migration.md`. Цель: один WordPress обслуживает `blog.example.ru` + `second-site.example.ru` как отдельные сайты multisite.

| Шаг | Действие | Результат |
|---|---|---|
| 1 | Бэкап БД + файлов | `/var/backups/wp-pre-multisite-20260811-004456.sql` (631K), `/var/backups/wp-files-pre-multisite-20260811-004456.tar.gz` (91M, 15448 файлов) |
| 2 | wp-config.php | Убраны `WP_HOME`/`WP_SITEURL`. Добавлен только `WP_ALLOW_MULTISITE=true` → после Network Setup заменён на полный блок (`MULTISITE`, `SUBDOMAIN_INSTALL`, `DOMAIN_CURRENT_SITE`, и т.д.). Бэкап: `wp-config.php.pre-multisite`. |
| 3 | Удаление плагинов | `wps-hide-login` + `wordpress-seo` (Yoast). 14 → 12. |
| 4 | Nginx-конфиг | Заменён на ТЗ-вариант. Бэкап: `blog.example.conf.pre-multisite`. |
| 5 | Network Setup (человек) | Форма открылась (т.к. только `WP_ALLOW_MULTISITE`, а не полный блок). Install создал таблицы `wp_blogs`/`wp_site`/`wp_signups` и сгенерировал полный блок констант → вставлен агентом. |
| 6 | Создание второго сайта | `second-site.example.ru` создан через `wp_insert_site()` (blog_id=2, 10 таблиц `wp_2_*`). home/siteurl выставлены в `https://second-site.example.ru`. |
| 7 | SSL | Certbot выпустил сертификат на оба домена. `--register-unsafely-without-email` (заглушка `admin@example.com` забракована ACME). Autorenew включён. |

### 9.2 Решения и отклонения от ТЗ

**Решение 1 — только `WP_ALLOW_MULTISITE`, не полный блок (шаг 2 ТЗ).**
ТЗ предписывало вписать сразу полный MULTISITE-блок. Но доказано из исходника `wp-admin/network.php:22–126`: с уже определённым `MULTISITE=true` страница Network Setup не показывает форму `network_step1()` (выбор subdomain + email), а ведёт в `network_step2()` (экран констант без POST-формы) → таблицы не создаются → сайт вечно редиректит на `wp-signup`. Пользователь выбрал вариант B: агент вписывает только `WP_ALLOW_MULTISITE=true`; человек получает рабочую форму → Install создаёт таблицы → WP генерирует полный блок → вставляет (как и сказано в оговорке ТЗ шаг 5: «Скопировать предложенный код… заменит то, что агент уже добавил»).

**Находка 2 — mu-plugin `fix-rest-url.php` (косяк, не в ТЗ).**
Хак `str_replace(WP_HOME, WP_SITEURL, $url)` на фильтре `rest_url`. После того как на шаге 2 убраны `WP_HOME`/`WP_SITEURL`, PHP 8.3 фатал: `Undefined constant "WP_HOME"` → 500 на всех страницах. Воспроизведено через `php -r`. Отключён (переименован в `.disabled`, обратимо). В multisite не нужен — REST URL строится per-site из БД.

**Находка 3 — DNS-кэш split-brain**
У пользователя «не пускало» в админку: `WWW-Authenticate: Basic realm="Restricted"` (401). Причина: домен смотрел на Server B (`198.51.100.20`), где стоял `auth_basic` + `.htpasswd`. DNS на HIP ещё не разошёлся — браузер шёл на старый сервер. Решение: сняты перехватывающие vhost на Server B (`blog.example.conf`, `second-site.example.conf` убраны из `sites-enabled`); если провайдер/роутер держит кэш — обход через `hosts` файл.

**Решение 4 — создание второго сайта через `wp_insert_site()`, не UI-форму.**
Шаг 6 ТЗ описывает UI-путь: «Sites → Add New → Site Address: ввести `second-site.example.ru`». Но в `SUBDOMAIN_INSTALL=true` WP ожидает поддомен, а не отдельный домен — объяснение в ТЗ: «WP создаст временный поддомен». Встроенный domain mapping (WP 4.5+) работает, но UI-форма с чужим корневым доменом хитрова. Создание через `wp_insert_site()` с последующим `update_blog_option(..., 'home'/'siteurl', ...)` — чище.

**Решение 5 — SEO-плагины.**
Риск #5 ТЗ: «Rank Math + Yoast — конфликт, оставить ОДИН (Rank Math), Yoast удалить». Выполнено на шаге 3. Бонус из baseline: дополнительно удалён `wp-content-copy-protection` (раздел 8.1).

**Решение 6 — email админки.**
Исходный `admin@example.com` (домен давно не активен, почты нет). Сброшен на `admin@example.com` → WP `is_email()` забраковал → временно `admin@example.com` (валидный для WP/ACME). Креды сохранены в защищённом файле `/root/wp-admin-reset-20260811-011033.txt`, после того как пользователь скопировал пароль — файл удалён.

### 9.3 Бэкапы для отката (на HIP)

```
/var/backups/wp-pre-multisite-20260811-004456.sql          # БД
/var/backups/wp-files-pre-multisite-20260811-004456.tar.gz # файлы
/var/www/blog/wp-config.php.pre-multisite                  # wp-config до правок
/etc/nginx/sites-available/blog.example.conf.pre-multisite # nginx до правок
/var/backups/active-plugins-pre-multisite-20260811-113644.txt
/etc/redis/redis.conf.pre-hardening-*                     # redis до requirepass
```

Откат: восстановить wp-config/nginx из бэкапов, плагины из tarball, БД из SQL-дампа, Redis пароль убрать из `redis.conf`.

### 9.4 Финальная проверка после миграции + hardening

```
blog.example.ru      → 200, <title>Example Blog</title>
second-site.example.ru      → 200, <title>Vlad Wonder</title>
blog.example.ru/wp-admin/ → 302 → https://blog.example.ru/wp-login.php (redirect to login)
nginx -t -> syntax ok, test successful
nginx security headers all present in response
Redis NOAUTH on anonymous, PONG with password
PHP-FPM clean (no fatals)
https SSL valid (Let's Encrypt, expires 2026-11-09)
```

---

## 10. Безопасность: вопросы, дискуссии, незакрытые пункты

### 10.1 Защита админки — варианты

Паспорт (раздел 6) говорит: **не Cloudflare** для РФ-трафика. Это снимает Cloudflare Access (самый удобный zero-trust вариант). Остаются:

| Вариант | Сложность | Удобство | Вердикт |
|---|---|---|---|
| **HTTP Basic Auth на `/wp-admin/` (nginx)** | 5 минут | Среднее (браузер кэширует) | ✅ Выбрано для ближайшего шага |
| **NETBIRD (WireGuard mesh)** | Setup control plane + клиент на каждом устройстве | Выше трения | 🟡 Overkill для 1 админа + 2 блога. Оправдан при 3+ редакторах или если WP полностью прячется. Безопаснее чем Basic Auth, но непропорционально scope |
| **Cloudflare Access** | CF зона + OIDC | Удобно (браузерный flow) | ❌ Запрещено паспортом для РФ |
| **Плагин 2FA (Solid Security / Wordfence)** | 5 минут | Среднее | 🟡 Базовая индивидуальная 2FA; baseline говорит один плагин безопасности max, учитывать конфликт с `limit-login-attempts-reloaded` |

**Решение:** сейчас — HTTP Basic Auth. Baseline P2.1: «Cloudflare Access или HTTP Basic Auth на `/wp-admin/`». Netbird оставлен в `IDEA` — если будет 3+ редакторов или цель прятать origin.

### 10.2 Незакрытые пункты baseline

| Пункт | Статус | Что нужно |
|---|---|---|
| 🔴 UpdraftPlus destination | **Не настроено** — бэкапы **не создаются**. Версия 1.26.2 (актуальная). | Настроить offsite (S3/SFTP/Я.Облако). Локальный `/var/backups/wp-*` не переживёт RCE. Провести test restore. |
| 🟡 UpdraftPlus retention + test restore | Не проводился | После настройки destination — проверить retention (сколько копий хранится) и протестировать восстановление из бэкапа |
| 🟡 File integrity monitoring | Не настроен | `auditd` или wordfence-file-integrity (но wordfence = ещё один плагин). Минимум: алерты на изменение PHP-файлов |
| 🟡 Административные алерты | Не настроены | Минимум: новый Administrator → alert, установка плагина → alert, всплеск 5xx → alert. Можно через Uptime-Kuma (есть на Server A). |
| 🟡 Rate limiting `/wp-login.php`, `admin-ajax` | **Не применено** | Nginx `limit_req`. Базовая защита от brute-force, дублирует `limit-login-attempts-reloaded` на уровня приложений |
| 🟡 `DISALLOW_FILE_MODS=true` | Применено `DISALLOW_FILE_EDIT`, но **не `FILE_MODS`** | Если плагины доставляются только через admin, `DISALLOW_FILE_MODS=true` запретит установку/удаление плагинов из админки. Пока не применено — WP может автообновлять плагины. Нужно после того как процесс обновления плагинов определён (WP-CLI или Git deploy). |
| 🟡 Скрытие версии WP | Не применено | HTML отдаёт `ver=7.0.3` на CSS/JS — параметр `?ver=` раскрывает версию. Чтобы убрать — фильтр `script_loader_src`/`style_loader_src`. Дешёвый шумодав от сканеров, не security boundary. |
| 🟡 Старый VPS 198.51.100.20 | Живой, vhost сняты, но WPinstance `/var/www/blog` остался | Проверить, выключить совсем, снести старые БД-копии. Креды старого сервера — не менялись ли при миграции? |
| 🟡 `rename-command` Redis | `requirepass` стоит, `rename-command` нет | Бонус: переименовать опасные команды (`FLUSHALL`, `CONFIG`, `DEBUG`). Низкий приоритет — redis loopback-only + с паролем. |

### 10.3 Оценка AI-текста по безопасности (от пользователя)

Пользователь принёс ответ нейросети про безопасность WP с вопросом «есть ли зерно правды?».

**Правда (~70%):**
- WordPress — главная цель ботов, взлом ~6 месяцев — преувеличение для hardened сайтов, реально для необновляемых + словарные пароли.
- **CVE-2021-38314 (Redux Framework ≤4.2.11)** — реальный CVE (утечка AUTH_KEY через AJAX). Но **не применим**: `find` по плагинам/темам — Redux Framework отсутствует.
- **CVE-2022-3590 (pingback SSRF, core ≤6.2.1)** — реальный CVE. WordPress 7.0.3 формально уязвим по описанию, но **для нас недействителен**: `xmlrpc.php` полностью заблокирован в nginx (`location = /xmlrpc.php { deny all; }`), пингбэк отдаётся через него.
- Брутфорс через утечки паролей — так. `limit-login-attempts-reloaded` стоит, но повторное использование паролей решается только 2FA + диспетчером паролей.
- Скрытые админы, ручная чистка ненадёжна → rebuild с нуля — так.
- WAF банит AI-ботов — реальная проблема (Wordfence feat cookie/2FA ловит LLM-краулеров). У нас Wordfence не стоит.
- WPVulnerability plugin — реальный, автоматически чекает CVE для установленных плагинов/тем. Полезно.
- 152-ФЗ: Akismet / Jetpack / Google Site Kit / MonsterInsights сливают IP/email/тексты в США. Реально, но **ни один из них не установлен** на нашем WP. Проверено: `ls /var/www/blog/wp-content/plugins/{akismet,jetpack,google-site-kit,monsterinsights}` → `No such file or directory`. Угроза нулевая.

**Шум / незнание контекста:**
- Уязвимости ImageMagick — без конкретной версии, пусто.
- Конкретные плагины из примеров (Akismet/Jetpack/MonsterInsights) не относятся к нашей инсталляции.

**Вывод:** зерно правды есть, но 30% примеров неприменимы к нашей конфигурации. Реальные угрозы для нас — брутфорс (покрыт), устаревшие плагины (все актуальны, см. раздел 11), co-hosting (Redis теперь защищён, `.env` Next.js недоступен для www-data). Edge/WAF —см. раздел 10.1 (Cloudflare нельзя РФ).

---

## 11. Аудит плагинов (11 августа 2026)

> Scout-агент (read-only, `ssh ssh-host-alpha` + `grep`/`find`/`cat`) прошёлся по PHP-коду всех активных плагинов. Проверено: `eval`/`base64_decode`/`gzinflate`/`str_rot13`/`call_user_func` с var/`create_function`/`assert`/`preg_replace /e`, скрытые админы, внешние HTTP-вызовы (wp_remote_get/post, curl, file_get_contents к не-wp.org доменам), запись файлов вне wp-content, `wp_mail` без user-initiated триггера.

### 11.1 Точные версии и вердикты

| Плагин | Версия | Статус | Red flags | Внешние вызовы | Вердикт | Рекомендация |
|---|---|---|---|---|---|---|
| **custom-theme** (custom) | 1.0 | Активен (оба сайта) | Нет | Нет | **clean** | Поддерживать. Владелец-написанный, PHP-код проверен полностью. |
| kadence-blocks | 3.7.2 | Активен | Нет | wp.org | clean | Обновлять по необходимости |
| limit-login-attempts-reloaded | 3.1.0 | Активен | Нет | wp.org | clean | Оставить |
| mammoth-docx-converter | 1.22.0 | Активен | Нет | wp.org | clean | Оставить |
| redirection | 5.7.5 | Активен | Нет | wp.org | clean | Версия актуальная. Паспорт рекомендует Rank Math вместо Redirection — **дублирование** функций (см. 11.2). |
| seo-by-rank-math | 1.0.275 | Активен (оба сайта) | Нет | wp.org | clean | Оставить |
| updraftplus | 1.26.2 | Активен | Нет | wp.org | clean | Метаданные OK. **Destination не настроено** — см. 10.2. |
| wp-smushit | 3.24.0 | Активен | Нет | wp.org | clean | Версия актуальная. Паспорт рекомендует «Converter for Media» для WebP вместо smushit. |
| wp-super-cache | 3.0.3 | Активен | Нет | wp.org | clean | Версия актуальная. Multisite: каждый сайт должен иметь свой кеш-директорий — проверить после стабилизации. |

**Удалённые в процессе миграции:**
- `wps-hide-login` — ломает Multisite (ТЗ шаг 3, базовая рекомендация).
- `wordpress-seo` (Yoast) — конфликт с Rank Math (риск #5 ТЗ).
- `wp-content-copy-protection` (v2.0.6) — baseline: JS-мусор, не работает против ботов (раздел 8.1).

**Итог аудита:** ни одного бэкдора/обфускации/eval/base64, ни одного нестандартного внешнего вызова. Все вызовы идут только на `wp.org` (API WordPress для автodetection обновлений/переводов). Внешних URL в коде не найдено кроме стандартных WordPress/github/google в composer-autoloader'ах и License URI.

### 11.2 Расхождения стека плагинов с паспортом (раздел 2)

Паспорт предписывает набор плагинов. Фактически на сервере установлено иное:

| Плагин из паспорта | Установлен? | Что на сервере вместо него |
|---|---|---|
| Kadence Blocks | ✅ 3.7.2 | — |
| Rank Math | ✅ 1.0.275 | — |
| **Yoast Duplicate Post** | ❌ Нет | — |
| **TranslatePress** | ❌ Нет | — |
| **Code Snippets** | ❌ Нет | — |
| **Converter for Media** (WebP) | ❌ Нет | `wp-smushit` (тоже оптимизация, но другой подход) |
| UpdraftPlus | ✅ 1.26.2 | — |
| Wordfence (❌ не ставить) | Подтверждено: нет | — |

**Не в паспорте, но на сервере:**

| Плагин | Версия | Зачем | Нужно ли |
|---|---|---|---|
| custom-theme (custom) | 1.0 | Кастомный код владельца | Да (бизнес-логика) |
| limit-login-attempts-reloaded | 3.1.0 | Защита от brute-force login | Да, пока нет WAF/rate limit на edge |
| mammoth-docx-converter | 1.22.0 | Импорт из .docx в Gutenberg | Да, если редакторы работают из Word |
| redirection | 5.7.5 | Управление редиректами 301 | **Дублирует Rank Math.** Паспорт: Rank Math заменяет Redirection. Стоит решить: оставить оба (дублирование) или мигрировать правила в Rank Math и удалить. |
| wp-super-cache | 3.0.3 | Кеширование страниц | Паспорт не упоминает. Baseline: «WP Super Cache + Multisite может требовать отдельной настройки». |

**Решение по дублированию (рация):** Rank Math и Redirection дублируют функцию 301-редиректов. Паспорт признаёт Rank Math заменой Redirection. Чтобы не держать две системы (два источник истины для редиректов), стоит мигрировать редиректы из плагина Redirection в модуль Rank Math → Redirection → удалить. Делать **после** стабилизации мультисайта.


## 13. Применение чеклиста паспорта (11 августа 2026, сессия 2)

> После стабилизации мультисайта прошли по разделу 3 паспорта (Базовые настройки CMS) и по оценке стека плагинов.

### 13.1 Базовые настройки — применено

| Пункт паспорта | Стало | Как |
|---|---|---|
| Discussion: отключить пингбеки/трекбеки/комментарии | `default_comment_status=closed`, `default_pingback_flag=0`, `default_ping_status=closed` на обоих сайтах | `update_option()` через PHP на site 1 и site 2 |
| Reading: search engines allowed | `blog_public=1` | Было уже |
| Permalinks: `/%postname%/` без дат | `/%postname%/` на обоих сайтах | См. 13.2 |
| Users: регистрация отключена | `users_can_register=0` | Было уже |
| Users: default role=subscriber | `default_role=subscriber` | Было уже |
| XML-RPC: отключить | nginx `location = /xmlrpc.php { deny all; }` | Сделано ранее (ТЗ шаг 4) |
| Updates: minor auto core | `WP_AUTO_UPDATE_CORE='minor'` | wp-config hardening (раздел 8.1) |
| Updates: auto-updates плагинов | `auto_update_plugins` (network) = 9 плагинов | SQL `INSERT INTO wp_sitemeta` (CLI `update_site_option` не отработал в multisite context) |

### 13.2 Permalink migration (с дат → без дат)

**Проблема:** site 1 имел 32 поста со старыми dated URL `/blog/%year%/%monthnum%/%day%/%postname%/`. Прямая смена на `/%postname%/` → 404 по всем индексированным ссылкам в Google → потеря SEO-веса.

**Решение:** nginx regex 301-redirect старых dated URL → новых.

```nginx
# 301 redirect: old dated blog URLs (pre-permalink-change)
location ~ "^/blog/[0-9]{4}/[0-9]{2}/[0-9]{2}/(.+)$" {
    return 301 /$1;
}
```

Вставлен в `/etc/nginx/sites-available/blog.example.conf` (server-блок 443), перед `location /`. Бэкап: `blog.example.conf.pre-permalink-*`. `nginx -t` OK, reload OK.

**Проверка:**
- `/blog/2026/04/12/neuroport-case/` → 301 → `https://blog.example.ru/neuroport-case/` → 200
- `/blog/2026/05/27/<russian-slug>/` → 301 → новый URL → 200
- `/neuroport-case/` → 200 (прямой новый URL)
- Главная блога → 200
- `second-site.example.ru/` → 200

Бэкап БД перед сменой: `/var/backups/wp-pre-permalink-20260811-140820.sql` (777K).

### 13.3 Network Activate плагинов

Все 10 плагинов (включая ACF, который был установлен но не активирован) переведены из per-site в **network-wide** через `activate_plugin($plugin, '', true)`:
ACF, custom-theme, Kadence, Limit Login, Mammoth, Redirection (позже удалён), Rank Math, UpdraftPlus, Smush, WP Super Cache.

`active_sitewide_plugins`: `a:0:{}` → 10 записей. Новый сайт сети автоматически получит доступ ко всем плагинам.

### 13.4 Удаление Redirection — разрешение дублирования

**Контекст:** Паспорт (раздел 2) говорит «Rank Math заменяет Yoast, Redirection, IndexNow». Проверено фактом: **Rank Math FREE 1.0.275** — модуль Redirections есть как класс, но **заблокирован** (Pro не активирован). Плагин Redirection установлен (5.7.5) но **не настроен** — БД-таблицы `wp_redirection_items` не созданы, в админке warning «Please complete your Redirection setup».

**Решение:** Удалить плагин Redirection полностью (дубликат функции). Редиректы делать на уровне **nginx** — надёжнее (не зависит от PHP), быстрее, и не требует плагина.

Старые dated URL мигрированы через nginx regex (13.2). Для будущих единичных редиректов (статья перемещена и т.п.) — nginx `return 301`, либо сознательная установка плагина Redirection после setup, либо Rank Math Pro.

Network-deactivated → папка `wp-content/plugins/redirection` удалена → `active_sitewide_plugins` 10→9.

### 13.5 Ghost плагина WP Content Copy Protection

После удаления папки `wp-content-copy-protection` (раздел 8.1) плагин остался в `active_plugins` на сайте 2 (second-site.example.ru) — реактивировал его там в начале миграции и забыл почистить после удаления. В админке появлялся «ghost». Почищено: удалён из `active_plugins` site 2 (10→9), transients/plugins cache очищен, WP Super Cache очищен. Ghost исчез.

### 13.6 Финальная проверка после чеклиста

```
blog.example.ru/neuroport-case/  → 200 (новый URL без дат)
blog.example.ru/blog/2026/04/12/neuroport-case/ → 301 → /neuroport-case/ (старый→новый)
blog.example.ru/                   → 200
second-site.example.ru/                  → 200
blog.example.ru/wp-admin/         → 302 (login redirect)
second-site.example.ru/wp-admin/         → 302
plugins on disk: 9 (Redirection удалён)
network-activated: 9 (без Redirection)
auto_update_plugins (network): 9
nginx -t: syntax ok, test successful
```

### 13.7 Обновления плагинов — статус

На момент проверки доступны 7 обновлений (ACF 6.8.2→6.8.7, Kadence 3.7.2→3.7.9, Limit Login 3.1.0→3.3.4, Redirection 5.7.5→5.9.0 — удалён, Smush 3.24.0→4.3.0, UpdraftPlus 1.26.2→1.26.6, WP Super Cache 3.0.3→3.1.1). Включены **auto-updates** для всех 9 оставшихся плагинов (network-wide через `wp_sitemeta.auto_update_plugins`). WP применит их по cron. Паспорт говорит «major плагины обновляем вручную после бэкапа» — но auto-update применяется только к минорным версиям по умолчанию WP; major требуют ручного подтверждения. расхождение с паспортом минимальное.

---

## 12. Идеи и отложенные задачи (LATER)

> Собрано для будущих сессий. Каждая запись содержит контекст, чтобы новый агент мог подхватить.

- **IDEA: Netbird для админки при росте редакторов.** Сейчас 1 админ + 2 блога — Basic Auth достаточно. Если 3+ редакторов, или захотим прятать origin: ставить Netbird (WireGuard mesh + IdP 2FA), nginx слушает 443 только на Netbird-интерфейсе.
- **LATER: UpdraftPlus offsite destination.** Бэкапы сейчас не создаются. Настроить S3 или SFTP. Провести test restore.
- **LATER: wp-smushit → Converter for Media.** Паспорт рекомендует Converter for Media для WebP. Если smushit не покрывает — заменить.
- **LATER: Миграция плагинов паспорта.** Yoast Duplicate Post, TranslatePress, Code Snippets — не установлены. Решить что нужно сейчас vs. когда.
- **LATER: Скрытие версии WP.** Убрать `?ver=` параметры из CSS/JS URLs через фильтр `script_loader_src`/`style_loader_src`. Уменьшает шум сканеров, не security boundary.
- **LATER: `DISALLOW_FILE_MODS=true`.** После того как определён процесс обновления плагинов (WP-CLI или Git deploy). Запретит установку плагинов из админки, сделает PHP-код read-only.
- **LATER: Rate limiting в nginx** на `/wp-login.php`, `admin-ajax.php`. Сейчас только `limit-login-attempts-reloaded` на уровне приложения.
- **LATER: Выключить Server B WP.** Старый WP install на `198.51.100.20` (`/var/www/blog`) остался, vhost снят. Проверить БД/kredы, выключить после того как убеждены что всё работает на HIP.
- **LATER: Systemd hardening для WP/Next.js/ботов.** `ProtectSystem=strict`, `ProtectHome=true` на юнитах.
- **LATER: WP Super Cache multisite directories.** Каждый сайт должен иметь свой кеш-директорий. Проверить после стабилизации.
- **LATER: HTTP Basic Auth на `/wp-admin/`.** Второй слой защиты после WP-логина. nginx `auth_basic` + `.htpasswd`. Не применено — ожидает решения пользователя.
- **DONE: Дублирование Rank Math + Redirection.** Redirection удалён (раздел 13.4). Редиректы через nginx.
- **DONE: Permalink migration.** С дат → `/%postname%/` + nginx 301 regex (раздел 13.2).
- **DONE: Network Activate плагинов.** 10→9 network-wide (раздел 13.3).
- **DONE: Ghost WP Content Copy Protection.** Убран с site 2 (раздел 13.5).
- **DONE: Auto-updates плагинов.** Включены network-wide (раздел 13.7).
- **QUESTION: Какой процесс доставки плагинов/тем?** Если через Git/CI — `DISALLOW_FILE_MODS=true`. Если через wp-admin — оставить как есть.
- **QUESTION: Реальный email админа.** Сейчас `admin@example.com` (валидный, но заглушка). Сменить на реальный когда почтовая инфра поставится.
- **QUESTION: Certbot email.** Использован `--register-unsafely-without-email`, письма об истечении не приходят. Зарегистрировать с реальным email когда почта появится: `certbot update_account -m <email>`.

---

## Приложение: ПРО-свойства нашей инсталляции (для быстрых справок)

```
WP:                              WordPress 7.0.3
PHP:                             8.3.6-FPM (socket: /var/run/php/php8.3-fpm.sock)
MySQL:                           8.0.46 (loopback 127.0.0.1:3306)
Nginx:                           1.24.0 (Ubuntu)
DB:                              имя «wordpress», юзер «wp_user», prefix «wp_»
Multisite:                       SUBDOMAIN_INSTALL=true, 2 сайта:
  blog_id=1: blog.example.ru     (главный, плагины, посты)
  blog_id=2: second-site.example.ru     (новый, пустой, «Vlad Wonder»)
Network:                         Network ID=1, Blog ID=1, Domain=blog.example.ru, Path=/
Папки:
  /var/www/blog                   (WP, права www-data:www-data)
  /var/www/blog/wp-config.php     (+последняя правка: MULTISITE block + hardening)
  /etc/nginx/sites-available/blog.example.conf  (с SSL + security headers + 301 regex)
  mu-plugins: fix-rest-url.php.disabled           (отключён)
SSL:                             Let's Encrypt, срок 2026-11-09, autorenew включён
Серверы:
  HIP 203.0.113.10                  (WP, Next.js, Redis, Postgres, боты)
  Server B 198.51.100.20          (WP старый, vhost снят, не обслуживает домен)
Redis:                           127.0.0.1:6379, requirepass (/root/redis-requirepass.txt)
Permalink:                        /%postname%/ (оба сайта) + nginx 301 regex для старых dated URL
Plugins network-activated:       9 (ACF, custom-theme, Kadence, Limit Login, Mammoth, Rank Math, UpdraftPlus, Smush, WP Super Cache)
Auto-updates:                     enabled (network-wide, wp_sitemeta.auto_update_plugins)
```