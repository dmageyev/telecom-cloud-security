# Лекція 7 — Безпека веб-застосунків: Детальний конспект

> **Курс:** Інформаційна безпека хмарних технологій  
> **Лекція:** 7 | **Модуль:** 2 — Захист сервісів, інциденти та відповідність

---

## 1. Ландшафт загроз веб-застосунків

### 1.1 Масштаб проблеми

Веб-застосунки є найпоширенішим вектором атак у сучасному цифровому світі. За даними Verizon Data Breach Investigations Report (DBIR) 2024, 43% усіх підтверджених зломів безпосередньо пов'язані з вразливостями веб-застосунків. IBM Cost of a Data Breach Report фіксує середню вартість одного інциденту на рівні $4.45 млн у 2023 році.

Основні категорії атак за частотою:
- **Injection attacks (SQL, XSS, Command)** — 32% інцидентів
- **API abuse** — 21% (зростаючий тренд через proliferation REST/GraphQL)
- **Broken Authentication** — 18%
- **DDoS на веб-рівень (L7)** — 16%
- **Supply chain / 3rd-party** — 13%

### 1.2 Поверхня атаки веб-застосунку

Кожен елемент веб-застосунку є потенційним вектором атаки:

**Вхідні дані:** форми реєстрації та авторизації, GET/POST параметри, HTTP заголовки (User-Agent, Cookie, Referer, X-Forwarded-For), завантаження файлів, JSON/XML тіло запиту.

**API endpoints:** REST API (найпоширеніший), GraphQL (особлива складність через introspection та batching), SOAP (legacy але поширений у телекомі), WebSocket (persistent connections).

**Залежності:** npm/pip/Maven пакети третіх сторін, CDN-вбудований JavaScript (Supply Chain атаки), рекламні скрипти та аналітика.

**Інфраструктура:** конфігурація веб-сервера (nginx, Apache), дефолтні сторінки помилок (stack trace розкриває архітектуру), відкриті адміністративні інтерфейси (/admin, /phpmyadmin).

**Телеком-специфіка:** BSS/OSS-портали (білінг, управління), OAM-інтерфейси (Operations, Administration, Maintenance), 5G NF REST API (Service-Based Interface), між-операторські API роумінгу.

---

## 2. OWASP Top 10 (2021) — деталі

### 2.1 A01: Broken Access Control

Найпоширеніша категорія у 2021 році, що підіймається з 5-го місця (2017). Означає, що система не коректно обмежує, що дозволено робити автентифікованим користувачам.

**IDOR (Insecure Direct Object Reference)** — підкатегорія, де системи повертають дані за прямим посиланням без перевірки прав.

Класичний приклад: REST endpoint `/api/invoices/1042` повертає рахунок поточного користувача. Якщо змінити ID на 1041, і сервер повертає дані іншого клієнта — це IDOR. Захист: перевірка на backend — чи належить запитуваний ресурс поточному authenticated user.

**Вертикальна ескалація привілеїв:** звичайний користувач отримує доступ до адміністративних функцій через маніпуляцію параметрами запиту (`role=admin`, або зміна HTTP методу з GET на POST для прихованих endpoints).

**Горизонтальна ескалація:** доступ до ресурсів іншого користувача того самого рівня привілеїв.

**Захист:**
- Перевірка прав **на рівні кожного backend-запиту**, не лише на рівні роутингу
- Принцип Deny by Default: все заборонено, якщо явно не дозволено
- RBAC (Role-Based Access Control) або ABAC (Attribute-Based) — централізована логіка
- Використання UUID/ULID замість sequential integer ID
- Автоматизовані тести для перевірки access control сценаріїв

### 2.2 A03: Injection

SQL Injection залишається актуальною понад 25 років. Атакуючий вводить шкідливий SQL-код у поле введення, і цей код виконується СУБД.

**Класичний SQLi:**
```sql
SELECT * FROM users WHERE login = 'admin' OR '1'='1' -- ' AND password = '...'
```
Результат: обхід перевірки пароля, login='admin' OR '1'='1' завжди true.

**Time-Based Blind SQLi:** коли відповідь не відрізняється, атакуючий використовує затримку: `'; IF (1=1) WAITFOR DELAY '00:00:05'--` — якщо відповідь затримується на 5 секунд, умова виконалась.

**Command Injection:** особливо небезпечна, якщо застосунок виконує команди ОС з користувацьким введенням. Наприклад, ping-сервіс: `os.system("ping " + user_input)` де `user_input = "8.8.8.8; cat /etc/passwd"`.

**Захист:**
- Параметризовані запити (Prepared Statements) — основний захист
- ORM (SQLAlchemy, Hibernate, ActiveRecord) — автоматична параметризація
- Whitelist валідація вхідних даних — дозволяти тільки очікуваний формат
- Мінімальні права БД-акаунту застосунку (SELECT/INSERT, не DROP)
- Escaping — для контекстів де параметризація неможлива

