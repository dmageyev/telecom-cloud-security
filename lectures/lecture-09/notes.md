# Конспект лекції 9: Захист від DDoS та забезпечення доступності

**Курс:** Інформаційна безпека телекомунікаційних та хмарних технологій  
**Модуль 2 — Лекція 9**

---

## 1. DDoS — визначення, масштаб загрози та статистика

**DoS (Denial of Service)** — атака, що має на меті зробити ресурс недоступним, використовуючи один або кілька джерел.

**DDoS (Distributed Denial of Service)** — скоординована атака з тисяч або мільйонів вузлів одночасно, що ускладнює блокування за IP-адресою чи підмережею.

**Статистика DDoS 2024 (Cloudflare, NETSCOUT):**

| Показник | Значення |
|----------|----------|
| Глобальних DDoS-атак/рік | ~15 мільйонів |
| Рекордна атака | 5.6 Tbps (Cloudflare, 2024) |
| Рекорд HTTP-запитів | 71 мільйон RPS (Cloudflare, 2023) |
| Середня тривалість | ~30 хвилин |
| Вартість простою/годину | $300K–$1M (Gartner) |
| Частка телеком/ISP серед цілей | 35% (NETSCOUT) |

**Тенденції 2024:**
- Зростання атак Layer 7 (+65%)
- Скорочення тривалості (short burst attacks)
- Збільшення частоти повторних атак проти одного ресурсу

**Мотиви зловмисників:**
- **Ransom DDoS** — вимагання: "заплати або ми продовжимо"
- **Конкурентна боротьба** — опустити конкурента в пік сезону
- **Хактивізм** — протест чи ідеологічні мотиви
- **Diversion** — відволікти SOC, поки відбувається реальна атака
- **Геополітика** — державні актори проти критичної інфраструктури

> **Telecom як ціль №1:** атака на оператора = атака проти тисяч клієнтів одночасно, оскільки оператор є транзитним вузлом для всього трафіку.

---

## 2. Класифікація DDoS-атак за моделлю OSI

Усі DDoS-атаки поділяються на три великі категорії залежно від цільового рівня моделі OSI:

| Категорія | Рівень OSI | Ціль атаки | Вимірюється | Приклади |
|-----------|------------|------------|-------------|---------|
| **Об'ємні (Volumetric)** | L3/L4 | Переповнення канала | Gbps / Tbps | UDP Flood, ICMP Flood, Amplification |
| **Протокольні (Protocol)** | L3/L4 | Виснаження ресурсів TCP-стека | PPS (пакети/с) | SYN Flood, ACK Flood, Fragmented Packets |
| **Прикладні (Application)** | L7 | Виснаження вебсервера або бази даних | RPS (запити/с) | HTTP Flood, Slowloris, ReDoS, Cache-bypass |

### Об'ємні атаки
Мета — заповнити канал зв'язку жертви трафіком, щоб легітимні пакети не могли потрапити до сервера.
- **UDP Flood** — масові UDP-пакети на довільні порти; сервер витрачає ресурси на відповіді ICMP Unreachable
- **ICMP Flood (Smurf)** — масові ping-запити, в т.ч. broadcast amplification
- **Amplification** — зловмисник використовує сторонні відкриті сервери для багаторазового підсилення трафіку (детально у розділі 4)

### Протокольні атаки
Мета — вичерпати системні ресурси (таблиці з'єднань, пам'ять, CPU) сервера або мережевого обладнання.
- **SYN Flood** — надсилання SYN-пакетів без завершення TCP handshake (без ACK). Сервер зберігає "напіввідкриті" з'єднання в таблиці до таймауту. При масовому потоці таблиця переповнюється, нові з'єднання відхиляються
- **ACK Flood** — масові ACK-пакети для завантаження TCP-стека
- **Fragmented Packets** — IP-фрагменти, які не збираються у повний пакет, витрачаючи буфери

### Прикладні атаки (Layer 7)
Найскладніший клас — трафік виглядає як легітимний, тому важко відрізнити від реальних користувачів.
- **HTTP GET/POST Flood** — масові HTTP-запити до вебсервера
- **Slowloris** — відкриває з'єднання і надсилає заголовки дуже повільно (детально у розділі 6)
- **ReDoS** — складні регулярні вирази у вхідних даних блокують CPU
- **Cache-bypass flood** — унікальні URL (`?v=random`) для обходу CDN-кешу та навантаження origin-сервера

---

## 3. Amplification-атаки — механізм підсилення

**Принцип Amplification:**
1. Зловмисник підробляє src IP (spoofing) — вказує IP жертви замість свого
2. Надсилає малий запит до відкритого публічного сервера (DNS, NTP, Memcached)
3. Сервер відповідає великою відповіддю на адресу жертви
4. Жертва отримує у BAF разів більше трафіку, ніж атакуючий надіслав

**BAF (Bandwidth Amplification Factor)** = розмір відповіді / розмір запиту

