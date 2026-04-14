# Лекція 7 — Безпека веб- та телеком-застосунків: Детальний конспект

> **Курс:** Інформаційна безпека телекомунікаційних та хмарних технологій  
> **Лекція:** 7 з 11 | **Модуль:** 2 — Захист сервісів, інциденти та відповідність

---

## 1. Безпека веб-застосунків

### 1.1 Ландшафт загроз

Веб-застосунки є найбільшою поверхнею атаки для більшості організацій. За даними Verizon DBIR 2023, 43% порушень даних пов'язані з атаками на веб-застосунки. Для телеком-операторів веб-вектор особливо критичний, оскільки вони обслуговують:

- **Self-care портали** — управління тарифами, балансом, послугами
- **Білінгові системи** — виставлення рахунків, платіжні шлюзи
- **OSS/BSS системи** — операційні та бізнес-підтримуючі системи
- **REST/SOAP API** для інтеграції з партнерами
- **Адміністративні панелі** мережевих елементів

Вектори атак поділяються на три категорії:

| Вектор | Приклади |
|--------|---------|
| **Client-side** | XSS, CSRF, Clickjacking, DOM manipulation |
| **Server-side** | SQL Injection, SSRF, Deserialization, RCE |
| **Infrastructure** | DDoS, TLS Downgrade, DNS Hijacking, MITM |

---

### 1.2 OWASP Top 10 (2021)

OWASP (Open Web Application Security Project) публікує Top 10 найпоширеніших вразливостей кожні кілька років на основі реальних даних аудитів.

#### A01 — Broken Access Control (Зламаний контроль доступу)
Піднявся з 5-го місця на 1-е. Охоплює IDOR, обхід авторизації, CORS-помилки.

**Приклад:** `GET /api/users/1042` повертає дані іншого користувача без перевірки прав.

#### A02 — Cryptographic Failures (Криптографічні збої)
Раніше "Sensitive Data Exposure". Слабке шифрування, HTTP замість HTTPS, MD5 для паролів.

#### A03 — Injection (Ін'єкція)
SQL Injection, Command Injection, LDAP Injection, NoSQL Injection. Вхідні дані потрапляють до інтерпретатора без санації.

#### A04 — Insecure Design (Небезпечна архітектура)
Нова категорія. Відсутність threat modeling, security requirements, перевірки бізнес-логіки.

#### A05 — Security Misconfiguration (Хибна конфігурація)
Дефолтні паролі, відкриті S3 bucket, XML External Entity (XXE), непотрібні функції.

#### A06 — Vulnerable and Outdated Components
Log4Shell (CVE-2021-44228) — яскравий приклад катастрофічних наслідків вразливості в залежності.

#### A07 — Identification and Authentication Failures
Слабкі паролі, брутфорс без блокування, відсутність MFA.

#### A08 — Software and Data Integrity Failures
Компрометація CI/CD pipeline, неперевірені оновлення, десеріалізація.

#### A09 — Security Logging and Monitoring Failures
Відсутність логів для виявлення атак, неповне логування, невчасне реагування.

#### A10 — Server-Side Request Forgery (SSRF)
Сервер виконує запити за адресою, вказаною зловмисником, отримуючи доступ до внутрішніх ресурсів.

---

### 1.3 SQL Injection — детальний розбір

#### Механізм атаки

```sql
-- Код застосунку (PHP)
$query = "SELECT * FROM users WHERE username='" . $_POST['user'] . "'";

-- Введення зловмисника: admin' --
-- Результуючий запит:
SELECT * FROM users WHERE username='admin' --'
-- Коментар -- ігнорує решту умов → вхід без пароля!
```

#### Типи SQL Injection

| Тип | Характеристика |
|-----|---------------|
| In-band | Результат відразу видно у відповіді |
| Blind Boolean | Умовні відповіді (true/false) |
| Blind Time-based | Затримка відповіді (`SLEEP(5)`) |
| Out-of-band | Дані надсилаються до зовнішнього сервера (DNS) |

#### Захист

1. **Parameterized queries** (prepared statements) — єдиний надійний метод
2. **ORM** — але уважно з raw queries та string interpolation
3. **Input validation** — whitelist дозволених символів
4. **Least privilege** — обліковий запис БД без DROP, CREATE
5. **WAF** — додатковий рівень захисту (не заміна!)

```java
PreparedStatement ps = conn.prepareStatement(
    "SELECT * FROM users WHERE username=? AND password=?"
);
ps.setString(1, username);
ps.setString(2, password);
```

