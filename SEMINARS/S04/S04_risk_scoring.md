# S04 - Приоритизация рисков L×I

| Risk ID | Source (DFD/Row)            | Consolidated Description                          | Threat | NFR link (ID) | L  | Rationale-L                          | I  | Rationale-I                          | Score | Top-5? | ADR candidate                |
|---------|-----------------------------|--------------------------------------------------|--------|---------------|----|--------------------------------------|----|--------------------------------------|-------|--------|------------------------------|
| R-01    | Edge: Service → External API | Утечка PII при передаче внешним системам         | I      | NFR-004       | 4  | Частые интеграции с внешними API     | 5  | Массовая утечка PII (GDPR)           | 20    | Top-1  | PII Masking + DLP Scanning    |
| R-02    | Node: PostgreSQL            | Доступ к raw PII при компрометации БД            | I      | NFR-004       | 3  | Требуется доступ к БД                | 5  | Критичная утечка данных              | 15    | Top-2  | Column-level Encryption       |
| R-03    | Edge: Client → API          | MITM-атака с модификацией JWT                    | T      | NFR-004       | 4  | Публичное API                        | 3  | Компрометация сессии                 | 12    | Top-3  | HSTS + Certificate Pinning    |
| R-04    | Node: API Gateway           | Подмена/повтор JWT токена                        | S      | NFR-005       | 3  | Стандартная уязвимость               | 4  | Полный доступ к системе              | 12    | Top-4  | JWT TTL+Refresh Rotation      |
| R-05    | Edge: Service → External API | Долгие ответы блокируют сервис                   | D      | NFR-001       | 4  | Ненадёжный внешний API               | 3  | Простой бизнес-процессов             | 12    | Top-5  | Async Calls with Buffering    |
| R-06    | Node: Integration Service   | Обход RBAC через невалидированные DTO            | E      | need NFR      | 2  | Сложная эксплуатация                 | 5  | Полный доступ к системе              | 10    |        | DTO-Level AuthZ Validation    |
| R-07    | Edge: Service → Queue       | Потеря аудит-логов при сбое очереди              | R      | need NFR      | 2  | Редкие сбои очереди                  | 4  | Невозможность расследования          | 8     |        | Audit Logs Replication        |
| R-08    | Node: Redis Cache           | Подмена кэшированных данных                      | T      | NFR-006       | 2  | Требуется доступ к Redis             | 3  | Коррупция бизнес-данных              | 6     |        | Signed Cache Values           |
| R-09    | Edge: Service → DB          | SQL-инъекции с утечкой PII                       | I      | NFR-002,004   | 1  | Защищено ORM                         | 5  | Теоретически критично                | 5     |        | ORM Hardening                |

## Сводка Top-5 рисков

1. **Top-1: R-01 (Score 20)**  
   *Высокий риск утечки PII при интеграциях (L=4, I=5)*  
   **ADR:** PII Masking + DLP Scanning для внешних API

2. **Top-2: R-02 (Score 15)**  
   *Компрометация PII в БД (L=3, I=5)*  
   **ADR:** Column-level Encryption для sensitive полей

3. **Top-3: R-03 (Score 12)**  
   *MITM-атаки на JWT (L=4, I=3)*  
   **ADR:** HSTS + Certificate Pinning

4. **Top-4: R-04 (Score 12)**  
   *Подмена JWT токенов (L=3, I=4)*  
   **ADR:** JWT TTL+Refresh Rotation

5. **Top-5: R-05 (Score 12)**  
   *Зависание на внешних API (L=4, I=3)*  
   **ADR:** Асинхронные вызовы с буферизацией

## Критерии тай-брейкинга
Для рисков с одинаковым Score 12:
1. **R-03** выбран перед R-04 из-за более высокой вероятности (публичное API)
2. **R-04** перед R-05 из-за большего impact (полный доступ vs простои)