| Протокол | BAF | Порт | Механізм |
|---------|-----|------|---------|
| DNS | ×28–54 | 53/UDP | ANY query → велика відповідь |
| NTP | ×556 | 123/UDP | monlist команда → перелік клієнтів |
| Memcached | ×51,200 | 11211/UDP | get key → дамп кешу |
| SSDP | ×30 | 1900/UDP | UPnP discovery |
| CLDAP | ×70 | 389/UDP | LDAP search |

**Memcached 2018:** атака 1.7 Tbps проти GitHub — найбільша на той момент. Один Memcached-сервер надіслав у 51,200 разів більше трафіку, ніж запит атакуючого.

**Захист від Amplification:**
- **BCP38 (Network Ingress Filtering)** — ISP/оператор не пропускає пакети з підробленими src IP
- **Response Rate Limiting (RRL)** — DNS-сервери обмежують кількість відповідей одному клієнту
- Закрити UDP-сервіси (Memcached, NTP monlist) від зовнішнього доступу
- AWS Shield Advanced автоматично виявляє та блокує amplification-атаки

---

## 4. Ботнети та DDoS-as-a-Service

### Архітектура ботнету
**Ботнет** — мережа заражених комп'ютерів (ботів/зомбі), яка централізовано керується зловмисником.

- **Bot Herder** — оператор ботнету, управляє через C2-сервер
- **Bots (Zombies)** — заражені пристрої: ПК, сервери, IoT-пристрої (камери, роутери)
- **C2 (Command & Control)** — канал керування: IRC, HTTP, P2P, Tor, Domain Generation Algorithm (DGA)
- **Target** — жертва атаки

### Відомі ботнети

| Ботнет | Розмір | Особливість |
|--------|--------|-------------|
| Mirai (2016) | 600,000 IoT | Перший масовий IoT-ботнет; зупинив DynDNS, паралізував частину Інтернету |
| Mēris (2021) | 250,000 роутерів | 21.8M RPS проти Яндекс |
| HTTP/2 Rapid Reset | Мало ботів | 398M RPS (2023), CVE-2023-44487 |
| NoName057(16) | ~50K | Геополітичний, цілі: Україна/ЄС |

### HTTP/2 Rapid Reset (CVE-2023-44487)
Критична вразливість у механізмі скасування потоків HTTP/2. Зловмисник відкриває тисячі потоків і одразу їх скасовує через RST_STREAM, змушуючи сервер обробляти кожен. Кілька ботів генерували 398 мільйонів запитів/с. AWS, Google, Cloudflare одночасно виявили та залатали вразливість.

### DDoS-as-a-Service (Booter/Stresser)
- Нелегальні сервіси у darknet від **$5–$200/годину**
- "Легальні" IP Stresser — позиціонуються як інструменти тестування навантаження
- Технічні знання не потрібні: вибрати ціль, оплатити криптовалютою, натиснути кнопку
- Доступні потужності — до 1 Tbps "на замовлення"

### IoT як вектор DDoS
Мільярди незахищених IoT-пристроїв з дефолтними паролями Telnet/SSH стали сировиною для Mirai-подібних ботнетів. AWS IoT Device Defender допомагає операторам IoT-платформ виявляти аномалії в поведінці пристроїв.

---

## 5. Layer 7 атаки — Slowloris та HTTP Flood

### Slowloris Attack
Slowloris — атака, що відкриває якомога більше з'єднань до цільового вебсервера та тримає їх відкритими якомога довше, надсилаючи часткові HTTP-заголовки.

**Механізм:**
```
GET / HTTP/1.1
Host: target.com
X-Custom-Header-1: ...
(пауза 10-15 сек, не надсилаємо завершальний \r\n\r\n)
```

- Сервер чекає завершення заголовка → пул з'єднань вичерпується
- Для атаки потрібен один звичайний комп'ютер, без ботнету
- Особливо ефективний проти Apache (thread-per-connection model)
- nginx стійкіший (event-driven model)

**Захист від Slowloris:**
- `RequestReadTimeout` в Apache/nginx — закривати повільні з'єднання за таймаутом
- Обмеження `LimitRequestFields` та `LimitRequestFieldSize`
- AWS WAF: Connection timeout rules, rate limiting на кількість з'єднань з одного IP

### HTTP Flood
- **Simple HTTP flood** — масові GET/POST запити на одну сторінку
- **Cache-bypass flood** — унікальні URL (`?nocache=RANDOM`) щоб обійти CDN та потрапити до origin
- **POST body flood** — масові POST з великим body (форми, завантаження файлів, API-виклики)

**Ознаки L7 атаки (виявлення):**

| Ознака | Характеристика |
|--------|---------------|
| User-Agent | Однотипний або відсутній |
| Referer | Відсутній або підробний |
| HTTP версія | HTTP/1.0 замість 1.1/2 |
| Accept-Language | Однаковий для всіх запитів |
| Cookie | Відсутній (не браузер) |
| TLS fingerprint (JA3) | Повторюється масово |

---

## 6. DDoS у телекомунікаційних мережах

Телекомунікаційна галузь — особлива: оператори мають власні специфічні протоколи (SIP, SS7, Diameter), для яких стандартні WAF та Shield не дають повного захисту.

