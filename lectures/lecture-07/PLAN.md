# PLAN.md — Лекція 7: Безпека веб-застосунків
## Призначення цього файлу

Цей документ призначений для ШІ-агентів, які продовжуватимуть роботу над презентацією.
У ньому зафіксовано: повний план слайдів, стильові конвенції, логіку структури та
детальний зміст кожного блоку, який ще не реалізований.

---

## Загальні відомості

| Параметр | Значення |
|----------|----------|
| Файл презентації | `lectures/lecture-07/presentation_cloud.html` |
| Мова | Українська |
| Кількість слайдів | **38** |
| Бібліотека | Reveal.js (CDN jsDelivr) |
| Тема | `black` |
| Аудиторія | Магістри спеціальності «Кібербезпека» |
| Курс | Інформаційна безпека хмарних технологій |
| Попередня лекція | Лекція 6 — Безпека обчислювальних сервісів і контейнерів |
| Наступна лекція | Лекція 8 — Моніторинг, логування та виявлення загроз |

---

## Стильові конвенції (ОБОВ'ЯЗКОВО дотримуватись)

### HTML-шапка (ідентична лекціям 5 та 6)

```html
<!doctype html>
<html lang="uk">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Презентація: Безпека веб-застосунків</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js/dist/reveal.css">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js/dist/theme/black.css">
    <script src="https://cdn.jsdelivr.net/npm/reveal.js/dist/reveal.js"></script>
```

### CSS (копіювати точно, не змінювати)

```css
.reveal h1 { font-size: 1.45em; }
.reveal h2 { font-size: 1.18em; color: #58a6ff; }
.reveal h3 { font-size: 0.9em; color: #e8b04b; margin: 0.2em 0 0.15em; }
.reveal ul, .reveal ol { text-align: left; }
.reveal li { margin-bottom: 0.1em; font-size: 1.0em; }
.reveal p { margin: 0.18em 0; font-size: 1.0em; }
.two-col { display: flex; gap: 1em; }
.two-col > div { flex: 1; min-width: 0; }
.highlight { color: #e8b04b; font-weight: bold; }
.tag { background: #1f6feb; border-radius: 4px; padding: 1px 5px; font-size: 0.75em; margin: 1px; display: inline-block; }
.tag-green { background: #238636; border-radius: 4px; padding: 1px 5px; font-size: 0.75em; margin: 1px; display: inline-block; }
.tag-red { background: #da3633; border-radius: 4px; padding: 1px 5px; font-size: 0.75em; margin: 1px; display: inline-block; }
.box { border: 1px solid #444; border-radius: 5px; padding: 0.2em 0.6em; margin: 0.15em 0; }
table { font-size: 0.7em; border-collapse: collapse; width: 100%; }
th { background: #1f6feb; padding: 3px 5px; }
td { padding: 3px 5px; border-bottom: 1px solid #444; }
code { background: #161b22; padding: 1px 4px; border-radius: 4px; font-size: 0.88em; color: #58a6ff; }
pre code { display: block; padding: 0.4em; font-size: 0.8em; color: #c9d1d9; line-height: 1.25; white-space: pre; }
.reveal .slides section {
    overflow-y: auto !important;
    max-height: 100% !important;
    box-sizing: border-box !important;
}
```

### Reveal.js initialize (точний конфіг)

```javascript
Reveal.initialize({
    hash: true,
    transition: 'slide',
    slideNumber: 'c/t',
    center: false,
    width: 1650,
    height: 900,
    margin: 0.04,
    minScale: 0.2,
    maxScale: 1.0,
    scrollActivationWidth: 700
});
```

### Коментарі до слайдів (формат)

```html
<!-- ===== СЛАЙД N: Назва слайду ===== -->
```

### Кольорова палітра

| Елемент | Колір |
|---------|-------|
| h2 заголовки | `#58a6ff` (синій) |
| h3 заголовки | `#e8b04b` (помаранчевий/золотий) |
| `.highlight` | `#e8b04b` |
| `.tag` (нейтральний) | `#1f6feb` (синій) |
| `.tag-green` (позитивний) | `#238636` (зелений) |
| `.tag-red` (небезпека) | `#da3633` (червоний) |
| `.box` border-color (нейтральний) | `#444` |
| `.box` border-color (позитивний) | `#238636` |
| `.box` border-color (небезпека) | `#da3633` |
| `.box` border-color (інформаційний) | `#1f6feb` |
| Примітки | `color:#888` |