---

### 1.4 XSS — Cross-Site Scripting

XSS дозволяє атакуючому виконати довільний JavaScript у браузері жертви.

**Типи:**
1. **Reflected XSS** — скрипт передається у GET/POST і відображається у відповіді
2. **Stored XSS** — скрипт зберігається в БД, виконується для всіх відвідувачів
3. **DOM-based XSS** — маніпуляція DOM без запитів до сервера

**DOM-based XSS приклад:**
```javascript
// Вразливий код:
document.getElementById('output').innerHTML = location.hash.substring(1);
// URL: https://app.example/#<img src=x onerror=alert(1)>
```

**Захист:**
```html
<!-- Content Security Policy -->
<meta http-equiv="Content-Security-Policy"
      content="default-src 'self'; script-src 'self' 'nonce-<generated-per-response>'">

<!-- Output encoding у PHP -->
echo htmlspecialchars($user_input, ENT_QUOTES, 'UTF-8');
```

```http
Content-Security-Policy: default-src 'self';
  script-src 'self' https://trusted-cdn.example;
  style-src 'self' 'nonce-{random}';
  img-src 'self' data:;
  frame-ancestors 'none';
```

CSP різко ускладнює виконання ін'єктованих скриптів.

---

### 1.5 CSRF та SSRF

#### CSRF (Cross-Site Request Forgery)

Змушує браузер автентифікованого користувача виконати небажаний запит:

```html
<!-- Сторінка attacker.com -->
<img src="https://bank.com/transfer?to=attacker&amount=5000">
<!-- Браузер автоматично надсилає cookie сесії банку! -->
```

**Захист:**
- **CSRF Token** — унікальний токен у кожній формі, перевіряється сервером
- **SameSite=Strict** — браузер не надсилає cookie для cross-site запитів
- Перевірка заголовків `Origin` та `Referer`

#### SSRF (Server-Side Request Forgery)

Сервер виконує HTTP-запит до URL, вказаного атакуючим:

```
POST /api/fetch-url
Body: {"url": "http://169.254.169.254/latest/meta-data/iam/credentials"}

Відповідь: {"AccessKeyId": "ASIA...", "SecretAccessKey": "...", "Token": "..."}
```

**Реальний інцидент:** Capital One (2019) — SSRF через WAF → AWS IMDS v1 → витік IAM ролі → 100M+ записів клієнтів.

**IMDSv2** вирішує цю проблему: для отримання metadata спочатку потрібно отримати токен через `PUT` запит (не підробляється через SSRF).

---

### 1.6 Автентифікація та JWT

#### Проблеми автентифікації

- Слабкі паролі без обмежень (менше 8 символів, без спецсимволів)
- Відсутність MFA для привілейованих операцій
- Схеми відновлення пароля на основі "секретних питань"
- Витік сесійних токенів у URL (`https://app.example/reset?token=abc123`)
- Відсутність блокування після N невдалих спроб (brute force)

#### JWT — поширені атаки

**`alg: none` атака:**
```json
// Заголовок:
{"alg": "none", "typ": "JWT"}
// Якщо сервер приймає `none` — підпис не перевіряється!
```

**Algorithm Confusion (RS256 → HS256):**
- Сервер підписує RS256 (приватним ключем)
- Зловмисник змінює алгоритм на HS256
- Використовує публічний ключ RS256 як HMAC-секрет
- Сервер перевіряє підпис публічним ключем → успіх!

#### Захист JWT

- Строго перевіряти і фіксувати алгоритм (`alg` whitelist)
- Секрет HMAC ≥ 256 біт, згенерований криптографічно
- Короткий TTL (`exp`), з Refresh token ротацією
- Зберігати в `httpOnly; Secure; SameSite=Strict` cookie

---

### 1.7 IDOR та Broken Access Control

#### Горизонтальне vs Вертикальне підвищення привілеїв

- **Горизонтальне:** Користувач A отримує доступ до даних Користувача B (однаковий рівень ролі)
- **Вертикальне:** Звичайний користувач виконує адміністративні функції

#### Приклад IDOR

```
GET /api/orders/10001 → 200 OK (моє замовлення)
GET /api/orders/10002 → 200 OK (замовлення іншого клієнта!)

Виправлення:
if order.user_id != current_user.id:
    return 403 Forbidden
```

#### UUID vs числові ID

