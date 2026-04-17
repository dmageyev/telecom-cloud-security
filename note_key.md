# note_key.md — Стандарти оформлення лекційних презентацій

> Цей файл є специфікацією для ШІ-агентів, що створюють або редагують файли `_NN.html` у цьому репозиторії.  
> Всі агенти зобов'язані точно дотримуватися цих вимог, щоб забезпечити однаковий стиль у всіх лекціях.

---

## 1. Загальна структура _NN.html

Кожна презентація є **standalone Reveal.js deck** (CDN-посилання на reveal.js), що складається з:

- Один `<html lang="uk">` файл
- Тема: `black` (`theme/black.css`)
- CDN: `https://cdn.jsdelivr.net/npm/reveal.js/`
- **36 слайдів** (може бути 34–50 залежно від теми)
- Файл розміщується у: `lectures/lecture-NN/_NN.html` (наприклад, `lectures/lecture-08/_08.html`)

---

## 2. Reveal.initialize — параметри

Єдиний стандарт для всіх лекцій (оптимізований для 1920×1080):

```js
Reveal.initialize({
    hash: true,
    transition: 'slide',
    slideNumber: 'c/t',
    center: false,
    width: 1850,
    height: 950,
    margin: 0.04,
    minScale: 0.2,
    maxScale: 1.05,
    scrollActivationWidth: 900
});
```

---

## 3. CSS — базові розміри шрифтів та стилі

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
```

---

## 4. Вимога: прокрутка та вміщення слайду на екрані

### 4.1 Прокрутка (overflow)

Кожен слайд **повинен мати можливість вертикальної прокрутки**, якщо вміст перевищує висоту екрана. Додати до CSS:

```css
/* Allow slide content to scroll vertically when it overflows */
.reveal .slides section {
    overflow-y: auto !important;
    max-height: 100% !important;
    box-sizing: border-box !important;
}
```

### 4.2 Вміщення контенту на екрані

- Основний контент слайду **має вміщатися без прокрутки** при роздільній здатності 1920×1080.
- Якщо контент переповнює слайд по вертикалі, зменшити `font-size` секції:
  ```html
  <section style="font-size:0.86em;">
  ```
  Це масштабує весь контент секції приблизно на 14%.
- Таблиці: `font-size: 0.7em` (вже в CSS)
- Списки в щільних слайдах: `font-size: 0.82em` або `0.75em` inline
- `pre code`: `font-size: 0.8em`

---

## 5. Структура останніх двох слайдів

### Слайд N-1: Типові помилки / Антипатерни + Чеклист

Передостанній слайд **завжди** має формат:
- Ліва колонка: **❌ Антипатерни** — таблиця з помилками, наслідками та рішеннями
- Права колонка: **✅ Чеклист** — список пунктів з позначками `☑`

Приклад структури:
```html
<section>
    <h2>Типові помилки та антипатерни</h2>
    <div class="two-col">
        <div>
            <h3>❌ Антипатерни</h3>
            <table>
                <thead><tr><th>Помилка</th><th>Наслідок</th><th>Рішення</th></tr></thead>
                <tbody>
                    <tr><td>...</td><td>...</td><td>...</td></tr>
                </tbody>
            </table>
        </div>
        <div>
            <h3>✅ Чеклист</h3>
            <ul style="font-size:0.8em;">
                <li>☑ ...</li>
            </ul>
        </div>
    </div>
</section>
```

### Останній слайд: Підсумок

Останній слайд **завжди** містить:
- Заголовок: `<h2>Підсумок лекції</h2>`
- Ліва колонка: теги-ключові концепції, згруповані по частинах лекції (`<span class="tag">`, `<span class="tag-green">`)
- Права колонка:
  - Блок **Золоте правило** або ключовий висновок у `.box` з `border-color:#238636`
  - Рекомендовані ресурси (`<ul>` зі списком посилань/документів)
  - Навігаційний блок з посиланнями ← попередня / → наступна лекція

Приклад навігаційного блоку:
```html
<div class="box" style="border-color:#444; font-size:0.75em; margin-top:0.4em;">
    📊 35 слайдів &nbsp;·&nbsp; Лекція N з 11 &nbsp;·&nbsp;
    ← <a href="../lecture-NN/_NN.html" style="color:#58a6ff;">Лекція N-1: Назва</a> &nbsp;·&nbsp;
    → Лекція N+1: Назва
</div>
```

---

## 6. Кнопка "Конспект" (Notes Button) — повна специфікація

### 6.1 HTML-розмітка кнопки та панелі