### 2.3 A02: Cryptographic Failures

Стосується захисту чутливих даних: паролів, платіжних даних, персональної інформації.

**Типові помилки:**
- Зберігання паролів у MD5 або SHA-1 (навіть з salt — занадто швидкі для брутфорсу)
- Передача sensitive даних по HTTP (plaintext)
- Хардкодовані ключі та секрети у вихідному коді (часто потрапляють до git)
- Застарілі алгоритми: DES (56-bit, зламується за хвилини), RC4 (BEAST атака), ECB mode (паттерн-витік)
- Самопідписані або прострочені TLS-сертифікати
- Відключена перевірка сертифіката: `requests.get(url, verify=False)` — MITM-вразливість

**Правильне хешування паролів:**
- **bcrypt** — адаптивна функція, cost factor (rounds) — рекомендовано 12+
- **Argon2id** — переможець Password Hashing Competition (2015), рекомендований NIST SP 800-63B
- **scrypt** — memory-hard функція, захист від GPU-брутфорсу

**Симетричне шифрування даних:**
- AES-256-GCM — автентифіковане шифрування (AEAD), захист від модифікації
- ChaCha20-Poly1305 — альтернатива AES, краща продуктивність на пристроях без AES-NI

### 2.4 A07: Authentication and Identification Failures

Включає вразливості механізмів автентифікації та управління сесіями.

**Credential Stuffing:** автоматизоване тестування пар login/password з витоків інших сервісів. Ефективне через масове повторне використання паролів (дослідження показують 52-65% користувачів використовують один пароль на кількох сайтах).

**Брутфорс:** систематичне перебирання паролів. Без rate limiting та lockout — trivial атака.

**Session Fixation:** атакуючий встановлює жертві відомий session ID до автентифікації, після чого отримує авторизований доступ.

**JWT-специфічні вразливості:**
- `alg: none` — старі бібліотеки приймали токени без підпису
- Algorithm confusion: підміна RS256 на HS256, де публічний ключ RSA використовується як секрет HMAC
- Відсутність перевірки claims `iss`, `aud`, `exp`

**Захист сесій:**
- Session ID: мінімум 128 біт ентропії, генерувати криптографічно безпечним PRNG
- Cookie flags: `HttpOnly` (недоступний JS), `Secure` (тільки HTTPS), `SameSite=Strict` або `Lax`
- Server-side інвалідація сесії при logout (не лише видалення cookie на клієнті)
- Idle timeout (15-30 хвилин) та absolute timeout (8 годин)

---

## 3. Специфічні атаки на веб

### 3.1 XSS — Cross-Site Scripting

XSS дозволяє атакуючому виконати JavaScript у браузері жертви в контексті довіреного сайту. Наслідки: крадіжка cookies/токенів, кейлоггінг, defacement, redirect.

**Reflected XSS:** шкідливий payload у запиті відображається у відповіді без зберігання. Типово: URL зі скриптом в параметрі пошуку.

**Stored (Persistent) XSS:** payload зберігається в БД і виконується у всіх відвідувачів сторінки. Найнебезпечніший тип — один запит атакує тисячі користувачів.

**DOM-based XSS:** маніпуляція DOM на стороні клієнта без взаємодії з сервером. Вразливі конструкції: `document.innerHTML = location.hash`, `eval(userInput)`, `document.write()`.

**Захист:**
- Output encoding — HTMLentities при виведенні: `<` → `&lt;`, `>` → `&gt;`, `"` → `&quot;`
- Content Security Policy (CSP) — whitelist дозволених джерел скриптів, заборона inline-JS
- HTTPOnly cookie flag — JS не може читати session token
- DOMPurify — бібліотека sanitization для HTML-вмісту
- Сучасні фреймворки (React, Vue, Angular) — автоматичне escaping у шаблонах

### 3.2 CSRF — Cross-Site Request Forgery

Атака змушує браузер жертви відправити автентифікований запит до цільового сайту без відома користувача. Браузер автоматично додає cookies до запитів, незалежно від джерела.

Приклад: жертва авторизована у банку, відвідує evil.com, де є прихований `<img src="https://bank.com/transfer?to=attacker&amount=10000">`. Браузер надсилає запит з cookies банку.

**CSRF Token:** сервер генерує непередбачуваний токен, вбудовує у форму (hidden field), перевіряє при кожному POST-запиті. Атакуючий не може дізнатися токен іншого сайту (Same-Origin Policy).

**SameSite Cookie:** сучасний та ефективний захист. `SameSite=Strict` — cookie не надсилається з інших сайтів взагалі. `SameSite=Lax` — дозволяє GET-переходи (для UX), блокує POST/PUT/DELETE.

### 3.3 SSRF — Server-Side Request Forgery

Атакуючий змушує сервер виконати HTTP-запит до внутрішнього або стороннього ресурсу.

