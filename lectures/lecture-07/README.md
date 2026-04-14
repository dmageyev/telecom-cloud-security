# Лекція 7 — Безпека веб- та телеком-застосунків

**Курс:** Інформаційна безпека телекомунікаційних та хмарних технологій  
**Модуль:** 2 — Захист сервісів, інциденти та відповідність  
**Аудиторія:** Магістри спеціальності «Кібербезпека»  
**Тривалість:** 2 академічні години (90 хв)  
**Кількість слайдів:** 48

---

## Мета лекції

Сформувати у студентів комплексне розуміння вразливостей та механізмів захисту веб-застосунків і телекомунікаційних протоколів — від класичних атак OWASP Top 10 до специфічних загроз протоколів SS7, SIP, GTP та 5G. Студенти навчаться застосовувати відповідні засоби захисту (WAF, API Gateway, AWS Shield, DevSecOps) у хмарному середовищі AWS.

---

## Цілі навчання

Після завершення лекції студент зможе:

1. Описати OWASP Top 10 (2021) та пояснити механізм і методи захисту для кожної вразливості
2. Виявляти та запобігати атакам ін'єкцій (SQL, NoSQL, LDAP, Command) із використанням параметризованих запитів
3. Налаштовувати захищену автентифікацію: OAuth 2.0, JWT, MFA, PKCE
4. Пояснювати атаки XSS, CSRF, SSRF, IDOR та застосовувати відповідні засоби захисту
5. Описати вразливості протоколів SS7 та Diameter (відстеження, перехоплення, DoS) і захист через Signaling Firewall та Edge Agent
6. Аналізувати атаки на SIP/VoIP та обґрунтовувати необхідність SBC, SRTP та SIPS
7. Пояснити ризики GTP та принципи захисту мобільного ядра мережі через GTP Firewall
8. Порівняти механізми безпеки 4G та 5G (SUCI, SEPP, SBA, Network Slicing, 5G-AKA, NRF)
9. Ідентифікувати типи телеком-шахрайства та механізми FMS-виявлення
10. Захищати Telecom API та REST/GraphQL інтерфейси; налаштовувати AWS API Gateway з Cognito та Lambda Authorizer
11. Налаштовувати AWS WAF, AWS Shield та Amazon CloudFront для захисту веб- та телеком-інфраструктури
12. Будувати DevSecOps-пайплайн із SAST, DAST, SCA для веб- та телеком-застосунків
13. Зіставляти вимоги NIS2, PCI DSS v4.0, GDPR, NIST, OWASP ASVS та GSMA FS.11 для веб- та телеком-систем

---

## Структура лекції (~48 слайдів)

### Частина 1: Безпека веб-застосунків (слайди 1–14, ~30%)

| # | Тема | Ключові концепції |
|---|------|-------------------|
| 1 | Титульний слайд | — |
| 2 | План лекції | Три розділи, 48 слайдів |
| 3 | Веб-застосунки — Огляд | Ландшафт загроз, стандарти OWASP/NIST/PCI DSS |
| 4 | OWASP Top 10 — огляд | A01–A10: CWE, ризики, пріоритизація |
| 5 | Ін'єкції (Injection) | SQL, NoSQL, LDAP, Command, ORM |
| 6 | Порушення автентифікації | OAuth 2.0, JWT, MFA, PKCE, session fixation |
| 7 | Міжсайтовий скриптинг (XSS) | Reflected, Stored, DOM-based, CSP |
| 8 | CSRF та Clickjacking | SameSite cookie, CSRF-token, X-Frame-Options |
| 9 | SSRF та IDOR | Метадані AWS, горизонтальна/вертикальна ескалація |
| 10 | HTTPS/TLS та безпека транспорту | TLS 1.3, HSTS, Certificate Pinning, mTLS |
| 11 | Безпека веб-API (REST та GraphQL) | Rate limiting, versioning, schema validation |
| 12 | WAF — Web Application Firewall | AWS WAF, правила, OWASP Core Rule Set |
| 13 | Кращі практики захисту веб-застосунків | Security Headers, CSP, Input Validation |
| 14 | Практичний кейс: захист веб-портала | AWS WAF + CloudFront + ALB |

### Частина 2: Безпека телеком-застосунків (слайди 15–39, ~55%)

