# Лекція 7 — Безпека веб-застосунків

**Курс:** Інформаційна безпека хмарних технологій  
**Модуль:** 2 — Захист сервісів, інциденти та відповідність  
**Кількість слайдів:** 39

---

## Опис лекції

Лекція охоплює комплексний підхід до безпеки веб-застосунків у контексті хмарних і телекомунікаційних технологій. Від класичних веб-вразливостей (OWASP Top 10) до специфічних загроз телекомунікаційного середовища (SIP, 5G SBA, SS7) та інструментів захисту на платформі AWS (WAF, Shield, CloudFront, API Gateway).

---

## Цілі навчання

Після завершення лекції студент повинен вміти:

1. Ідентифікувати та описати вразливості OWASP Web Top 10 та API Security Top 10
2. Пояснити механізми атак SQL Injection, XSS, CSRF, SSRF, IDOR та методи захисту
3. Налаштовувати безпечну аутентифікацію з OAuth 2.0, JWT та MFA
4. Описати специфічні вразливості телекомунікаційних протоколів SIP, SS7, Diameter
5. Пояснити архітектуру безпеки 5G Service-Based Architecture (SBA)
6. Знати специфіку OWASP Mobile Top 10 та захист мобільних застосунків
7. Налаштовувати AWS WAF (Managed Rules, Rate-based rules, Custom rules)
8. Описати захист від DDoS атак через AWS Shield та CloudFront
9. Побудувати DevSecOps pipeline з SAST/DAST для веб-застосунків
10. Застосовувати вимоги PCI DSS v4.0, GDPR та ISO 27034 до веб-розробки
11. Проектувати архітектуру Defence in Depth для захищеного веб-застосунку

---

## Структура лекції

| # | Тема | Слайд |
|---|------|-------|
| — | Вступ і план лекції | 1–2 |
| **Частина 1: Безпека веб-застосунків** | | |
| 1 | Ландшафт загроз та поверхня атаки | 3 |
| 2 | OWASP Top 10 (2021) — огляд | 4 |
| 3 | A01: Broken Access Control / IDOR | 5 |
| 4 | A03: SQL Injection, Command Injection | 6 |
| 5 | A02: Cryptographic Failures | 7 |
| 6 | A07: XSS — Cross-Site Scripting | 8 |
| 7 | CSRF — Cross-Site Request Forgery | 9 |
| 8 | A10: SSRF — Server-Side Request Forgery | 10 |
| 9 | Аутентифікація та управління сесіями | 11 |
| 10 | OWASP API Security Top 10 | 12 |
| 11 | HTTPS та TLS — транспортна безпека | 13 |
| 12 | WAF — Web Application Firewall | 14 |
| 13 | HTTP Security Headers та CSP | 15 |
| 14 | OAuth 2.0, JWT, OpenID Connect | 16 |
| 15 | Secure SDLC та Shift-Left Security | 17 |
| 16 | Інструменти тестування безпеки | 18 |
| 17 | Моніторинг та виявлення атак | 19 |
| 18 | DDoS-атаки та захист веб-рівня | 20 |
| 19 | Кращі практики захисту веб-застосунків | 21 |
| 20 | Архітектура захищеного веб-застосунку | 22 |
| **Частина 2: Телеком веб-застосунки** | | |
| 21 | Телеком веб — специфіка та поверхня атаки | 23 |
| 22 | VoIP/SIP — протоколи та вразливості | 24 |
| 23 | 5G SBA та безпека NF-інтерфейсів | 25 |
| 24 | Telecom API: NEF, CAMARA, Open Gateway | 26 |
| 25 | SS7/Diameter — legacy вразливості | 27 |
| 26 | OWASP Mobile Top 10 | 28 |
| 27 | AWS API Gateway — архітектура та захист | 29 |
| **Частина 3: Cloud/DevSecOps** | | |
| 28 | AWS WAF — Managed Rules та архітектура | 30 |
| 29 | AWS Shield — DDoS-захист | 31 |
| 30 | Amazon CloudFront — CDN та безпека | 32 |
| 31 | DevSecOps Pipeline для веб в AWS | 33 |
| 32 | Регуляторна відповідність (PCI DSS, GDPR) | 34 |
| 33 | Практичний кейс: телеком-портал в AWS | 35 |
| 34 | Типові помилки та антипатерни | 36 |
| 35 | AWS Security Services — зведена таблиця | 37 |
| 36 | Підсумок лекції | 38 |
| 37 | Анонс Лекції 8 + Дякую + Питання | 39 |

---

## Файли лекції

| Файл | Опис |
|------|------|
| [`presentation_cloud.html`](presentation_cloud.html) | Презентація Reveal.js (39 слайдів, кнопка «Конспект» 📄) |
| [`notes_cloud.md`](notes_cloud.md) | Детальний конспект лекції (Markdown, 40 000–60 000 знаків) |
| [`references_cloud.md`](references_cloud.md) | Список рекомендованої літератури та джерел |
| [`PLAN.md`](PLAN.md) | Технічний план для ШІ-агентів (конвенції, слайд-план) |

---

## Перегляд презентації

Відкрийте файл `presentation_cloud.html` у браузері. Навігація:

- **→ / Space** — наступний слайд
- **← / Backspace** — попередній слайд
- **📄** (кнопка праворуч внизу) — відкрити/закрити панель конспекту поточного слайду
- **F** — повноекранний режим
- **S** — режим доповідача (speaker notes)
- **Esc** — огляд усіх слайдів (overview)
- **Scroll** — прокрутка вмісту слайду, якщо він не вміщується на екрані

---

## Пов'язані AWS Academy модулі

- AWS Academy Cloud Security Foundations — Domain 6: Application Security
- AWS Well-Architected Framework — Security Pillar
- AWS re:Invent: "Building Secure Web Applications on AWS" (SEC403)

---

## Попередня / Наступна лекція

← [Лекція 6 — Безпека обчислювальних сервісів і контейнерів](../lecture-06/presentation.html)  
→ Лекція 8 — Моніторинг, логування та виявлення загроз *(в розробці)*
