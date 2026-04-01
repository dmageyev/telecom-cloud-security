# Лекція 7 — Безпека веб- та телеком-застосунків: Детальний конспект

> **Курс:** Інформаційна безпека телекомунікаційних та хмарних технологій  
> **Лекція:** 7 | **Модуль:** 2 — Захист сервісів, інциденти та відповідність

---

## 1. Чому веб-застосунки є ціллю №1

Веб-застосунки відкриті для Internet 24/7, містять бізнес-логіку та дані мільйонів користувачів. За даними Verizon DBIR 2023, 43% усіх кібератак спрямовані на веб-застосунки.

**Ключові чинники ризику:**
- Публічна доступність з будь-якого куточка світу
- Постійно зростаюча складність (мікросервіси, SPA, GraphQL API)
- Швидкий цикл розробки (CI/CD) без достатнього security review
- Залежність від третіх сторін (npm, pip, Maven)

---

## 2. OWASP Top 10 — 2021

OWASP (Open Web Application Security Project) публікує Top 10 найпоширеніших вразливостей кожні кілька років на основі реальних даних аудитів.

### A01 — Broken Access Control (Зламаний контроль доступу)
Піднявся з 5-го місця на 1-е. Охоплює IDOR, обхід авторизації, CORS-помилки.

**Приклад:** `GET /api/users/1042` повертає дані іншого користувача без перевірки прав.

### A02 — Cryptographic Failures (Криптографічні збої)
Раніше "Sensitive Data Exposure". Слабке шифрування, HTTP замість HTTPS, MD5 для паролів.

### A03 — Injection (Ін'єкція)
SQL Injection, Command Injection, LDAP Injection, NoSQL Injection. Вхідні дані потрапляють до інтерпретатора без санації.

### A04 — Insecure Design (Небезпечна архітектура)
Нова категорія. Відсутність threat modeling, security requirements, перевірки бізнес-логіки.

### A05 — Security Misconfiguration (Хибна конфігурація)
Дефолтні паролі, відкриті S3 bucket, XML External Entity (XXE), непотрібні функції.

### A06 — Vulnerable and Outdated Components
Log4Shell (CVE-2021-44228) — яскравий приклад катастрофічних наслідків вразливості в залежності.

### A07 — Identification and Authentication Failures
Слабкі паролі, брутфорс без блокування, відсутність MFA.

### A08 — Software and Data Integrity Failures
Компрометація CI/CD pipeline, неперевірені оновлення, десеріалізація.

### A09 — Security Logging and Monitoring Failures
Відсутність логів для виявлення атак, неповне логування, невчасне реагування.

### A10 — Server-Side Request Forgery (SSRF)
Сервер виконує запити за адресою, вказаною зловмисником, отримуючи доступ до внутрішніх ресурсів.

---

## 3. SQL Injection — детальний розбір

### Механізм атаки

```sql
-- Код застосунку (PHP)
$query = "SELECT * FROM users WHERE username='" . $_POST['user'] . "'";

-- Введення зловмисника: admin' --
-- Результуючий запит:
SELECT * FROM users WHERE username='admin' --'
-- Коментар -- ігнорує решту умов → вхід без пароля!
```

### Типи SQL Injection

| Тип | Характеристика |
|-----|---------------|
| In-band | Результат відразу видно у відповіді |
| Blind Boolean | Умовні відповіді (true/false) |
| Blind Time-based | Затримка відповіді (`SLEEP(5)`) |
| Out-of-band | Дані надсилаються до зовнішнього сервера (DNS) |

### Захист

1. **Parameterized queries** (prepared statements) — єдиний надійний метод
2. **ORM** — але уважно з raw queries та string interpolation
3. **Input validation** — whitelist дозволених символів
4. **Least privilege** — обліковий запис БД без DROP, CREATE
5. **WAF** — додатковий рівень захисту (не заміна!)

---

## 4. Автентифікація та JWT

### Проблеми автентифікації

- Слабкі паролі без обмежень (менше 8 символів, без спецсимволів)
- Відсутність MFA для привілейованих операцій
- Схеми відновлення пароля на основі "секретних питань"
- Витік сесійних токенів у URL (`https://app.example/reset?token=abc123`)
- Відсутність блокування після N невдалих спроб (brute force)

### JWT — поширені атаки

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

### Захист JWT

- Строго перевіряти і фіксувати алгоритм (`alg` whitelist)
- Секрет HMAC ≥ 256 біт, згенерований криптографічно
- Короткий TTL (`exp`), з Refresh token ротацією
- Зберігати в `httpOnly; Secure; SameSite=Strict` cookie

---

## 5. XSS — Cross-Site Scripting

### Reflected XSS

```
1. Зловмисник надсилає жертві URL:
   https://shop.example/search?q=<script>stealCookies()</script>

2. Сервер відображає параметр без екранування:
   <p>Результати пошуку: <script>stealCookies()</script></p>

3. Браузер виконує скрипт у контексті shop.example
4. Скрипт надсилає cookies на сервер зловмисника
```