Використання UUID (v4) замість auto-increment ID ускладнює IDOR, але **не усуває** — авторизація на рівні сервера обов'язкова.

---

### 1.8 Безпека API

#### OWASP API Security Top 10 (2023)

**API1 (BOLA)** — Broken Object Level Authorization:
```http
GET /api/v1/accounts/ACC001/transactions   ← свій рахунок
GET /api/v1/accounts/ACC002/transactions   ← чужий рахунок → 200 OK (вразливість!)
```

**API4 (Unrestricted Resource Consumption)** — відсутність rate limiting:
```
// Атакуючий може надіслати 10 000 запитів/сек
// Тоді як легітимний користувач ← 10 запитів/хв
```

**GraphQL-специфічні атаки:**

```graphql
# Query depth attack — DoS через глибоко вкладені запити
query {
  user {
    friends {
      friends {
        friends {
          friends { ... }  # до нескінченності
        }
      }
    }
  }
}
```

Захист: обмеження глибини запиту (`maxDepth: 5`), інтроспекцію відключити в продакшні.

---

## 2. Безпека телеком-застосунків

### 2.1 SS7 — Signaling System 7

#### Архітектура та передумови

SS7 — протокол сигналізації, розроблений у 1975 році для телефонних мереж. Ключова проблема: **відсутність автентифікації між операторами**. Кожен учасник SS7-мережі довіряє повідомленням від будь-якого іншого учасника.

Компоненти:
- **MSC** (Mobile Switching Center) — комутація дзвінків
- **HLR** (Home Location Register) — база даних абонентів оператора
- **VLR** (Visitor Location Register) — тимчасові дані роумінгових абонентів
- **SMSC** (SMS Center) — маршрутизація SMS

#### SS7 Атаки

**Атака відстеження локації (Location Tracking):**

```
Зловмисник (з SS7 доступом):
1. Надсилає: SendRoutingInfo (SRI) → HLR жертви
   Відповідь: MSRN, IMSI (ідентифікатори)

2. Надсилає: ProvideSubscriberInfo (PSI) → MSC де перебуває жертва
   Відповідь: Cell ID (номер стільника) → GPS точність через базу Cell ID
```

Точність: 50–300 метрів у міських районах.

**Атака перехоплення SMS:**

```
1. Зловмисник надсилає: UpdateLocation → HLR жертви
   (Фальшивий MSC/SGSN з адресою зловмисника)

2. HLR оновлює VLR запис — SMS тепер йдуть на "MSC зловмисника"

3. Всі вхідні SMS (включно з OTP для банку) → зловмисник
```

#### Захист SS7

```
Рівень 1 — Базовий фільтрінг:
  ✓ SMS Home Routing — всі SMS через SMSC оператора
  ✓ Блокувати SRI/PSI від невідомих GTT (Global Title Translation)
  ✓ Блокувати UpdateLocation від мереж-не-партнерів

Рівень 2 — Аналітика:
  ✓ Виявлення аномалій: UpdateLocation без попереднього Location Update
  ✓ Кореляція SS7 подій з CDR
  ✓ Сповіщення при підозрілих SRI → HLR запитах

Рівень 3 — Машинне навчання:
  ✓ ML-моделі поведінкового аналізу
  ✓ Автоматичне блокування підозрілих джерел
```

Нормативна база: **GSMA FS.11** (SS7 Network Security), категорії загроз 1-7.

---

### 2.2 Diameter та GTP

#### Diameter — 4G/LTE Signaling

Diameter замінив SS7 MAP у LTE мережах для аутентифікації та авторизації. Хоча він кращий за SS7 (підтримує TLS/IPSec), на практиці більшість операторів не шифрують Diameter трафік між собою.

**Основні вразливості:**
- Spoofing Origin-Host/Origin-Realm — видавати себе за інший вузол
- `Cancel-Location-Request` (CLR) — примусово "відключити" абонента
- `Update-Location-Request` (ULR) — аналог SS7 UpdateLocation

**Захист:** Diameter Edge Agent (DEA) з фільтрацією + IPSec між вузлами.

#### GTP — GPRS Tunneling Protocol

GTP тунелює пакети між мобільним пристроєм та мережею. GTP-C (Control) — управління сесіями; GTP-U (User) — передача даних.

**Вразливість Cell Spoofing:**
```
Зловмисник надсилає підроблений GTP-C Create Session Request
з IMSI жертви → Creates PDP context на ім'я жертви
→ Інтернет трафік → тунелюється через зловмисника
```

