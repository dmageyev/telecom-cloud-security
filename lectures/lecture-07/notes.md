# Лекція 7 — Безпека веб- та телеком-застосунків: Детальний конспект

> **Курс:** Інформаційна безпека телекомунікаційних та хмарних технологій  
> **Лекція:** 7 з 11 | **Модуль:** 2 — Захист сервісів, інциденти та відповідність

---

## Вступ

Лекція 7 охоплює три взаємопов'язані домени сучасної кібербезпеки: безпеку веб-застосунків, захист телекомунікаційних мереж і хмарний підхід DevSecOps. Ці теми є фундаментальними для фахівців з інформаційної безпеки, адже саме веб-сервіси та телеком-інфраструктура є найчастіше атакованими цілями у сучасному цифровому середовищі.

Важливою особливістю лекції є розгляд спадкових протоколів — SS7 та Diameter, що були розроблені в епоху, коли концепція кібербезпеки ще не існувала як самостійна дисципліна. Ці протоколи досі використовуються у мільярдах телефонних з'єднань щодня, і їхні вразливості є реальним ризиком як для операторів, так і для кінцевих користувачів. Паралельно розглядається 5G — нове покоління мобільних мереж, де проблеми безпеки були враховані від початку проектування архітектури.

Хмарна складова лекції охоплює практичне застосування сервісів AWS (WAF, Shield, CloudFront) для захисту веб-застосунків і телеком API, а також принципи DevSecOps — методологію, що інтегрує безпеку у кожен етап розробки та доставки програмного забезпечення.

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
-- Код застосунку (PHP) — форма входу перевіряє і username, і password
$query = "SELECT * FROM users WHERE username='" . $_POST['user'] . "'"
       . " AND password='" . $_POST['pass'] . "'";

-- Легітимний запит:
SELECT * FROM users WHERE username='alice' AND password='secret'

-- Введення зловмисника: username = admin' --   password = (будь-що)
-- Результуючий запит:
SELECT * FROM users WHERE username='admin' --' AND password='...'
-- Коментар -- відкидає умову AND password='...' → вхід без знання пароля!
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
      content="default-src 'self'; script-src 'self' 'nonce-random123'">

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

**IMDSv2** суттєво зменшує ризик SSRF-доступу до metadata: для отримання metadata спочатку потрібно отримати session token через `PUT` запит. Це блокує багато типових SSRF-сценаріїв, але не робить доступ до metadata неможливим у всіх випадках, оскільки деякі SSRF-примітиви дозволяють виконувати `PUT` або додавати потрібні заголовки.

Тому IMDSv2 слід використовувати разом із додатковими контрзаходами: **egress filtering**, блокування доступу до `169.254.169.254` там, де він не потрібен, та **allowlist** для вихідних запитів.

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

### 1.9 HTTPS, TLS та HTTP Security Headers

#### TLS — протокол та версії

TLS (Transport Layer Security) є обов'язковим для всіх веб-застосунків, які передають будь-які дані. Версії та їх статус:

| Версія | Рік | Статус |
|--------|-----|--------|
| SSL 3.0 | 1996 | Deprecated (POODLE attack) |
| TLS 1.0 | 1999 | Deprecated (RFC 8996, 2021) |
| TLS 1.1 | 2006 | Deprecated (RFC 8996, 2021) |
| TLS 1.2 | 2008 | Підтримується, мінімальна вимога |
| TLS 1.3 | 2018 | **Рекомендований**, значно швидший |

TLS 1.3 усунув слабкі cipher suites (RC4, DES, 3DES, RSA key exchange), обов'язково Forward Secrecy (ECDHE), скоротив handshake до 1-RTT (або навіть 0-RTT для resume сесій).

**Типові помилки конфігурації TLS:**
- Дозволяти TLS 1.0/1.1 для "сумісності" зі старими клієнтами
- Відключати перевірку сертифіката у внутрішніх сервісах
- Використовувати самопідписані сертифікати без перевірки CA
- Mixed Content — завантаження HTTP ресурсів на HTTPS-сторінці

#### HTTP Security Headers

Правильний набір HTTP-заголовків безпеки значно ускладнює атаки:

| Заголовок | Призначення | Рекомендоване значення |
|-----------|-------------|----------------------|
| `Strict-Transport-Security` | Примусовий HTTPS (HSTS) | `max-age=31536000; includeSubDomains; preload` |
| `Content-Security-Policy` | Захист від XSS | `default-src 'self'; script-src 'self' 'nonce-...';` |
| `X-Frame-Options` | Захист від Clickjacking | `DENY` або `SAMEORIGIN` |
| `X-Content-Type-Options` | Захист від MIME sniffing | `nosniff` |
| `Referrer-Policy` | Контроль Referer заголовку | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | Обмеження API браузера | `camera=(), microphone=(), geolocation=()` |

**HSTS Preload:** Якщо домен включено до HSTS Preload List (hstspreload.org), браузер ніколи не звертатиметься до нього по HTTP, навіть при першому відвідуванні.

**Certificate Transparency (CT):** Всі SSL/TLS сертифікати публічно логуються в CT Log (RFC 9162). Це дозволяє виявити неавторизовані сертифікати для вашого домену.

---

### 1.10 WAF та кращі практики веб-безпеки

#### Web Application Firewall — принцип роботи

WAF (Web Application Firewall) аналізує HTTP/HTTPS трафік на рівні застосунку (L7) і блокує шкідливі запити. Режими роботи:

- **Detection mode** — лише логування, без блокування (для tuning)
- **Prevention mode** — активне блокування (production)
- **Hybrid** — деякі правила блокують, інші тільки логують