### Stored XSS
Зловмисник зберігає скрипт у БД (коментар, профіль). Скрипт виконується для кожного відвідувача.

### DOM-based XSS
```javascript
// Вразливий код:
document.getElementById('output').innerHTML = location.hash.substring(1);
// URL: https://app.example/#<img src=x onerror=alert(1)>
```

### Content Security Policy (CSP)

```http
Content-Security-Policy: default-src 'self'; 
  script-src 'self' https://trusted-cdn.example;
  style-src 'self' 'nonce-{random}';
  img-src 'self' data:;
  frame-ancestors 'none';
```

CSP різко ускладнює виконання ін'єктованих скриптів.

---

## 6. CSRF та SSRF

### CSRF — Механізм атаки

```html
<!-- На сайті зловмисника evil.com -->
<form action="https://bank.example/transfer" method="POST" id="csrf-form">
  <input type="hidden" name="to" value="attacker-account">
  <input type="hidden" name="amount" value="5000">
</form>
<script>document.getElementById('csrf-form').submit();</script>
```

Якщо користувач авторизований на bank.example і заходить на evil.com — запит надсилається з його cookies!

### Захист від CSRF

1. **Synchronizer Token Pattern** — унікальний токен у кожній формі
2. **Double Submit Cookie** — токен у cookie та body; сервер порівнює
3. **SameSite=Strict** — браузер не надсилає cookie для cross-site запитів
4. **Перевірка Origin/Referer** — заголовок надсилається браузером

### SSRF — Атаки на AWS Metadata

```
POST /api/fetch HTTP/1.1
{"url": "http://169.254.169.254/latest/meta-data/iam/security-credentials/EC2Role"}

Відповідь:
{
  "AccessKeyId": "ASIA...",
  "SecretAccessKey": "...",
  "Token": "..."
}
```

Зловмисник отримує тимчасові AWS credentials і може виконувати дії від імені IAM ролі.

**IMDSv2** вирішує цю проблему: для отримання metadata спочатку потрібно отримати токен через `PUT` запит (не підробляється через SSRF).

---

## 7. IDOR та Broken Access Control

### Горизонтальне vs Вертикальне підвищення привілеїв

- **Горизонтальне:** Користувач A отримує доступ до даних Користувача B (однаковий рівень ролі)
- **Вертикальне:** Звичайний користувач виконує адміністративні функції

### Приклад IDOR

```
GET /api/orders/10001 → 200 OK (мій замовлення)
GET /api/orders/10002 → 200 OK (замовлення іншого клієнта!)

Виправлення:
if order.user_id != current_user.id:
    return 403 Forbidden
```

### UUID vs числові ID

Використання UUID (v4) замість auto-increment ID ускладнює IDOR, але **не усуває** — авторизація на рівні сервера обов'язкова.

---

## 8. SS7 — Деталі вразливостей

### Чому SS7 досі вразливий?

SS7 розроблявся у 1975 році для закритих телефонних мереж між операторами, де всі учасники вважалися довіреними. З появою Інтернету та лібералізацією ринку доступ до SS7 отримали тисячі операторів по всьому світу, включаючи зловмисників.

### Відстеження місцезнаходження через SS7

```
Зловмисник (з доступом до SS7) → MSC (мережа оператора)
  1. Надсилає: SendRoutingInfoForSM (SRI-SM) → отримує IMSI + MSC address
  2. Надсилає: ProvideSubscriberInfo (PSI) до MSC
  3. Отримує: Cell ID → конвертує через відкриті бази до GPS-координат
```

Точність: 50–300 метрів у міських районах.

### Перехоплення SMS (2FA bypass)

```
1. Зловмисник запускає SRI-SM → отримує IMSI жертви
2. RegisterSS: переадресація SMS на власний номер
3. Запитує "забув пароль" на банківському сайті
4. Отримує OTP SMS → входить в акаунт
```

### Захист від SS7 атак

- **SS7 Firewall** — фільтрація аномальних MAP-повідомлень
- **Аномалія-виявлення:** незвичайні SRI-SM, PSI, RegisterSS запити
- **GSMA FS.11** — мінімальний базовий набір правил фільтрації
- Перехід на Diameter (4G) та HTTP/2 SBI (5G) — не вирішує повністю

---

## 9. 5G Security Architecture — деталі

### SUCI — Subscription Concealed Identifier

У 4G/LTE IMSI передавався відкрито → можливий перехоплення через IMSI catcher (Stingray).

У 5G IMSI (тепер SUPI) захищається:
```
SUPI: 255-01-1234567890
SUCI: шифрується публічним ключем оператора (ECIES)
→ Навіть базова станція не знає реального SUPI
```

### 5G Network Slicing

Оператор може ділити мережу на ізольовані "слайси" для різних клієнтів (MBB, URLLC, mMTC).

