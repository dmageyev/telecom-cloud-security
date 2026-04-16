# Конспект лекції 8: Моніторинг, логування та виявлення загроз

**Курс:** Інформаційна безпека телекомунікаційних та хмарних технологій  
**Модуль 2 — Лекція 8**

---

## 1. Навіщо моніторинг безпеки — масштаб проблеми

Моніторинг безпеки є одним із фундаментальних стовпів сучасної стратегії кіберзахисту. Принцип «неможливо захистити те, чого не бачиш» лаконічно відображає ключову цінність цього напрямку: лише тоді, коли організація має повну видимість (visibility) власного середовища, вона здатна своєчасно виявляти загрози, мінімізувати збитки та забезпечувати виконання регуляторних вимог. Відсутність належного моніторингу перетворює навіть найбільш захищену архітектуру на «чорну скриньку», де зловмисник може діяти непоміченим місяцями.

Статистика підтверджує масштаб проблеми. Відповідно до Verizon Data Breach Investigations Report (DBIR) 2024, медіанний час перебування зловмисника в скомпрометованій системі (dwell time) без виявлення становить 194 дні — понад шість місяців. Ще більш тривожним є інший показник того ж звіту: 68% зламів виявляються не власними службами безпеки, а зовнішніми агентами — клієнтами, партнерами або регуляторами. IBM Cost of a Data Breach Report 2024 фіксує середню вартість витоку даних на рівні 4,88 мільйона доларів, причому організації з розвиненим SOC (Security Operations Center) скорочують цей показник у середньому на 49%.

Середній час виявлення загрози (Mean Time to Detect, MTTD) і середній час реагування (Mean Time to Respond, MTTR) є ключовими KPI будь-якого підрозділу безпеки. Організації, що впровадили повноцінний SIEM з автоматизацією (SOAR), демонструють MTTD на рівні 15–30 хвилин проти 194 днів для організацій без моніторингу; MTTR — від кількох хвилин (автоматизована відповідь) проти 73 днів у випадку ручного реагування.

| Показник | Без моніторингу | З повноцінним SOC |
|----------|-----------------|-------------------|
| MTTD | 194 дні | 30 хвилин |
| MTTR | 73 дні | 5 хвилин (авто) |
| Вартість витоку | $4.88M | -49% при ранньому виявленні |
| Зовнішнє виявлення | 68% | <20% |

Мета моніторингу безпеки охоплює кілька взаємопов'язаних завдань. Виявлення (Detection) — ідентифікація аномальної або підозрілої активності в режимі реального часу. Розслідування (Investigation) — збір цифрових доказів та відтворення хронології інциденту. Реагування (Response) — автоматична або ручна нейтралізація загрози. Compliance — документальне підтвердження виконання вимог PCI DSS, GDPR, ISO 27001, NIS2. AWS Well-Architected Framework визначає Detection як обов'язковий елемент Security Pillar поряд з Identity Management та Infrastructure Protection, що відображає загальногалузевий консенсус щодо пріоритетності цього напрямку.

Важливо розуміти, що моніторинг безпеки є не разовим проектом, а безперервним операційним процесом, який потребує постійного вдосконалення. Загрозовий ландшафт змінюється щодня: нові CVE-вразливості публікуються сотнями щотижня, нові техніки атак відкриваються та каталогізуються MITRE, нові класи зловмисників з'являються в міру зростання цінності хмарних даних. Тому архітектура системи моніторингу повинна бути гнучкою та адаптивною: правила виявлення регулярно оновлюються за результатами Threat Intelligence, порогові значення перекалібровуються відповідно до реальних поведінкових базисів, а SOAR-playbook'и удосконалюються після кожного інциденту в рамках циклу Post-Incident Review. Організації, що розглядають систему моніторингу як «встановив та забув», ризикують виявити, що їхні детектори безнадійно застаріли саме в той момент, коли вони найбільш необхідні.

---

## 2. Модель видимості: шари моніторингу в хмарному середовищі

Ефективний моніторинг безпеки хмарного середовища будується за принципом багатошарової видимості, що охоплює всі рівні технологічного стека — від інфраструктурного до прикладного. Кожен шар генерує специфічні типи подій та вимагає специфічних інструментів збору та аналізу.

Мережевий моніторинг (Network Layer) охоплює аналіз VPC Flow Logs, трафіку DNS, мережевих з'єднань та виявлення аномальних потоків між підмережами. Сервіс AWS GuardDuty на цьому рівні аналізує VPC Flow Logs та CloudTrail DNS Logs для виявлення компромісу EC2 (command & control зв'язок, внутрішнє сканування, незвичний трафік). Рівень хоста (Host Layer) охоплює системні логи операційної системи, виклики системних викликів (syscall), журнали автентифікації, файлові операції. Amazon Inspector v2 та AWS Systems Manager надають видимість на рівні EC2. Прикладний рівень (Application Layer) — це логи застосунків, HTTP access logs, логи баз даних, API Gateway logs. Рівень ідентичності (Identity Layer) фіксується через AWS CloudTrail та IAM Access Analyzer: кожен API-виклик до будь-якого AWS-сервісу залишає записи CloudTrail із зазначенням ідентифікатора користувача/ролі, часу, source IP та параметрів. Рівень даних (Data Layer) — доступ до S3, RDS, DynamoDB, Secrets Manager — моніторується Amazon Macie для виявлення PII та AWS Config для перевірки публічного доступу.

Цикл моніторингу безпеки описується як безперервний процес: Collect → Normalize → Correlate → Analyze → Alert → Investigate → Respond → Learn. На етапі Collect дані збираються з усіх джерел (VPC Flow Logs, CloudTrail, ALB logs, CloudWatch Logs тощо) до централізованого сховища. На етапі Normalize різноформатні логи приводяться до єдиної схеми (наприклад, через Kinesis Data Firehose з Lambda-трансформацією або через OCSF — Open Cybersecurity Schema Framework). Correlate — зіставлення подій із різних джерел за часовим вікном та ідентифікаторами для виявлення складних атак (наприклад, reconnaissance → exploitation → exfiltration). Alert — генерація пріоритизованих сповіщень. Наступний цикл Learn включає оновлення правил виявлення за результатами розслідування інциденту, що забезпечує постійне вдосконалення системи.

---

## 3. Основи логування: формати, рівні, структура

Лог-файли є первинним артефактом моніторингу безпеки. Якість та повнота логування безпосередньо визначають можливість виявлення та розслідування інцидентів. Розуміння базових принципів логування є передумовою для побудови ефективної системи моніторингу.