### SIP/VoIP DDoS
- **INVITE Flood** — масові SIP INVITE → виснаження ресурсів Session Border Controller (SBC)
- **REGISTER Flood** — масові реєстрації абонентів → перевантаження реєстратора
- **OPTIONS Flood** — ping-like атака на SIP-сервери для розвідки
- **BYE Flood** — примусове розривання активних дзвінків

**Захист SIP:** SBC rate-limiting, fail2ban (аналіз логів + автоблокування IP), ACL за IP/ASN

### SS7 DoS-атаки
- **MAP Cancel Location** — деавторизація абонента в мережі, абонент стає недоступним
- **MAP Insert Subscriber Data** — підміна профілю абонента
- **SCCP flooding** — перевантаження Signal Transfer Point (STP)
- Наслідок: абонент фізично недоступний для дзвінків та SMS

**Захист SS7:** SS7 Firewall (категорії 1–3 за GSMA FS.11), фільтрація за Category (Category 1 — базова, Category 3 — повна).

### Специфіка захисту телеком-оператора

| Рівень | Загроза | Захист |
|--------|---------|--------|
| BGP/IP | Volumetric DDoS | RTBH, Flowspec, Scrubbing center |
| DNS | DNS amplification | RRL, Anycast DNS |
| SIP/VoIP | INVITE flood | SBC rate limit, fail2ban |
| SS7/Diameter | MAP flooding | SS7 Firewall (cat.1-3) |
| 5G HTTP/2 | Rapid Reset | Patch + WAF L7 |
| Абоненти | Ботнет-трафік | Anomaly detection, BCP38 |

**RTBH (Remotely Triggered Black Hole):** оператор через BGP community повідомляє upstream про блокування IP жертви. Ефективно при об'ємних атаках, але жертва також тимчасово недоступна.

---

## 7. Стратегії захисту від DDoS — Defense in Depth

Ефективний захист будується шарами, де кожен шар обробляє той клас атак, для якого він найбільш придатний:

1. **ISP/Upstream** — RTBH, Flowspec, scrubbing на рівні магістральних каналів
2. **CDN/Anycast** — розподіл трафіку по PoP, поглинання об'єму (>1 Tbps ємності)
3. **DNS layer** — DNS failover, geo-routing, rate limit на запити
4. **Network perimeter** — BGP communities, SYN cookies, rate limiting
5. **Load Balancer** — connection limits, SYN proxy, idle timeout
6. **WAF** — Layer 7 фільтрація, challenge (CAPTCHA/JS challenge)
7. **Application** — caching, queue-based processing, circuit breaker

| Техніка | Рівень | Ефективність |
|---------|--------|-------------|
| Anycast routing | L3 | Об'ємні атаки |
| SYN cookies | L4 | SYN flood |
| Rate limiting (per-IP) | L4/L7 | Flood атаки |
| IP reputation lists | L3/L7 | Відомі ботнети |
| CAPTCHA/JS challenge | L7 | Bot HTTP flood |
| Geo-blocking | L3/L7 | Атаки з певних регіонів |
| Behavioral analysis | L7 | Sophisticated bots |

---

## 8. AWS Shield — захист від DDoS

### AWS Shield Standard
- **Безкоштовно** для всіх AWS-клієнтів, вмикається автоматично
- Захист від найпоширеніших **Layer 3/4** атак: SYN Flood, UDP Flood, Reflection attacks
- Захищає: **CloudFront, Route 53, ELB, Global Accelerator**
- Нульова настройка — вмикається автоматично для захищених ресурсів

### AWS Shield Advanced
- **$3,000/місяць** + data transfer fees (або Enterprise Support plan)
- Захист Layer 3/4 **та Layer 7** (у поєднанні з WAF)
- **DDoS Response Team (DRT)** — AWS-інженери доступні 24/7 під час атаки
- **Cost Protection** — компенсація витрат (EC2, CloudFront, Route 53, ELB) під час атаки
- **Advanced diagnostics** — детальна телеметрія в CloudWatch (DDoS Attack Vectors dashboard)
- **Proactive Engagement** — DRT сам ескалює при виявленні великих атак

```hcl
# Terraform: Shield Advanced для ALB
resource "aws_shield_protection" "alb" {
  name         = "ALB-DDoS-Protection"
  resource_arn = aws_lb.main.arn
}

# Proactive Engagement
resource "aws_shield_proactive_engagement" "main" {
  enabled = true
  emergency_contact {
    email        = "soc@company.com"
    phone        = "+380501234567"
  }
}
```

| Функція | Shield Standard | Shield Advanced |
|---------|----------------|-----------------|
| L3/L4 захист | ✅ | ✅ |
| L7 захист (WAF) | ❌ | ✅ (безкоштовний WAF) |
| Threat visibility | ❌ | ✅ CloudWatch metrics |
| DRT підтримка | ❌ | ✅ 24/7 |
| Cost protection | ❌ | ✅ credits |
| SLA | Best effort | Guaranteed mitigation |

---

## 9. AWS WAF — захист на рівні застосунку

**AWS WAF (Web Application Firewall)** — Layer 7 фільтр для HTTP/HTTPS трафіку, який розгортається перед: CloudFront, ALB, API Gateway, AppSync.

