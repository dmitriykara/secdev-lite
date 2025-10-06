# S03 - Реестр NFR (Given-When-Then)

Этот файл - рабочий **реестр NFR** на семинаре.
Заполняйте его у себя в репозитории в: `SEMINARS/S03/S03_register.md`.

---

## Поля реестра (data dictionary)

* **ID** - короткий идентификатор, например `NFR-001`.
* **User Story / Feature** - к какой истории/фиче относится требование (ссылка допустима).
* **Category** - выберите из банка (напр.: `Performance`, `Security-AuthZ/RBAC`, `RateLimiting`, `Privacy/PII`, `Observability/Logging`, …).
* **Requirement (NFR)** - *измеримое* требование (числа/пороги/границы действия).
* **Rationale / Risk** - зачем это нужно, какой риск/ценность покрываем.
* **Acceptance (G-W-T)** - проверяемая формулировка: *Given … When … Then …*.
* **Evidence (test/log/scan/policy)** - чем подтвердим выполнение (тест, лог-шаблон, сканер, политика).
* **Trace (issue/link)** - ссылка на задачу, обсуждение, артефакт.
* **Owner** - ответственный.
* **Status** - `Draft` | `Proposed` | `Approved` | `Implemented` | `Verified`.
* **Priority** - `P1 - High` | `P2 - Medium` | `P3 - Low`.
* **Severity** - `S1 - Critical` | `S2 - Major` | `S3 - Minor`.
* **Tags** - произвольные метки (через запятую).

---

## Таблица реестра

| ID      | User Story / Feature                     | Category                          | Requirement (NFR)                                                                 | Rationale / Risk                                                                 | Acceptance (G-W-T)                                                                                                                      | Evidence (test/log/scan/policy)                     | Trace (issue/link) | Owner  | Status   | Priority    | Severity   | Tags                     |
|---------|------------------------------------------|-----------------------------------|-----------------------------------------------------------------------------------|----------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------|--------------------|--------|----------|-------------|------------|--------------------------|
| NFR-001 | US-018: Интеграция с внешним API         | Timeouts/Retry/CircuitBreaker     | Timeout ≤ 2s; retry ≤ 3 с джиттером; circuit-breaker при ≥50% ошибок за 1 мин     | Защита от зависаний при сбоях внешнего API                                       | **Given** внешний API недоступен<br>**When** сервис вызывает внешний API при обращении к `/api/integrations/vendor-x/*`<br>**Then** ожидание ≤6s, 3 retry, затем circuit breaker | test: `integration-failure`; log: breaker state    | #125               | backend | Draft | P1 - High   | S2 - Major | resilience,integration   |
| NFR-002 | US-018: Интеграция с внешним API         | Performance                      | P95 латентность ≤500ms при 20 RPS к `/api/integrations/vendor-x/*`                | UX для клиентов, зависящих от данных                                            | **Given** здоровый внешний API<br>**When** 20 RPS на интеграцию 5 мин<br>**Then** P95 ≤500ms, ошибок ≤1%                              | test: `load-20rps`; metrics: latency quantiles     | #126               | backend | Draft    | P2 - Medium | S2 - Major | perf,slo                 |
| NFR-003 | US-018: Интеграция с внешним API         | Observability/Logging            | Все вызовы логируются с `correlation_id` и статусом (успех/таймаут/ошибка)        | Трассировка проблем в цепочке вызовов                                           | **Given** запрос с `X-Correlation-ID=123`<br>**When** интеграция завершена<br>**Then** в логах есть `correlation_id=123` и статус      | log: JSON-структура с полями                       | #127               | backend+infra | Draft | P2 - Medium | S3 - Minor | logging,observability    |
| NFR-004 | US-018: Интеграция с внешним API         | Privacy/PII                      | Ответы внешнего API с PII не логируются в сыром виде                              | Комплаенс (GDPR, CCPA)                                                          | **Given** ответ с PII (email, phone)<br>**When** логируется<br>**Then** PII маскируется (***@domain.com)                               | audit: sample log entry                            | #128               | backend+devsecops | Draft    | P3 - Low    | S2 - Major | privacy,compliance       |
| NFR-005 | US-018: Интеграция с внешним API         | RateLimiting                 | Лимит: 100 вызовов/мин на ключ → 429 + Retry-After при превышении             | Защита от превышения квот внешнего API                                          | **Given** ключ с 100+ вызовами за 60 сек<br>**When** новый запрос к `/api/integrations/vendor-x/*`<br>**Then** 429 + заголовок `Retry-After: 60` | test: `rate-limit-check`; log: 429 entries         | #129               | backend | Draft | P2 - Medium | S2 - Major | quotas,throttling        |
| NFR-006 | US-018: Интеграция с внешним API         | Caching                      | Кэш данных API: TTL=5 мин, инвалидация при изменении                              | Снижение нагрузки на внешний API и улучшение отзывчивости                       | **Given** кэшированный ответ<br>**When** данные не обновлялись в течение установленного времени жизни кэша, например, 5 мин<br>**Then** следующий запрос обновляет кэш; клиент получает актуальные данные | test: `cache-ttl`; metrics: cache hit/miss         | #130               | backend+infra | Draft    | P3 - Low    | S3 - Minor | caching,performance      |


## Памятка по заполнению

* **Измеримость.** В `Requirement` фиксируйте числа и границы (мс, RPS, минуты, MiB, коды 4xx/5xx, CVSS).
* **Проверяемость.** В `Acceptance (G-W-T)` используйте объективные условия и наблюдаемые факты (код ответа, квантиль, наличие заголовка, запись в лог).
* **Связность.** Сверяйте, чтобы NFR не конфликтовали (timeouts vs retry, rate limits vs SLO, privacy vs logging).
* **План проверки.** В `Evidence` укажите, чем это будет подтверждаться позже (test/log/scan/policy). В рамках семинара **реальные артефакты не требуются**.
* **Трассировка.** В `Trace` добавляйте ссылки на Issues/документы, чтобы потом не искать контекст.

---

## После семинара

* Перенесите/доработайте **8-10 утверждённых NFR** (по сути, те же строки) в раздел **NFR** вашего `GRADING/TM.md`.
* На S04-S05 свяжете эти NFR с угрозами (STRIDE) и ADR - по ID.