Рівні логування (Log Levels) відповідно до стандарту RFC 5424 (syslog) впорядковані за серйозністю від 0 до 7. У контексті безпеки найбільш релевантними є: ERROR — помилки, що потребують уваги (помилки автентифікації, збої з'єднання); WARNING — потенційно аномальні події (перевищення порогів, незвичні запити); INFO — нормальні події, що підлягають аудиту (логін/логаут, зміни конфігурації); DEBUG — деталізована інформація для розробки. Практично для цілей безпеки та compliance рекомендується логувати мінімум рівні INFO та вище з постійним зберіганням, тоді як DEBUG-логи можуть зберігатись лише тимчасово.

Структуроване логування у форматі JSON є галузевим стандартом для сучасних систем завдяки машинозчитуваності та простоті парсингу. Приклад структурованого лог-запису:

```json
{
  "timestamp": "2024-03-15T14:22:31.123Z",
  "level": "WARNING",
  "service": "auth-service",
  "event_type": "authentication_failed",
  "user_id": "user-1234",
  "source_ip": "198.51.100.42",
  "attempt": 5,
  "user_agent": "python-requests/2.31.0",
  "request_id": "req-abc-xyz"
}
```

На відміну від неструктурованих текстових логів, JSON-формат дозволяє автоматично індексувати поля, виконувати агрегаційні запити в CloudWatch Logs Insights або Amazon Athena без попереднього парсингу, а також безпосередньо передавати в SIEM без трансформації.

Вимоги до обов'язкового логування для безпеки охоплюють кілька категорій подій. Події автентифікації — усі успішні та невдалі спроби входу з зазначенням часу, source IP, User-Agent. Зміни привілеїв — надання, відкликання та ескалація прав доступу. Зміни конфігурації — модифікація Security Group, NACL, IAM-політик, S3 Bucket Policy. Доступ до чутливих даних — читання та запис до конфіденційних сховищ. Операції з ресурсами — створення, зміна та видалення критичних ресурсів AWS. Мережева активність — необхідний для детекції lateral movement та C2.

Критично важливим аспектом є дотримання вимог щодо конфіденційності в логах: паролі, ключі API, номери банківських карток (PAN), SSN та інші персональні дані (PII) мають бути виключені з логів або заміщені замасковані представники (токени). Це вимога не лише безпеки, але й комплаєнсу PCI DSS (Requirement 3.4) та GDPR (Article 25 — Privacy by Design). Amazon Macie автоматично виявляє паттерни PII в S3-об'єктах (включно з log-файлами) і сповіщає Security Hub.

---

## 4. Архітектура збору та зберігання логів в AWS

Централізована архітектура логування є необхідною умовою ефективного моніторингу в хмарному середовищі, особливо для організацій із множинними AWS-акаунтами. Розрізнені логи в різних сервісах унеможливлюють кореляцію подій та подовжують час розслідування.

Типова архітектура централізованого логування в AWS будується за таким принципом: усі джерела (CloudTrail, VPC Flow Logs, ALB Access Logs, CloudWatch Logs, DNS Logs) передають події до Amazon Kinesis Data Firehose або через CloudWatch Logs Subscription Filter до Firehose. Firehose трансформує та доставляє дані до Amazon S3 (як primary storage для довгострокового зберігання та Athena-аналізу) і паралельно до Amazon OpenSearch Service для оперативного пошуку та dashboards. AWS Lambda виконує нормалізацію та збагачення подій (додавання геолокації, threat intelligence тощо) перед завантаженням у OpenSearch.

| Компонент | Роль | Зберігання |
|-----------|------|-----------|
| CloudTrail Trail | Аудит API-викликів | S3 (1–7 років) |
| VPC Flow Logs | Мережевий трафік | CloudWatch Logs / S3 |
| ALB/CloudFront Logs | HTTP трафік | S3 |
| CloudWatch Logs | Застосунки, OS | 1–365 днів (tax retention) |
| S3 (log archive) | Централізоване сховище | Object Lock (WORM) |

Цілісність логів є критичною умовою їх використання як доказів при розслідуванні інцидентів та для compliance. S3 Object Lock у режимі WORM (Write Once, Read Many) унеможливлює зміну або видалення об'єктів протягом визначеного retention period. CloudTrail Log File Validation генерує SHA-256 digest-файли для кожного лог-файлу, що дозволяє верифікувати цілісність через CLI-команду `aws cloudtrail validate-logs`. Terraform-конфігурація для CloudWatch Log Group:

```hcl
resource "aws_cloudwatch_log_group" "security" {
  name              = "/security/audit"
  retention_in_days = 365
  kms_key_id        = aws_kms_key.logs.arn
}
```

Шифрування логів за допомогою KMS-ключів є обов'язковою вимогою PCI DSS та ISO 27001 для захисту конфіденційних даних, що можуть зберігатися в логах (наприклад, номери транзакцій, IP-адреси). Server-Side Encryption (SSE-KMS) для S3 та CMK у CloudWatch Logs забезпечують шифрування в стані спокою (at rest), тоді як TLS/HTTPS — при передачі (in transit).

---

## 5. SIEM: Security Information and Event Management

SIEM (Security Information and Event Management) є центральним компонентом будь-якого зрілого SOC і виконує функцію «мозку» системи безпеки, агрегуючи, кореляціюючи та аналізуючи події безпеки з усього середовища в режимі реального часу. Концепція SIEM виникла на початку 2000-х років шляхом поєднання Security Information Management (SIM, довгострокове зберігання та аналіз логів) та Security Event Management (SEM, моніторинг у реальному часі), і відтоді пройшла декілька поколінь еволюції.

Функціональна архітектура сучасного SIEM включає шість ключових компонентів. Збір даних (Data Collection) — прийом та парсинг логів із сотень різнорідних джерел через агенти, API, syslog, Kafka. Нормалізація (Normalization) — приведення різноформатних подій до єдиної схеми для уніфікованого аналізу. Зберігання (Storage) — індексований пошуковий рушій (Elasticsearch / OpenSearch) для оперативного доступу та холодне сховище S3 для архіву. Кореляція (Correlation) — застосування правил виявлення до потоку подій для ідентифікації паттернів атак. Alerting — пріоритизована генерація сповіщень для SOC. Reporting — автоматична генерація звітів для compliance та management.

Популярні SIEM-платформи поділяються на три категорії: класичні enterprise (Splunk, IBM QRadar, Microsoft Sentinel), open-source (Elastic SIEM, Wazuh) та cloud-native (AWS Security Hub + OpenSearch). Архітектура SIEM в AWS включає такий ланцюжок: Log Sources → Kinesis Firehose → S3/OpenSearch → Security Hub → EventBridge → Lambda (автоматична ремедіація) → SNS/PagerDuty (оповіщення).

| Сервіс | Роль у SIEM |
|--------|------------|
| CloudTrail | Джерело IAM/API подій |
| GuardDuty | ML-детектор загроз |
| Security Hub | Агрегатор findings |
| OpenSearch | SIEM-рушій, пошук, дашборди |
| EventBridge | Маршрутизація та тригери автоматизації |
| Lambda | SOAR playbook виконання |

Одним із найважливіших показників ефективності SIEM є False Positive Rate — відсоток хибних спрацьовувань. Надмірна кількість хибних спрацьовувань призводить до «alert fatigue» — стану, коли аналітик SOC починає ігнорувати або механічно закривати алерти, що може спричинити пропуск реального інциденту. За даними досліджень, аналітики SOC ігнорують до 44% алертів через їх надмірну кількість. Мистецтво тюнінгу SIEM полягає в балансуванні між чутливістю (sensitivity) та специфічністю (specificity) правил виявлення.

Еволюція SIEM пройшла чотири основних покоління. Перше покоління (2000–2006) — базовий збір та нормалізація syslog-логів без реальної кореляції. Друге покоління (2006–2013) — впровадження реального часу кореляції подій за фіксованими правилами, поява dashboards та compliance-звітів. Третє покоління (2013–2019) — інтеграція Threat Intelligence (TI) Feeds, UEBA-аналітика, перші ML-підходи для виявлення аномалій. Четверте покоління (2019–сьогодні) — Cloud-native SIEM (Microsoft Sentinel, AWS Security Hub + OpenSearch), повна інтеграція SOAR, NLP-аналіз логів через Large Language Models, OCSF-стандартизація для inter-vendor сумісності. Кожне покоління значно підвищувало здатність виявляти складні APT-атаки, але одночасно ускладнювало оперативне управління системою. Сучасна тенденція — перехід до SIEM-as-a-Service, де постачальник хмарних послуг бере на себе управління інфраструктурою та оновлення ML-моделей, звільняючи SOC від завдань операційного обслуговування платформи.

---

## 6. Кореляція подій та правила виявлення (Sigma Rules)

Кореляція подій є технологічним серцем SIEM — це процес зіставлення множини окремих подій з різних джерел для виявлення складних паттернів атак, жодна окрема подія з яких не є сама по собі індикатором компромісу. Атаки рідко виявляються через одну аномальну подію; натомість вони залишають ланцюжки пов'язаних слідів у різних системах.

Правило кореляції у типовому SIEM визначає: набір вхідних подій (джерела, типи), умову поєднання (AND, OR, sequence), часове вікно (наприклад, 5 хвилин), пороговий показник (count, threshold) та дію (alert, block, notify). Простий приклад: «Якщо з однієї IP-адреси протягом 60 секунд відбулося більше 10 невдалих спроб автентифікації (тип події: auth_failed) — генерувати HIGH-severity алерт».

Sigma — відкритий стандарт для написання правил виявлення загроз у Generic-форматі, що може транслюватися в запити для різних платформ (Splunk SPL, Elasticsearch DSL, CloudWatch Logs Insights тощо). Приклад Sigma-правила для виявлення privilege escalation в AWS:

```yaml
title: AWS IAM Privilege Escalation via Policy Attachment
status: stable
logsource:
  product: aws
  service: cloudtrail
detection:
  selection:
    eventSource: iam.amazonaws.com
    eventName:
      - AttachUserPolicy
      - AttachGroupPolicy
      - PutUserPolicy
    requestParameters.policyArn|contains:
      - 'AdministratorAccess'
      - 'PowerUserAccess'
  timeframe: 15m
  condition: selection
falsepositives:
  - Legitimate administrative changes (verify with change management)
level: high
tags:
  - attack.privilege_escalation
  - attack.t1098
```

Типові правила виявлення для AWS-середовища охоплюють кілька критичних сценаріїв. Публічний доступ S3: спрацьовування при `s3:PutBucketPublicAccessBlock` з дозволеним публічним доступом — потенційний data leak. Зміна SG/NACL: `ec2:AuthorizeSecurityGroupIngress` на порт 0-65535 від 0.0.0.0/0 — відкриття всіх портів назовні. Видалення CloudTrail: `cloudtrail:DeleteTrail` або `cloudtrail:StopLogging` — спроба приховати сліди активності. Підозрілий IAM: виконання більше 20 різних API-дій за 1 хвилину — reconnaissance-поведінка. Консоль AWS з незвичного регіону: вхід в Management Console з IP-адреси у незвичному географічному регіоні.

---

## 7. UEBA: User and Entity Behavior Analytics

User and Entity Behavior Analytics (UEBA) представляє наступний еволюційний крок після традиційних правил SIEM — перехід від детекції за фіксованими сигнатурами та порогами до виявлення аномалій на основі машинного навчання та статистичного моделювання. Ключова інновація UEBA полягає у формуванні індивідуального «поведінкового базису» (behavioral baseline) для кожного користувача та пристрою (entity), а подальшому виявленні відхилень від цього базису як потенційних індикаторів компромісу.

Технічні підходи UEBA включають: аналіз принципу мінімальних привілеїв (чи отримав користувач доступ до даних, яких зазвичай не використовує?), peer grouping (порівняння поведінки з типовою поведінкою схожих за роллю колег), time-series analysis (чи типово для цього облікового запису виконувати дії о 3 годині ночі?), entity graph analysis (які зв'язки між користувачами, пристроями та даними є незвичними?).

Класичні ознаки аномальної поведінки, що є мішенями UEBA: дії у незвичний час (нічні операції для співробітника, що завжди працює вдень); доступ до незвичних ресурсів (маркетолог, що звертається до сховища з кредитними картами); масове завантаження даних (DevOps інженер скачує 50 ГБ з S3 за 30 хвилин); геолокаційна аномалія (вхід з Бразилії через 5 хвилин після входу з Польщі — неможливо фізично); доступ з незвичного пристрою або браузера; незвичний патерн API-викликів.

Amazon GuardDuty є фактично UEBA-сервісом для AWS-контексту: він аналізує VPC Flow Logs, CloudTrail Management та Data Events, а також Route 53 DNS Logs за допомогою ML-моделей, побудованих на основі аналізу поведінки мільярдів подій у глобальній мережі AWS. GuardDuty автоматично встановлює базову поведінку для кожного ресурсу (IAM identity, EC2 instance, S3 bucket) і виявляє відхилення, не вимагаючи ручного налаштування порогів. Оцінка знахідок GuardDuty базується на severity score від 0.1 до 8.9, що відображає впевненість ML-моделі та потенційний ризик.

Реальна цінність UEBA реалізується лише при правильному налаштуванні baseline learning period. Для нового GuardDuty-акаунту або нового ресурсу ML-модель потребує приблизно 7–14 днів для навчання на «нормальному» трафіку. Якщо GuardDuty вмикається під час або відразу після компромісу, модель може навчитись на зловмисній активності як на «нормі», що призведе до занижених severity scores для подальших атак того ж типу. Тому рекомендується вмикати GuardDuty в нових акаунтах у момент їх створення (через AWS Control Tower guardrails або Organizations policy), а не ретроспективно після виявлення підозрілої активності. Регулярні GuardDuty penetration tests (з використанням офіційних sample findings через AWS CLI `generate-sample-findings`) дозволяють верифікувати коректність налаштування detection pipeline без реального ризику для production-середовища.

---

## 8. MITRE ATT&CK: Framework для виявлення загроз

MITRE ATT&CK (Adversarial Tactics, Techniques, and Common Knowledge) є загальновизнаним структурованим каталогом тактик та технік, що використовуються реальними APT-угрупуваннями під час кіберзловмисництва, побудованим на основі аналізу реальних атак. Цей фреймворк радикально трансформує підхід до виявлення загроз: замість реагування на сигнатури конкретних інструментів (що швидко застарівають), захисники фокусуються на виявленні поведінкових патернів — тобто на тому, ЩО робить зловмисник, а не ЧИМ.

ATT&CK Enterprise Matrix організована за чотирнадцятьма тактиками (Tactics), що відображають стадії атаки: Reconnaissance → Resource Development → Initial Access → Execution → Persistence → Privilege Escalation → Defense Evasion → Credential Access → Discovery → Lateral Movement → Collection → Command and Control → Exfiltration → Impact. Кожна тактика містить множину технік (Techniques), а кожна техніка — конкретні sub-techniques та процедури з прив'язкою до реальних APT-угруповань.

Для хмарного середовища AWS надзвичайно актуальні такі техніки ATT&CK. T1078 — Valid Accounts: компрометація IAM-credentials та їх використання для несанкціонованого доступу виявляється через CloudTrail-аномалії (географічна аномалія, незвичний час). T1562.001 — Impair Defenses: вимикання CloudTrail або GuardDuty для приховування активності детектується через моніторинг самих сервісів безпеки. T1530 — Data from Cloud Storage Object: несанкціонований доступ до S3 bucket виявляється через S3 Data Events + Macie. T1098 — Account Manipulation: прикріплення `AdministratorAccess`-policy виявляється через CloudTrail IAM-events + Sigma-правила.

Практичне застосування ATT&CK полягає в побудові threat model для конкретного середовища: організація визначає найбільш вірогідних для неї зловмисників (наприклад, фінансово мотивовані групи, конкуренти, інсайдери), ідентифікує техніки, які вони використовують, і будує детективні правила (Detection Signatures) та превентивні контролі (Mitigations) для кожної техніки. MITRE ATT&CK Navigator є веб-інструментом для візуалізації покриття (coverage) матриці.

Coverage analysis — ключовий інструмент зрілих команд безпеки: для кожної техніки ATT&CK визначається, чи є у організації відповідний детектив (так → зелений), є детектив але не верифікований у production (жовтий), або детектива немає (червоний). Результуюча «heat map» наочно показує прогалини у виявленні та допомагає пріоритизувати розробку нових правил. Для хмарної матриці ATT&CK Cloud (attack.mitre.org/matrices/cloud/) наявні специфічні техніки для AWS, Azure та GCP, що відображають принципові відмінності хмарної поверхні атаки від традиційної on-premise інфраструктури. Регулярний (щоквартальний) перегляд coverage у контексті нових технік, доданих до ATT&CK, є необхідною умовою підтримки актуальності системи виявлення.

Приклад ланцюжка атаки в AWS (реальний сценарій): Phishing → компрометація AWS Access Key (T1566.001) → Reconnaissance через `aws iam list-users`, `aws s3 ls` (T1580) → Privilege Escalation через `AttachUserPolicy:AdministratorAccess` (T1098) → Defense Evasion через `StopLogging` CloudTrail (T1562.001) → Exfiltration через S3 `GetObject` масового скачування (T1530). Кожна ланка цього ланцюжка генерує характерні CloudTrail-події, які можна корелювати для виявлення атаки навіть при частковому збігу.

---

## 9. AWS CloudWatch: метрики та тривоги

Amazon CloudWatch є центральним сервісом моніторингу та спостережуваності AWS, що охоплює збір метрик, логів, подій та трейсів у єдину платформу. З погляду безпеки CloudWatch є не лише інструментом операційного моніторингу, але й критичною складовою виявлення загроз.

CloudWatch Metrics — сервіс збору, зберігання та візуалізації часових рядів метрик. AWS-сервіси автоматично публікують десятки стандартних метрик: CPU Utilization EC2, Request Count ALB, PutObject S3, GetSecretValue Secrets Manager тощо. Кастомні метрики публікуються через CloudWatch API:

```bash
aws cloudwatch put-metric-data \
  --namespace "Security/AuthMetrics" \
  --metric-name "FailedLoginAttempts" \
  --value 7 \
  --dimensions Service=auth-api,Environment=prod
```

CloudWatch Alarms дозволяють налаштовувати автоматичні тригери при перевищенні метриками заданих порогів. Для безпеки особливо цінними є Alarms на: кількість помилок автентифікації (metric filter по CloudWatch Logs), активність Root Account (CloudTrail metric filter), зміни Security Groups (Config Rule), будь-яку активність CloudTrail в регіоні, де не ведеться бізнес-діяльність. Підтримуються три стани Alarm: OK, ALARM та INSUFFICIENT_DATA; action при переході в ALARM може бути SNS notification, EC2 action, Auto Scaling action або Lambda execution.

Terraform-приклад Alarm для виявлення Root-активності:

```hcl
resource "aws_cloudwatch_metric_alarm" "root_activity" {
  alarm_name          = "root-account-usage"
  metric_name         = "RootAccountUsageCount"
  namespace           = "CloudTrailMetrics"
  statistic           = "Sum"
  period              = 300
  evaluation_periods  = 1
  threshold           = 1
  comparison_operator = "GreaterThanOrEqualToThreshold"
  alarm_actions       = [aws_sns_topic.security_alerts.arn]
  treat_missing_data  = "notBreaching"
}
```

Додатковою потужною функцією є CloudWatch Anomaly Detection — ML-алгоритм, що моделює очікувану поведінку метрики з урахуванням сезонності (денні/нічні/тижневі патерни) і автоматично встановлює динамічні пороги. Це особливо цінно для метрик, що не мають фіксованих «нормальних» значень: трафік API-шлюзу варіюється в залежності від дня тижня, тому статичний поріг виявлятиме хибні спрацьовування або пропускатиме справжні аномалії.

---

## 10. AWS CloudWatch Logs та Logs Insights

CloudWatch Logs є сервісом централізованого зберігання та аналізу лог-даних в AWS. Він приймає логи з EC2 (через CloudWatch Agent), Lambda (автоматично), ECS, EKS, CloudTrail, VPC Flow Logs та будь-яких застосунків через API. Log Groups організовують логи за джерелом; Log Streams — за конкретним ресурсом або часовим вікном всередині групи.

Retention Policy дозволяє налаштувати автоматичне видалення логів через 1, 3, 5, 7, 14, 30, 60, 90 днів або 1, 2, 3, 5, 7, 10 років. Для compliance-вимог PCI DSS рекомендується зберігати логи не менше 12 місяців (3 місяці онлайн + 9 у архіві). Шифрування CMK (Customer Managed Keys) через KMS застосовується на рівні Log Group.

CloudWatch Logs Insights є інтерактивним сервісом аналізу логів із власною мовою запитів. Приклад запиту для виявлення підозрілої активності:

```
fields @timestamp, sourceIPAddress, userAgent, errorCode
| filter eventSource = "signin.amazonaws.com" 
  and eventName = "ConsoleLogin"
  and errorCode = "Failed authentication"
| stats count(*) as failCount by sourceIPAddress, userIdentity.arn
| sort failCount desc
| limit 20
```

Цей запит повертає топ-20 IP-адрес із найбільшою кількістю невдалих спроб входу до AWS Console за вибраний часовий діапазон — базовий детектив для brute-force та credential stuffing атак. Logs Insights підтримує агрегації (stats, count, sum, avg, max, min), парсинг JSON-полів (`| parse @message`), регулярні вирази та фільтрацію.

Metric Filters дозволяють перетворювати патерни в лог-потоках на CloudWatch Metrics, що потім можуть тригерувати Alarms. Приклад фільтра для виявлення несанкціонованих API-викликів:

```hcl
resource "aws_cloudwatch_log_metric_filter" "unauthorized_api" {
  name           = "UnauthorizedAPICalls"
  log_group_name = aws_cloudwatch_log_group.cloudtrail.name
  pattern        = "{ ($.errorCode = \"*UnauthorizedAccess*\") || ($.errorCode = \"AccessDenied*\") }"
  metric_transformation {
    name      = "UnauthorizedAPICallsCount"
    namespace = "CloudTrailMetrics"
    value     = "1"
  }
}
```

Subscription Filters дозволяють у реальному часі доставляти відфільтровані лог-записи до Kinesis Data Firehose (для SIEM) або Lambda (для автоматичної реакції), що є ключовим компонентом real-time детекції.

---

## 11. AWS CloudTrail: аудит API-викликів

AWS CloudTrail є сервісом аудиту та логування всіх API-викликів до сервісів AWS у вашому акаунті. Кожен API-виклик — від консолі, CLI, SDK або будь-якого AWS-сервісу — залишає запис у CloudTrail із зазначенням: хто (userIdentity — IAM user/role/service), що (eventName — CreateBucket, DescribeInstances тощо), де (sourceIPAddress, awsRegion), коли (eventTime, ISO 8601), з якими параметрами (requestParameters) та з яким результатом (responseElements, errorCode).

CloudTrail розрізняє три типи подій. Management Events (Control Plane) — адміністративні операції з ресурсами (CreateVPC, AttachRolePolicy, ModifyDBInstance). Увімкнені за замовчуванням та зберігаються 90 днів у CloudTrail Event History безкоштовно. Data Events (Data Plane) — операції над даними всередині ресурсів (S3 GetObject/PutObject, Lambda Invoke, DynamoDB GetItem). Потребують явного увімкнення. Insights Events — автоматичне виявлення незвичних паттернів у Management Events (наприклад, різке зростання кількості `TerminateInstances` API-викликів).

За замовчуванням CloudTrail Event History зберігається 90 днів. Для compliance потрібна конфігурація Trail — окремого компонента, що доставляє події до S3:

```hcl
resource "aws_cloudtrail" "main" {
  name                          = "main-trail"
  s3_bucket_name                = aws_s3_bucket.trail_logs.id
  include_global_service_events = true
  is_multi_region_trail         = true
  enable_log_file_validation    = true
  cloud_watch_logs_group_arn    = "${aws_cloudwatch_log_group.trail.arn}:*"
  cloud_watch_logs_role_arn     = aws_iam_role.trail_cloudwatch.arn
  kms_key_id                    = aws_kms_key.trail.arn
}
```

Параметр `enable_log_file_validation = true` забезпечує генерацію SHA-256 digest-файлів для кожного log file, що дозволяє перевірити цілісність логів і виявити будь-яку спробу маніпуляції. `is_multi_region_trail = true` забезпечує, що Trail охоплює всі регіони AWS — без цього зловмисник може діяти в іншому регіоні поза зоною видимості.

Типовий CloudTrail-запис підозрілої activity виглядає наступним чином:

```json
{
  "eventTime": "2024-03-15T03:14:22Z",
  "userIdentity": {
    "type": "IAMUser",
    "userName": "dev-alice",
    "arn": "arn:aws:iam::123456789:user/dev-alice"
  },
  "eventName": "AttachUserPolicy",
  "sourceIPAddress": "198.51.100.42",
  "requestParameters": {
    "userName": "dev-alice",
    "policyArn": "arn:aws:iam::aws:policy/AdministratorAccess"
  }
}
```

Цей запис свідчить про те, що о 3:14 AM користувач dev-alice самостійно прикріпила AdministratorAccess policy — класичний індикатор privilege escalation. Зверніть увагу на нічний час (03:14 AM), незвичну дію для dev-роль, і адміністративну природу policy.

---

## 12. CloudTrail: аналіз та розслідування інцидентів

Ефективне розслідування інцидентів в AWS-середовищі базується передусім на аналізі CloudTrail-записів. Для аналізу великих обсягів CloudTrail-даних (мільйони подій за кілька місяців) найбільш ефективним є використання Amazon Athena — інтерактивного SQL-сервісу для запитів до S3.

Типовий аналітичний запит в Athena для пошуку всіх дій конкретного IAM-користувача за визначений період:

```sql
SELECT eventtime, eventsource, eventname, 
       sourceipaddress, requestparameters, errormessage
FROM cloudtrail_logs
WHERE useridentity.arn LIKE '%dev-alice%'
  AND eventtime > '2024-03-14'
  AND eventtime < '2024-03-16'
ORDER BY eventtime;
```

Цей запит повертає повну хронологію дій облікового запису `dev-alice` за 24-годинний період, що дозволяє відтворити послідовність дій зловмисника: що відбувалось до підозрілого AttachPolicy-запису, які ресурси було доступлено після.

Для розслідування data exfiltration через S3 корисний такий запит:

```sql
SELECT sourceipaddress, useragent, sum(1) as request_count
FROM cloudtrail_logs
WHERE eventsource = 's3.amazonaws.com'
  AND eventname IN ('GetObject', 'ListBucket')
  AND errorcode IS NULL
  AND requestparameters.bucketName = 'customer-data-bucket'
  AND eventtime > '2024-03-15'
GROUP BY sourceipaddress, useragent
ORDER BY request_count DESC;
```

Цей запит виявляє аномальну кількість GET-запитів до конкретного S3 bucket, згрупованих за source IP та User-Agent. Раптовий сплеск від незнайомої IP-адреси або незвичного User-Agent є сильним індикатором несанкціонованого скачування.

Amazon Detective автоматизує цей аналітичний процес, будуючи граф відносин між AWS-ресурсами, мережевими з'єднаннями та діями на основі CloudTrail та VPC Flow Logs. Detective дозволяє в кілька кліків відповісти на питання: «Які ресурси спілкувались з цим EC2 інстансом?», «Які API-виклики виконувала ця IAM роль?», «З яких IP-адрес входив цей IAM user?».

Важливим елементом ефективного розслідування є збереження ланцюжка доказів (Chain of Custody). CloudTrail log files з увімкненою валідацією (Log File Validation) та S3 Object Lock забезпечують незмінність доказів навіть при наявності у зловмисника доступу до адміністративних ролей — він не може видалити або змінити вже записані log files. При передачі справи до правоохоронних органів або зовнішніх аудиторів AWS-аккаунт Power of Attorney або AWS Artifact може надати офіційно завірені звіти про стан ресурсів у визначений момент часу, що є юридично прийнятними доказами. Практика «preserve before investigate» — першочергове збереження повного стану підозрілих ресурсів (EBS snapshot, memory dump через SSM, network packet capture) до початку будь-яких ремедіаційних дій — є обов'язковою умовою якісного forensic-розслідування.

---

## 13. AWS Config: безперервний моніторинг конфігурацій

AWS Config є сервісом безперервного моніторингу, аудиту та оцінки конфігурацій ресурсів AWS на відповідність визначеним правилам (Config Rules). Де CloudTrail фіксує WHO зробив WHAT (API-виклики), Config фіксує СТАН кожного ресурсу у кожен момент часу, утворюючи повну хронологію змін конфігурацій.

Config автоматично виявляє та записує конфігурацію ресурсів AWS: параметри Security Group (дозволені порти та протоколи), стан S3 bucket (публічний/приватний, шифрування, versioning), IAM-Policy (прикріплені дозволи), стан шифрування EBS, налаштування RDS тощо. При кожній зміні конфігурації Config генерує Configuration Item — знімок поточного стану ресурсу — та публікує Change Notification через SNS або EventBridge.

Managed Config Rules — набір вбудованих правил оцінки відповідності з більш ніж 200 пунктів, зокрема:

| Config Rule | Що перевіряє |
|-------------|-------------|
| `s3-bucket-public-read-prohibited` | S3 bucket не дозволяє публічне читання |
| `encrypted-volumes` | EBS томи зашифровані |
| `mfa-enabled-for-iam-console-access` | MFA активовано для консольного доступу |
| `vpc-sg-open-only-to-authorized-ports` | Security Group не відкриває всі порти |
| `cloudtrail-enabled` | CloudTrail активований у всіх регіонах |
| `rds-storage-encrypted` | RDS-інстанси зашифровані |

Автоматична ремедіація (Auto Remediation) через SSM Automation — найбільш потужна функція Config з погляду безпеки. При спрацьовуванні Config Rule можна автоматично запустити SSM Automation Document, що виправить відповідність: наприклад, автоматично видалити правило Security Group `0.0.0.0/0 -> SSH port 22`, або примусово увімкнути шифрування S3 bucket. Terraform-конфігурація:

```hcl
resource "aws_config_remediation_configuration" "sg_remediation" {
  config_rule_name = "vpc-sg-open-only-to-authorized-ports"
  target_id        = "AWS-DisablePublicAccessForSecurityGroup"
  target_type      = "SSM_DOCUMENT"
  automatic        = true
  maximum_automatic_attempts = 5
}
```

Config Aggregator дозволяє збирати дані конфігурацій з множини AWS-акаунтів та регіонів у централізований Security акаунт, що є обов'язковою функцією для enterprise-середовищ з multi-account AWS Organizations.

---

## 14. AWS Security Hub: єдина панель безпеки

AWS Security Hub є центральним агрегатором findings (знахідок безпеки) від усіх AWS-сервісів безпеки та сторонніх партнерів, що реалізує принцип «single pane of glass» для команди безпеки. Замість того щоб перевіряти GuardDuty, Inspector, Macie, Config та інші сервіси окремо, Security Hub збирає, нормалізує та пріоритизує всі findings в єдиному інтерфейсі.

Security Hub використовує стандартний формат ASFF (Amazon Security Finding Format) для уніфікованого представлення знахідок із різних джерел. Findings нормалізуються за severity: CRITICAL, HIGH, MEDIUM, LOW, INFORMATIONAL. Автоматична оцінка відповідності стандартам (Security Standards) дозволяє Security Hub оцінювати середовище на відповідність AWS Foundational Security Best Practices, CIS AWS Foundations Benchmark v1.4.0, PCI DSS v3.2.1, NIST SP 800-53 Rev 5 — з автоматичним скором та переліком конкретних невідповідностей.

Екосистема інтеграцій Security Hub охоплює: GuardDuty (threat findings), Inspector v2 (vulnerabilities), Macie (data privacy), Config (compliance findings), IAM Access Analyzer (public access findings), Firewall Manager, сторонні SIEM та SOAR через API. Автоматизований workflow через EventBridge + Lambda дозволяє реалізувати будь-який Playbook безпосередньо при появі finding:

```python
# Lambda triggered by EventBridge Rule on Security Hub Finding
def lambda_handler(event, context):
    finding = event['detail']['findings'][0]
    severity = finding['Severity']['Label']
    resource_arn = finding['Resources'][0]['Id']
    
    if severity in ['CRITICAL', 'HIGH']:
        if finding['Types'][0] == 'Software and Configuration Checks/AWS/GuardDuty':
            quarantine_ec2(resource_arn)
        revoke_iam_keys(finding)
        notify_soc(finding)
```

Security Hub insights — агреговані фільтровані подання, що відповідають на типові питання аналітика: «Які EC2 інстанси мають найбільше критичних знахідок?», «Які IAM entities генерують найбільше підозрілих подій?», «Яке compliance-покриття по акаунтах?».

---

## 15. AWS GuardDuty: ML-детекція загроз

Amazon GuardDuty є managed service для виявлення загроз на базі машинного навчання, що аналізує три типи джерел даних: VPC Flow Logs (мережева активність), CloudTrail Management Events (API-активність) та Route 53 DNS Logs (DNS-запити). GuardDuty не потребує встановлення агентів і жодних змін у налаштуваннях мережі — він отримує копії даних безпосередньо від відповідних AWS-сервісів, що забезпечує нульовий вплив на продуктивність.

ML-моделі GuardDuty навчені на базі петабайтів даних AWS-інфраструктури і здатні виявляти як відомі (сигнатурні) загрози — звернення до відомих зловмисних IP-адрес та доменів — так і невідомі аномалії поведінки. GuardDuty категоризує знахідки у кілька класів, кожен із яких відповідає певному типу загрози.

Клас Backdoor вказує на EC2-інстанс, що здійснює аутбаунд-з'єднання на зовнішній C2-сервер, проводить port scanning або є частиною botnet. Клас Recon фіксує розвідувальну діяльність: сканування портів, незвичні API-виклики для отримання списків ресурсів. Клас Trojan виявляє DNS-запити до відомих доменів malware або фішингу. Клас UnauthorizedAccess визначає підозрілий вхід з незвичної геолокації або використання Tor Exit Nodes. Клас Persistence фіксує спроби закріплення: створення backdoor-юзерів, нестандартних scheduled tasks, додавання backdoor SSH keys.

| Категорія знахідки | Приклад | Дія |
|--------------------|---------|----|
| Backdoor:EC2/C&CActivity | EC2 → known C2 IP | Isolate EC2, forensics |
| UnauthorizedAccess:IAMUser/TorIPCaller | API з Tor Exit Node | Revoke key, investigate |
| Recon:IAMUser/MaliciousIPCaller | AWS API з known-bad IP | Block IP, investigate IAM |
| CryptoCurrency:EC2/BitcoinTool | EC2 → mining pools | Terminate + investigate |
| Exfiltration:S3/AnomalousBehavior | S3 bulk download | Investigate data access |

Автоматична реакція на GuardDuty findings реалізується через EventBridge → Lambda playbook. GuardDuty підтримує suppression rules — аналог whitelisting — для виключення відомо-легітимних паттернів із генерації findings. Наприклад, якщо тестовий EC2-інстанс регулярно звертається до зовнішнього endpoint для інтеграційних тестів, відповідне GuardDuty finding можна suppressed на рівні конкретного resource ARN або source IP range, щоб уникнути шуму. Важливо задокументувати всі suppressed findings та регулярно переглядати їх актуальність.

GuardDuty Malware Protection (доданий у 2022) розширює можливості сервісу за межі мережевого та API-аналізу: при виявленні підозрілої активності EC2 або ECS GuardDuty автоматично виконує non-disruptive сканування EBS-тому на наявність шкідливого ПЗ, використовуючи ізольований snapshot-процес без зупинки інстансу. GuardDuty S3 Protection аналізує S3 Data Events для виявлення підозрілого доступу до об'єктів, що доповнює мережевий та API-аналіз. Разом ці компоненти перетворюють GuardDuty на найближчий відповідник EDR (Endpoint Detection and Response) у хмарному контексті.

Приклад для compromised EC2:

```hcl
resource "aws_cloudwatch_event_rule" "guardduty_high" {
  name        = "guardduty-high-severity"
  event_pattern = jsonencode({
    source      = ["aws.guardduty"]
    detail-type = ["GuardDuty Finding"]
    detail = { severity = [{ numeric = [">=", 7] }] }
  })
}
```

---

## 16. AWS Inspector, Macie та Detective

Три спеціалізованих AWS-сервіси безпеки доповнюють GuardDuty різними векторами аналізу: Inspector зосереджується на вразливостях, Macie — на захисті персональних даних, Detective — на forensic-розслідуваннях.

AWS Inspector v2 здійснює безперервне сканування на вразливості EC2-інстансів та контейнерних образів Amazon ECR. Для EC2 Inspector використовує Systems Manager (SSM) Agent для аналізу встановленого ПЗ та порівняння із базою CVE (Common Vulnerabilities and Exposures). Для ECR — сканує образи при push або за розкладом. Inspector v2 виставляє числовий Risk Score (0–10) для кожної знахідки, що допомагає пріоритизувати remediation. Критично важливою функцією є відображення reachability (досяжності): Inspector аналізує конфігурацію Security Groups та NACLs, щоб визначити, чи доступна уразлива служба з Інтернету.

Amazon Macie — ML-сервіс для автоматичного виявлення та захисту чутливих даних в Amazon S3. Macie виявляє PII (Personally Identifiable Information): номери кредитних карток, SSN, паспортні дані, адреси, медичні дані (PHI), фінансові дані тощо. Macie також виявляє публічно доступні S3 buckets та buckets з вимкненим шифруванням. Для GDPR-compliance Macie є незамінним інструментом: автоматичне сканування S3-сховищ на наявність даних, що підпадають під GDPR, і генерація findings у Security Hub при виявленні.

Amazon Detective є сервісом forensic-розслідування, що автоматично будує граф взаємозв'язків між AWS-ресурсами, IP-адресами та часовими вікнами на основі CloudTrail та VPC Flow Logs. Замість ручного написання Athena-запитів, Detective надає інтерактивні візуалізації: «Які ресурси спілкувались із цим EC2 за останні 7 днів?», «З яких IP-адрес здійснювались API-виклики від цієї IAM ролі?», «Чи є це IP нормальним для даного ресурсу?». Detective особливо цінний для answer «scope of blast radius» — визначення повного масштабу компромісу.

---

## 17. VPC Flow Logs: мережевий моніторинг

VPC Flow Logs є механізмом захоплення метаданих мережевих потоків (flows) через мережеві інтерфейси EC2, RDS, ENI, NAT Gateway та інших ресурсів у VPC. Потоки VPC Flow Logs не захоплюють вміст пакетів (payload), натомість фіксують metadata кожного TCP/UDP/ICMP-з'єднання: source та destination IP, порт, протокол, тривалість, кількість байтів та пакетів, а також статус (ACCEPT або REJECT).

Формат запису VPC Flow Log виглядає наступним чином:

```
version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes start end action log-status
2 123456789012 eni-0abc1234 10.0.1.15 192.168.1.1 45621 443 6 15 7548 1710498000 1710498060 ACCEPT OK
2 123456789012 eni-0abc1234 198.51.100.42 10.0.1.15 0 22 6 4 240 1710498120 1710498140 REJECT OK
```

Перший рядок показує успішне з'єднання від внутрішнього IP до зовнішнього на порт 443 (HTTPS). Другий — спробу з'єднання ззовні на порт 22 (SSH), яку Security Group заблокувала (REJECT). Аналіз REJECT-записів є цінним для виявлення розвідувальної активності та спроб несанкціонованого доступу.

Аналіз VPC Flow Logs в Amazon Athena дозволяє виявляти мережеві аномалії:

```sql
-- Виявлення port scanning: той самий src_addr → багато різних dst_port
SELECT srcaddr, count(distinct dstport) as ports_scanned
FROM vpc_flow_logs
WHERE action = 'REJECT'
  AND start > unix_timestamp(NOW() - INTERVAL 1 HOUR)
GROUP BY srcaddr
HAVING ports_scanned > 50
ORDER BY ports_scanned DESC;
```

Сценарії виявлення загроз через VPC Flow Logs включають: внутрішнє сканування (lateral movement — одна EC2 сканує інші), незвичний аутбаунд трафік (potential data exfiltration — великі обсяги даних до зовнішніх IP), аномальний DNS трафік (DNS tunneling), з'єднання до відомих зловмисних IP, несподівані мережеві з'єднання між ізольованими підмережами.

GuardDuty автоматично аналізує VPC Flow Logs та генерує знахідки при виявленні аномалій, що звільняє аналітика від необхідності ручно писати SQL-запити. Однак самостійний аналіз через Athena залишається незамінним при forensic-розслідуваннях для відтворення повної картини мережевої активності.

Рекомендована практика зберігання VPC Flow Logs — доставка в обидва сховища одночасно: CloudWatch Logs Group для оперативного аналізу та Metric Filters (наприклад, відстеження зростання REJECT-лічильника), і Amazon S3 для довгострокового архіву та Athena-аналізу. При великих обсягах трафіку (сотні гігабайтів Flow Logs щодобово) рекомендується використовувати Parquet-формат через Kinesis Data Firehose із партиціонуванням за датою та VPC ID — це знижує витрати на Athena-запити в 10–100 разів порівняно з неструктурованими текстовими файлами. Retention period для VPC Flow Logs слід встановлювати відповідно до вимог compliance: PCI DSS вимагає щонайменше 12 місяців, тоді як для forensic-цілей бажано мати 90 днів «онлайн» в CloudWatch Logs для швидкого доступу.

---

## 18. Threat Hunting: проактивне виявлення загроз

Threat Hunting (полювання на загрози) — це проактивна дисципліна пошуку прихованих зловмисників або артефактів у середовищі, які не були виявлені автоматичними системами виявлення. На відміну від реактивного реагування на алерти, Threat Hunting базується на гіпотезах: «Що якщо зловмисник вже присутній у нашій мережі і успішно обходить наші детектори?»

Методологія Threat Hunting включає кілька обов'язкових кроків. Формулювання гіпотези (Hypothesis) — наприклад, «Зловмисник використовує скомпрометовані IAM-credentials для виконання пасивної розвідки». Збір та аналіз даних — CloudTrail за певний period, VPC Flow Logs, CloudWatch Logs. Пошук індикаторів (IoC/IoA пошук) — аномальні паттерни, що відповідають гіпотезі. Верифікація — підтвердження або спростування гіпотези. Документування результатів — або закриття гіпотези, або ескалація до повноцінного інциденту.

Приклад Threat Hunt в AWS-середовищі: гіпотеза — «Зловмисник, що скомпрометував IAM-credentials, зараз виконує S3 discovery». Athena-запит для верифікації:

```sql
SELECT useridentity.arn, sourceipaddress, count(*) as api_count,
       count(distinct eventsource) as services_touched,
       min(eventtime) as first_seen, max(eventtime) as last_seen
FROM cloudtrail_logs
WHERE eventname IN ('ListBuckets', 'ListObjects', 
                    'GetBucketAcl', 'GetBucketPolicy')
  AND eventtime > '2024-03-01'
GROUP BY useridentity.arn, sourceipaddress
HAVING api_count > 100
ORDER BY api_count DESC;
```

Якщо запит повертає IAM-user або роль із нехарактерно великою кількістю S3-discovery операцій — це підтверджує гіпотезу і потребує глибшого розслідування.

Інструменти для Threat Hunting в AWS: Amazon Athena + CloudTrail (SQL-аналіз), CloudWatch Logs Insights (real-time пошук у логах), Amazon Detective (граф-аналіз та forensic timeline), AWS Security Lake (централізоване озеро безпекових даних в OCSF-форматі для multi-source кореляції).

Зрілість Threat Hunting програми оцінюється за так званою Threat Hunting Maturity Model (HMM), що включає п'ять рівнів. Рівень 0 — організація покладається виключно на автоматизовані алерти, активного пошуку немає. Рівень 1 — нерегулярний ситуативний пошук при конкретних підозрах. Рівень 2 — процедурний Threat Hunting за заздалегідь визначеними гіпотезами на регулярній основі. Рівень 3 — інноваційний Threat Hunting з розробкою нових технік та інтеграцією зовнішнього Threat Intelligence. Рівень 4 — автоматизований Threat Hunting з ML-моделями, що безперервно генерують та перевіряють гіпотези. Для більшості організацій реалістичною метою є досягнення рівня 2–3 протягом першого року після впровадження SIEM, з поступовим рухом до рівня 4 при використанні GuardDuty та Amazon Detective як automated hunting компонентів.

---

## 19. SOAR: автоматизація реакції на інциденти

Security Orchestration, Automation and Response (SOAR) — клас технологій, що дозволяє автоматизувати повторювані процеси реагування на інциденти (playbooks), оркеструвати дії між різними системами безпеки (SIEM, firewall, identity provider, ticketing system) та підвищити ефективність SOC. Ключова цінність SOAR: скорочення MTTR з годин до хвилин через автоматизацію рутинних кроків розслідування та ремедіації.

AWS SOAR архітектура базується на EventBridge → Step Functions або Lambda → AWS-сервіси. EventBridge отримує Security Hub findings або GuardDuty notifications, маршрутизує їх за правилами до відповідного Lambda-playbook або Step Functions workflow. Step Functions особливо зручні для складних multi-step playbooks із гілками та паралельними кроками.

Приклад Playbook для compromised IAM Key:

```python
def compromised_iam_key_playbook(event, context):
    finding = event['detail']['findings'][0]
    iam_user = extract_iam_user(finding)
    
    # Step 1: Негайно відкликати access key
    iam.update_access_key(
        UserName=iam_user,
        AccessKeyId=finding['Resources'][0]['Details']['AwsIamAccessKey']['AccessKeyId'],
        Status='Inactive'
    )
    
    # Step 2: Прикріпити Deny-all policy для запобігання подальших дій
    attach_deny_policy(iam_user)
    
    # Step 3: Зберегти snapshot активності для forensic
    save_cloudtrail_snapshot(iam_user, hours_back=24)
    
    # Step 4: Відкрити тікет в Jira/ServiceNow
    create_incident_ticket(severity='HIGH', user=iam_user, finding=finding)
    
    # Step 5: Сповістити SOC
    notify_soc_channel(f"Compromised IAM key for {iam_user} - auto-revoked")
```

Цей playbook виконується за 5–30 секунд автоматично після появи GuardDuty finding — задовго до того, як аналітик SOC навіть відкриє консоль. Це принципово важливо: при компрометації IAM Key зловмисник може діяти дуже швидко, тому автоматична реакція є єдиним способом мінімізувати збитки.

SOAR-рішення в AWS можуть будуватись як native (Lambda + Step Functions), так і з використанням спеціалізованих платформ: AWS Security Hub Automated Response and Remediation (SHARR), Splunk SOAR, Palo Alto XSOAR, або Microsoft Sentinel Playbooks.

---

## 20. Alert Fatigue та пріоритизація алертів

Alert fatigue (втома від алертів) є одним із найбільш поширених і небезпечних явищ у сучасних SOC. Коли аналітик отримує тисячі алертів щодня, більшість із яких є хибними спрацьовуваннями або мають низький пріоритет, він неминуче починає обробляти їх механічно або ігнорувати, що підвищує ризик пропустити справжній інцидент. За даними EY Global Information Security Survey, SOC-аналітики ігнорують у середньому 44% алертів, що надходять.

Причини alert fatigue є системними: надто агресивні пороги у правилах виявлення (занадто чутливі, генерують хибні спрацьовування); відсутність контексту в алерті (незрозуміло, чи є це критичним або норм); дублювання алертів із різних систем (GuardDuty + SIEM + Config виявляють одну і ту саму проблему тричі); відсутність автоматичного закриття очевидно хибних спрацьовувань.

Стратегії зниження шуму включають кілька підходів. Тюнінг правил на основі false positive analysis — аналіз закритих як False Positive алертів за минулий місяць для виявлення патернів та виправлення правил. Контекстуальне збагачення — додавання бізнес-контексту до кожного алерту (чи є цей ресурс critical? Чи є це change-window?). Автоматичне закриття INFORMATIONAL знахідок — Security Hub best practice: findings з severity INFORMATIONAL автоматично закриваються через 30 днів, якщо не підтверджені аналітиком. Suppression Rules — правила ігнорування відомо-дозволених паттернів (наприклад, scheduled job, що завжди запускається о 3 AM і завжди генерує однаковий алерт).

Пріоритизація в Security Hub реалізується через поєднання двох критеріїв: severity finding (CRITICAL/HIGH/MEDIUM/LOW) та workflow status (NEW/NOTIFIED/RESOLVED/SUPPRESSED). Оптимальна черга роботи SOC-аналітика: обробляти спочатку всі CRITICAL + NEW, потім HIGH + NEW, Medium тільки при наявності вільного часу. AWS Security Hub Score (0–100) відображає загальний стан відповідності та допомагає командам безпеки відстежувати прогрес у покращенні постури безпеки.

---

## 21. Моніторинг телекомунікаційних та 5G мереж

Телекомунікаційні мережі характеризуються специфічними протоколами та архітектурними особливостями, що принципово відрізняють їх моніторинг від стандартного ІТ-середовища. Сигналізаційні протоколи SS7, Diameter та SIGTRAN, що забезпечують inter-carrier комунікацію, не мають вбудованих механізмів автентифікації і є джерелом специфічних загроз, невидимих для стандартних SIEM.

CDR (Call Detail Record) Analysis є фундаментальним методом моніторингу безпеки в GSM/PSTN мережах. CDR-записи фіксують: номер абонента (A-party, B-party), час початку та завершення дзвінка, тривалість, тип з'єднання (голос/SMS), IMSI/IMEI, використаний BSC/BTS. Аномалії в CDR-даних є індикатором атак SS7: IMSI Catchers (масові Location Update без реального руху абонента), SIM-card fraud (різкий стрибок міжнародних дзвінків з однієї SIM), clone attack (два телефони з однаковим IMSI), SMS spoofing (A-party підміна).

SS7 Firewall та моніторинг є обов'язковими компонентами захисту телеком-оператора відповідно до стандарту GSMA FS.11. Firewall аналізує кожне SS7-повідомлення та блокує підозрілі, генеруючи Security Events для SIEM. Характерні Sigma-правила для SS7: масові Location Update без відповідних HO (handover) подій; UpdateLocation з незнайомих GT (Global Title) адрес; MAP Cancel Location, що переключає абонента в іншу мережу.

SIP/VoIP моніторинг включає аналіз SIP-повідомлень через Session Border Controller (SBC). SBC лог-файли містять повні SIP-трейси — INVITE, REGISTER, OPTIONS, BYE — з якими виявляються toll fraud (несанкціоновані дорогі дзвінки), SIP brute-force (масові REGISTER-спроби з неправильними паролями), SIP flooding (DoS через INVITE flood). Fail2ban-інтеграція з SBC-логами дозволяє автоматично блокувати атакуючі IP після визначеної кількості невдалих REGISTER-спроб.

5G Core Monitoring базується на стандарті логування 3GPP TS 33.501, що визначає обов'язкові події безпеки для кожної Network Function. AMF (Access and Mobility Management Function) логує Authentication, Authorization та Registration Events. SMF (Session Management Function) — PDU Session Establishment та Modification. UDM (Unified Data Management) — subscriber profile access та credential operations. 5G NF генерують структуровані JSON-логи через N-інтерфейси, що безпосередньо передаються до CloudWatch через Kinesis Agent — один pipeline для cloud та edge компонентів. Ключова метрика безпеки: Authentication Success Rate. Падіння нижче 95% є індикатором аномалії, що може вказувати на IMSI Catcher атаку або масовий SIM-card fraud.

---

## 22. SOC: Security Operations Center

Security Operations Center (SOC) — спеціалізований підрозділ або функція, відповідальна за безперервний моніторинг, виявлення, розслідування та реагування на події безпеки в режимі 24/7/365. SOC є операційним ядром системи безпеки організації, де технологічний стек (SIEM, SOAR, Threat Intelligence) зустрічається з людськими аналітиками.

Організаційна структура SOC зазвичай включає трирівневу ієрархію. Tier 1 (Alert Analyst) — перша лінія реагування: обробляє вхідний потік алертів, проводить початкову тріаж (класифікація та пріоритизація) та виконує стандартні runbook-дії для типових інцидентів. Tier 2 (Incident Responder) — розслідує складні інциденти, виконані Tier 1 для ескалації; здійснює forensic-аналіз, containment та eradication. Tier 3 (Threat Hunter / Senior Analyst) — проактивно шукає приховані загрози, розробляє нові правила виявлення, досліджує нові TTP (Tactics, Techniques, Procedures), виконує red team exercises. Threat Intelligence Team — збирає та аналізує дані про загрози: IoC (Indicator of Compromise) списки, TIP (Threat Intelligence Platform), CVE-відстеження.

Ключові метрики операційної ефективності SOC: MTTD (Mean Time to Detect) — час від початку інциденту до його виявлення, цільовий показник ≤30 хвилин; MTTR (Mean Time to Respond/Remediate) — час від виявлення до нейтралізації, ціль ≤4 годин для HIGH; False Positive Rate — відсоток хибних спрацьовувань серед усіх алертів, ціль <20%; Alert Coverage — відсоток алертів, оброблених у межах SLA.

Cloud-Native SOC в AWS базується на комбінації GuardDuty (ML-детекція), Security Hub (централізація findings), EventBridge + Lambda (SOAR), OpenSearch Dashboards (візуалізація та пошук), PagerDuty або Opsgenie (on-call management). Перевага Cloud SOC: відсутність потреби у власній SIEM-інфраструктурі — GuardDuty + Security Hub = managed SIEM із ML за фіксованою вартістю, що масштабується автоматично зі зростанням середовища.

Одним із ключових операційних викликів SOC є управління on-call escalation. Більшість серйозних атак відбуваються у вихідні дні або в нічний час, коли SOC укомплектований мінімально. Ефективна on-call система включає чіткі escalation paths (Tier 1 → Tier 2 → CISO за визначений час без відповіді), автоматизовану paging інтеграцію (PagerDuty / Opsgenie підключені до Security Hub через Lambda), та runbooks для найпоширеніших сценаріїв, доступних аналітику з мобільного пристрою. Важливим елементом є також психологічна безпека команди: SOC-аналітики зазнають значного когнітивного навантаження та burnout через тривалий аналіз загроз і постійну готовність до найгіршого сценарію. Ротація ролей між Tier 1, Tier 2 та Threat Hunter, а також регулярні tabletop exercises дозволяють підтримувати мотивацію та знижувати рівень стресу.

Метрика Analyst Utilization Rate показує, яка частка робочого часу SOC-аналітика витрачається на реальну аналітичну роботу проти адміністративних задач. Лідируючі SOC досягають 70%+ корисної аналітичної завантаженості завдяки SOAR-автоматизації, що бере на себе triage та початкове збагачення алертів, залишаючи людину лише для інтелектуальних рішень вищого порядку.

---

## 23. Compliance моніторинг: PCI DSS, GDPR, ISO 27001

Моніторинг безпеки є не лише технічною необхідністю, але й регуляторною вимогою для організацій у ряді галузей. Розуміння конкретних вимог кожного стандарту дозволяє будувати систему моніторингу, що одночасно забезпечує операційну безпеку та дотримання compliance.

PCI DSS v4.0 (Payment Card Industry Data Security Standard) визначає детальні вимоги до логування та моніторингу для організацій, що обробляють платіжні картки. Requirement 10 повністю присвячено логуванню та моніторингу: необхідне логування всіх доступів до Cardholder Data Environment (CDE); збереження логів не менше 12 місяців (3 місяці доступні для негайного аналізу); захист логів від неавторизованих змін; щоденний огляд логів систем, що обробляють картки; автоматизовані механізми виявлення аномалій. AWS Security Hub має вбудований стандарт PCI DSS v3.2.1, що автоматично перевіряє відповідність конфігурацій AWS-ресурсів вимогам стандарту.

GDPR (General Data Protection Regulation) у контексті моніторингу вимагає: документального підтвердження обробки персональних даних (Article 30 — Records of Processing Activities); логування доступу до персональних даних для забезпечення accountability; повідомлення supervisory authority протягом 72 годин при витоку (Article 33); реалізації Privacy by Design — виключення PII з логів або їх маскування. Amazon Macie є ключовим інструментом GDPR-compliance: автоматично виявляє PII в S3-сховищах та генерує findings у Security Hub для негайного реагування.

ISO 27001 Annex A включає численні контролі, що безпосередньо реалізуються через AWS-інструменти моніторингу: A.12.4 Logging and Monitoring (CloudTrail, CloudWatch), A.16 Information Security Incident Management (Security Hub, SOAR), A.18.1 Compliance with Legal Requirements (AWS Artifact, Audit Manager). AWS Audit Manager автоматично збирає докази відповідності (evidence) для ISO 27001, PCI DSS, SOC 2 та інших стандартів, генеруючи аудиторські звіти безпосередньо з конфігурацій AWS-ресурсів.

NIS2 Directive (Network and Information Security Directive 2, набула чинності у 2023) є новим регуляторним фреймворком ЄС, що розширює кількість секторів, які підпадають під вимоги кібербезпеки, включаючи телекомунікаційний сектор. NIS2 вимагає від операторів суттєвих послуг впровадження заходів управління кіберризиками, обов'язкового повідомлення про значні інциденти протягом 24 годин, проведення регулярних аудитів та тестів на проникнення. З практичної точки зору NIS2 означає, що телеком-оператори в ЄС повинні мати задокументовану систему виявлення інцидентів (SIEM) та реагування (SOAR) як необхідний елемент демонстрації відповідності регулятору. Поєднання AWS Security Hub, Config та Audit Manager дозволяє генерувати необхідну документацію автоматично, суттєво спрощуючи підготовку до аудитів.

---

## 24. Централізований моніторинг Multi-Account AWS

Великі організації, що використовують AWS Organizations із десятками або сотнями AWS-акаунтів, стикаються з принциповою проблемою моніторингу: без централізації кожен акаунт потрібно перевіряти окремо, що унеможливлює кореляцію cross-account атак та подовжує час розслідування.

AWS Organizations та AWS Security Hub реалізують делеговане адміністрування (Delegated Administrator): один Security Tooling Account отримує агреговані findings від усіх member accounts у organizations, що дозволяє SOC-аналітику мати єдину точку видимості для всієї організації. GuardDuty, Config, CloudTrail та IAM Access Analyzer підтримують Organizations-level delegated admin.

AWS Control Tower автоматизує налаштування безпекових guardrails при провіжнингу нових AWS-акаунтів: обов'язковий CloudTrail у всіх регіонах, обов'язковий AWS Config, обов'язковий GuardDuty, централізовані лог-архіви до Log Archive Account. Landing Zone — стандартна архітектурна основа для multi-account setup — включає Management Account, Log Archive Account та Security Tooling Account як мінімально необхідні.

AWS Security Lake (2023) — відносно новий managed service, що агрегує дані безпеки зі всіх акаунтів та регіонів в єдиний Amazon S3-based data lake у стандартизованому форматі OCSF (Open Cybersecurity Schema Framework). OCSF — відкритий стандарт, що дозволяє нормалізувати події безпеки з різних постачальників (AWS, third-party security tools) у єдину схему, уможливлюючи cross-source кореляцію без попередньої трансформації.

```hcl
resource "aws_securitylake_data_lake" "main" {
  configuration {
    region = "eu-west-1"
    encryption_configuration {
      kms_key_id = aws_kms_key.security_lake.arn
    }
    lifecycle_configuration {
      expiration { days = 365 }
    }
  }
}
```

---

## 25. Виявлення інсайдерських загроз та ML-аномалії

Інсайдерські загрози (Insider Threats) є одними з найбільш важко виявних та потенційно найбільш деструктивних загроз для організації. На відміну від зовнішніх атак, інсайдер вже має легітимний доступ до систем і даних, що значно ускладнює відрізнення зловмисних дій від нормальних робочих операцій. Статистика IBM підтверджує: 20% усіх витоків даних пов'язані з інсайдерами.

Типи інсайдерів розрізняють за мотивацією та намірами. Злочинний інсайдер — умисно краде або шкодить даним задля фінансової вигоди, конкурентного шпіонажу або помсти. Недбалий інсайдер — не має злочинного умислу, але через помилку або недостатню обізнаність спричиняє інцидент (наприклад, випадково публікує S3 bucket). Компрометований інсайдер — легітимний співробітник, акаунт якого захоплено зовнішнім зловмисником.

Індикатори підозрілої поведінки, що виявляються через CloudTrail та UEBA: масове завантаження даних перед звільненням (Download before resignation pattern — різке зростання S3 GetObject за 2 тижні до закінчення tranzaction); доступ до незвичних ресурсів (Database Administrator, що раптово читає HR-документи); часові аномалії (системний адміністратор, що звично виконує завдання лише в робочий час, о 2 AM виконує масові DELETE-операції); використання особистих хмарних сховищ або email для передачі корпоративних даних.

AWS IAM Access Analyzer виявляє ресурси, до яких є публічний або cross-account доступ, що є потенційним вектором для інсайдерської ексфільтрації (витоку через публічно відкриті ресурси). CloudWatch Logs Insights дозволяє реалізувати регулярні автоматичні аудиторські запити для виявлення IAM-entities з мінімальною активністю (які не потрібно мати активних) або навпаки — з раптовим різким зростанням активності.

Технічні контролі для запобігання інсайдерським загрозам включають принцип мінімальних привілеїв (Least Privilege) на рівні IAM Permission Boundaries та Service Control Policies (SCP) в AWS Organizations, обов'язкову MFA для консольного та API-доступу, just-in-time (JIT) привілейований доступ через AWS IAM Identity Center (SSO) замість постійних IAM Keys, та data loss prevention (DLP) контролі — Macie для S3, VPC Endpoints без доступу до Інтернету для ізоляції workloads. Окремо слід зазначити People-процесні контролі: обов'язкові вихідні перевірки (offboarding checklist) з негайним відкликанням усіх прав при звільненні, background перевірки персоналу при прийомі на роботу з доступом до критичних систем, а також програми підвищення обізнаності (security awareness training), що знижують ризик недбалих інсайдерів — найпоширенішого класу внутрішніх загроз.

Машинне навчання у security реалізується через кілька підходів AWS. CloudWatch Anomaly Detection (описаний раніше) застосовує ML для числових метрик. GuardDuty використовує supervised та unsupervised ML для аналізу мережевого та API-трафіку. Amazon SageMaker може бути використаний для побудови custom ML-детекторів — наприклад, для класифікації CloudTrail-активності як normal/suspicious на основі feature engineering (година доби, тип дії, попередня активність того ж entity, геолокація тощо). Amazon Detective використовує graph ML (graph neural networks) для виявлення аномальних зв'язків між AWS-ресурсами.

---

## 26. DevSecOps: моніторинг у CI/CD pipeline

DevSecOps (Development, Security, Operations) є підходом до розробки ПЗ, де практики безпеки інтегруються безпосередньо в CI/CD pipeline на кожному етапі, замість традиційного «перевірки безпеки в кінці». З погляду моніторингу DevSecOps означає, що безпека повинна відстежуватись не лише в production, але й у pipeline, де вразливості можна виявити та виправити значно дешевше.

Стек Security Observability в CI/CD охоплює кілька шарів перевірок. SAST (Static Application Security Testing) — аналіз вихідного коду на вразливості (AWS CodeGuru Security, SonarQube, Checkmarx). SCA (Software Composition Analysis) — аналіз вразливостей у залежностях (Dependabot, Snyk, OWASP Dependency Check). DAST (Dynamic Application Security Testing) — тестування запущеного застосунку на вразливості (OWASP ZAP, Burp Suite Pro). Container Scanning — аналіз Docker-образів перед push до ECR (Amazon Inspector v2, Trivy, Grype). IaC Security — аналіз Terraform/CloudFormation на небезпечні конфігурації (Checkov, tfsec, AWS CloudFormation Guard).

Приклад інтеграції безпеки в AWS CodePipeline:

```hcl
resource "aws_codepipeline" "secure_pipeline" {
  stage {
    name = "Security-Checks"
    action {
      name     = "SAST-CodeGuru"
      category = "Test"
      provider = "CodeBuild"
      configuration = {
        ProjectName = aws_codebuild_project.sast.name
      }
    }
    action {
      name     = "Container-Scan"
      category = "Test"
      provider = "CodeBuild"
      configuration = {
        ProjectName = aws_codebuild_project.inspector_ecr.name
      }
    }
  }
}
```

Runtime Security Monitoring у production охоплює відстеження того, що фактично виконується в рамках workloads: abnormal process execution (Lambda або ECS task запускає bash або curl — потенційний backdoor), незвичний мережевий трафік від сервісного акаунту, аномальні системні виклики (контейнер намагається підключитись до host network). AWS CloudTrail Data Events для Lambda функцій та ECS/EKS забезпечують видимість операцій на цьому рівні.

Ще одним важливим виміром DevSecOps є Security Metrics in Development — відстеження метрик безпеки в контексті розробки. Це включає: кількість відкритих Critical/High CVE у залежностях (з SBOM — Software Bill of Materials), відсоток коду, покритого SAST-сканером, час від виявлення вразливості до patch deployment (Mean Time to Patch, MTTP). MTTP є критичним KPI для відповідності NIS2 та ISO 27001: CVSS ≥9.0 (Critical) повинні патчитися в межах 72 годин, CVSS 7.0–8.9 (High) — 7 днів. AWS Inspector v2 автоматично відстежує ці метрики та публікує їх у Security Hub для централізованого моніторингу, що дозволяє замкнути петлю зворотного зв'язку між командою розробки та службою безпеки в рамках єдиного dashboard.

---

## 27. Практичний кейс: Виявлення атаки в AWS

Розглянемо реалістичний сценарій складної атаки та детальну хронологію її виявлення через описані інструменти. Сценарій: Compromised EC2 Instance → Data Exfiltration з S3.

Стадія 1 (Initial Access, T1566 — Phishing): зловмисник надсилає фішинговий лист розробнику. Розробник переходить за посиланням, яке завантажує credentials з Instance Metadata Service (IMDS). На EC2-інстансі виконується команда `curl http://169.254.169.254/latest/meta-data/iam/security-credentials/prod-role`. **Виявлення:** GuardDuty finding `CredentialAccess:EC2/MetadataCredentialTheft` — IMDS credentials були використані з IP-адреси, відмінної від тієї EC2.

Стадія 2 (Discovery, T1580 — Cloud Infrastructure Discovery): зловмисник використовує скрадені credentials для розвідки: `aws s3 ls`, `aws iam list-users`, `aws ec2 describe-instances --region us-east-1`. **Виявлення:** CloudWatch Metric Filter спрацьовує на велику кількість Read-Only API-викликів з незнайомої IP. GuardDuty finding `Recon:IAMUser/MaliciousIPCaller`.

Стадія 3 (Exfiltration, T1530 — Data from Cloud Storage): зловмисник масово скачує файли з S3: `aws s3 sync s3://customer-data-bucket/ /tmp/data/ --no-sign-request` (400 GB). **Виявлення:** CloudTrail S3 Data Events фіксують 50 000 GetObject-запитів з незнайомого IP за 30 хвилин. Macie генерує finding про аномальний доступ до PII-даних. GuardDuty finding `Exfiltration:S3/AnomalousBehavior` — volume significantly exceeds baseline.

| Час | Подія | Виявлення |
|-----|-------|-----------|
| 14:00 | Phishing → IMDS theft | GuardDuty: MetadataCredentialTheft |
| 14:05 | Discovery API calls | CloudWatch Alarm: UnauthorizedAPICalls |
| 14:10 | S3 bulk download | GuardDuty: Exfiltration + Macie |
| 14:11 | EventBridge trigger | Lambda: revoke credentials, block IP |
| 14:13 | Containment завершено | Security Hub: HIGH finding resolved |
| 14:30 | SOC розслідування | Detective: full timeline |

Автоматична реакція EventBridge → Lambda за 1 хвилину після початку ексфільтрації: відкликання тимчасових IAM credentials (роль EC2 instance profile перебудована з новою version), блокування source IP в NACL, ізоляція EC2 (Security Group замінена на deny-all), збереження знімку CloudTrail та VPC Flow Logs для forensic. MTTD: 10 хвилин. MTTR (автоматизована containment): 1 хвилина після detection.

---

## 28. Типові помилки та чеклист налаштування моніторингу

Практика впровадження систем моніторингу безпеки в реальних організаціях виявляє ряд типових антипатернів, що систематично знижують ефективність захисту навіть при наявності правильних інструментів. Розуміння цих пасток є необхідною умовою побудови дійсно ефективної системи.

| Антипатерн | Наслідок | Рішення |
|-----------|---------|---------|
| CloudTrail тільки в одному регіоні | Blind spot у інших регіонах | `is_multi_region_trail = true` |
| CloudWatch Logs без retention | Необмежені витрати на зберігання | Встановити retention 90–365 днів |
| GuardDuty вимкнено або в тестовому режимі | Відсутність ML-детекції | Увімкнути + перевірити підписку на findings |
| Security Hub без стандартів | Немає compliance-оцінки | Увімкнути AWS Foundational Best Practices |
| WAF тільки в COUNT mode | Трафік проходить без блокування | Перевести критичні правила в BLOCK |
| Немає алерту на Root Account | Критична активність непомічена | CloudWatch Alarm на RootAccountUsageCount |
| S3 Data Events вимкнено | Не видно скачування даних | Увімкнути для critical buckets |
| Логи без шифрування | Потенційне розкриття sensitive даних | KMS encryption для S3 + CloudWatch Logs |
| Alert Queue без SLA | Алерти оброблюються роками | Встановити SLA: CRITICAL ≤15 хв |

Особлива небезпека криється у «Security Theatre» — ілюзії безпеки, коли інструменти формально наявні (GuardDuty enabled, Security Hub увімкнений), але насправді неефективні через відсутність тюнінгу, SLA або автоматизації реагування. GuardDuty, що генерує findings, які ніхто не читає, є гірше, ніж його відсутність: він створює false sense of security.

### ✅ Чеклист налаштування моніторингу AWS

- ☑ CloudTrail: Multi-region Trail → S3 з log file validation + encryption
- ☑ CloudTrail: Log Insights query для daily review або автоматизований аналіз
- ☑ GuardDuty: Enabled в усіх регіонах, findings → Security Hub
- ☑ Security Hub: Enabled, стандарти AWS FSBP + CIS увімкнені
- ☑ Config: Enabled з managed rules + Auto Remediation
- ☑ VPC Flow Logs: Enabled для всіх VPC → CloudWatch Logs / S3
- ☑ CloudWatch Alarms: Root usage, unauthorized API, SG changes
- ☑ S3 Data Events: Enabled для sensitive buckets
- ☑ Macie: Enabled, sensitive data discovery job на критичних buckets
- ☑ Inspector v2: Enabled для EC2 та ECR
- ☑ Log retention policies: встановлені для всіх Log Groups
- ☑ KMS encryption: для S3 log bucket та CloudWatch Log Groups
- ☑ SOAR playbooks: EventBridge → Lambda для HIGH+ findings
- ☑ Security Hub findings SLA: документований та дотримується

---

## Підсумок

Восьма лекція курсу присвячена одній із найбільш практично значущих тем у сфері кібербезпеки хмарних середовищ: моніторингу, логуванню та виявленню загроз. Центральною ідеєю лекції є принцип «неможливо захистити те, чого не бачиш» — без повноцінної видимості всього технологічного стека організація фактично діє наосліп, дозволяючи зловмисникам залишатись непоміченими в середньому 194 дні.

Критично важливим є усвідомлення взаємозалежності між окремими компонентами системи моніторингу. CloudTrail без відповідних Athena-запитів або Metric Filters є лише сховищем даних, що не генерує жодних сигналів безпеки. GuardDuty без налаштованого EventBridge-маршрутизатора та SOAR-playbook'ів перетворюється на джерело findings, яких ніхто не читає. Security Hub без встановлених SLA для обробки CRITICAL/HIGH знахідок — лише красивий дашборд. Лише повна інтеграція всіх компонентів у єдину функціональну систему, де кожна знахідка автоматично запускає відповідний workflow, і де є команда, яка цей workflow контролює та вдосконалює, забезпечує реальний захист.

Технологічна основа моніторингу безпеки в AWS будується на взаємодоповнюючому стеку сервісів: CloudTrail забезпечує аудит всіх API-викликів; VPC Flow Logs — мережеву видимість; GuardDuty застосовує ML для виявлення аномалій у поведінці ресурсів та ідентифікацій; Security Hub агрегує findings у єдину пріоритизовану чергу; Config гарантує безперервну відповідність конфігурацій визначеним стандартам; Inspector та Macie забезпечують захист вразливостей та персональних даних відповідно. CloudWatch Logs та CloudWatch Alarms забезпечують операційну видимість та тригери для автоматизації.

Концептуальна рамка MITRE ATT&CK пов'язує всі технічні компоненти в єдину стратегію виявлення, орієнтовану на поведінкові паттерни реальних зловмисників, а не на сигнатури конкретних інструментів. Правила виявлення у стандарті Sigma, прив'язані до технік ATT&CK, забезпечують переносимість між різними SIEM-платформами. SOAR-автоматизація через EventBridge та Lambda дозволяє скоротити MTTR з годин до хвилин, що є принципово важливим для протидії сучасним атакам.

Специфіка телекомунікаційної галузі додає до цієї картини необхідність моніторингу сигналізаційних протоколів SS7, Diameter та SIP, для яких стандартні хмарні інструменти не є достатніми. 5G Core на AWS-інфраструктурі відкриває можливість уніфікованого pipeline логування: JSON-логи NF через Kinesis Agent → CloudWatch → GuardDuty → Security Hub, що дозволяє операторам мати єдину точку видимості для cloud та edge компонентів.

Compliance-аспект є невід'ємною частиною стратегії моніторингу: PCI DSS Requirement 10, GDPR Article 33, ISO 27001 A.12.4 та NIS2 Directive встановлюють конкретні вимоги до логування, зберігання та моніторингу, дотримання яких одночасно підвищує рівень безпеки та захищає організацію від регуляторних санкцій. AWS Audit Manager автоматизує збір доказів відповідності, перетворюючи compliance з рутинної ручної роботи на безперервний автоматизований процес.