**Web ACL (Access Control List)** — набір впорядкованих правил, кожне з яких перевіряє умову та виконує дію:
- `ALLOW` — пропустити запит
- `BLOCK` — заблокувати (повернути 403)
- `COUNT` — порахувати, але не блокувати (режим аудиту)
- `CAPTCHA` — показати CAPTCHA-виклик
- `CHALLENGE` — показати JavaScript challenge

### AWS Managed Rule Groups

| Група | Захищає від |
|-------|-------------|
| AWSManagedRulesCommonRuleSet | OWASP Top 10 загроз |
| AWSManagedRulesSQLiRuleSet | SQL injection |
| AWSManagedRulesKnownBadInputs | Log4j, ShellShock та інші |
| AWSManagedRulesAmazonIpReputation | Відомі шкідливі IP |
| AWSManagedRulesBotControlRuleSet | Bot detection (common + targeted mode) |

### Rate Limiting в AWS WAF

```hcl
resource "aws_wafv2_web_acl" "main" {
  name  = "main-waf"
  scope = "CLOUDFRONT"
  default_action { allow {} }

  rule {
    name     = "RateLimitPerIP"
    priority = 1
    action   { block {} }
    statement {
      rate_based_statement {
        limit              = 2000  # 2000 запитів за 5 хвилин з одного IP
        aggregate_key_type = "IP"
      }
    }
    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "RateLimit"
      sampled_requests_enabled   = true
    }
  }
}
```

### WAF Logging та аналіз
WAF логи → **Kinesis Firehose → S3** → аналіз в Athena або OpenSearch.

```sql
-- Athena: топ правил за 24 години
SELECT terminatingruleid, count(*) as blocked_requests
FROM waf_logs
WHERE action = 'BLOCK'
  AND timestamp > to_unixtime(current_timestamp - interval '1' day) * 1000
GROUP BY terminatingruleid
ORDER BY blocked_requests DESC
LIMIT 20;
```

### Bot Control
- **Common mode** — сигнатурне виявлення ботів
- **Targeted mode** — advanced ML + browser challenge (перевіряє, чи є JavaScript-підтримка)
- Автоматичний дозвіл для SEO-ботів (Googlebot, Bingbot) — перевіряється ASN

---

## 10. Концепції доступності — RTO, RPO, SLA

### Ключові метрики

| Метрика | Визначення | Приклад |
|---------|------------|---------|
| **RTO** (Recovery Time Objective) | Максимально допустимий час відновлення сервісу | ≤4 годин |
| **RPO** (Recovery Point Objective) | Максимально допустима втрата даних у часі | ≤1 годину |
| **MTTR** (Mean Time to Recover) | Середній час відновлення після відмови | 45 хвилин |
| **MTBF** (Mean Time Between Failures) | Середній час між відмовами | 720 годин |

### SLA рівні доступності

| Uptime | Downtime/рік | Downtime/місяць |
|--------|-------------|----------------|
| 99% | 3.65 дні | 7.3 год |
| 99.9% ("три дев'ятки") | 8.76 год | 43.8 хв |
| 99.99% ("чотири дев'ятки") | 52.6 хв | 4.4 хв |
| 99.999% ("п'ять дев'яток") | 5.26 хв | 26 сек |

> **Telecom:** регулятор вимагає 99.999% для голосових послуг (ліцензійні умови).

### Стратегії відновлення (AWS Well-Architected)

| Стратегія | RTO / RPO | Вартість |
|-----------|-----------|---------|
| **Backup & Restore** | Години/Дні | $ |
| **Pilot Light** | 10–30 хв | $$ |
| **Warm Standby** | Хвилини | $$$ |
| **Multi-site Active-Active** | Секунди / ~0 | $$$$ |

---

## 11. AWS CloudFront — CDN та захист на периметрі

**CloudFront** — глобальна мережа доставки контенту (CDN) з понад **450 PoP** по всьому світу.

**Роль у захисті від DDoS:**
- **Anycast routing** — трафік іде до найближчого PoP, розподіляючи навантаження
- **Поглинання об'єму** — ємність мережі CloudFront >1 Tbps
- **Origin Shield** — додатковий шар кешування, зменшує навантаження на origin
- **Shield Standard вбудований** безкоштовно
- **Приховує реальний IP origin** — зловмисник не може атакувати origin напряму
- **HTTPS termination** на PoP → розвантаження TLS від origin

**Захист origin-сервера:** Security Group/NACL повинна дозволяти трафік **виключно від CloudFront** (через `aws_ec2_managed_prefix_list` для CloudFront).

```hcl
# Origin Shield + WAF + HTTPS-only
resource "aws_cloudfront_distribution" "main" {
  enabled    = true
  web_acl_id = aws_wafv2_web_acl.main.arn

  origin {
    domain_name = aws_lb.main.dns_name
    origin_id   = "alb-origin"
    origin_shield {
      enabled              = true
      origin_shield_region = "eu-west-1"
    }
    custom_origin_config {
      http_port              = 80
      https_port             = 443
      origin_protocol_policy = "https-only"
    }
  }

  viewer_certificate {
    acm_certificate_arn      = aws_acm_certificate.main.arn
    ssl_support_method       = "sni-only"
    minimum_protocol_version = "TLSv1.2_2021"
  }
}
```