**Типи WAF:**
- **Network WAF** — апаратний пристрій у мережі (Imperva, F5 BIG-IP)
- **Host-based WAF** — модуль у веб-сервері (ModSecurity для Apache/nginx)
- **Cloud WAF** — SaaS або CDN-інтегрований (AWS WAF, Cloudflare, Akamai)

**OWASP CRS (Core Rule Set)** — набір правил для ModSecurity та AWS WAF, що покриває OWASP Top 10. Використовується як базовий рівень для більшості WAF рішень.

#### Defense in Depth — багаторівневий захист

Жоден окремий засіб захисту не є достатнім. Правильна архітектура:

```
Internet
    ↓ CloudFront + AWS Shield (DDoS, Edge)
    ↓ AWS WAF (L7 filtering, OWASP rules)
    ↓ Load Balancer (TLS termination)
    ↓ Application (prepared statements, output encoding)
    ↓ Database (least privilege, encryption at rest)
```

Кожен рівень перехоплює різні типи атак. Компрометація одного рівня не означає компрометацію системи.

---

## 2. Безпека телеком-застосунків

Телекомунікаційна інфраструктура є критично важливою для будь-якої сучасної держави. Атаки на телеком-мережі можуть призвести не тільки до фінансових збитків, але й до порушення роботи екстрених служб, перехоплення урядових комунікацій і масової слідкованини за громадянами. Унікальність телеком-безпеки полягає в тому, що більшість використовуваних протоколів (SS7, Diameter) були розроблені ще до появи концепції нульової довіри та в епоху, коли весь трафік між операторами вважався надійним за визначенням.

Сучасний телеком-оператор обслуговує одночасно легасі-інфраструктуру (2G/3G/SS7) та нові покоління мереж (4G LTE, 5G NR), що вимагає підтримки безпеки на всіх рівнях паралельно. Крім того, операторський бізнес включає складну екосистему партнерів: MVNO, роумінгові партнери, IPX-провайдери — кожен з яких є потенційним вектором атаки.

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

Захист: GTP Firewall між GGSN та інтернетом, **GSMA PRD FS.20 — GTP Security**.

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

### 2.7 Безпека мобільних застосунків

Мобільні застосунки операторів (self-care додатки, корпоративні VPN-клієнти) є специфічним вектором атак, оскільки поєднують клієнтський код, API-взаємодію та зберігання чутливих даних на пристрої.

#### OWASP MASVS — Mobile Application Security Verification Standard

| Рівень | Застосування |
|--------|-------------|
| MASVS-L1 | Базова безпека (всі застосунки) |
| MASVS-L2 | Захист від потужних зловмисників (банківські, телеком) |
| MASVS-R | Захист від реверс-інжинірингу та тампінгу |

**Основні ризики мобільних застосунків:**

- **Незахищене зберігання:** Паролі, токени, MSISDN у SharedPreferences або незашифрованих файлах
- **Слабка автентифікація:** Відсутність biometric або PIN lock після timeout
- **Небезпечна комунікація:** Відсутність certificate pinning → MITM атака через proxy
- **Витік через логи:** Запис чутливих даних у логи Android/iOS
- **Реверс-інжиніринг:** Декомпіляція APK та отримання hardcoded ключів API

**Захист:**
- Certificate pinning для API-комунікації (але ускладнює оновлення)
- Зберігання credentials у Android Keystore / iOS Secure Enclave
- Root/jailbreak detection (бібліотеки RootBeer, DTTJailbreakDetection)
- Obfuscation + RASP (Runtime Application Self-Protection)
- Мінімальні дозволи (Android permissions): запитувати лише необхідні

---

### 2.8 MVNO та безпека роумінгу

#### MVNO (Mobile Virtual Network Operator)

MVNO — оператор, що надає послуги мобільного зв'язку, орендуючи інфраструктуру у MNO (Mobile Network Operator). Моделі MVNO:

- **Full MVNO** — власний HLR/HSS, SIM-карти, IP-ядро
- **Light MVNO** — власні SIM та тарифи, але частина ядра від MNO
- **Reseller** — повністю залежить від MNO

**Ризики MVNO:**
- MVNO має доступ до SS7-мережі MNO → потенційний вектор атаки
- Слабкий контроль ідентичності абонентів MVNO з боку MNO
- Спільне використання HLR/HSS може призводити до витоку даних між MVNO

**Безпека роумінгу:**
Роумінг здійснюється через **IPX (IP eXchange)** провайдерів — посередників між операторами різних країн.

| Рівень | Протокол | Ризик |
|--------|----------|-------|
| Сигналізація | SS7 MAP / Diameter | Location tracking, SMS interception |
| Транспорт | IPX IP мережа | MITM, traffic analysis |
| Дані | GTP-U тунелі | User data interception |
| 5G роумінг | N32 (SEPP) | JOSE manipulation |

**Захист:**
- IPsec між IPX вузлами
- SS7/Diameter Signaling Firewall на кордоні з IPX
- SEPP для 5G роумінгу (N32 інтерфейс з JOSE захистом)
- Регулярний аудит роумінгових угод та IPX-партнерів

---

### 2.9 AWS API Gateway для телеком

AWS API Gateway є ключовим компонентом для побудови захищених телеком API у хмарі — BSS/OSS API для партнерів, MVNO-порталів, IoT платформ.

**Ключові функції безпеки API Gateway:**

| Функція | Опис |
|---------|------|
| Lambda Authorizer | JWT, API Key, IMSI-based автентифікація через Lambda |
| WAF Integration | AWS WAF на шлюзі (L7 захист від SQLi/XSS) |
| Throttling | Rate limit per метод/ресурс/API Key |
| Usage Plans | Квоти для партнерів/MVNO (daily/monthly quota) |
| VPC Link | Приватний доступ до телеком VPC без інтернету |
| Resource Policy | IP whitelist / account-based доступ |