Захист: GTP Firewall між GGSN та інтернетом, **GSMA IR.77**.

---

### 2.3 VoIP та SIP безпека

#### SIP Protocol

SIP (Session Initiation Protocol, RFC 3261) — протокол сигналізації для VoIP. Використовує текстовий формат (схожий на HTTP), що робить його вразливим до різних атак.

**SIP Registration Hijacking:**
```
Легітимна реєстрація:
REGISTER sip:telecom.ua SIP/2.0
From: <sip:alice@telecom.ua>
Contact: <sip:alice@192.168.1.100>

Hijacking attack:
REGISTER sip:telecom.ua SIP/2.0
From: <sip:alice@telecom.ua>
Contact: <sip:alice@attacker.com>  ← підроблений Contact
→ Тепер дзвінки для Alice йдуть до зловмисника!
```

**INVITE Flooding (SIP DoS):**
```bash
# SIPVicious — відомий інструмент атаки
svwar -e100-200 192.168.1.1  # Enumerate extensions
inviteflood eth0 target user 192.168.1.1 1000  # 1000 INVITE/сек
```

**Захист:**
- **SBC (Session Border Controller)** — нормалізація SIP, rate limiting, topology hiding
- **SIPS** (SIP over TLS) для шифрування сигналізації
- **SRTP** для шифрування медіа (RTP)
- **Digest Authentication** з перевіркою nonce

---

### 2.4 SMS-безпека та STIR/SHAKEN

#### SIM Swapping — детальний розбір

SIM Swap — одна з найнебезпечніших атак, що поєднує соціальну інженерію та технічні вектори:

```
Метод 1 — Соціальна інженерія:
  1. Збір OSINT: ім'я, дата народження, адреса жертви
  2. Дзвінок оператору: "Я загубив телефон, прошу перевипустити SIM"
  3. Відповідь на "секретні питання" → новий SIM контролює зловмисник

Метод 2 — SS7 (технічний):
  1. Зловмисник має SS7 доступ
  2. MO-Forward-SM перенаправляє SMS на власний SMSC
  3. Отримує OTP без взаємодії з оператором

Результат: контроль номера → 2FA bypass → банківський акаунт
```

**Статистика:** FBI IC3 2022 — SIM swap fraud: $72.7M збитків в США.

**NIST SP 800-63B** рекомендує **уникати SMS OTP** для криптографічно чутливих операцій.

#### STIR/SHAKEN — захист від CLI Spoofing

Стандарти IETF RFC 8224/8226 та ATIS для верифікації Caller ID:

```
Оригінатор → Підписує JWT з номером → Підпис перевіряється отримувачем
[ORIG]    → [attestation: A/B/C]  → [TERM] → [Display: ✅ Verified]
```

- **Attestation A:** Повна верифікація (відомий абонент)
- **Attestation B:** Часткова (відомий шлюз, але не абонент)
- **Attestation C:** Маршрут верифіковано, але початок — ні

---

### 2.5 5G безпека

#### Архітектурні покращення 5G

**SUCI (Subscription Concealed Identifier):**
```
4G: Телефон → BTS: IMSI = 255010123456789 (відкрито!)
    → IMSI Catcher може перехопити

5G: Телефон → gNB: SUCI = encrypted(SUPI, ephemeral_key)
    → Навіть якщо перехоплено — SUPI (реальний IMSI) захищений
    → Тільки HSS/UDM може розшифрувати SUCI → SUPI
```

**SEPP (Security Edge Protection Proxy):**
```
Оператор A (Україна) ←── N32 ──→ Оператор B (Польща)
         SEPP-A                     SEPP-B

N32 інтерфейс:
  - TLS для транспорту
  - JOSE (JSON Object Signing and Encryption) для application-layer
  - Фільтрація та перевірка роумінгового трафіку
  - Прихування топології мережі
```

**Service-Based Architecture (SBA):**
```
NRF (Network Repository Function):
  AMF, SMF, PCF, UDM реєструються в NRF

Взаємодія: OAuth 2.0 Client Credentials Grant
  AMF запитує токен у NRF
  AMF → токен → UDM (автентифікований запит)
```

```
AMF ←──── HTTP/2 + OAuth 2.0 ────→ SMF
NRF ←──── HTTP/2 + mTLS ──────→ PCF
```

