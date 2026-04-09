# Лекція 7: Безпека веб- та телеком-застосунків

**Курс:** Інформаційна безпека телекомунікаційних та хмарних технологій  
**Модуль:** 2 — Захист сервісів, інциденти та відповідність  
**Аудиторія:** Магістри спеціальності «Кібербезпека»  
**Тривалість:** 2 академічні години (90 хв)

---

## Мета лекції

Сформувати у студентів комплексне розуміння вразливостей та механізмів захисту веб-застосунків і телекомунікаційних протоколів — від класичних атак OWASP Top 10 до специфічних загроз протоколів SS7, SIP, GTP та 5G. Студенти навчаться застосовувати відповідні засоби захисту (WAF, API Gateway, AWS Shield, DevSecOps) у хмарному середовищі AWS.

---

## Цілі навчання

Після завершення лекції студент зможе:

- Описати OWASP Top 10 та пояснити механізм кожної вразливості
- Виявляти та запобігати атакам ін'єкцій (SQL, NoSQL, LDAP, Command)
- Налаштовувати захищену автентифікацію: OAuth 2.0, JWT, MFA, PKCE
- Пояснювати атаки XSS, CSRF, SSRF, IDOR та способи їх нейтралізації
- Описати специфічні вразливості протоколів SS7, Diameter, SIP, GTP та SMS
- Розуміти архітектуру безпеки 5G та механізми захисту (5G-AKA, SEPP, NRF)
- Захищати Telecom API та REST/GraphQL інтерфейси
- Налаштовувати AWS WAF, AWS Shield, Amazon CloudFront та API Gateway
- Будувати DevSecOps-пайплайн для веб- та телеком-застосунків
- Застосовувати вимоги PCI DSS, GDPR та NIST для веб- та телеком-систем

---

## Структура лекції (38 слайдів)

### Частина 1: Безпека веб-застосунків (слайди 1–13, ~30%)

| # | Тема | Ключові концепції |
|---|------|-------------------|
| 1 | Титульний слайд | — |
| 2 | План лекції | Три розділи, 38 слайдів |
| 3 | OWASP Top 10 — огляд | A01–A10: CWE, ризики, пріоритизація |
| 4 | Ін'єкції (Injection) | SQL, NoSQL, LDAP, Command, ORM |
| 5 | Порушення автентифікації | OAuth 2.0, JWT, MFA, PKCE, session fixation |
| 6 | Міжсайтовий скриптинг (XSS) | Reflected, Stored, DOM-based, CSP |
| 7 | CSRF та Clickjacking | SameSite cookie, CSRF-token, X-Frame-Options |
| 8 | SSRF та IDOR | Метадані AWS, Horizontal/Vertical privilege escalation |
| 9 | HTTPS/TLS та безпека транспорту | TLS 1.3, HSTS, Certificate Pinning, mTLS |
| 10 | Безпека веб-API (REST та GraphQL) | Rate limiting, versioning, schema validation |
| 11 | WAF — Web Application Firewall | AWS WAF, правила, OWASP Core Rule Set |
| 12 | Кращі практики захисту веб-застосунків | Security Headers, CSP, Input Validation |
| 13 | Практичний кейс: захист веб-портала | AWS WAF + CloudFront + ALB |

### Частина 2: Безпека телеком-застосунків (слайди 14–33, ~55%)

| # | Тема | Ключові концепції |
|---|------|-------------------|
| 14 | Архітектура телеком-мережі та загрози | Core network, BSS/OSS, SS7, Diameter, SIP, GTP |
| 15 | Безпека протоколу SS7 | MAP-атаки, location tracking, IMSI catching |
| 16 | Diameter — вразливості та захист | AVP injection, roaming fraud, IPX filtering |
| 17 | VoIP та безпека SIP | INVITE flood, Registration hijacking, RTP |
| 18 | Безпека SMS та SMS-фішинг (SMishing) | SIM swap, SS7 SMS interception, A2P fraud |
| 19 | Безпека GTP та пакетного ядра | GTP-C/U атаки, PGW flooding, tunnel hijacking |
| 20 | Архітектура безпеки 5G | 5G-AKA, SEPP, NRF, SBA security |
| 21 | Атаки на 5G мережі | Fake base station, NAS spoofing, slicing threats |
| 22 | Telecom API та відкриті інтерфейси | CAMARA, Open Gateway, NEF, API abuse |
| 23 | Безпека мобільних застосунків | OWASP Mobile Top 10, RASP, certificate pinning |
| 24 | Захист MVNO та роумінгу | IPX fraud, anti-spoofing, roaming firewall |
| 25 | AWS API Gateway для телеком | Throttling, Authorizers, Usage Plans, WAF |
| 26 | Сигнальні фаєрволи та SMS Firewall | A2P filtering, keyword blocking, GSMA guidelines |
| 27 | Моніторинг аномалій у телеком | SIEM, ML-виявлення, CDR-аналіз, GuardDuty |
| 28 | Криптографія у телеком-протоколах | IPsec, SRTP, TLS для SIP, ZRTP |
| 29 | Практичний кейс: захист BSS/OSS | AWS PrivateLink, VPC, IAM для telecom API |
| 30 | Regulatory & Compliance у телеком | GSMA FS.11, 3GPP TS 33.501, NIST SP 800-187 |
| 31 | IoT та M2M безпека у телеком | LwM2M, DTLS, AWS IoT Core, SIM-based auth |
| 32 | Атаки на абонентів: соціальна інженерія | Vishing, Smishing, SIM swap, OTP interception |
| 33 | Zero Trust для телеком-інфраструктури | BeyondCorp, mTLS mesh, Network Slicing isolation |

### Частина 3: Хмарний захист та DevSecOps (слайди 34–38, ~15%)

| # | Тема | Ключові концепції |
|---|------|-------------------|
| 34 | AWS WAF, Shield та CloudFront | L3/L4/L7 захист, DDoS mitigation |
| 35 | DevSecOps для веб- та телеком-систем | SAST, DAST, SCA, shift-left, IaC security |
| 36 | Compliance: PCI DSS, GDPR, NIST | Mapping вимог до AWS-сервісів |
| 37 | Кращі практики та чек-лист безпеки | Web + Telecom security checklist |
| 38 | Підсумок та питання | Ключові висновки, зв'язок з лекцією 8 |

---

## Файли

| Файл | Опис |
|------|------|
| [`7_presentation.html`](./7_presentation.html) | Reveal.js презентація (38 слайдів) |
| [`7_README.md`](./7_README.md) | Цей файл — опис та навігація по лекції |

---

## Як відкрити презентацію

```bash
# Локальний HTTP-сервер (Python)
cd lectures/lecture-07
python3 -m http.server 8080
# Відкрити: http://localhost:8080/7_presentation.html
```

Або відкрийте `7_presentation.html` безпосередньо в браузері.

**Навігація в презентації:**
- `→` або `Space` — наступний слайд
- `←` — попередній слайд
- `F` — повноекранний режим
- `S` — режим доповідача
- `Esc` або `O` — огляд усіх слайдів

---

## Передумови

Студенти мають бути знайомі з:
- Базовими концепціями AWS (IAM, VPC, S3) — Лекції 1–4
- Принципами захисту даних та криптографією — Лекція 5
- Безпекою обчислювальних сервісів та контейнерів — Лекція 6

---

## Зв'язки з іншими темами

- **← Лекція 6:** Безпека обчислювальних сервісів і контейнерів (EC2, EKS, Lambda, WAF)
- **→ Лекція 8:** Моніторинг, логування та виявлення загроз (GuardDuty, CloudTrail, SIEM)