---

## 12. Route 53 — DNS failover та висока доступність

**Route 53** — глобальна Anycast DNS-служба AWS з **100% SLA**.

**Типи маршрутизації для HA:**
- **Failover routing** — автоматичний перехід на резервний endpoint за результатами health check
- **Latency routing** — маршрут до найближчого (за затримкою) регіону
- **Geolocation routing** — маршрут залежно від місцезнаходження користувача
- **Weighted routing** — розподіл трафіку між endpoints у заданих пропорціях (A/B тестування, поступовий rollout)

**Health Checks:** перевіряють endpoint кожні 10–30 сек; після 3 послідовних невдалих перевірок — failover.

```hcl
resource "aws_route53_health_check" "primary" {
  fqdn              = "app.example.com"
  port              = 443
  type              = "HTTPS"
  resource_path     = "/health"
  failure_threshold = 3
  request_interval  = 10
}

resource "aws_route53_record" "primary" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "app.example.com"
  type    = "A"
  failover_routing_policy { type = "PRIMARY" }
  alias {
    name                   = aws_lb.primary.dns_name
    zone_id                = aws_lb.primary.zone_id
    evaluate_target_health = true
  }
  health_check_id = aws_route53_health_check.primary.id
  set_identifier  = "primary"
}
```

**DNSSEC в Route 53:** підпис зон KMS-ключем захищає від DNS hijacking та cache poisoning.

---

## 13. Elastic Load Balancing — типи та захист від DDoS

| Тип | Рівень | Використання | DDoS захист |
|-----|--------|-------------|-------------|
| **ALB** (Application) | L7 HTTP/HTTPS | Веб, мікросервіси | WAF integration, rate limiting |
| **NLB** (Network) | L4 TCP/UDP | High performance, IoT | SYN cookies, Shield |
| **GLB** (Gateway) | L3/L4 | Security appliances | Traffic inspection |

### ALB вбудований захист
- **Connection rate limiting** — обмеження нових з'єднань за секунду
- **HTTP Desync mitigation** — захист від HTTP Request Smuggling (режим `strictest`)
- **HTTP/2 support** — ефективніше використання з'єднань
- **Idle timeout** — автоматичне закриття повільних з'єднань (Slowloris)
- **Drop invalid HTTP headers** — відкидання некоректних заголовків

```hcl
resource "aws_lb" "main" {
  name               = "main-alb"
  load_balancer_type = "application"
  enable_http2                = true
  desync_mitigation_mode      = "strictest"
  drop_invalid_header_fields  = true
  enable_deletion_protection  = true
}
```

---

## 14. Auto Scaling — еластичність під DDoS-навантаженням

**Auto Scaling Group** дозволяє горизонтально масштабуватися під час атаки — absorb тактика. Але масштабування при DDoS може збільшити витрати.

**Правильна стратегія:**
1. WAF блокує основну частину атаки на рівні CloudFront/ALB
2. Auto Scaling масштабується лише для легітимного трафіку
3. `MaxCapacity` — жорсткий ліміт для контролю витрат

> **"Bill Attack":** зловмисник може використати Auto Scaling для підвищення AWS-рахунку. Захист: `MaxCapacity` + AWS Budget Alerts.

**Типи Scaling Policies:**
- **Target Tracking** — тримати CPU/RPS на заданому рівні
- **Step Scaling** — різні кроки при різних рівнях метрики
- **Predictive Scaling** — ML-модель передбачає пікове навантаження

```hcl
resource "aws_autoscaling_group" "web" {
  min_size         = 2
  max_size         = 20  # жорсткий ліміт
  desired_capacity = 4
  health_check_type = "ELB"
}

resource "aws_autoscaling_policy" "target_cpu" {
  policy_type = "TargetTrackingScaling"
  target_tracking_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ASGAverageCPUUtilization"
    }
    target_value = 70.0
  }
}
```

---

## 15. Multi-AZ архітектура

**AZ (Availability Zone)** — фізично відокремлений дата-центр в регіоні AWS. Один регіон = 3–6 AZ з незалежним живленням, мережею та охолодженням.

**Правило 2+ AZ:** ніколи не розміщувати всі ресурси в одній AZ.

### Multi-AZ для баз даних

| Сервіс | Multi-AZ | Час failover |
|--------|---------|------------|
| RDS | Синхронна реплікація | ~60–120 сек |
| Aurora | 6 копій у 3 AZ | <30 сек |
| ElastiCache | Replica в 2 AZ | Автоматично |
| EFS | Вбудовано | N/A |
| S3 | ≥3 AZ за замовчуванням | N/A |

```hcl
resource "aws_db_instance" "main" {
  engine            = "postgres"
  instance_class    = "db.r6g.large"
  multi_az          = true        # синхронна реплікація
  storage_encrypted = true
  backup_retention_period = 7
  deletion_protection     = true
}
```

---

## 16. Multi-Region — глобальна висока доступність