**Загрози SBI:**
- JWT token forgery між NF
- OAuth 2.0 misconfiguration
- HTTP/2 DoS (HPACK bomb, RST flood)
- Rogue NF — підробна мережева функція в ядрі

**Загрози ізоляції слайсів (Network Slicing):**
- Витік між слайсами через спільні NF (Network Functions)
- SSRF через SBI API від одного слайсу до іншого
- Атаки на NSSF (Network Slice Selection Function)

---

### 2.6 Telecom Fraud

Телеком-шахрайство — глобальна проблема з річними збитками **$38.95 мільярдів** (CFCA 2021).

#### IRSF (International Revenue Share Fraud)

Найдорожчий вид телеком-шахрайства: **$6.1B/рік** глобально.

**Механізм:**
1. Шахрай орендує premium-номер (наприклад, в Латвії, Сомалі)
2. Отримує 60–80% від вартості дзвінка
3. Генерує масові дзвінки: через ботів, компрометовані АТС, скомпрометовані акаунти
4. Оператор жертви виплачує interconnect fees → шахрай отримує гроші

```
Схема:
1. Зловмисник реєструє "premium" номери в країні X (+890XXXX)
2. Оператор Y перераховує 70% вартості дзвінків → оператору X → зловмиснику
3. Зловмисник використовує IRSF-as-a-service або ботнети для дзвінків
4. Всі дзвінки — недійсні, але оплата вже перерахована
```

#### Wangiri Fraud

```
1. Зловмисник дзвонить на тисячі номерів, кидає після 1 дзвінку
2. Жертва передзвонює (бачить пропущений дзвінок)
3. Попадає на premium номер → рахунок за дзвінок
4. Зловмисник отримує revenue share
```

#### Виявлення шахрайства

```python
# CDR-based IRSF detection (спрощений алгоритм)
def detect_irsf(cdr_records):
    # Velocity check: >10 дзвінків за 5 хвилин на один префікс
    for subscriber, calls in group_by_subscriber(cdr_records):
        per_prefix = group_by_destination_prefix(calls)
        for prefix, calls_to_prefix in per_prefix.items():
            if is_high_risk_prefix(prefix) and rate(calls_to_prefix) > 10:
                flag_for_review(subscriber, f"IRSF_{prefix}")
```

---

## 3. Хмара та DevSecOps

### 3.1 AWS WAF та Shield

#### Web ACL структура

```
Web ACL
├── Rule Group: AWS-AWSManagedRulesCommonRuleSet (Priority: 10)
│   ├── NoUserAgent_HEADER
│   ├── UserAgent_BadBots_HEADER
│   └── SizeRestrictions_BODY
├── Rule Group: AWS-AWSManagedRulesSQLiRuleSet (Priority: 20)
├── Custom Rule: Rate-Based (Priority: 30)
│   └── Rate: 2000 req / 5 min per IP → Block
├── IP Set Rule: BlockList (Priority: 40)
└── Default Action: Allow
```

#### Кастомне правило (rate-based) — JSON

```json
{
  "Name": "RateLimitLoginEndpoint",
  "Priority": 1,
  "Action": { "Block": {} },
  "Statement": {
    "RateBasedStatement": {
      "Limit": 100,
      "AggregateKeyType": "IP",
      "ScopeDownStatement": {
        "ByteMatchStatement": {
          "FieldToMatch": { "UriPath": {} },
          "PositionalConstraint": "STARTS_WITH",
          "SearchString": "/api/auth/login",
          "TextTransformations": [{"Type": "LOWERCASE", "Priority": 0}]
        }
      }
    }
  }
}
```

Це правило обмежує 100 запитів/5хв на `/api/auth/login` з одного IP — захист від brute force.

#### Інтеграція з CloudFront

```json
{
  "Type": "AWS::WAFv2::WebACLAssociation",
  "Properties": {
    "ResourceArn": {"Ref": "CloudFrontDistribution"},
    "WebACLArn": {"Ref": "MyWebACL"}
  }
}
```

WAF на CloudFront діє **глобально** (CLOUDFRONT scope), на ALB — регіонально (REGIONAL scope).

#### AWS Shield Advanced — ключові переваги

- **SRT (Shield Response Team)** — 24/7 підтримка під час DDoS атак
- **Cost Protection** — відшкодування scaled-up ресурсів під час атаки
- **Proactive Engagement** — AWS сам звертається при виявленні атаки
- **Real-time Attack Visibility** — дашборд DDoS метрик

---

### 3.2 DevSecOps Pipeline