**Архітектура BSS/OSS API:**
```
Partner / MVNO
    │ HTTPS + JWT
    ↓
API Gateway (WAF + Lambda Authorizer)
    │ VPC Link
    ↓
NLB (Network Load Balancer)
    │
    ↓
BSS Microservices (ECS Fargate)
    │
    ↓
Aurora PostgreSQL / DynamoDB
```

**Конфігурація Usage Plan (Terraform):**
```hcl
resource "aws_api_gateway_usage_plan" "partner" {
  name = "partner-quota"
  throttle_settings {
    burst_limit = 100
    rate_limit  = 50      # запитів/сек
  }
  quota_settings {
    limit  = 10000
    period = "DAY"        # 10k запитів/день
  }
}
```

---

### 2.10 Сигнальні фаєрволи та SMS Firewall

#### Signaling Firewall

Сигнальний фаєрвол — захисний пристрій (або програмне рішення), що інспектує та фільтрує сигнальні повідомлення (SS7 MAP, Diameter, SIP, GTP) на кордоні мережі оператора. Відповідно до GSMA FS.11, впровадження сигнального фаєрвола є обов'язковою вимогою для операторів, що беруть участь у міжнародному роумінгу.

**Функції Signaling Firewall:**

| Функція | Опис |
|---------|------|
| GT Validation | Перевірка відповідності Global Title (GT) та IMSI у MAP-повідомленнях |
| Category Blocking | Блокування MAP-операцій категорій 1, 2, 3 (GSMA FS.11) |
| Anomaly Detection | Виявлення location-tracking patterns, UpdateLocation-атак |
| Geo-blocking | Відхилення запитів з неочікуваних мереж / регіонів |
| Rate Limiting | Обмеження кількості повідомлень per Origin-Point Code (OPC) |
| Reporting | Логування та звітність для аудиту GSMA FS.11 compliance |

Провідні вендори Signaling Firewall: Cellusys, P1 Security, Mobileum, Tektronix, NetCracker, Subex.

**Категорії захисту GSMA FS.11:**
- **Категорія 1 (Базова):** Фільтрація очевидно шкідливих повідомлень — updateLocation від нерозпізнаних мереж, запити на кшталт AnyTimeInterrogation.
- **Категорія 2 (Розширена):** Корелятивна фільтрація — виявлення складних атак через кореляцію кількох повідомлень.
- **Категорія 3 (Аналітична):** ML-моделі для виявлення нетипової поведінки, підключення до T-ISAC (Telecom Information Sharing and Analysis Center).

#### SMS Firewall

SMS Firewall фільтрує вхідні та вихідні SMS на рівні SMSC для захисту абонентів від SMishing та запобігання A2P-шахрайству.

**Правила фільтрації SMS Firewall:**
- **Keyword Blocking** — шаблони шкідливого контенту, URL-аналіз у реальному часі через Threat Intelligence
- **Sender ID Validation** — перевірка реєстрації Sender ID у центральному реєстрі оператора
- **A2P Rate Limiting** — обмеження кількості SMS від одного джерела за одиницю часу
- **Grey Route Detection** — виявлення нелегальних маршрутів A2P трафіку в обхід тарифів
- **Flash SMS Blocking** — блокування SMS Class 0 (miflashing), що не зберігаються на пристрої
- **MSRN / IMSI Masking** — захист ідентифікаторів абонента під час маршрутизації

Регуляторна база: GSMA PRD IR.70 (Interconnection SMS), GSMA PRD IR.71 (SMS Firewall вимоги).

---

### 2.11 Моніторинг аномалій у телекомі

Виявлення атак у реальному часі є критичним для телеком-операторів. Традиційні IDS не підходять для специфічного трафіку SS7/Diameter/GTP, тому потрібні спеціалізовані рішення.

**CDR-аналіз для виявлення шахрайства:**

```python
# Спрощений алгоритм виявлення IRSF на основі CDR
import pandas as pd

def detect_irsf_anomalies(cdr_df):
    """
    cdr_df: DataFrame з колонками [msisdn, called_number, duration, timestamp]
    """
    HIGH_RISK_PREFIXES = ['+252', '+881', '+960', '+964']  # known IRSF prefixes

    # Velocity check: >10 дзвінків на high-risk prefix за 5 хвилин
    cdr_df['window'] = cdr_df['timestamp'].dt.floor('5min')
    grouped = cdr_df.groupby(['msisdn', 'window', 'prefix'])

    alerts = []
    for (msisdn, window, prefix), group in grouped:
        if prefix in HIGH_RISK_PREFIXES and len(group) > 10:
            alerts.append({
                'msisdn': msisdn,
                'type': 'IRSF_VELOCITY',
                'count': len(group),
                'prefix': prefix,
                'window': window
            })
    return alerts
```

**Metrics-based підходи:**

| Метрика | Порогове значення | Тип атаки |
|---------|-------------------|-----------|
| UpdateLocation rate | >5/год на IMSI | SS7 Location Tracking |
| SRI запити з нового SCCP | >100/год | SS7 Profiling |
| SMS до одного MSISDN | >50/хв | SMishing Campaign |
| GTP Create Session з однакового IMSI | >10/хв | SIM Cloning |
| Wangiri дзвінки (тривалість <3 сек) | >1000/год | Wangiri Fraud |