Розміщується **безпосередньо перед `<script>`**, після закриваючого `</div>` слайдів:

```html
<!-- ===== Notes Button & Panel ===== -->
<button id="notes-btn" onclick="toggleNotesPanel()" title="Конспект до слайду">📄</button>
<div id="notes-panel">
    <div id="notes-panel-header">
        <span id="notes-panel-title">📄 Конспект лекції</span>
        <button id="notes-panel-close" onclick="toggleNotesPanel()" title="Закрити">✕</button>
    </div>
    <div id="notes-panel-body">Оберіть слайд для перегляду конспекту.</div>
</div>
```

### 6.2 CSS кнопки та панелі

```css
#notes-btn {
    position: fixed; bottom: 25px; right: 40px; z-index: 10000;
    width: 30px; height: 30px; border-radius: 50%;
    background: #1f6feb; border: none; cursor: pointer;
    font-size: 1.1em; color: #fff; box-shadow: 0 2px 8px rgba(0,0,0,0.5);
    display: flex; align-items: center; justify-content: center;
}
#notes-btn:hover { background: #388bfd; }
#notes-panel {
    position: fixed; top: 0; right: -420px; width: 410px; height: 100vh;
    background: #0d1117; border-left: 2px solid #1f6feb;
    z-index: 9999; overflow-y: auto; transition: right 0.3s ease;
    padding: 0; box-shadow: -4px 0 18px rgba(0,0,0,0.6);
}
#notes-panel.open { right: 0; }
#notes-panel-header {
    display: flex; align-items: center; justify-content: space-between;
    padding: 12px 16px; background: #161b22;
    border-bottom: 1px solid #30363d; position: sticky; top: 0;
}
#notes-panel-title { color: #58a6ff; font-size: 0.9em; font-weight: bold; }
#notes-panel-close {
    background: none; border: none; color: #8b949e;
    cursor: pointer; font-size: 1.2em; padding: 0 4px;
}
#notes-panel-close:hover { color: #f0f6fc; }
#notes-panel-body {
    padding: 16px; color: #c9d1d9; font-size: 0.85em; line-height: 1.6;
}
#notes-panel-body h4 { color: #58a6ff; margin: 0.8em 0 0.3em; }
#notes-panel-body p { margin: 0.4em 0; }
#notes-panel-body ul { padding-left: 1.4em; margin: 0.3em 0; }
#notes-panel-body li { margin-bottom: 0.2em; }
#notes-panel-body code { background: #161b22; padding: 1px 4px; border-radius: 3px; color: #79c0ff; font-size: 0.92em; }
#notes-panel-body strong { color: #e8b04b; }
```

### 6.3 JavaScript — SLIDE_NOTES та обробники

```js
const SLIDE_NOTES = [
    null,          // 0 — титульний слайд (без конспекту)
    null,          // 1 — план лекції (без конспекту)
    `...`,         // 2 — перший змістовий слайд
    `...`,         // 3
    // ... (по одному рядку на кожен слайд)
    null,          // останній — підсумок (без конспекту)
];

let notesPanelOpen = false;
function toggleNotesPanel() {
    const panel = document.getElementById('notes-panel');
    notesPanelOpen = !notesPanelOpen;
    panel.classList.toggle('open', notesPanelOpen);
    if (notesPanelOpen) updateNotesContent();
}
function updateNotesContent() {
    const idx = Reveal.getIndices().h;
    const body = document.getElementById('notes-panel-body');
    const title = document.getElementById('notes-panel-title');
    const content = SLIDE_NOTES[idx];
    title.textContent = `📄 Слайд ${idx + 1} / ${SLIDE_NOTES.length}`;
    body.innerHTML = content ||
        '<p style="color:#8b949e;font-style:italic;">Конспект для цього слайду відсутній.</p>';
}
Reveal.on('slidechanged', () => { if (notesPanelOpen) updateNotesContent(); });
```

---

## 7. Вимоги до вмісту SLIDE_NOTES

### 7.1 Загальні правила

- Масив `SLIDE_NOTES` має **рівно стільки елементів, скільки слайдів** у презентації (індекси 0..N-1).
- Слайд 0 (титульний) і слайд 1 (план лекції) — завжди `null`.
- Останній слайд (підсумок) — завжди `null`.
- Кожен змістовий слайд (індекси 2..N-2) **повинен мати повний академічний конспект**.

### 7.2 Структура одного запису SLIDE_NOTES

Кожен запис — це HTML-рядок у backtick template literal:

```html
<h4>N. Назва теми слайду</h4>
<p>Перший абзац — визначення або ключова концепція (2-4 речення).</p>
<p>Другий абзац — деталі, механізми, класифікація.</p>
<table style="font-size:0.88em;border-collapse:collapse;width:100%">
    <tr><th>...</th><th>...</th></tr>
    <tr><td>...</td><td>...</td></tr>
</table>
<p>Третій абзац — практичне застосування, AWS-реалізація або стандарти.</p>
<ul>
    <li><strong>Термін:</strong> пояснення</li>
</ul>
```

**Мінімальний обсяг**: 1500–3000 символів HTML на слайд.

**Обов'язкові елементи**:
- `<h4>` з номером та назвою теми
- 3–6 абзаців `<p>` з академічним текстом
- За наявності порівнянь або параметрів — `<table>`
- Ключові терміни виділені `<strong>`

### 7.3 ⚠️ КРИТИЧНО: Екранування в template literals

**Всі** входження `${...}` всередині backtick-рядків `SLIDE_NOTES` **мають бути екрановані** як `\${...}`.

Якщо цього не зробити — браузер отримає `ReferenceError`, і сторінка відобразиться білим екраном.

Правило: при конвертації тексту notes_NN.md у SLIDE_NOTES — автоматично замінювати всі `${` на `\${`.

**Приклад:**
```js
// ❌ ПОМИЛКА — викликає ReferenceError:
`<code>aws s3 sync ${bucket}/ /tmp/</code>`

// ✅ ПРАВИЛЬНО — екрановано:
`<code>aws s3 sync \${bucket}/ /tmp/</code>`
```

### 7.4 Backtick (`) у HTML-рядках

Якщо конспект містить backtick-символ у тексті, його також треба екранувати: `` \` ``.

---

## 8. Вимоги до пояснень до слайдів

### 8.1 Принцип відповідності слайду

Кожен запис `SLIDE_NOTES[idx]` **пояснює саме той слайд**, якому відповідає індекс `idx`. Заголовок `<h4>` має збігатися з темою слайду.

### 8.2 Академічний стиль

Конспект до слайду — це **теоретичний академічний текст**, а не маркована шпаргалка:
- Повні речення з поясненнями причинно-наслідкових зв'язків
- Посилання на стандарти та документи (ISO 27001, PCI DSS, NIST, GSMA, 3GPP, RFC)
- Статистика з джерелами (IBM DBIR, Verizon, Cloudflare, NETSCOUT тощо)
- Порівняльні таблиці, якщо тема передбачає порівняння сервісів або методів
- Практична інтерпретація теоретичного матеріалу в контексті AWS або телеком-мереж

### 8.3 Розподіл контенту між великими секціями

Якщо тема одного розділу notes_NN.md охоплює 2+ слайди — розподілити контент розділу між відповідними слайдами (перша половина → перший слайд, друга → другий).

### 8.4 Заборони

- **Не** використовувати прості маркери без пояснень (замість `<li>GuardDuty` — `<li><strong>GuardDuty</strong> — сервіс ML-виявлення аномалій...`)
- **Не** дублювати текст між сусідніми слайдами
- **Не** писати конспект коротше 1000 символів (занадто мало для академічного конспекту)

---

## 9. Референсний шаблон — повний _NN.html

```html
<!doctype html>
<html lang="uk">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Презентація: [Назва лекції]</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js/dist/reveal.css">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js/dist/theme/black.css">
    <script src="https://cdn.jsdelivr.net/npm/reveal.js/dist/reveal.js"></script>
    <style>
        /* === Базові стилі === */
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
        /* Прокрутка */
        .reveal .slides section {
            overflow-y: auto !important;
            max-height: 100% !important;
            box-sizing: border-box !important;
        }
        /* Notes Button */
        #notes-btn {
            position: fixed; bottom: 25px; right: 40px; z-index: 10000;
            width: 30px; height: 30px; border-radius: 50%;
            background: #1f6feb; border: none; cursor: pointer;
            font-size: 1.1em; color: #fff; box-shadow: 0 2px 8px rgba(0,0,0,0.5);
            display: flex; align-items: center; justify-content: center;
        }
        #notes-btn:hover { background: #388bfd; }
        #notes-panel {
            position: fixed; top: 0; right: -420px; width: 410px; height: 100vh;
            background: #0d1117; border-left: 2px solid #1f6feb;
            z-index: 9999; overflow-y: auto; transition: right 0.3s ease;
            padding: 0; box-shadow: -4px 0 18px rgba(0,0,0,0.6);
        }
        #notes-panel.open { right: 0; }
        #notes-panel-header {
            display: flex; align-items: center; justify-content: space-between;
            padding: 12px 16px; background: #161b22;
            border-bottom: 1px solid #30363d; position: sticky; top: 0;
        }
        #notes-panel-title { color: #58a6ff; font-size: 0.9em; font-weight: bold; }
        #notes-panel-close {
            background: none; border: none; color: #8b949e;
            cursor: pointer; font-size: 1.2em; padding: 0 4px;
        }
        #notes-panel-close:hover { color: #f0f6fc; }
        #notes-panel-body {
            padding: 16px; color: #c9d1d9; font-size: 0.85em; line-height: 1.6;
        }
        #notes-panel-body h4 { color: #58a6ff; margin: 0.8em 0 0.3em; }
        #notes-panel-body p { margin: 0.4em 0; }
        #notes-panel-body ul { padding-left: 1.4em; margin: 0.3em 0; }
        #notes-panel-body li { margin-bottom: 0.2em; }
        #notes-panel-body code { background: #161b22; padding: 1px 4px; border-radius: 3px; color: #79c0ff; font-size: 0.92em; }
        #notes-panel-body strong { color: #e8b04b; }
    </style>