**Multi-Region** потрібен коли:
- RTO < 1 годину, RPO ≈ 0 (active-active)
- Законодавчі вимоги (data residency)
- Глобальна аудиторія — зниження latency
- Захист від регіональних DDoS-атак
- Критична інфраструктура (телеком, банки)

### Компоненти Multi-Region Active-Active

| Компонент | Реплікація | AWS Сервіс |
|-----------|-----------|-----------|
| DNS | Latency routing | Route 53 |
| CDN | Global PoP | CloudFront |
| Database | Global Tables | DynamoDB Global |
| Database | Global Cluster | Aurora Global |
| Storage | Cross-region replication | S3 CRR |

**Aurora Global Database:** запис у primary region, до 5 читальних регіонів. Реплікація <1 сек. При відмові primary — promote secondary за <1 хвилину (RPO ~0, RTO ~1 хв).

### AWS Global Accelerator
- **2 статичні Anycast IP-адреси** для всього світу
- Трафік маршрутизується через AWS backbone (швидше за публічний Інтернет)
- Health checks + автоматичний failover між регіонами
- Вбудований Shield Advanced

| Характеристика | CloudFront | Global Accelerator |
|---------------|------------|-------------------|
| Протоколи | HTTP/HTTPS | TCP/UDP/HTTP |
| Кешування | ✅ Так | ❌ Ні |
| IP-адреси | Динамічні | Статичні (2 Anycast) |
| Failover | DNS-based (~60s) | Миттєво (<30s) |
| Використання | Веб/API/CDN | Gaming, IoT, real-time |

---

## 17. Caching стратегії — зниження навантаження на origin

Кешування є одним із найефективніших засобів захисту origin-сервера під час DDoS — якщо 95%+ трафіку обслуговується кешем, до origin доходить лише 5%.

| Шар | Сервіс | TTL | Захист від DDoS |
|-----|--------|-----|----------------|
| DNS | Route 53 | 300 сек | DNS amplification захист |
| CDN | CloudFront | Хвилини–дні | Origin shield, absorb burst |
| In-memory | ElastiCache Redis | Секунди–год | DB offload під атакою |
| App cache | Application-level | Секунди | Reduce DB queries |

**Cache-bypass атака:** зловмисники використовують унікальні параметри URL (`?nocache=random`) щоб обійти CDN та навантажити origin.  
**Захист:** WAF нормалізує URL та видаляє зайві query-параметри, зберігаючи тільки відомі (`lang`, `version`).

---

## 18. Rate Limiting та Circuit Breaker

### Rate Limiting — рівні
- **Network level** — pps/bps ліміти (NLB, Shield)
- **WAF level** — req/5min per IP (AWS WAF Rate Rule)
- **API Gateway** — req/sec per API key або per account
- **Application level** — Redis token bucket / leaky bucket

### API Gateway Throttling
```hcl
resource "aws_api_gateway_stage" "prod" {
  default_route_settings {
    throttling_burst_limit = 5000   # max concurrent
    throttling_rate_limit  = 1000   # req/sec
  }
  route_settings {
    route_key              = "POST /payments"
    throttling_burst_limit = 100
    throttling_rate_limit  = 50
  }
}
```

### Circuit Breaker патерн
**CLOSED** → нормальна робота → **OPEN** (після N помилок) → **HALF-OPEN** (через таймаут, тестовий запит) → **CLOSED**

Захищає downstream-сервіси від **каскадних відмов** — якщо сервіс A перевантажений, Circuit Breaker запобігає перевантаженню сервісів B, C, що залежать від A.

| Сервіс | Механізм |
|--------|---------|
| ALB | Connection draining + health checks |
| App Mesh | Outlier detection (Envoy proxy) |
| ECS Service Connect | Retry + circuit breaker вбудовано |
| Lambda | Concurrency limits = circuit breaker |

---

## 19. Захист DNS-інфраструктури

**DNS — ціль та інструмент DDoS:**
- **DNS DDoS (ціль):** перевантажити DNS-сервер → домен недоступний
- **DNS Amplification (інструмент):** відкриті резолвери → ×28–54 підсилення
- **DNS Hijacking:** підміна відповідей для редиректу трафіку
- **NXDOMAIN flood:** масові запити неіснуючих доменів → виснаження ресурсів резолвера

**Захист DNS:**
- **Anycast DNS** (Route 53) — розподілена відмовостійкість
- **DNSSEC** — підпис зон KMS-ключем, захист від підміни
- Закрити recursive resolver від зовнішнього доступу
- Response Rate Limiting (RRL) на авторитативних серверах

---

## 20. DDoS Response Plan — план реагування

### Фази реагування

| Фаза | Дія | Час |
|------|-----|-----|
| **Detect** | CloudWatch Alarm, Shield notification | Хвилина 0 |
| **Classify** | L3/L4 vs L7, volumetric vs app-layer | Хвилини 1–2 |
| **Escalate** | Notify SOC, підключити AWS DRT | Хвилина 2–3 |
| **Mitigate** | WAF rules, rate limits, geo-block | Хвилини 3–5 |
| **Monitor** | Відстеження ефективності mitigation | Тривало |
| **Recover** | Перевірка нормальної роботи | Після завершення |
| **Post-mortem** | Аналіз, покращення захисту | Наступного дня |