#### Shift Left Security

Чим раніше виявлена вразливість, тим дешевше її виправити:
- Код: $80
- QA: $240
- Production: $960
- Після інциденту: $7680

#### Приклад GitHub Actions pipeline

```yaml
name: Security Pipeline
on: [push, pull_request]

jobs:
  sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Semgrep SAST
        uses: returntocorp/semgrep-action@v1
        with:
          config: p/owasp-top-ten

  sca:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Trivy vulnerability scan
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          severity: 'HIGH,CRITICAL'
          exit-code: '1'

  iac:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Checkov IaC scan
        uses: bridgecrewio/checkov-action@master
        with:
          directory: terraform/
          soft_fail: false

  secrets:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Gitleaks secret scan
        uses: gitleaks/gitleaks-action@v2
```

#### Телеком-специфічне тестування

| Фаза | Інструмент | Що тестується |
|------|-----------|---------------|
| Build | Codenomicon Defensics | SIP, Diameter, GTP fuzzing |
| Test | SIPVicious | SIP security assessment |
| Runtime | SS7 Monitor (FW logs) | Аномальні SS7 повідомлення |
| Runtime | FMS | CDR anomaly detection |

---

### 3.3 Compliance

#### GDPR — вимоги до веб- та телеком-застосунків

| Стаття | Вимога | Технічна реалізація |
|--------|--------|---------------------|
| Art. 25 | Privacy by Design | Мінімізація даних, шифрування за замовчуванням |
| Art. 32 | Технічні заходи | TLS, шифрування, RBAC, аудит |
| Art. 33 | Повідомлення про порушення | Процедура за 72 години |
| Art. 35 | DPIA | Оцінка ризиків для privacy-чутливих систем |

#### PCI DSS v4.0 — Requirement 6 (Secure Software)

- **6.2:** Процес виявлення та управління вразливостями
- **6.3:** Захист від відомих вразливостей (патчинг ≤ 30 днів для критичних)
- **6.4:** Веб-застосунки захищені від OWASP Top 10
- **6.4.2:** WAF або автоматизований технічний засіб
- **11.6:** Перевірка HTTP заголовків на зміни

#### 3GPP TS 33.501 — ключові вимоги для 5G

- 5G-AKA або EAP-AKA' для автентифікації абонента
- SUPI concealment через SUCI (ECIES)
- NDS/IP (IPsec) для захисту N2, N3 інтерфейсів
- Захист сигналізації між NF: TLS 1.2+ або DTLS 1.2+
- OAuth 2.0 для авторизації між NF

#### OWASP ASVS — рівні перевірки

```
L1 (Базовий) — автоматизовані перевірки:
  → Відсутність ін'єкцій
  → HTTPS скрізь
  → Основна автентифікація

L2 (Стандартний) — для більшості застосунків:
  → MFA, управління сесіями
  → Контроль доступу
  → Логування безпекових подій

L3 (Просунутий) — для критичних систем:
  → Криптографічні вимоги
  → Defense in Depth
  → Threat modeling
```

#### NIS2 та телеком-оператори

Директива NIS2 (Network and Information Security, 2022/2555) включає телеком-операторів до переліку **критичних суб'єктів**, що зобов'язані:

- Впровадити **заходи управління ризиками кібербезпеки**
- Повідомляти про значні інциденти протягом **24 годин** (початкове сповіщення) та **72 годин** (повне)
- Призначити відповідального за кібербезпеку
- Проводити регулярні оцінки ризиків

Штрафи: до **€10 мільйонів** або **2% річного обороту** (подібно до GDPR).

---

## 4. Підсумок

### Ключові концепції лекції

| Тема | Основні поняття |
|------|----------------|
| **OWASP Top 10** | A01 Access Control, A03 Injection, A10 SSRF |
| **SS7** | Location tracking, SMS interception, UpdateLocation attack |
| **5G Security** | SUCI, SEPP, SBA + OAuth 2.0, Network Slicing |
| **SIP/VoIP** | Registration hijacking, INVITE flood, SBC, SRTP |
| **Telecom Fraud** | IRSF, SIM swap, Wangiri, CDR analysis |
| **AWS WAF/Shield** | WebACL, managed rules, Shield Advanced SRT |
| **DevSecOps** | SAST+SCA в CI, DAST на staging, shift-left |
| **Compliance** | PCI DSS v4.0, NIS2, OWASP ASVS, GSMA FS.11 |
