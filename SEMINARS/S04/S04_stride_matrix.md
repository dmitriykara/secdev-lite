# S04 - STRIDE per element матрица

| Element                | Data/Boundary          | Threat | Description                                                                 | NFR link (ID)          | Mitigation idea (ADR later)                     |
|------------------------|------------------------|--------|-----------------------------------------------------------------------------|------------------------|------------------------------------------------|
| **Node: API Gateway**  | AuthN/Z                | S      | Подмена JWT токена или использование скомпрометированных учетных данных    | NFR-005                | JWT с коротким TTL + Refresh tokens            |
|                        | Rate limiting          | D      | Исчерпание лимита запросов (DoS через API)                                 | NFR-005                | Динамическое масштабирование лимитов           |
| **Edge: Client → API** | JWT/HTTPS              | T      | Модификация запросов MITM                                                  | NFR-004                | HSTS + сертификаты с коротким сроком действия  |
| **Node: Integration Service** | Circuit Breaker    | D      | Неправильная настройка CB приводит к ложным блокировкам                   | NFR-001                | Автоматическая калибровка CB параметров        |
|                        | Business logic         | E      | Обход RBAC через невалидированные DTO                                      | need NFR (AuthZ)       | Валидация прав на уровне DTO                   |
| **Edge: Service → DB** | SQL/ORM                | I      | Утечка PII через SQL-инъекции                                              | NFR-002, NFR-004       | Параметризованные запросы + шифрование PII     |
|                        |                        | T      | Модификация данных в БД                                                    | NFR-002                | Hash-суммы критичных данных                    |
| **Edge: Service → External API** | HTTP/gRPC       | D      | Долгие ответы внешнего API блокируют сервис                                | NFR-001                | Асинхронные вызовы с буферизацией              |
|                        | PII data               | I      | Утечка PII при передаче внешним системам                                   | NFR-004                | Маскирование + DLP-сканирование                |
| **Node: Redis Cache**  | Cached data            | T      | Подмена кэшированных данных                                                | NFR-006                | Подпись кэшированных значений                  |
| **Edge: Service → Queue** | Audit events       | R      | Потеря аудит-логов при сбое очереди                                        | need NFR (Auditability)| Дублирование в холодное хранилище              |
| **Node: PostgreSQL**   | PII storage            | I      | Доступ к raw PII при компрометации БД                                      | NFR-004                | Column-level encryption                        |
|                        | Audit logs             | R      | Отсутствие доказательств изменений                                         | need NFR (Non-Repudiation)| Blockchain-аудит                             |