### Автоматичний Runbook (Lambda)
```python
def respond_to_ddos(event, context):
    # 1. Отримати топ-IP за 15 хв з WAF логів
    top_ips = get_top_attackers_from_logs(15)
    # 2. Додати до WAF IP Set блокування
    waf.update_ip_set(Id=BLOCK_IPSET_ID, Addresses=top_ips)
    # 3. Знизити WAF rate limit тимчасово
    update_rate_limit(500)  # з 2000 до 500
    # 4. Увімкнути JS challenge для всіх
    enable_js_challenge_for_all_requests()
    # 5. Сповістити SOC через SNS
    notify_soc(f"DDoS mitigation: {len(top_ips)} IPs blocked")
```

---

## 21. Chaos Engineering та AWS FIS

**Chaos Engineering** — дисципліна проведення керованих "аварій" у системі для виявлення слабких місць до того, як вони призведуть до реальних інцидентів.

**Принципи:**
- "Inject failure before failure finds you"
- Визначити **Steady State Hypothesis** — нормальний стан системи
- Контролювати **Blast Radius** — масштаб експерименту
- Мати автоматичний **rollback** при відхиленні від норми

### AWS Fault Injection Simulator (FIS)
- Managed chaos engineering tool від AWS
- Вбудовані actions: terminate EC2, CPU/memory stress, network disruption
- RDS failover, AZ disruption, packet loss, latency injection
- Stop Conditions: зупинка при спрацьовуванні CloudWatch Alarm

```hcl
resource "aws_fis_experiment_template" "az_failure" {
  description = "Simulate AZ-1a failure"
  stop_condition {
    source = "aws:cloudwatch:alarm"
    value  = aws_cloudwatch_alarm.error_rate.arn
  }
  action {
    name      = "TerminateAZ1aInstances"
    action_id = "aws:ec2:terminate-instances"
    target    { key = "Instances"; value = "az1a" }
  }
}
```

### AWS Resilience Hub
- Оцінює застосунок на відповідність цільовим RTO/RPO
- **Resiliency Score** — числова оцінка від 0 до 100
- Знаходить single points of failure
- Рекомендації для покращення: де додати Multi-AZ, резервування, backup

---

## 22. Захист API від DDoS та abuse

### API Gateway захист
- **Usage Plans + API Keys** — throttling per client (burst + rate limits)
- **Resource Policy** — IP-based access control на рівні API
- **Authorizers** — Lambda або Cognito JWT validation
- **Caching** — 300 сек cache для GET endpoints → знижує навантаження на backend

### Загрози та захист

| Загроза | Вектор | Захист |
|---------|--------|--------|
| API flooding | Масові запити | Rate limiting + WAF |
| Credential stuffing | Login bruteforce | Cognito + MFA + WAF |
| Scraping | Data harvest | Bot Control + CAPTCHA |
| Shadow API | Unlisted endpoints | API Gateway + OpenAPI validation |

---

## 23. HA у телекомунікаціях та 5G

### Вимоги до доступності
- Голос: **99.999%** (5.26 хв/рік) — регуляторна вимога
- SMS: 99.99% або вище
- Дані: 99.9–99.99% залежно від класу сервісу

### 5G Core HA в AWS

| NF (Network Function) | HA Pattern | AWS реалізація |
|----------------------|------------|---------------|
| AMF | Active-Active | EKS Multi-AZ + NLB |
| SMF | Active-Active | EKS + DynamoDB sessions |
| UPF | N:1 redundancy | EKS + AWS VPC CNI |
| UDM/HSS | Active-Active | Aurora Global + EKS |
| NRF | Active-Active | EKS + ElastiCache |

**AWS Outposts + 5G:** NF на Outposts у дата-центрі оператора + зв'язок з AWS регіоном через Direct Connect. Latency <1 мс для control plane + DDoS захист через AWS Shield.

---

## 24. Health Checks та Synthetic Monitoring

### Шари health checks

| Компонент | Health Check | Дія при відмові |
|-----------|-------------|----------------|
| EC2 | ASG EC2 Status Check | Terminate + replace |
| Container | ECS/EKS liveness probe | Restart container |
| Target | ALB Target Health | Remove від LB |
| Endpoint | Route 53 Health Check | DNS failover |
| RDS | Multi-AZ replication lag | Failover до replica |

### CloudWatch Synthetics (Canary)
- Виконує скрипти (Playwright/Puppeteer) для перевірки end-to-end user journey
- Тестує повний шлях: login → вибір товару → оплата → підтвердження
- Запускається кожні 1–5 хвилин
- Сповіщення: CloudWatch → SNS → PagerDuty

```hcl
resource "aws_synthetics_canary" "checkout" {
  name            = "checkout-flow-canary"
  runtime_version = "syn-nodejs-puppeteer-7.0"
  handler         = "checkout.handler"
  schedule { expression = "rate(5 minutes)" }
  run_config {
    timeout_in_seconds = 120
    active_tracing     = true
  }
}
```