**Платформи телеком SOC:**
- **Mobileum RAID** — Fraud Management System з AI/ML
- **Cellusys Analytics** — SS7/Diameter моніторинг
- **AWS GuardDuty** — виявлення загроз у хмарних компонентах телеком
- **Amazon Kinesis + OpenSearch** — real-time CDR streaming та аналіз

---

### 2.12 Перехоплення трафіку та IMSI Catcher

#### Lawful Interception (LI)

Законне перехоплення — регуляторна вимога, що зобов'язує операторів надавати правоохоронним органам (LEA) доступ до комунікацій конкретних абонентів за рішенням суду.

**Архітектура LI (ETSI TS 101 331):**

```
Оператор:
  ├── ADMF (Administration Function) ← команди від LEA
  ├── IRI (Intercept Related Information) → LEA (метадані)
  └── CC (Content of Communication) → LEA (контент)
                ↑
    Law Enforcement Agency (правоохоронці)
```

- **3GPP TS 33.108** — LI для мобільних мереж (2G/3G/4G)
- **3GPP TS 33.127** — LI для 5G (IRI-POI, CC-POI в AMF/SMF/UPF)
- **ETSI TS 102 232** — загальна архітектура LI

**Ризики LI систем:**
- Несанкціонований доступ до LI-інтерфейсів (резонансний інцидент: Vodafone Greece, 2005 — прослуховування прем'єр-міністра)
- Backdoor у LI-обладнанні від постачальників
- Компрометація ключів шифрування LI-каналу до LEA
- Інсайдер з доступом до ADMF

#### IMSI Catcher (Stingray)

IMSI Catcher — пристрій, що імітує легітимну базову станцію і виконує MITM-атаку між телефоном та мережею.

**Принцип роботи:**
```
Легітимна BTS → BSC → MSC → HLR
IMSI Catcher (фіктивна вишка):
  1. Випромінює потужніший сигнал ніж реальні BTS
  2. Телефон підключається до "найсильнішої" станції
  3. Отримує IMSI/TMSI — ідентифікатор абонента
  4. Ініціює downgrade до 2G (де A5/1 зламаний)
  5. Декриптує та прослуховує дзвінки / SMS
```

**Еволюція методів перехоплення:**

| Метод | Покоління | Доступність |
|-------|-----------|-------------|
| IMSI Catcher | 2G–4G | Держструктури, злочинці ($2K–$100K) |
| SS7 Call Forward | 2G–3G | Будь-хто з SS7-доступом |
| Diameter Intercept | 4G | Спецслужби |
| GSM Downgrade | 2G fallback | Спеціалізоване ПО |

**Захист кінцевого користувача:**
- **End-to-End шифрування** — Signal Protocol, WhatsApp (Noise Protocol Framework)
- **VPN** — захист усього IP-трафіку
- **5G SUCI** — шифрований ідентифікатор: навіть фіктивна 5G-базова станція не отримає реальний IMSI
- **FIDO2/WebAuthn** — замість SMS OTP для критичних акаунтів

---

### 2.13 DDoS-атаки на телеком-мережі

Телеком-мережі є мішенню специфічних DDoS-атак, спрямованих не лише на пропускну здатність, але й на сигнальні протоколи.

**Телеком-специфічні DDoS:**

| Тип атаки | Протокол | Інструмент | Захист |
|-----------|----------|-----------|--------|
| SIP INVITE Flood | SIP/UDP | SIPVicious, inviteflood | SBC rate limiting |
| SS7 Signal Flood | SS7/SCTP | Кастомні скрипти | SS7 Firewall |
| GTP Flood | GTP/UDP | EPC Attack Toolkit | GTP Firewall |
| DNS Amplification | DNS/UDP | Ботнети | RPF, rate limit |
| RTP Flood | RTP/UDP | rtpflood | Медіа rate limit |
| Volumetric | IP | Ботнети, IoT | Anycast BGP, scrubbing |

**Amplification фактори:**
```
DNS:  8B запит   → 3000B відповідь  → 50×
NTP:  8B запит   → 468B відповідь   → 58×
SSDP: 30B запит  → 300B відповідь   → 10×
```

**SIP DDoS — специфіка:**

SIP Flood складніший за звичайний IP DDoS: кожен INVITE є легітимним запитом з коректними SIP-заголовками. SBC повинен ідентифікувати аномальну частоту від одного IP/SIP UA.

```
Порогові значення SBC:
  > 50 INVITE/сек з одного IP → тимчасовий бан (300 сек)
  > 1000 REGISTER/хв з одного IP → блокування
  Розмір тіла SDP > 4KB → підозра на DoS
```

**Захист телеком-оператора:**
- **Anycast BGP** — розподіл атаки по глобальній мережі (BGP community blackhole)
- **Scrubbing Centers** — очищення трафіку через Akamai Prolexic, Cloudflare Magic Transit
- **RTBH (Remote Triggered Black Hole)** — аварійне блокування атакованого префіксу
- **BCP38 / RPF** — фільтрація підробленого IP-відправника (Unicast Reverse Path Forwarding)
- **AWS Shield Advanced** — захист хмарних компонентів телеком (SRT підтримка, cost protection)

---

### 2.14 Криптографія у телеком-протоколах

#### Огляд криптографії за поколіннями

| Покоління | Протокол | Алгоритм шифрування | Статус |
|-----------|----------|---------------------|--------|
| 2G GSM | A5/1, A5/2 | Stream cipher 64-bit | **Зламаний** |
| 3G UMTS | KASUMI (UEA1/UIA1) | Block cipher 128-bit | Слабкий, deprecated |
| 3G/4G | SNOW 3G (UEA2/UIA2) | Stream cipher 128-bit | Прийнятний |
| 4G/5G | AES-EEA2 / AES-EIA2 | AES-CTR/CMAC 128-bit | Добрий |
| 5G | ZUC (EEA3/EIA3) | Stream cipher 128-bit | Добрий |
| SIP/RTP | SRTP (RFC 3711) | AES-128, HMAC-SHA1 | Стандарт |
| Роумінг | IPsec / TLS | AES-256-GCM, SHA-256 | Вимога |

**GSM A5/1 зламаний:** існують rainbow tables для повного bruteforce offline-злому GSM дзвінків. Вартість обладнання для атаки — від $2000.

#### Ключові протоколи захисту

**IPsec для телеком-з'єднань:**
```
Режим тунелювання IPsec (Transport/Tunnel Mode):
  - Tunnel: захист GTP між vPGW та партнерами
  - IKEv2: встановлення Security Association (SA)
  - ESP з AES-256-GCM: автентифіковане шифрування
  - 3GPP TS 33.210: вимоги до захисту IP-мережі
```

**SRTP для VoIP (RFC 3711):**
```
SRTP = RTP + шифрування + автентифікація:
  - AES-128 CTR: шифрування медіапотоку RTP
  - HMAC-SHA1 (80-bit): цілісність
  - DTLS-SRTP (RFC 5764): для WebRTC-дзвінків
  - ZRTP (RFC 6189): E2E без PKI (Zimmermann Protocol)
```

**Post-Quantum Cryptography (PQC) у телекомі:**

3GPP та GSMA вивчають PQC для захисту від атак квантових комп'ютерів у 5G Advanced та 6G:
- **CRYSTALS-Kyber** (NIST FIPS 203) — для Key Encapsulation Mechanism (KEM)
- **CRYSTALS-Dilithium** (NIST FIPS 204) — для цифрових підписів
- Перехід планується поетапно для N32 (SEPP) та NDS/IP інтерфейсів

---

### 2.15 IoT та M2M безпека у телекомі

Телеком-оператори є основними провайдерами IoT-підключень: NB-IoT та LTE-M через ядро LTE/5G. Очікується понад **30 мільярдів підключених IoT-пристроїв** до 2030 року (GSMA Intelligence).

#### Основні ризики IoT/M2M

| Ризик | Приклад | Наслідок |
|-------|---------|---------|
| Слабкі credentials | Default паролі (`admin/admin`) | Компрометація, ботнет |
| Відсутність OTA update | Незакриті вразливості роками | Masive exploitation |
| Незахищений транспорт | Дані без TLS/DTLS | Перехоплення, підробка |
| SIM misuse | IoT SIM для дзвінків або IRSF | Фінансові збитки оператора |
| IoT Botnet | Mirai (2016) — 600 Gbps DDoS | Знищення інфраструктури |
| Supply chain | Backdoor у IoT-чипах | Тихе шпигунство |

**Mirai botnet (2016):** Захопив 600,000+ IoT-пристроїв (камери, роутери) через default credentials. DDoS атака на Dyn DNS — 1.2 Tbps, недоступність Twitter, Netflix, Amazon.

#### Захист IoT/M2M

**Стандарти та технології:**
- **LwM2M (OMA)** — протокол управління IoT пристроями, DTLS для захисту
- **DTLS 1.3** — TLS для UDP (CoAP/MQTT протоколи IoT)
- **eSIM / iSIM** — захищена ідентифікація (GSMA SGP.02, SGP.32)
- **PKI для IoT** — certificate-based автентифікація замість паролів
- **OTA firmware** з цифровим підписом (SUIT standard — RFC 9019)

**Сегрегація IoT в мережі:**
```
IoT Security Architecture:
  ┌─────────────────────────────────────────┐
  │ IoT SIM → Окремий APN (iot.operator.ua) │
  │ → Dedicated PDN Gateway                 │
  │ → IoT Security Gateway (DPI + firewall) │
  │ → Ізольований VPC / VLAN               │
  │ → Rate limit: 100 KB/день per SIM       │
  │ → AWS IoT Core з X.509 cert auth        │
  └─────────────────────────────────────────┘
```

**AWS IoT Core для телеком-операторів:**
- **X.509 cert auth + IoT Device Policies** — кожен пристрій отримує унікальний сертифікат
- **AWS IoT Device Defender** — виявлення аномалій поведінки пристрою (detect, audit)
- **AWS IoT Jobs** — безпечна доставка OTA-оновлень з rollback-механізмом
- **Amazon Kinesis** — стрімінг IoT телеметрії для аналізу

---

### 2.16 Соціальна інженерія проти абонентів

#### Основні вектори атак

**Vishing (Voice Phishing):**

Телефонний дзвінок від "банку", "поліції" або "служби підтримки оператора". Зловмисники використовують:
- **Caller ID Spoofing** — підробка номера легітимного банку/організації
- **AI-generated голоси (2024+)** — deepfake голосу знайомої людини або офіційної особи
- **Pre-texting** — ретельно підготовлений сценарій розмови на основі OSINT

**SMishing (SMS Phishing):**
- Шкідливі SMS з фішинговими посиланнями
- Підробка Sender ID банків та держорганів
- OTP-перехоплення через соціальну інженерію ("скажіть код, що прийшов")

#### SIM Swap Attack — детальна схема

SIM Swap поєднує соціальну інженерію та технічні методи:

```
Крок 1: OSINT збір
  - Ім'я, дата народження, адреса жертви (Facebook, LinkedIn, OSINT)
  - Контрольні питання для верифікації у оператора

Крок 2: Дзвінок оператору
  - "Я загубив телефон, прошу перевипустити SIM"
  - Відповіді на секретні питання (з OSINT-даних)
  - Оператор переносить номер на SIM зловмисника

Крок 3: Exploitation
  - Всі вхідні SMS + дзвінки → SIM зловмисника
  - Скидання паролів через SMS OTP
  - Захоплення email → банків → крипто-гаманців
```

**Статистика:** FBI IC3 2022 — SIM Swap fraud: **$72.7 мільйони збитків** в США.

#### Захист від соціальної інженерії

**STIR/SHAKEN — стандарт автентифікації Caller ID (RFC 8224/8226, ATIS-1000074):**
```
Оригінатор → Підписує PASSporT токен (JWT) з номером
Транзитний оператор → Передає токен
Оператор отримувача → Перевіряє підпис через PKI
Дисплей телефону: "✅ Verified A" або "⚠️ Unverified"

Рівні атестації:
  A — Повна верифікація (відомий абонент + номер)
  B — Часткова (відомий шлюз, але не кінцевий абонент)
  C — Маршрут верифіковано, початок — ні
```

Обов'язково в США (FCC STIR/SHAKEN mandate), впроваджується в ЄС та Україні.

**SIM Swap Detection API** (GSMA Open Gateway):
```http
GET /sim-swap/check
Authorization: Bearer <operator-token>
X-MSISDN: +380671234567

Response:
{
  "swapped": false,
  "lastSwapDate": "2024-01-15T10:30:00Z"
}
```

Банки можуть перевіряти чи не відбувся SIM Swap нещодавно перед підтвердженням транзакції.

---

### 2.17 Zero Trust для телеком-інфраструктури

**Zero Trust принцип:** "Ніколи не довіряй, завжди перевіряй" — жоден вузол, сервіс або абонент не вважається довіреним за замовчуванням, навіть всередині корпоративної мережі оператора.

#### Zero Trust для 5G Core (SBA)

```
5G Service-Based Architecture + Zero Trust:
  ┌──────────────────────────────────────────────────────┐
  │ Кожна NF:                                            │
  │   ✓ Автентифікується: X.509 cert + NRF OAuth 2.0    │
  │   ✓ Авторизується: token з мінімальними правами      │
  │   ✓ mTLS до кожного сусіднього NF (Istio Service Mesh)│
  │   ✓ Ізольована в окремому Kubernetes namespace       │
  │   ✓ Моніторинг: Envoy sidecar → Jaeger tracing       │
  └──────────────────────────────────────────────────────┘
```

**Istio Service Mesh для 5G SBA:**
- Автоматичне інжектування Envoy sidecar proxy до кожного NF pod
- mTLS увімкнено між усіма podами (PeerAuthentication: STRICT)
- Policy enforcement через `AuthorizationPolicy`
- Доповнює NRF OAuth 2.0 на application layer

#### Zero Trust архітектура телеком у AWS

```
AWS Verified Access (Zero Trust без VPN для BSS/OSS)
         ↓
AWS IAM Identity Center (SSO + SAML/OIDC)
         ↓
Amazon EKS + Istio (mTLS Service Mesh для NF)
         ↓
AWS Network Firewall (сегментація між VPC/subnet)
         ↓
VPC Flow Logs + GuardDuty (continuous monitoring)
         ↓
AWS Security Hub (єдиний Zero Trust posture dashboard)
```

**Network Slicing та Zero Trust:**

Кожен network slice (IoT, eMBB, URLLC) ізольований на рівні UPF + SMF + NSSAI. Zero Trust гарантує відсутність lateral movement між слайсами навіть при компрометації одного з них.

---

### 2.18 NFV та SDN — безпека віртуалізованих мереж

#### Еволюція та нові ризики

| Характеристика | Традиційна мережа | NFV/SDN |
|----------------|-------------------|---------|
| Обладнання | Спеціалізоване HW | COTS сервери |
| NF розгортання | Фізичне | VNF/CNF (VM/Container) |
| Управління | CLI, SNMP | REST API, YANG/NETCONF |
| Атаки | Фізичні, протокольні | Cloud + протокольні + hypervisor |

**Нові вектори атак NFV/SDN:**
- **Hypervisor escape** — вихід VNF на фізичний хост (VMware ESXi, KVM вразливості)
- **Tenant isolation failure** — витік між VNF різних клієнтів/слайсів
- **SDN Controller compromise** — центральний мозок мережі, Single Point of Failure
- **Оркестратор MANO** — несанкціонований деплой шкідливого VNF через VNFM API
- **Supply chain VNF** — шкідливі образи NFV у marketplace / репозиторії

**Захист NFV/SDN (ETSI NFV SEC):**
- **ETSI GS NFV-SEC 001** — NFV Security Problem Statement
- **ETSI GS NFV-SEC 010** — Report on Security Aspects in the NFV architecture
- Ізоляція VNF через vSphere/KVM + sVirt (SELinux / AppArmor)
- Сканування образів VNF перед деплоєм: Trivy, Amazon ECR scanning
- **OAuth 2.0 + mTLS для MANO API** — автентифікація оркестратора
- Immutable infrastructure: VNF образи підписані, не модифікуються в runtime

---

## 3. Хмара та DevSecOps

Хмарні технології кардинально змінили підхід до розгортання телеком-сервісів. Сучасні оператори переносять BSS/OSS системи в хмару (Cloud-Native Network Functions — CNF), використовують AWS або Azure для обслуговування API партнерів і MVNO. Це відкриває нові можливості масштабування, але й нові вектори атак.

DevSecOps — культура та набір практик, що інтегрує заходи безпеки у весь lifecycle розробки програмного забезпечення. Ключовий принцип: **Shift Left** — виявляти вразливості якомога раніше, коли вартість їх виправлення мінімальна. Дослідження IBM показує, що вартість виявлення вразливості на етапі коду — $80, тоді як після продакшн-релізу — $960, а після реального інциденту — $7680+.

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

### 3.3 Тестування безпеки веб- та телеком-застосунків

#### Методології тестування

| Методологія | Область | Стандарт |
|-------------|---------|---------|
| OWASP WSTG | Веб-застосунки | OWASP Testing Guide v4.2 |
| OWASP MASTG | Мобільні застосунки | OWASP Mobile Security Testing Guide |
| PTES | Penetration Testing | Penetration Testing Execution Standard |
| GSMA FS.11 | SS7 Security | GSMA Baseline Security Controls |
| 3GPP TS 33.117 | 5G Security | SECAM/SCAS |

**3GPP SECAM/SCAS:**
Обов'язкове тестування NF (AMF, SMF, UDM, PCF, SMF) перед деплоєм у 5G Core через Security Assurance Specification (SCAS). Тестування проводиться акредитованими лабораторіями.

#### Інструменти тестування

| Категорія | Веб | Телеком |
|-----------|-----|---------|
| Проксі / MITM | Burp Suite, OWASP ZAP | SigPloit, ss7MAPer |
| Fuzzing | ffuf, wfuzz, Boofuzz | SIPVicious, Asterisk, Codenomicon Defensics |
| Сканування | Nikto, Nuclei, Nessus | Nmap, Shodan, Censys |
| Мобільні | MobSF, APKTool, Frida | IMSI Catcher detector |
| API | Postman, Insomnia, kiterunner | Diameter test tools |
| Статичний аналіз | Semgrep, SonarQube, Bandit | Протокол-специфічні парсери |

**SigPloit — SS7 Penetration Testing:**
```python
# SigPloit — Python-фреймворк для тестування SS7
from sigtran import M2UA, SCCP, MAP

# Тест: надсилання UpdateLocation на чужий IMSI
map_msg = MAP()
map_msg.set_operation("updateLocation")
map_msg.set_imsi("255010123456789")
map_msg.set_vlr_number("+380671234567")
# Відправка через SS7 GW...
```

**Burp Suite для API-тестування телеком BSS:**
```
1. Interceptor → Capture JWT token від partner portal
2. Repeater → Тест IDOR: змінити account_id в запиті
3. Intruder → Brute force rate limit на /api/auth/login
4. Scanner → Автоматичний SAST API endpoints
5. Extensions → JSON Web Tokens (JWT Editor), AuthMatrix
```

---

### 3.4 AWS WAF, Shield та CloudFront — комплексний захист

#### Рівні захисту від DDoS та атак

| Рівень | Сервіс | Захищає від |
|--------|--------|-------------|
| L3/L4 | AWS Shield Standard | SYN/UDP flood, Volumetric — безкоштовно |
| L3/L4 | AWS Shield Advanced | Складні DDoS, SRT підтримка, cost protection |
| L7 | AWS WAF | SQLi, XSS, OWASP Top 10, bots |
| CDN | CloudFront | Кешування, geo-routing, TLS termination |
| Network | Network Firewall | IPS/IDS, stateful inspection, SIP deep inspection |
| DNS | Route 53 Resolver | DNS hijacking, DNSSEC |

**Shield Advanced — додаткові можливості:**
- **AWS Shield Response Team (SRT)** — 24/7 підтримка під час атаки, прямий доступ
- **DDoS cost protection** — компенсація AWS-витрат під час DDoS (EC2, CloudFront)
- **Global threat environment dashboard** — реальний час
- **Proactive engagement** — SRT сам контактує при виявленні атаки

#### CloudFront для веб та телеком API

- **450+ Points of Presence** — геодистрибуція, мінімальна затримка
- **Origin Shield** — додатковий рівень кешування, захист origin від прямих запитів
- **Lambda@Edge / CloudFront Functions** — обробка запитів на edge (auth, redirect, headers)
- **Real-time logs** — streaming до Kinesis для SIEM
- **Field-level encryption** — шифрування окремих полів HTTP POST (наприклад, PAN-number)

**CloudFront Security Headers (Terraform):**
```hcl
resource "aws_cloudfront_response_headers_policy" "security" {
  name = "telecom-security-headers"
  security_headers_config {
    strict_transport_security {
      access_control_max_age_sec = 31536000
      include_subdomains = true
      preload = true
      override = true
    }
    content_type_options { override = true }
    frame_options { frame_option = "DENY" override = true }
    xss_protection { protection = true mode_block = true override = true }
    referrer_policy {
      referrer_policy = "strict-origin-when-cross-origin"
      override = true
    }
  }
}
```

---

### 3.5 Compliance

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
- **11.6:** Механізми виявлення змін/втручання (change/tamper detection) для payment pages та пов'язаних скриптів

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

#### NIST Cybersecurity Framework 2.0

NIST CSF 2.0 (лютий 2024) є стандартом де-факто для управління ризиками кібербезпеки у США та за їх межами. Він визначає шість функцій:

| Функція | Опис | Приклади заходів для телеком |
|---------|------|------------------------------|
| **Govern** | Управління ризиками, стратегія, supply chain | CSP, ISMS, третя сторона аудити |
| **Identify** | Інвентаризація активів, оцінка ризиків | Карта мережевих активів, CDR класифікація |
| **Protect** | Захист активів | IAM, WAF, Signaling Firewall, шифрування |
| **Detect** | Виявлення подій | GuardDuty, SS7 monitor, SIEM |
| **Respond** | Реагування на інциденти | IR Playbooks, EventBridge + Lambda |
| **Recover** | Відновлення після інцидентів | Multi-AZ, Backup, DRS, DR-тести |

**AWS Compliance Automation:**

AWS Artifact надає для завантаження compliance звіти (SOC 2 Type II, ISO 27001 Certificate, PCI DSS AOC) для підтвердження відповідності регуляторам. AWS Config Rules автоматично перевіряє відповідність конфігурацій таким вимогам:

- `restricted-ssh` — перевіряє відсутність відкритого SSH на 0.0.0.0/0
- `s3-bucket-public-read-prohibited` — заборона публічних S3 buckets
- `rds-storage-encrypted` — перевірка шифрування RDS
- `vpc-flow-logs-enabled` — увімкнення VPC Flow Logs

Ці правила є частиною автоматизованих перевірок у конформних хмарних середовищах для операторів зв'язку.

---

## 4. Підсумок

### 4.1 Чек-лист безпеки

#### Web Application Security Checklist

- ✅ HTTPS скрізь (HSTS preloaded, TLS 1.2+, HSTS max-age ≥ 31536000)
- ✅ Prepared Statements / Parameterized Queries для всіх DB-запитів
- ✅ Output Encoding при виведенні в HTML (`htmlspecialchars`, DOMPurify)
- ✅ Content Security Policy (CSP) з nonce або hash (уникати `unsafe-inline`)
- ✅ SameSite=Strict cookie + HttpOnly + Secure атрибути
- ✅ MFA для адміністраторів та критичних операцій
- ✅ Rate limiting на auth endpoints та API (≤ 5 спроб/хвилину на логін)
- ✅ SAST + DAST + SCA в CI/CD pipeline (Semgrep + ZAP + Snyk)
- ✅ AWS WAF з OWASP Core Rule Set на ALB/CloudFront
- ✅ Security headers (X-Frame-Options, HSTS, CSP, X-Content-Type-Options)
- ✅ JWT: RS256/ES256, короткий exp (≤ 1 год), перевірка `iss` та `aud`
- ✅ Dependency scanning та оновлення (Dependabot або Snyk автоматично)
- ✅ Threat modeling (STRIDE/PASTA) для нових функцій

#### Telecom Security Checklist

- ✅ SS7/Diameter Signaling Firewall (GSMA FS.11/FS.19)
- ✅ GTP Firewall з peer whitelist (GSMA FS.20)
- ✅ SMS Firewall з A2P filtering та URL scanning
- ✅ IPsec / TLS між всіма роумінговими з'єднаннями
- ✅ SEPP для 5G роумінгового трафіку (N32 інтерфейс)
- ✅ NRF OAuth 2.0 між 5G NF (3GPP TS 33.501)
- ✅ SIM Swap Detection API для критичних API (GSMA Open Gateway)
- ✅ STIR/SHAKEN для захисту Caller ID
- ✅ CDR-аналіз для виявлення IRSF/Wangiri/SIM Box
- ✅ SBI: TLS між усіма NF у 5G Core
- ✅ IoT SIM сегрегація: окремий APN + rate limit на трафік
- ✅ Регулярний аудит роумінгових партнерів (IPX-провайдери)
- ✅ LI-інтерфейси ізольовані та моніторяться

### 4.2 Ключові концепції лекції

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
| **Zero Trust** | NF OAuth 2.0, mTLS/Istio, micro-segmentation |
| **IoT Security** | LwM2M, eSIM, Device Defender, segmented APN |
| **Соц. інженерія** | Vishing, SIM Swap, STIR/SHAKEN, FIDO2 |

### 4.3 Зв'язки з курсом

| Лекція | Зв'язок |
|--------|---------|
| ← Лекція 5 | KMS для шифрування API-ключів та CDR в S3 |
| ← Лекція 6 | EKS для деплою 5G NF, Lambda для Telecom API |
| → Лекція 8 | GuardDuty, CloudTrail, SIEM для моніторингу загроз |


---

## Висновок

Лекція 7 охопила широкий спектр загроз та контрзаходів у трьох взаємопов'язаних доменах. Веб-безпека базується на систематичному підході OWASP: розуміння Top 10 вразливостей є відправною точкою для будь-якого веб-розробника та security engineer. Кожна вразливість — від ін'єкцій до SSRF — має конкретні технічні контрзаходи, які необхідно застосовувати комплексно.

Телеком-безпека є унікальним доменом, де спадкові протоколи (SS7, GTP) існують паралельно з сучасними рішеннями (5G SBA, OAuth 2.0, SEPP). Захист цих мереж вимагає глибокого розуміння специфічних протоколів, стандартів GSMA, 3GPP, та специфічних рішень — Signaling Firewall, SMS Firewall, GTP Firewall. Телеком-шахрайство залишається масштабною проблемою з щорічними збитками майже $40 мільярдів, що вимагає систематичного CDR-аналізу та Fraud Management Systems.

Хмарна та DevSecOps складова показує, як сучасні оператори можуть будувати безпечну інфраструктуру на AWS, використовуючи managed сервіси (WAF, Shield, GuardDuty) замість власних рішень, а також інтегрувати безпеку у кожен крок CI/CD пайплайна. Zero Trust архітектура, де кожен компонент автентифікується незалежно від місця у мережі, є сучасним стандартом для 5G Core та хмарних телеком-розгортань.

Виконання регуляторних вимог (NIS2, GDPR, PCI DSS, GSMA FS.11) — це не лише юридичне зобов'язання, але й практичний мінімум заходів безпеки, перевірений галузевим досвідом. Фахівець з кібербезпеки в телекомі повинен орієнтуватися як у веб-вразливостях та хмарних архітектурах, так і в специфічних телеком-протоколах та регуляторному середовищі.