**Загрози ізоляції слайсів:**
- Витік між слайсами через спільні NF (Network Functions)
- SSRF через SBI API від одного слайсу до іншого
- Атаки на NSSF (Network Slice Selection Function)

### Service-Based Architecture (SBA) та безпека

5G Core замінив Diameter на HTTP/2 + JSON/REST для взаємодії між NF:

```
AMF ←──── HTTP/2 + OAuth 2.0 ────→ SMF
NRF ←──── HTTP/2 + mTLS ──────→ PCF
```

**Загрози SBI:**
- JWT token forgery між NF
- OAuth 2.0 misconfiguration
- HTTP/2 DoS (HPACK bomb, RST flood)
- Rogue NF — підробна мережева функція в ядрі

---

## 10. Telecom Fraud — деталі

### IRSF (International Revenue Share Fraud)

Найдорожчий вид телеком-шахрайства: $6 млрд/рік (CFCA).

**Механізм:**
1. Шахрай орендує premium-номер (наприклад, в Латвії, Сомалі)
2. Отримує 60-80% від вартості дзвінка
3. Генерує масові дзвінки: через ботів, компрометовані АТС, скомпрометовані акаунти
4. Оператор жертви виплачує interconnect fees → шахрай отримує гроші

### STIR/SHAKEN — захист від CLI Spoofing

Стандарти IETF RFC 8224/8226 та ATIS для верифікації Caller ID:

```
Оригінатор → Підписує JWT з номером → Підпис перевіряється отримувачем
[ORIG]    → [attestation: A/B/C]  → [TERM] → [Display: ✅ Verified]
```

- **Attestation A:** Повна верифікація (відомий абонент)
- **Attestation B:** Часткова (відомий шлюз, але не абонент)
- **Attestation C:** Маршрут верифіковано, але початок — ні

---

## 11. AWS WAF — практична конфігурація

### Web ACL структура

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

### Інтеграція з CloudFront

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

---

## 12. DevSecOps Pipeline — повний цикл

### Shift Left Security

Чим раніше виявлена вразливість, тим дешевше її виправити:
- Код: $80
- QA: $240
- Production: $960
- Після інциденту: $7680

### Приклад GitHub Actions pipeline

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

---

## 13. Відповідність — ключові вимоги

### GDPR — вимоги до веб- та телеком-застосунків

| Стаття | Вимога | Технічна реалізація |
|--------|--------|---------------------|
| Art. 25 | Privacy by Design | Мінімізація даних, шифрування за замовчуванням |
| Art. 32 | Технічні заходи | TLS, шифрування, RBAC, аудит |
| Art. 33 | Повідомлення про порушення | Процедура за 72 години |
| Art. 35 | DPIA | Оцінка ризиків для privacy-чутливих систем |

### PCI DSS v4.0 — Requirement 6 (Secure Software)

- 6.2: Процес виявлення та управління вразливостями
- 6.3: Захист від відомих вразливостей (патчинг ≤ 30 днів для критичних)
- 6.4: Веб-застосунки захищені від OWASP Top 10
- 6.4.2: WAF або автоматизований технічний засіб

### 3GPP TS 33.501 — ключові вимоги для 5G

- 5G-AKA або EAP-AKA' для автентифікації абонента
- SUPI concealment через SUCI (ECIES)
- NDS/IP (IPsec) для захисту N2, N3 інтерфейсів
- Захист сигналізації між NF: TLS 1.2+ або DTLS 1.2+
- OAuth 2.0 для авторизації між NF

---

## 14. Підсумок та ключові висновки

### Безпека веб-застосунків (30%)

1. **OWASP Top 10** — стандартний каталог вразливостей, обов'язковий до вивчення
2. **SQLi** — parameterized queries, ORM, least privilege
3. **XSS** — output encoding, CSP, HttpOnly cookies
4. **CSRF** — CSRF tokens, SameSite cookies
5. **SSRF** — whitelist, IMDSv2, мережева сегментація
6. **IDOR** — перевірка власника на кожному запиті

### Безпека телеком-застосунків (55%)

1. **SS7** — legacy-протокол з критичними вразливостями, потребує SS7 Firewall
2. **Diameter** — захист роумінгових з'єднань, IPsec, GSMA FS.19
3. **SIP/VoIP** — SIPS + SRTP + SBC
4. **SMS** — ненадійний канал для 2FA, мігрувати на TOTP/FIDO2
5. **GTP** — GTP Firewall, IPsec між SGW/PGW
6. **5G** — нова архітектура (SBA, SUCI), нові ризики (rogue AMF, slice isolation)
7. **Telecom APIs** — CAMARA, GSMA Open Gateway, OAuth 2.0, rate limiting

### Хмара та DevSecOps (15%)

1. **AWS WAF + Shield** — захист від L3-L7 атак
2. **CloudFront** — CDN + безпека + TLS termination
3. **Shift Left** — SAST + SCA + DAST в CI/CD pipeline
4. **Compliance** — GDPR, PCI DSS, 3GPP TS 33.501