---

## 25. Практичний кейс: DDoS-атака на телеком-портал

**Сценарій:** HTTP flood через ботнет — 500,000 RPS на портал самообслуговування оператора.

**Хронологія реагування:**

| Час | Подія |
|-----|-------|
| 14:00 | Початок атаки: 500K RPS на ALB |
| 14:01 | CloudWatch Alarm: ALB RequestCount >100K/хв |
| 14:02 | Shield Advanced виявляє аномалію → GuardDuty finding |
| 14:03 | EventBridge → Lambda → WAF Rate Limit знижено до 500/5хв |
| 14:05 | 95% трафіку заблоковано WAF |
| 14:10 | AWS DRT підключається, аналізує патерн |
| 14:15 | DRT додає сигнатурне WAF-правило → блок 99.9% атаки |

**Результати:**

| Метрика | Під атакою | Після митигації |
|---------|-----------|----------------|
| Availability | 72% | 99.95% |
| Latency (p99) | 12 сек | 250 мс |
| Blocked requests | 0% | 99.8% (499K/s) |
| MTTD | — | 2 хвилини |
| MTTR (авто) | — | 3 хвилини |

> **Ключ до успіху:** Shield Advanced + WAF pre-configured + автоматизований playbook. Система захистилась самостійно — людина підключилась тільки через 10 хвилин.

---

## 26. Оптимізація витрат під час DDoS

### "Bill Attack" — DDoS проти бюджету
Атакуючий генерує трафік → Auto Scaling піднімає instance → AWS-рахунок зростає.

### Захист
- **AWS Shield Advanced Cost Protection** — компенсує Data Transfer та WAF costs під час атаки
- **ASG MaxCapacity** — жорсткий ліміт кількості інстанцій
- **AWS Budgets Alerts** — сповіщення при перевищенні бюджету
- **AWS Cost Anomaly Detection** — ML виявляє аномальні витрати

---

## 27. Compliance та регуляторні вимоги

| Стандарт | Вимога | AWS реалізація |
|----------|--------|---------------|
| PCI DSS v4 Req 12.3 | Захист від DDoS для CDE | Shield Advanced + WAF |
| ISO 27001 A.17 | Business continuity planning | Multi-AZ, Backup |
| DORA (EU) | ICT resilience & recovery testing | FIS + Resilience Hub |
| NIS2 Directive | Cyber resilience для операторів | Well-Architected + DRP |
| Telecom Regulation (UA) | 99.999% для голосу | Multi-AZ Aurora + ASG |

**DORA (Digital Operational Resilience Act):** вступив у дію 01.2025 для EU фінансового сектору. Вимагає регулярне тестування resilience (chaos engineering) та звітування про ICT incidents.

**Звітування про DDoS-інциденти:**
- NIS2: значні інциденти — звіт регулятору за **24 год**
- GDPR: якщо DDoS призвів до витоку даних → **72 год**
- Telecom regulators: великі збої = обов'язкова звітність

---

## 28. Типові помилки та чеклист DDoS Readiness

### Антипатерни

| Помилка | Наслідок | Рішення |
|---------|----------|---------|
| EC2 напряму в Інтернет | Недоступний при атаці | CloudFront + ALB перед EC2 |
| Один регіон, одна AZ | Single point of failure | Multi-AZ мінімум |
| Shield Standard для критичного | Немає SLA, DRT | Shield Advanced |
| WAF тільки у COUNT mode | Все пропускається | Перевести у BLOCK mode |
| Auto Scaling без ліміту | Bill Attack | MaxCapacity + Budget Alert |
| Реальний IP origin відкритий | Атакують в обхід CloudFront | SG дозволяє тільки CF prefix |
| Немає DDoS Response Plan | Хаотична реакція, довгий MTTR | Runbook + автоматизація |

### ✅ Чеклист DDoS Readiness
- ☑ Shield Advanced для всіх public-facing ресурсів
- ☑ WAF Web ACL з rate-limiting та managed rules у BLOCK mode
- ☑ Route 53 health checks + failover routing
- ☑ CloudFront перед origin з Origin Shield
- ☑ Auto Scaling з MaxCapacity та Budget Alert
- ☑ DDoS Response Plan задокументований та протестований
- ☑ Тестування щоквартально (FIS + load test)
- ☑ DNSSEC для захисту DNS
- ☑ Автоматизований runbook (EventBridge + Lambda)
- ☑ Synthetic canary monitoring

---

## Підсумок

Захист від DDoS — це не один продукт, а **система з шарів:** від мережевого рівня (Shield, CloudFront) до прикладного (WAF, Rate Limiting) до архітектурного (Multi-AZ, Auto Scaling). Доступність системи визначається не окремими компонентами, а найслабшою ланкою в ланцюзі.

**Золоте правило:** "Розподіляй навантаження, поглинай атаки на периметрі, масштабуй горизонтально, автоматизуй реагування, тестуй регулярно."

---

*Лекція 9 · Захист від DDoS та забезпечення доступності · Інформаційна безпека телекомунікаційних та хмарних технологій*