**SSRF в хмарному середовищі — критична загроза:** Instance Metadata Service (IMDS) за адресою `169.254.169.254` надає EC2-інстансу тимчасові IAM credentials. SSRF до цього endpoint → отримання AWS credentials → повний компроміс хмарних ресурсів.

**Реальний кейс — Capital One (2019):** зловмисниця використала SSRF через вразливий WAF для доступу до IMDS, отримала IAM credentials з надмірними правами, завантажила понад 100 мільйонів записів клієнтів з S3.

**DNS Rebinding attack:** обхід SSRF-фільтрів через DNS. Спочатку DNS резолвиться у зовнішній IP (проходить whitelist перевірку), потім сервер DNS повертає внутрішній IP при повторному резолвінгу.

**Захист:**
- Whitelist дозволених URL/хостів (а не blacklist заборонених)
- Блокування RFC 1918 приватних діапазонів після резолвінгу
- IMDSv2 (обов'язковий) — вимагає двокрокове отримання токена (PUT + GET)
- Мережева ізоляція backend-серверів (egress filtering через Security Groups)

---

## 4. Аутентифікація та API Security

### 4.1 OAuth 2.0 та OpenID Connect

**OAuth 2.0** — фреймворк авторизації (не аутентифікації). Дозволяє застосунку отримати обмежений доступ до ресурсів від імені користувача без передачі пароля.

**Ролі OAuth 2.0:**
- Resource Owner — користувач
- Client — застосунок (веб/мобільний)
- Authorization Server — видає токени (Cognito, Keycloak, Auth0)
- Resource Server — API, що захищається

**Authorization Code + PKCE** — єдиний рекомендований flow для веб та мобільних застосунків:
1. Client генерує `code_verifier` (random) та `code_challenge = SHA256(code_verifier)`
2. Redirect до Authorization Server з `code_challenge`
3. AS повертає `authorization_code`
4. Client обмінює `code` + `code_verifier` на `access_token`
5. PKCE захищає від перехоплення `authorization_code`

**Client Credentials** — для machine-to-machine (M2M): мікросервіси, 5G NF-to-NF комунікація. Немає участі людини.

**OpenID Connect** — надбудова над OAuth 2.0 для аутентифікації. Додає `id_token` (JWT з даними про користувача) та стандартні claims: `sub`, `name`, `email`, `phone_number`.

**JWT Security:**
- Використовувати RS256 (асиметричний) замість HS256 для публічних застосунків
- Завжди перевіряти `exp`, `iss`, `aud`
- Термін дії access token — 15 хвилин (мінімум), refresh token — дні/тижні
- Revocation: access token → short-lived (не зберігати revocation list). Refresh token → revocation list в БД

### 4.2 OWASP API Security Top 10

API-вразливості відрізняються від традиційних веб-вразливостей специфікою середовища.

**API1 — Broken Object Level Authorization (BOLA):** аналог IDOR, але для API. Endpoint `/orders/{id}` повертає замовлення без перевірки, чи належить воно поточному authenticated user.

**API4 — Unrestricted Resource Consumption:** відсутність rate limiting → DoS. Особливо небезпечно для resource-intensive endpoints (пошук, завантаження файлів, звіти). Захист: rate limiting за IP, user, endpoint; ліміт розміру запиту.

**API5 — Broken Function Level Authorization:** `/api/v1/admin/users` доступний рядовому користувачу, оскільки перевіряється лише authentication, не authorization.

**API9 — Improper Inventory Management:** стара версія API `/v1/` без security updates, тест/dev endpoints у production, забутий `/api/internal/` без auth.

---

## 5. Транспортна безпека та HTTP-заголовки

### 5.1 TLS 1.3 — ключові покращення

TLS 1.3 (RFC 8446, 2018) — значне спрощення та покращення безпеки порівняно з TLS 1.2:

- **1-RTT Handshake** (замість 2-RTT) — менша затримка
- **0-RTT (Early Data)** — відновлення сесії без round-trip (є обмеження replay)
- Видалено небезпечні cipher suites: RSA key exchange, MD5, SHA-1, RC4, CBC з MAC-then-Encrypt
- **Forward Secrecy обов'язкова** — ECDHE для всіх cipher suites
- Спрощений cipher suite список: TLS_AES_128_GCM_SHA256, TLS_AES_256_GCM_SHA384, TLS_CHACHA20_POLY1305_SHA256

**Типові проблеми TLS у телекомі:** SIP-сигналізація часто передається без TLS (plain UDP), media (RTP) без SRTP, legacy BSS-системи з само-підписаними сертифікатами, відключена перевірка CN/SAN у старому обладнанні.

### 5.2 HTTP Security Headers

Браузерна безпека значно покращується правильними HTTP-заголовками відповіді:

**Strict-Transport-Security (HSTS):** `max-age=31536000; includeSubDomains; preload` — браузер завжди використовує HTTPS для цього домену. Preload list — вбудований у браузери.

**Content-Security-Policy (CSP):** найпотужніший захист від XSS. Визначає, з яких джерел браузер може завантажувати скрипти, стилі, зображення. `script-src 'self' 'nonce-{random}'` — дозволяє inline-скрипти тільки з відповідним nonce.

**X-Frame-Options:** `DENY` або `SAMEORIGIN` — захист від Clickjacking (вбудовування в iframe на evil.com).

**X-Content-Type-Options:** `nosniff` — браузер не "вгадує" MIME-тип, виконує тільки відповідно до Content-Type.

**Referrer-Policy:** `strict-origin-when-cross-origin` — URL сторінки не витікає у Referer при cross-origin запитах (захист чутливих URL).

**Permissions-Policy:** обмеження доступу до API браузера: `camera=(), microphone=(), geolocation=()`.

---

## 6. Безпека телекомунікаційних веб-застосунків

### 6.1 SIP — специфіка та атаки

SIP (RFC 3261) — текстовий протокол, синтаксично схожий на HTTP. Це означає, що веб-вразливості ін'єкцій, authn bypass транслюються у SIP-контекст.

**Session Border Controller (SBC)** — ключовий елемент захисту SIP в операторських мережах. Виконує функції:
- **Topology hiding** — приховує внутрішню IP-адресацію
- **Сигнальний firewall** — фільтрація SIP-методів та rate limiting
- **Transcoding** — конвертація медіа-кодеків
- **NAT traversal** — вирішення проблем з firewall/NAT
- **DoS захист** — блокування flooding атак

**Toll Fraud (телефонне шахрайство)** — несанкціоновані дзвінки на преміум-номери за рахунок скомпрометованого облікового запису SIP. Збитки для операторів складають мільярди доларів на рік глобально.

Механізм захисту: whitelist дозволених країн для дзвінків, ліміти на кількість одночасних дзвінків, anomaly detection (дзвінки вночі, в незвичні країни), SIPS (SIP over TLS) для захисту credentials при реєстрації.

**SRTP (Secure RTP):** шифрує медіа-потік VoIP. AES-128-CM для payload, HMAC-SHA1 для автентифікації. Ключовий обмін через SDES (в SDP) або DTLS-SRTP (WebRTC).

### 6.2 5G Service-Based Architecture (SBA)

5G SBA є революційною зміною: замість традиційних двоточкових інтерфейсів (point-to-point), мережеві функції (NF) публікують сервіси через REST API (HTTP/2 + JSON + OpenAPI).

**Ключові NF та їх API:**
- **AMF (Access and Mobility Management Function):** управляє реєстрацією UE, handover. API: Namf_Communication, Namf_EventExposure
- **SMF (Session Management Function):** встановлення/зміна/видалення PDU сесій. API: Nsmf_PDUSession
- **UDM (Unified Data Management):** зберігає subscription data абонентів. API: Nudm_SubscriberDataManagement, Nudm_UEAuthentication
- **NRF (NF Repository Function):** реєстр всіх NF — discovery та авторизація. OAuth 2.0 Authorization Server для NF-to-NF

**Модель безпеки SBA (3GPP TS 33.501):**
- TLS 1.2+ для всіх SBI (Service Based Interfaces) в межах одного оператора
- mTLS (Mutual TLS) для N32 інтерфейсу між різними операторами (roaming)
- OAuth 2.0 Client Credentials Flow для авторизації NF: NRF видає токени, NF перевіряють їх при кожному API-виклику
- PRINS (Protected Roaming Interconnect Network Signalling) — шифрування на рівні застосунку для roaming API

**Вразливості 5G SBI:**
- Ненадійна перевірка OAuth-токенів між NF (scope validation)
- HTTP/2-специфічні атаки: CONTINUATION Flood (CVE-2024-27316), HPACK Bomb (Header Compression DoS)
- Незахищений NRF — підроблення NF-реєстрацій, redirect трафіку на шкідливі NF

### 6.3 Telecom API та CAMARA

**NEF (Network Exposure Function)** виступає безпечним шлюзом між зовнішніми розробниками та мережею 5G. Захищає внутрішні NF від прямого зовнішнього доступу.

**CAMARA** (Convergent Apis for Mobile tERminAl) — проект Linux Foundation у партнерстві з GSMA для стандартизації телеком API. Мета: єдиний набір API, що працює у всіх операторів.

**SIM Swap API** — критично важливий для фінансової безпеки. Банки запитують у оператора: "чи не було замінено SIM-карту цього номера за останні N годин?" перед відправкою SMS OTP. Якщо SIM змінено → транзакцію блокувати → SIM-своп атака виявлена.

Атаки на SIM Swap API: якщо API доступний без належної авторизації, зловмисник може перевірити, чи вже відбувся своп (reconnaissance), або маніпулювати відповіддю.

**Безпека CAMARA API:**
- OAuth 2.0 + PKCE — обов'язковий стандарт
- Consent management — абонент дає явну згоду на використання його даних (location, sim status)
- Rate limiting per API, per subscriber, per application
- Audit logging — кожен API-виклик з ідентифікацією клієнта

### 6.4 SS7 — Legacy загрози

**SS7** залишається в операційному стані у більшості телеком-мереж через backward compatibility з 2G/3G, SMS interworking та roaming. Протокол розроблявся за принципом "всі в мережі — довірені партнери", без аутентифікації джерела.

**MAP (Mobile Application Part) — небезпечні операції:**
- `SendRoutingInfo` → отримання IMSI та поточного MSC абонента → точне місцезнаходження
- `ProvideSubscriberInfo` → location area, IMEI, subscriber state
- `RegisterSS` → реєстрація call forwarding без відома абонента → перенаправлення всіх дзвінків

**Реальні інциденти:** дослідники Positive Technologies у 2014-2023 рр. демонстрували перехоплення SMS OTP банків через SS7, відстеження місцезнаходження VIP-осіб без фізичного доступу до пристрою.

**GSMA FS.11 Categories:**
- Category 1: повне блокування специфічних MAP-операцій між несанкціонованими мережами
- Category 2: обмеження на контент та напрямок — блокувати запити від операторів де абонент не роумінгує
- Category 3: виявлення аномалій — ML-детекція незвичних патернів сигналізації

---

## 7. Mobile App Security (OWASP Mobile Top 10)

### 7.1 Зберігання даних та аутентифікація

**Android insecure storage:**
- `SharedPreferences` без encryption — зчитується будь-яким застосунком на jailbreakнутому пристрої
- Зовнішній storage (SD-карта) — доступний будь-якому застосунку на pre-Android 10
- SQLite без шифрування — легко отримується при отриманні доступу до backup або USB debugging
- Logcat leak — sensitive дані у логах (common у debug builds)

**Правильне зберігання на Android:**
- `Android Keystore System` — ключі зберігаються в TEE (Trusted Execution Environment) або SE (Secure Element), недоступні навіть root
- `EncryptedSharedPreferences` (Jetpack Security) — прозоре шифрування
- `SQLCipher` — зашифрована SQLite

**iOS Keychain:**
- Ізольований від застосунків захищений контейнер
- `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` — найбезпечніший атрибут
- `SecureEnclave` — апаратний захист для приватних ключів (FaceID, TouchID)
- `App Transport Security (ATS)` — вимагає HTTPS з TLS 1.2+ для всіх мережевих з'єднань

### 7.2 Certificate Pinning

**Certificate Pinning** — застосунок перевіряє не тільки ланцюжок CA, але й конкретний сертифікат або публічний ключ сервера.

Android (Network Security Config):
```xml
<network-security-config>
  <domain-config>
    <domain includeSubdomains="true">api.example.com</domain>
    <pin-set>
      <pin digest="SHA-256">base64encodedHash==</pin>
    </pin-set>
  </domain-config>
</network-security-config>
```

**Ризики Certificate Pinning:** якщо сертифікат змінюється (ротація) без оновлення застосунку — сервіс ламається. Рекомендація: pinning публічного ключа (не сертифіката), та мати backup pin.

**Обхід pinning (для тестувальників безпеки):** Frida + SSL Unpinning скрипти, Objection framework, Xposed Framework на root Android.

---

## 8. AWS API Gateway — архітектура та безпека

### 8.1 Типи та компоненти

**REST API** — найбільш функціональний тип: повна підтримка feature set включаючи кешування, трансформацію запитів/відповідей (Velocity Templates), request validation, usage plans.

**HTTP API** — легковаговий, нижча затримка (~60% дешевше): підтримує OAuth 2.0, JWT Authorizers, Lambda proxy. Не має кешування та деяких advanced features.

**WebSocket API** — для real-time двостороннього зв'язку: routing на основі message content, Lambda для кожного route.

### 8.2 Авторизація

**Lambda Authorizer (раніше Custom Authorizer):**
- Lambda-функція отримує токен з `Authorization` заголовку
- Перевіряє токен (JWT signature, expiry, claims, revocation list)
- Повертає IAM Policy Document (Allow/Deny)
- API GW кешує policy за токеном (TTL: 0-3600 сек)
- Гнучкий: підходить для будь-якого custom auth схеми

**Cognito Authorizer:**
- Пряма інтеграція з Cognito User Pool
- API GW сам перевіряє JWT signature та claims
- Не потребує Lambda функції — менша затримка та вартість
- Обмеження: тільки стандартний Cognito JWT, немає custom claim logic

**IAM Authorization:**
- Для AWS-to-AWS комунікації (сервіси в тому ж або іншому AWS account)
- SigV4 підпис запиту (AWS SDK робить це автоматично)
- Cross-account через Resource Policy

**mTLS для B2B:**
- Клієнт надає клієнтський сертифікат (mutual TLS handshake)
- API GW перевіряє клієнтський сертифікат проти truststore в S3
- Використовується для партнерських API між операторами

### 8.3 Throttling та захист від overload

Default limits: 10,000 req/sec account-level throttle, burst limit 5,000 req. Stage-level throttling — налаштовується окремо.

**Usage Plans + API Keys:** Usage Plan прив'язується до API Stage та набору API Keys. Кожен API Key отримує квоту (запитів/день або місяць) та throttle limit. Важливо: API Keys НЕ є механізмом аутентифікації — вони ідентифікують клієнта для billing/quota purposes, але не підтверджують identity.

---

## 9. AWS WAF — детальна архітектура

### 9.1 Web ACL та Rule Engine

**Web ACL (Access Control List)** — головна одиниця конфігурації WAF. Складається з упорядкованих правил. Кожен запит оцінюється по черзі. Дія за замовчуванням (Default Action) — Allow або Block.

**WCU (WAF Capacity Units)** — міра "складності" набору правил. Кожне правило споживає WCU. Ліміт Web ACL — 1500 WCU. Managed Rule Groups мають фіксований WCU.

**Приоритизація:** правила виконуються в порядку Priority (нижче число — вища пріоритет). Перше правило що match → дія (Allow/Block/Count/CAPTCHA). Наступні правила не виконуються.

### 9.2 Managed Rules — деталі

**AWSManagedRulesCommonRuleSet (Core Rule Set):** захист від XSS, LFI (Local File Inclusion), RFI (Remote File Inclusion), RCE (Remote Code Execution), Scanner/Probe. Найпопулярніший набір.

**AWSManagedRulesATPRuleSet (Account Takeover Prevention):** аналізує login endpoints на предмет credential stuffing та brute force. Додає custom headers з score. Вимагає налаштування login path.

**AWSManagedRulesBotControlRuleSet:** класифікує ботів: verified bots (Googlebot, Bingbot — Allow), unverified bots, scraper bots, browser automation (Selenium → Block). Додає label `awswaf:managed:aws:bot-control:bot:*`.

### 9.3 Logging та аналітика

WAF Logging → Kinesis Data Firehose → S3 (з компресією). Кожен лог-запис містить: timestamp, action, clientIP, country, uri, args, headers, matchingRules.

Amazon Athena для аналізу: SQL запити по WAF логах в S3. Приклад: топ 10 заблокованих IP за 24 години, частота спрацювання кожного правила, географія атак.

---

## 10. AWS Shield та CloudFront

### 10.1 Shield Advanced — деталі

**SRT (Shield Response Team):** команда AWS security engineers, доступна 24/7 при активному Shield Advanced. Допомагає налаштувати WAF правила під час DDoS атаки, аналізує трафік в реальному часі.

**Cost Protection:** якщо DDoS атака викликала auto-scaling (EC2, ELB, CloudFront) — AWS компенсує додаткові витрати на трафік та обчислення. Подається через AWS Support.

**Application Layer DDoS Protection (L7):** Shield Advanced автоматично визначає та блокує L7 DDoS атаки. Вимагає: Shield Advanced + AWS WAF + ALB/CloudFront. Логіка: порівнює поточний трафік з базовим рівнем (auto-learned), при аномалії — auto-mitigation.

### 10.2 CloudFront як Security Layer

**Origin Access Control (OAC)** замінив Origin Access Identity (OAI) — новий стандарт для обмеження прямого доступу до S3. Принцип: S3 Bucket Policy дозволяє читання тільки для конкретного CloudFront Distribution (через AWS Principal та Condition).

**Field-Level Encryption:** шифрує специфічні поля запиту (наприклад, номер кредитної картки) до того, як запит досягне origin. Навіть backend-розробники не бачать чутливих полів у plaintext.

**Lambda@Edge for Authentication:** перевірка JWT або session token на CDN-рівні, до того як запит досягне origin. Зменшує навантаження на backend та дозволяє відхиляти неавторизовані запити ближче до користувача.

---

## 11. DevSecOps Pipeline

### 11.1 Shift-Left Security

**Концепція Shift-Left** означає введення security-заходів якомога раніше в SDLC. Вартість виправлення вразливості зростає: Вимоги ($1) → Проектування ($5) → Розробка ($15) → QA ($30) → Продуктив ($150+). Раннє виявлення — виправлення до 150x дешевше.

**Threat Modeling (STRIDE):** у фазі проектування ідентифікуємо загрози:
- **S**poofing — підробка identity
- **T**ampering — модифікація даних
- **R**epudiation — відмова від дій
- **I**nformation Disclosure — витік даних
- **D**enial of Service — відмова в обслуговуванні
- **E**levation of Privilege — підвищення привілеїв

### 11.2 Security Gates у CI/CD

**SAST (Static Analysis):** аналіз вихідного коду без виконання:
- **Semgrep** — швидкий, гнучкий, правила на YAML. `semgrep --config=p/owasp-top-ten .` — аналіз за OWASP правилами
- **Amazon CodeGuru Security** — AWS-native SAST, інтегрується з CodePipeline
- **GitHub Advanced Security / CodeQL** — deep semantic аналіз

**DAST (Dynamic Analysis):** тестування запущеного застосунку:
- **OWASP ZAP** у Docker в CodeBuild: `docker run -t owasp/zap2docker-stable zap-api-scan.py -t https://staging.api.example.com/openapi.json`
- Автоматичний Spider + Active Scan → звіт у форматі JUnit → failBuild при Critical findings

**Dependency Scanning:**
- **Snyk** або **Dependabot** — перевірка package.json, requirements.txt, pom.xml на відомі CVE
- Блокування builds з CRITICAL вразливостями у залежностях
- Автоматичні PR для оновлення залежностей

**Image Scanning:**
- **Amazon Inspector V2** — сканує ECR образи та Lambda functions
- **Trivy** (open source) — сканування в CodeBuild перед push до ECR

### 11.3 Управління секретами у pipeline

Критичне правило: **жодних секретів у вихідному коді та змінних середовища CI/CD у plaintext**.

Правильний підхід:
1. Зберігати в **AWS Secrets Manager** або **Parameter Store (SecureString)**
2. IAM роль CodeBuild має доступ до конкретних секретів
3. Отримувати на початку build через AWS CLI або AWS SDK
4. Не логувати значення (mask в buildspec.yml)

Автоматичне виявлення секретів: **git-secrets** (pre-commit hook), **truffleHog**, **detect-secrets** — сканування git history на витік ключів.

---

## 12. Регуляторна відповідність

### 12.1 PCI DSS v4.0 — Requirement 6 (Application Security)

PCI DSS v4.0 (2022) значно посилив вимоги до безпеки веб-застосунків:

**Requirement 6.2:** Entity має задокументований процес управління всіма software components (Inventory). Включає 3rd-party libraries та dependencies.

**Requirement 6.3.2 (новий у v4.0):** Inventory всіх bespoke та custom software з щорічною реvaluation.

**Requirement 6.4.1:** Для публічно доступних веб-застосунків: або WAF (що активно блокує атаки), або automated technical solution для виявлення та захисту від атак — постійно.

**Requirement 6.4.2 (новий у v4.0):** WAF повинен бути в Prevention Mode (блокування), а не тільки Detection Mode.

**Requirement 11.3.1.1:** Automated vulnerability scanning щонайменше раз на 3 місяці та після кожної значної зміни. Internal та external scans.

**Requirement 11.4:** Penetration testing щонайменше раз на рік та після значних змін в infrastructure або додатку.

### 12.2 GDPR для веб-розробника

**Privacy by Design (Стаття 25):** захист даних з самого початку проектування, не як afterthought. Для веб: мінімізація даних (collect only what you need), шифрування PII, anonymization для аналітики.

**Right to Erasure (Стаття 17):** "Right to be forgotten" — технічна реалізація: API для видалення user account та всіх пов'язаних даних, включаючи backup та logs (у розумні строки). Проблема: distributed systems, event sourcing, immutable logs.

**Data Breach Notification (Стаття 33):** повідомлення наглядового органу протягом 72 годин після виявлення порушення. AWS CloudWatch Alarm + SNS → автоматичне сповіщення при виявленні витоку GuardDuty.

**Lawful Basis for Processing:** згода (Consent), договір (Contract), законне зобов'язання (Legal Obligation), законний інтерес (Legitimate Interest). Для cookies — explicit consent через cookie banner.

### 12.3 ISO/IEC 27034 — Application Security

**ISO 27034** визначає Application Security Management Process (ASMP):
- Organizational Normative Framework (ONF) — корпоративна база знань про безпеку застосунків
- Application Normative Framework (ANF) — специфічні security controls для кожного застосунку
- Application Security Controls Library — каталог перевірених control implementations
- Рівні безпеки від 1 (мінімальний) до 5 (максимальний) — відповідно до критичності застосунку

---

## 13. Архітектура Defence in Depth

### 13.1 Принципи безпечної веб-архітектури

**Principle of Least Privilege:** кожен компонент (AWS Lambda, ECS Task, RDS user) отримує мінімально необхідні права. IAM Role для Lambda — тільки конкретний S3 bucket та Secrets Manager secret, не `*`.

**Fail Secure:** при помилці — відмова з забороною, не дозволом. Якщо Lambda Authorizer повертає помилку — API GW повертає 403, не 200.

**Separation of Concerns:** frontend (CloudFront + S3), API Gateway (auth/throttling), backend (ECS/Lambda), database (RDS) — ізольовані компоненти з мінімальними inter-service правами.

**Zero Trust Architecture:** "Never trust, always verify". Кожен запит автентифікується та авторизується незалежно від мережевого розташування. Навіть запити між внутрішніми мікросервісами мають JWT або mTLS.

### 13.2 Практична архітектура для телеком-порталу

Компоненти захищеного телеком self-service порталу:
1. **Route 53** + GeoDNS (обмеження доступу за географією), Health Checks
2. **CloudFront** з TLS 1.3, HSTS preloading, WAF з OWASP CRS + Bot Control
3. **AWS Cognito** з MSISDN-based authentication (OTP через оператора), TOTP MFA
4. **ALB** з sticky sessions для stateful компонентів, access logs до S3
5. **ECS Fargate** — non-root containers, read-only filesystem, no-privilege escalation
6. **API Gateway** з Lambda Authorizer (перевірка Cognito JWT + business claims)
7. **Lambda** для BSS-інтеграції з VPC endpoint (без публічного інтернету)
8. **RDS Aurora** в приватній підмережі, encryption at rest (KMS), no public access
9. **Secrets Manager** для BSS credentials, автоматична ротація
10. **GuardDuty** + **Security Hub** + **CloudTrail** — повний audit trail

**Security monitoring:**
- CloudWatch Alarm → SNS → PagerDuty при критичних WAF спрацьовуваннях
- GuardDuty Finding → EventBridge → Lambda для автоматичного блокування підозрілих IP через WAF
- Security Hub → weekly report для security team

---

## 14. Кращі практики та чеклист

### 14.1 Чеклист для розробника

**Input Validation:**
- [ ] Всі вхідні дані валідуються на server-side (whitelist)
- [ ] Параметризовані запити для всіх БД-взаємодій
- [ ] Output encoding у контексті виводу (HTML, JS, URL, SQL)
- [ ] Обмеження розміру та типу завантажуваних файлів

**Authentication:**
- [ ] bcrypt/Argon2id для паролів (cost factor ≥ 12)
- [ ] MFA для привілейованих операцій
- [ ] Rate limiting + lockout на login endpoint
- [ ] Безпечне відновлення паролю (short-lived токени, не security questions)

**Authorization:**
- [ ] Перевірка прав на кожному backend endpoint
- [ ] Deny by default
- [ ] Непередбачувані ідентифікатори (UUID)
- [ ] Логування всіх access control failures

**Session Management:**
- [ ] HttpOnly, Secure, SameSite=Strict cookies
- [ ] Idle та absolute session timeout
- [ ] Server-side invalidation при logout

**Secrets:**
- [ ] Жодних секретів у git (git-secrets pre-commit hook)
- [ ] AWS Secrets Manager або Parameter Store SecureString
- [ ] Ротація секретів (автоматична або планова)

**Infrastructure:**
- [ ] HTTPS скрізь + HSTS
- [ ] Security Headers на всіх відповідях
- [ ] WAF у Prevention Mode
- [ ] Regular pen testing

### 14.2 Top-10 рекомендацій для телеком-оператора

1. **Замінити SMS OTP** на TOTP або FIDO2 для критичних операцій (SS7-вразливість SMS)
2. **SBC обов'язковий** для захисту SIP-інфраструктури, включаючи rate limiting та topology hiding
3. **SS7 Firewall** (категорії 1-3) на всіх точках підключення до SS7 мережі
4. **5G SBA:** обов'язковий TLS + OAuth 2.0 між всіма NF (3GPP TS 33.501)
5. **CAMARA API:** consent management + rate limiting + OAuth 2.0 + PKCE
6. **WAF перед OAM-порталами** — адміністративні інтерфейси критичні
7. **IMDSv2** обов'язковий на всіх EC2 інстансах (захист від SSRF)
8. **Network Segmentation:** BSS/OSS у приватних підмережах, доступ тільки через VPN або PrivateLink
9. **DevSecOps Pipeline** з SAST/DAST/Dependency scanning для всього ПЗ що виходить у prod
10. **Incident Response Plan** для веб-інцидентів: playbooks для SQL injection, DDoS, account takeover

---

## Висновки

Безпека веб-застосунків у 2024 році — це комплексна дисципліна, що поєднує класичні веб-вразливості (OWASP Top 10), специфіку API та мобільних платформ, а в телекомунікаційному контексті — унікальні протоколи SIP, SS7, 5G SBA.

Жоден окремий захід не є достатнім: WAF не замінює безпечний код, HTTPS не виправляє логіку авторизації, а firewall не захищає від L7 атак. Defence in Depth — єдина ефективна стратегія.

AWS надає повний стек інструментів: WAF, Shield, CloudFront, API Gateway, Cognito, GuardDuty — але їх правильна конфігурація та інтеграція вимагає розуміння природи загроз, що ця лекція і намагалась сформувати.

**Ключова думка:** безпека — це не продукт, що купується одноразово, а безперервний процес: тестування, моніторинг, реагування та покращення. Shift-Left у SDLC, DevSecOps культура та Security Champions у кожній команді — організаційний фундамент технічної безпеки.
