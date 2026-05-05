# Bible-RST
Библия на русском языке в [синодальном переводе](https://ru.wikipedia.org/wiki/%D0%A1%D0%B8%D0%BD%D0%BE%D0%B4%D0%B0%D0%BB%D1%8C%D0%BD%D1%8B%D0%B9_%D0%BF%D0%B5%D1%80%D0%B5%D0%B2%D0%BE%D0%B4) для использования в [Obsidian](https://obsidian.md/).
- Основан на модуле RST+ от прекрасного приложения [MyBible](https://mybible.zone/ru/)
- Один файл - одна глава.
- Навигация по книгам и главам с помощью встроенного функционала Bases.
- Текст оформлен таким образом, чтобы можно было использовать внтренние ссылки на конкретные стихи.

## Навигация по Библии
Открыв файл Библия.base, выбираем представление в левом верхнем углу (Ветхий или Новый Завет). Видим главы, рассортированные по книгам. 
> Таким образом, отображение структуры Библии не зависит от того, какой порядок сортировки используется в вашем хранилище. 

## Snippet для удобного отображения текста
Каждый стих - это отдельный маркированный абзац. Следовательно, между ними будут большие интервалы. Для удобства чтения можно использовать snippet, который будет применять уменьшенный отступ только к текстам Библии.

В настройках хранилища выберите Оформление > Фрагменты CSS кода. Нажмите на иконку папки. Откроется папка, в которой нужно создать файл bible-spacing.css с таким содержимым: 

<details>
  <summary>bible-spacing.css</summary>
  
  ```
.bible {
    --p-spacing: 0.2rem !important; 
}

.bible .markdown-rendered p,
.bible .markdown-preview-view p {
    margin-top: 0.1em !important;
    margin-bottom: 0.1em !important;
    margin-block-start: 0.1em !important;
    margin-block-end: 0.1em !important;
}

.bible .el-p {
    margin-top: 0 !important;
    margin-bottom: 0 !important;
    padding-top: 0 !important;
    padding-bottom: 0 !important;
}

.bible .cm-line:empty,
.bible .cm-line:has(br) {
    line-height: 0.1em !important;
    min-height: 0.1em !important;
    padding: 0 !important;
    margin: 0 !important;
}
```
</details>
После этого закрываем папку, жмём на иконку Обновить и активируем соданный сниппет.

## Быстрое цитирование в заметку
Есть возможность буквально одним действием создавать такие ссылки в ваши заметки:
<img width="675" height="253" alt="Снимок экрана 2026-05-05 в 11 57 38" src="https://github.com/user-attachments/assets/36a8af30-5154-4ba7-984f-1c30bb9d3a6c" />
- Для этого понадобится установить плагин [Templater](https://github.com/SilentVoid13/Templater).
- В настройках плагина указываем папку, где будут храниться скрипты (например, создайте в хранилище папку Templates и укажите её).
- В навигаторе Obsidian откройте созданную папку Templates, добавьте туда заметку "Копировать стихи" и вставьте содержимое:

<details>
  <summary>Копировать стихи</summary>
  
  ```
<%*
// 1. Берем выделенный текст
let sel = tp.file.selection();

// 2. СРАЗУ возвращаем текст на место в Библии
tR += sel;

if (!sel) {
new Notice("Вы ничего не выделили!");
return;
}

// 3. Получаем имя файла
let fileName = tp.file.title;
let fileParts = fileName.match(/(.+?)\s+(\d+)/);

if (!fileParts) {
new Notice("Это не файл Библии!");
return;
}

let book = fileParts[1].trim();
let chapter = parseInt(fileParts[2]);

// 4. Ищем номера стихов в выделении
let verseRegex = /^v(\d+)/g;
let matches;
let verses = [];

while ((matches = verseRegex.exec(sel)) !== null) {
verses.push(parseInt(matches[1]));
}

if (verses.length === 0) {
new Notice("В выделении нет меток стихов! Захватите конец строки с ^v");
return;
}

verses.sort((a, b) => a - b);
let firstVerse = verses[0];
let lastVerse = verses[verses.length - 1];

// 5. Формируем красивое название (Римлянам 12:6-8)
let alias = ${book} ${chapter}:${firstVerse};
if (firstVerse !== lastVerse) {
alias += -${lastVerse};
}

// 6. ОЧИЩАЕМ И ФОРМАТИРУЕМ ТЕКСТ ЦИТАТЫ
// Удаляем символы ^v... в конце строк (они нужны только в исходнике)
let cleanText = sel.replace(/\s*^v\d+/g, "");

// Разбиваем текст на строки, убираем пустые, к каждой добавляем префикс "> "
let quoteLines = cleanText.split('\n')
.filter(line => line.trim().length > 0)
.map(line => > ${line.trim()})
.join('\n');

// 7. СОБИРАЕМ ФИНАЛЬНЫЙ БЛОК (CALLOUT)
// Заголовок выноски со ссылкой (используем #^v для надежности)
let calloutHeader = >[!quote] [[${fileName}#^v${firstVerse}|${alias}]];

// Итоговый текст для буфера обмена
let finalOutput = ${calloutHeader}\n${quoteLines}\n;

// 8. Копируем в буфер обмена
await navigator.clipboard.writeText(finalOutput);
new Notice("УСПЕХ! Скопирована цитата: " + alias);
%>
```
</details>

Далее возвращаемся в настройки плагина Templater. В разделе Template Hotkeys выбираем наш скрипт, и жмём плюсик. Нас перебрасывает в настройки горячих клавиш. Ищем в конце списка "Templater: Insert Копировать стихи" и задаём этому действию сочетание клавиш (например, Cmd + Shift + C).

Использование: выделяем фрагмент текста Библии > зажимаем комбинацию на клавиатуре > открываем рабочую заметку > вставляем текст, оформленный в виде цитаты. 

На смартфоне действия аналогичны. Только вместо установки сочетания клавиш переходим в Настройки > Мобильный инструментарий > в поиск вводим "Templater: Insert Копировать стихи" и добавляем это действие в панель над клавиатурой.
>Важно: для выполнения скрипта должен быть активен режим редактирования, а не чтения.