### Типові компоненти

- **`.two-col`** — двоколонкова розбивка (основний патерн для контентних слайдів)
- **`.box`** — рамочний блок з кольоровою рамкою (border-color)
- **`.tag`/`.tag-green`/`.tag-red`** — inline мітки-чіпи
- **`<table>`** — таблиці для порівняльного контенту
- **`<pre><code>`** — блоки коду/команд
- **`<code>`** — inline-код у тексті
- **Емодзі** — активно використовуються в h3 для візуалізації: 🔒 🛡️ ⚠️ ✅ 🎯 📊 🔑 🌐 📱 ☁️ 🚀

---

## Повний план слайдів (38 слайдів)

### ЧАСТИНА 1: Безпека веб-застосунків (Слайди 1–22) — РЕАЛІЗОВАНО

| # | Назва | Статус |
|---|-------|--------|
| 1 | Титульний слайд | ✅ Готово |
| 2 | План лекції | ✅ Готово |
| 3 | Вступ: Безпека веб-застосунків — ландшафт загроз | ✅ Готово |
| 4 | OWASP Top 10 (2021) — огляд | ✅ Готово |
| 5 | A01: Broken Access Control та IDOR | ✅ Готово |
| 6 | A03: Ін'єкції (SQL, Command, LDAP) | ✅ Готово |
| 7 | A02: Cryptographic Failures | ✅ Готово |
| 8 | A07: XSS — Cross-Site Scripting | ✅ Готово |
| 9 | CSRF — Cross-Site Request Forgery | ✅ Готово |
| 10 | A10: SSRF — Server-Side Request Forgery | ✅ Готово |
| 11 | Аутентифікація та управління сесіями | ✅ Готово |
| 12 | OWASP API Security Top 10 | ✅ Готово |
| 13 | HTTPS та TLS — транспортна безпека | ✅ Готово |
| 14 | WAF — Web Application Firewall | ✅ Готово |
| 15 | HTTP Security Headers та CSP | ✅ Готово |
| 16 | OAuth 2.0, JWT, OpenID Connect | ✅ Готово |
| 17 | Secure SDLC та Shift-Left Security | ✅ Готово |
| 18 | Інструменти тестування безпеки веб | ✅ Готово |
| 19 | Моніторинг та виявлення атак | ✅ Готово |
| 20 | DDoS-атаки та захист веб-рівня | ✅ Готово |
| 21 | Кращі практики захисту веб-застосунків | ✅ Готово |
| 22 | Архітектура захищеного веб-застосунку | ✅ Готово |

---

### ЧАСТИНА 2: Безпека телекомунікаційних веб-застосунків (Слайди 23–32) — ПОТРІБНО РЕАЛІЗУВАТИ