</head>
<body>
    <div class="reveal">
        <div class="slides">
            <!-- СЛАЙД 1: Титульний -->
            <section> ... </section>
            <!-- СЛАЙД 2: План лекції -->
            <section> ... </section>
            <!-- СЛАЙДИ 3..N-1: Змістові слайди -->
            <!-- ... -->
            <!-- СЛАЙД N-1: Антипатерни + Чеклист -->
            <section> ... </section>
            <!-- СЛАЙД N: Підсумок -->
            <section> ... </section>
        </div>
    </div>

    <!-- Notes Button & Panel -->
    <button id="notes-btn" onclick="toggleNotesPanel()" title="Конспект до слайду">📄</button>
    <div id="notes-panel">
        <div id="notes-panel-header">
            <span id="notes-panel-title">📄 Конспект лекції</span>
            <button id="notes-panel-close" onclick="toggleNotesPanel()" title="Закрити">✕</button>
        </div>
        <div id="notes-panel-body">Оберіть слайд для перегляду конспекту.</div>
    </div>

    <script>
        Reveal.initialize({
            hash: true,
            transition: 'slide',
            slideNumber: 'c/t',
            center: false,
            width: 1850,
            height: 950,
            margin: 0.04,
            minScale: 0.2,
            maxScale: 1.05,
            scrollActivationWidth: 900
        });

        const SLIDE_NOTES = [
            null, // 0 — титульний
            null, // 1 — план
            `<h4>1. Назва теми</h4><p>...</p>`,
            // ... (всі слайди)
            null, // останній — підсумок
        ];

        let notesPanelOpen = false;
        function toggleNotesPanel() {
            const panel = document.getElementById('notes-panel');
            notesPanelOpen = !notesPanelOpen;
            panel.classList.toggle('open', notesPanelOpen);
            if (notesPanelOpen) updateNotesContent();
        }
        function updateNotesContent() {
            const idx = Reveal.getIndices().h;
            const body = document.getElementById('notes-panel-body');
            const title = document.getElementById('notes-panel-title');
            const content = SLIDE_NOTES[idx];
            title.textContent = `📄 Слайд ${idx + 1} / ${SLIDE_NOTES.length}`;
            body.innerHTML = content ||
                '<p style="color:#8b949e;font-style:italic;">Конспект для цього слайду відсутній.</p>';
        }
        Reveal.on('slidechanged', () => { if (notesPanelOpen) updateNotesContent(); });
    </script>
</body>
</html>
```

---

## 10. Чеклист перед збереженням _NN.html

- [ ] `Reveal.initialize` містить всі параметри з розділу 2
- [ ] CSS містить `overflow-y: auto !important` для `.reveal .slides section`
- [ ] Кнопка `#notes-btn` розміщена правильно (bottom:25px, right:40px)
- [ ] Панель `#notes-panel` шириною 410px, початкова позиція `right:-420px`
- [ ] `SLIDE_NOTES` має рівно стільки елементів, скільки слайдів
- [ ] Перший і другий елементи `SLIDE_NOTES` — `null`
- [ ] Останній елемент `SLIDE_NOTES` — `null`
- [ ] Всі `${...}` у SLIDE_NOTES екрановані як `\${...}`
- [ ] Передостанній слайд: Антипатерни + Чеклист
- [ ] Останній слайд: Підсумок з тегами, ресурсами та навігацією
- [ ] Загальна кількість слайдів: 34–50