| # | Тема | Ключові концепції |
|---|------|-------------------|
| 15 | Архітектура телеком-мережі та загрози | Core network, BSS/OSS, SS7, Diameter, SIP, GTP |
| 16 | Безпека протоколу SS7 | MAP-атаки, location tracking, IMSI catching |
| **17** | **SS7 — Захист та рекомендації** | **3-рівнева система, GSMA FS.11/FS.07, T-ISAC** |
| 18 | Diameter — вразливості та захист | AVP injection, roaming fraud, DEA, IPX filtering |
| 19 | VoIP та безпека SIP | INVITE flood, Registration hijacking, RTP |
| **20** | **SIP — Атаки та Session Border Controller** | **Зведена таблиця атак, SBC функції, RFC 3711/5630** |
| 21 | Безпека SMS та SMS-фішинг (SMishing) | SIM swap, SS7 SMS interception, A2P fraud |
| 22 | Безпека GTP та пакетного ядра | GTP-C/U атаки, PGW flooding, tunnel hijacking |
| 23 | Архітектура безпеки 5G | 5G-AKA, SEPP, NRF, SBA security |
| 24 | Атаки на 5G мережі | Fake base station, NAS spoofing, slicing threats |
| 25 | Telecom API та відкриті інтерфейси | CAMARA, Open Gateway, NEF, API abuse |
| 26 | Безпека мобільних застосунків | OWASP Mobile Top 10, RASP, certificate pinning |
| 27 | Захист MVNO та роумінгу | IPX fraud, anti-spoofing, roaming firewall |
| 28 | AWS API Gateway для телеком | Throttling, Authorizers, Usage Plans, WAF |
| 29 | Сигнальні фаєрволи та SMS Firewall | A2P filtering, keyword blocking, GSMA guidelines |
| **30** | **Telecom Fraud — типи та виявлення** | **IRSF, Wangiri, SIM Box, CDR pipeline, $38.95B** |
| 31 | Моніторинг аномалій у телеком | SIEM, ML-виявлення, CDR-аналіз, GuardDuty |
| 32 | Перехоплення трафіку та Lawful Interception | LI стандарти, шифри A5/x, ETSI TS |
| **33** | **IMSI Catcher та Eavesdropping** | **Stingray, методи перехоплення, 5G захист** |
| **34** | **DDoS-атаки на телеком-мережі** | **SIP/SS7/GTP Flood, amplification, scrubbing** |
| 35 | Криптографія у телеком-протоколах | IPsec, SRTP, TLS для SIP, ZRTP |
| 36 | Практичний кейс: захист BSS/OSS | AWS PrivateLink, VPC, IAM для telecom API |
| 37 | Regulatory & Compliance у телеком | GSMA FS.11, 3GPP TS 33.501, NIST SP 800-187 |
| 38 | IoT та M2M безпека у телеком | LwM2M, DTLS, AWS IoT Core, SIM-based auth |
| 39 | Атаки на абонентів: соціальна інженерія | Vishing, Smishing, SIM swap, OTP interception |

### Частина 3: Хмарний захист та DevSecOps (слайди 40–48, ~15%)

| # | Тема | Ключові концепції |
|---|------|-------------------|
| 40 | Zero Trust для телеком-інфраструктури | BeyondCorp, mTLS mesh, Network Slicing isolation |
| 41 | NFV та SDN — безпека | Віртуалізація NF, Service Mesh, Istio |
| 42 | Тестування безпеки | OWASP WSTG, GSMA SECAM, protocol fuzzing |
| 43 | AWS WAF, Shield та CloudFront | L3/L4/L7 захист, DDoS mitigation, CDN |
| 44 | DevSecOps Pipeline | SAST, DAST, SCA, shift-left, IaC security |
| 45 | Compliance: PCI DSS, GDPR, NIST, NIS2 | Mapping вимог до AWS-сервісів |
| 46 | Кращі практики та зведений чек-лист | Web + Telecom security checklist |
| 47 | Підсумок лекції | Ключові висновки, зв'язок з лекцією 8 |
| 48 | Зв'язки з курсом та Питання? | Посилання, цитата Шнайєра, запитання |

---

## Файли лекції

| Файл | Опис |
|------|------|
| [`presentation.html`](presentation.html) | Reveal.js презентація (~48 слайдів) |
| [`slides.md`](slides.md) | Слайди у форматі Marp Markdown |
| [`notes.md`](notes.md) | Детальний конспект лекції (Markdown) |
| [`notes.html`](notes.html) | Конспект лекції для перегляду в браузері |
| [`references.md`](references.md) | Список літератури та джерел |

---

## Як відкрити презентацію

```bash
# Локальний HTTP-сервер (Python)
cd lectures/lecture-07
python3 -m http.server 8080
# Відкрити: http://localhost:8080/presentation.html
```

Або відкрийте `presentation.html` безпосередньо у VS Code чи браузері.

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