| # | Назва | Детальний зміст |
|---|-------|-----------------|
| 23 | Телеком веб-застосунки — специфіка | Відмінності телеком-контексту від корпоративного веб: висока доступність (99.999%), реальний час, legacy-протоколи, регуляторні вимоги (ETSI, 3GPP). Поверхня атаки: веб-портали самообслуговування, OAM-інтерфейси, BSS/OSS, NFV/SDN веб-управління, API між операторами. Таблиця з унікальними ризиками телеком-веб. |
| 24 | VoIP/SIP протоколи та вразливості | SIP (Session Initiation Protocol): архітектура — UA, Proxy, Registrar, B2BUA. HTTP-подібна природа SIP → ті ж вразливості (Injection, Auth bypass). Специфічні атаки: SIP Flooding, Registration Hijacking, Toll Fraud, Eavesdropping. Таблиця: метод SIP → можлива атака. Дві колонки: архітектура + вразливості. |
| 25 | SIP-атаки та механізми захисту | Детально: Toll Fraud (несанкціоновані дзвінки), SIP Scanning (пошук уразливих endpoints), BYE Injection, REGISTER hijacking. Захист: SIP-over-TLS (SIPS), SRTP для медіа, SIP-специфічні WAF-правила, fail2ban для SIP, rate limiting за IP/user, Digest Authentication з nonce. Блок коду: приклад SIP REGISTER запиту та атаки. |
| 26 | 5G веб-інтерфейси та RESTful API | 5G Service-Based Architecture (SBA): всі NF (Network Functions) взаємодіють через HTTP/2 + JSON. SBI (Service-Based Interface): AMF, SMF, UDM, PCF, NRF — всі мають REST API. Вразливості: незахищені SBI-інтерфейси, відсутність mTLS між NF, надмірні права OAUTH token. Захист: 3GPP TS 33.501 — обов'язковий TLS, OAuth 2.0, NRF для виявлення сервісів. |
| 27 | Telecom API — NEF, CAMARA, Open Gateway | NEF (Network Exposure Function) — шлюз між 3rd-party і мережею 5G. CAMARA Project (Linux Foundation + GSMA): стандартизовані API для телеком-функцій (Quality on Demand, SIM Swap, Number Verification). GSMA Open Gateway: монетизація мережевих API. Безпека CAMARA API: OAuth 2.0 + PKCE, rate limiting, API keys, захист від зловживань. Таблиця: API → можлива атака → захист. |
| 28 | Legacy телеком: SS7/Diameter вразливості | SS7 (Signaling System 7): протокол 1970-х, нема вбудованої автентифікації. Атаки: Location tracking (Send Routing Info), SMS interception (intercept OTP), Call forwarding fraud. Diameter (4G): теж вразливий — Diameter firewall обов'язковий. GTP (GPRS Tunneling Protocol) — атаки через GTP-C. Заходи: SS7 firewall (категорії 1–3), GSMA FS.11, моніторинг аномалій, поступовий перехід на 5G. |
| 29 | Mobile Apps безпека — OWASP Mobile Top 10 | OWASP Mobile Top 10 (2024): M1-M10. Особливості мобільних застосунків: зберігання даних на пристрої, сертифікати, бінарний захист. Дві колонки: Android-специфіка vs iOS-специфіка. Таблиця: M1 Improper Credential Usage → приклад → захист. Ключові інструменти: MobSF, Frida, objection, apktool. |
| 30 | Android та iOS — захист застосунків | Android: APK signing, ProGuard/R8 obfuscation, SafetyNet/Play Integrity API, Network Security Config (certificate pinning), Android Keystore для ключів. iOS: App Transport Security (ATS), Secure Enclave, Keychain, code signing, runtime protection. Блок коду: приклад Network Security Config (Android) та Info.plist ATS (iOS). Спільні практики: certificate pinning, jailbreak/root detection, biometric auth. |
| 31 | AWS API Gateway — архітектура та захист | AWS API Gateway: REST API vs HTTP API vs WebSocket. Компоненти: Edge-optimized / Regional / Private. Інтеграція з Lambda, EC2, Step Functions. Безпека: Resource Policies, Usage Plans (throttling/quota), API Keys (не для автентифікації!), Lambda Authorizer, Cognito Authorizer. Схема: клієнт → CloudFront → API GW → Lambda/Backend. Таблиця: тип автентифікації → коли використовувати. |
| 32 | AWS API Gateway — захист, моніторинг | Детально: Lambda Authorizer (custom JWT validation), Cognito User Pools (OAuth 2.0 flows), mTLS для B2B-інтеграцій. AWS WAF інтеграція з API Gateway. Throttling: default 10000 req/s, burst 5000. CloudWatch метрики: 4XXError, 5XXError, Latency, Count. X-Ray tracing для API-ланцюжків. Блок коду: Terraform для API Gateway + WAF association. |

---

### ЧАСТИНА 3: Cloud/DevSecOps для веб-застосунків (Слайди 33–37) — ПОТРІБНО РЕАЛІЗУВАТИ

| # | Назва | Детальний зміст |
|---|-------|-----------------|
| 33 | AWS WAF — архітектура та Managed Rules | AWS WAF v2: Web ACL → Rule Groups → Rules. Типи правил: IP set, Geo match, Rate-based, Regex pattern. AWS Managed Rules: Core Rule Set (CRS), Known Bad Inputs, SQL injection, Linux/POSIX, Bot Control, Fraud Control. Асоціація з: CloudFront, ALB, API Gateway, Cognito. Блок коду: Terraform для Web ACL з AWS Managed Rules та rate limiting. Метрики CloudWatch. |
| 34 | AWS Shield — DDoS-захист | Shield Standard (безкоштовно): автоматичний захист L3/L4 для всіх AWS. Shield Advanced (платний): SRT (Shield Response Team), cost protection, application-layer (L7) DDoS, advanced metrics. Порівняльна таблиця Standard vs Advanced. Інтеграція: Shield Advanced + AWS WAF + CloudFront + Route 53. Типи DDoS атак та яке правило AWS захищає. Архітектурна схема ешелонованого DDoS-захисту. |
| 35 | Amazon CloudFront — CDN та безпека | CloudFront: глобальна CDN + перший рівень захисту. Безпека: HTTPS-only (enforce), TLS 1.2+, Custom SSL сертифікати (ACM). Origin Access Control (OAC) — обмеження прямого доступу до S3. Geo-restriction. Field-level encryption. CloudFront Functions та Lambda@Edge для auth на рівні CDN. Таблиця: функція → переваги для безпеки. Блок коду: Terraform для CloudFront distribution з WAF + OAC. |
| 36 | DevSecOps Pipeline для веб в AWS | AWS CodePipeline + CodeBuild для Secure SDLC. Стадії: Source (CodeCommit/GitHub) → SAST (SonarQube/Semgrep) → Build (CodeBuild) → DAST (OWASP ZAP) → Deploy (CodeDeploy/ECS) → Production monitoring. AWS Inspector для сканування образів в pipeline. Amazon CodeGuru Reviewer для code review. Secrets Manager в pipeline (без хардкодингу). Terraform модуль pipeline. Таблиця: стадія → інструмент → що перевіряє. |
| 37 | Відповідність регуляторним вимогам | PCI DSS v4.0 для веб: Req 6 (Secure Systems), Req 8 (Strong Auth), Req 10 (Logging), Req 11 (Pen Testing). GDPR: захист персональних даних, right to erasure в веб-застосунках. ISO/IEC 27034 — Application Security. NIST SP 800-53 (AC, IA, SI controls). AWS Services для Compliance: Security Hub (standards), AWS Config (drift detection), Macie (PII), Audit Manager (evidence collection). Таблиця: стандарт → ключові вимоги до веб → AWS-інструмент. |

---

### СЛАЙД 38: Підсумок + Питання (ПОТРІБНО РЕАЛІЗУВАТИ)

Слайд 38 — підсумковий:
- Ліва колонка: три розділи лекції з ключовими концепціями (теги)
- Права колонка: "Золоте правило" (box з border-color:#238636) + список рекомендованих ресурсів
- Нижній рядок: кількість слайдів, зв'язок із Лекцією 8
- Закінчується: "Дякую за увагу! Питання?"

---

## Важливі технічні деталі для кожного розділу

### Розділ 2: Телеком (слайди 23–32)

**SIP (слайди 24-25)**
- SIP методи: INVITE, REGISTER, BYE, CANCEL, ACK, OPTIONS, SUBSCRIBE, NOTIFY
- SIP Flooding → захист: SBC (Session Border Controller), rate limiting
- Registration Hijacking → захист: SIP Digest Auth + TLS
- Toll Fraud → захист: whitelist країн, ліміти дзвінків, аномалій-детекція
- SRTCP для захисту медіа-потоків
- SBC (Session Border Controller) — головний елемент захисту SIP

**5G SBA (слайд 26)**
- NF список: AMF (доступ), SMF (сесія), UDM (дані користувача), PCF (політики), NRF (реєстр), NEF (виставлення), AUSF (автентифікація), UPF (рівень користувача)
- Всі NF взаємодіють через Nxx інтерфейси (HTTPS/2 + JSON)
- OAuth 2.0 client credentials flow для NF-to-NF
- 3GPP TS 33.501 — специфікація безпеки 5G

**CAMARA (слайд 27)**
- API категорії: Quality on Demand, SIM Swap, Number Verification, Device Location, Network Slicing
- SIM Swap API — критичний для безпеки (захист від SIM-свопінгу)
- Стандарт автентифікації: OIDC + OAuth 2.0 PKCE

**SS7/Diameter (слайд 28)**
- MAP (Mobile Application Part) — команди для пошуку абонента
- SendRoutingInfo → витік location
- GSMA FS.11 "SS7 and SIGTRAN Network Security"
- Категорії SS7 захисту: Cat 1 (блокування), Cat 2 (обмеження), Cat 3 (аномалії)

### Розділ 3: Cloud/DevSecOps (слайди 33–37)

**AWS WAF (слайд 33)**
- Вартість: $5/місяць за Web ACL + $1/мільйон запитів + правила
- WCU (WAF Capacity Units) — обмеження складності правил
- Logging: до S3 через Kinesis Data Firehose
- Bot Control: crawler, scraper, bot-score
- Fraud Control: account takeover, account creation

**CloudFront (слайд 35)**
- Origin Access Control (OAC) замінив Origin Access Identity (OAI) — новий стандарт
- Price Class: 100, 200, All
- Behaviors: default vs specific path patterns
- Signed URLs / Signed Cookies — для захищеного контенту

**DevSecOps Pipeline (слайд 36)**
- AWS CodePipeline безкоштовний для v2 (тільки оплата за дії)
- CodeBuild: buildspec.yml — місце для SAST-команд
- OWASP ZAP може запускатись у Docker в CodeBuild
- Amazon Inspector V2 — сканує Lambda + ECR

---

## Стан файлів

| Файл | Статус | Нотатки |
|------|--------|---------|
| `presentation_cloud.html` | ✅ Створено | Слайди 1–38 додані, структура документа завершена |
| `README_cloud.md` | ✅ Створено | Повністю готовий |
| `references_cloud.md` | ✅ Створено | Повністю готовий |
| `notes_cloud.md` | ✅ Створено | Академічний конспект ~9 000 слів |
| `notes.html` | 🟢 Залишилось створити | HTML-версія `notes_cloud.md` (standalone, без Reveal.js) |

---

## Джерела для references.md

### OWASP
- OWASP Top 10 2021: https://owasp.org/Top10/
- OWASP API Security Top 10: https://owasp.org/www-project-api-security/
- OWASP Mobile Top 10: https://owasp.org/www-project-mobile-top-10/
- OWASP Testing Guide v4.2: https://owasp.org/www-project-web-security-testing-guide/
- OWASP Cheat Sheet Series: https://cheatsheetseries.owasp.org/

### AWS
- AWS WAF Developer Guide: https://docs.aws.amazon.com/waf/latest/developerguide/
- AWS Shield: https://docs.aws.amazon.com/waf/latest/developerguide/shield-chapter.html
- Amazon CloudFront Security: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/security.html
- AWS API Gateway Security: https://docs.aws.amazon.com/apigateway/latest/developerguide/security.html
- AWS Security Best Practices for APIs: https://aws.amazon.com/blogs/security/

### Телеком стандарти
- 3GPP TS 33.501 (5G Security): https://www.3gpp.org/ftp/Specs/archive/33_series/33.501/
- GSMA FS.11 (SS7 Security): https://www.gsma.com/solutions-and-impact/technologies/security/
- RFC 3261 (SIP): https://tools.ietf.org/html/rfc3261
- ETSI TS 133 210 (Network Domain Security): https://www.etsi.org/
- CAMARA Project: https://camaraproject.org/

### Книги та стандарти
- "The Web Application Hacker's Handbook" — Stuttard, Pinto
- "Real-World Bug Hunting" — Yaworski
- NIST SP 800-53 Rev 5: https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final
- PCI DSS v4.0: https://www.pcisecuritystandards.org/

---

## Ключові повідомлення лекції (для підсумкового слайду)

1. **Web App Security**: OWASP Top 10 — основа, validation на всіх рівнях, WAF ≠ замінник безпечного коду
2. **API Security**: кожен API — потенційна поверхня атаки; authn + authz + rate limiting обов'язкові
3. **Telecom Web**: SIP, 5G SBA, SS7 — унікальні ризики, яких немає в корпоративному веб
4. **AWS захист**: WAF + Shield + CloudFront + API GW = ешелонована оборона для веб
5. **DevSecOps**: безпека вбудовується в pipeline, не додається після деплою

---

## Примітки щодо подальшої роботи

- `presentation_cloud.html` у поточному стані вже **повністю зібраний**: слайди 1–38 додані, структура документа завершена, HTML-теги коректно закриті.
- Інструкції про те, що файл містить лише слайди 1–22, має незакриті теги або потребує ручного додавання слайдів 23–38, **більше неактуальні**.
- Якщо презентацію буде змінено надалі, цей план слід оновлювати синхронно зі станом `presentation_cloud.html`, щоб уникати хибних вказівок для наступних авторів або агентів.
