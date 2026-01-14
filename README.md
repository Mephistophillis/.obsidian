# Obsidian Plugins Configuration

Документация по установленным плагинам и их настройкам в данном Obsidian vault.

## Содержание

- [Core Plugins (Встроенные)](#core-plugins-встроенные)
- [Community Plugins (Установленные)](#community-plugins-установленные)
- [Common Workflows](#common-workflows)

---

## Core Plugins (Встроенные)

### Активированные плагины

| Плагин | Описание |
|--------|----------|
| **File Explorer** | Навигация по файлам и папкам |
| **Global Search** | Поиск по всему vault |
| **Switcher** | Быстрое переключение между файлами (`Ctrl+O`) |
| **Graph View** | Визуализация связей между заметками |
| **Backlinks** | Показывает ссылки на текущую заметку |
| **Canvas** | Визуальный редактор для mind maps и схем |
| **Outgoing Links** | Показывает ссылки из текущей заметки |
| **Tag Pane** | Панель тегов в сайдбаре |
| **Properties** | Свойства (YAML frontmatter) для заметок |
| **Page Preview** | Предпросмотр заметок при наведении |
| **Daily Notes** | Создание ежедневных заметок |
| **Note Composer** | Комбинирование содержимого нескольких заметок |
| **Command Palette** | Выполнение команд (`Ctrl+P`) |
| **Editor Status** | Показывает статус редактора (режимы) |
| **Bookmarks** | Закладки для быстрого доступа |
| **Outline** | Содержание текущей заметки |
| **Word Count** | Подсчет слов и символов |
| **File Recovery** | Восстановление файлов |
| **Sync** | Синхронизация (требует Obsidian Sync) |
| **Bases** | Создание баз данных и представлений |

---

## Community Plugins (Установленные)

### 1. TaskNotes - Управление задачами

**Главная папка задач**: `TaskNotes/Tasks`

**Основные функции**:
- Создание и управление задачами через HTTP API
- Календарь, канбан доски и списки задач
- Статусы: `none`, `open`, `in-progress`, `done`
- Приоритеты: `none`, `low`, `normal`, `high`
- Зависимости между задачами (blockedBy/blocking)
- Подзадачи, проекты, контексты, теги
- Pomodoro таймер

#### API Configuration

- **API URL**: `http://localhost:8080/api/tasks`
- **Port**: 8080
- **Authentication**: Токен не настроен
- **API включен**: `true`

#### Task Creation via API

Критически важно использовать Python с UTF-8 кодировкой для создания задач с кириллицей:

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
import urllib.request
import json
import sys

sys.stdout.reconfigure(encoding='utf-8')

API_URL = "http://localhost:8080/api/tasks"

task = {
    "title": "Заголовок задачи",
    "status": "open",
    "priority": "normal",
    "scheduled": "2026-01-14",
    "projects": ["[[Project/Link]]"],
    "details": "Описание задачи",
    "contexts": ["@work"],
    "tags": ["#urgent"]
}

data = json.dumps(task).encode('utf-8')
req = urllib.request.Request(
    API_URL,
    data=data,
    headers={"Content-Type": "application/json; charset=utf-8"}
)
with urllib.request.urlopen(req) as response:
    result = json.loads(response.read().decode('utf-8'))
    print(f"Created: {result}")
```

#### Natural Language Input Triggers

- `#` → Tags (теги)
- `@` → Contexts (контексты)
- `+` → Projects (проекты)
- `*` → Status (статусы)

#### Task Views Commands

| Команда | Файл |
|---------|------|
| `open-calendar-view` | `TaskNotes/Views/mini-calendar-default.base` |
| `open-kanban-view` | `TaskNotes/Views/kanban-default.base` |
| `open-tasks-view` | `TaskNotes/Views/tasks-default.base` |
| `open-agenda-view` | `TaskNotes/Views/agenda-default.base` |

#### Task File Structure

```yaml
---
status: open|in-progress|done
priority: none|low|normal|high
due: YYYY-MM-DD
scheduled: YYYY-MM-DD
projects:
  - "[[Project/Link]]"
contexts:
  - "@context"
tags:
  - task
  - "#tag"
dateCreated: 2026-01-14T10:00:00.000Z
dateModified: 2026-01-14T10:00:00.000Z
---

# Task Title

Task description or details...
```

---

### 2. Templater Obsidian - Шаблоны заметок

**Папка шаблонов**: `Templates/`

**Автоматическое применение шаблонов**:
- `Journal/01 - Daily/` → `Templates/Daily Note Template.md`
- `Journal/02 - Weekly/` → `Templates/Weekly Note Template.md`

**Настройки**:
- `trigger_on_file_creation`: `true`
- `auto_jump_to_cursor`: `true`
- `enable_folder_templates`: `true`
- `syntax_highlighting`: `true`

#### Основные функции Templater

Создание динамического контента с помощью JavaScript:

```javascript
<% tp.date.now() %>           // Текущая дата
<% tp.file.title %>          // Заголовок файла
<% tp.file.folder %>         // Папка файла
<% tp.web.daily_quote %>     // Цитата дня
<% tp.system.prompt() %>     // Запрос ввода у пользователя
```

#### Примеры использования

**Daily Note Template** (`Templates/Daily Note Template.md`):
```markdown
---
date: <% tp.date.now("YYYY-MM-DD") %>
weekday: <% tp.date.now("dddd") %>
---

# <% tp.date.now("YYYY-MM-DD") %>

## Журнал

## Заметки

## Список задач на сегодня

## Новые знания
```

---

### 3. Dataview - Запросы данных

**Настройки**:
- `refreshEnabled`: `true`
- `refreshInterval`: `2500ms`
- `enableInlineDataview`: `true` (префикс: `=`)
- `enableDataviewJs`: `true` (ключевое слово: `dataviewjs`)

#### Inline Queries

Синтаксис: `= query`

```markdown
= this.file.name                 // Имя файла
= this.file.tags                 // Теги файла
= date(this.file.ctime)          // Дата создания
= this.file.tasks.filter(t => !t.completed)  // Невыполненные задачи
```

#### Dataview Query Language (DQL)

```markdown
```dataview
TABLE status, due, scheduled
FROM "TaskNotes/Tasks"
WHERE status = "open" AND scheduled <= date(today)
SORT due ASC
```

#### DataviewJS

```markdown
```dataviewjs
const tasks = dv.pages("#task")
  .filter(p => p.status === "in-progress")
  .sort(p => p.due);

dv.table(["Задача", "Статус", "Срок"],
  tasks.map(t => [t.file.link, t.status, t.due])
);
```

---

### 4. Obsidian Git - Git синхронизация

**Настройки**:
- `autoSaveInterval`: `5` (секунд)
- `autoPullOnBoot`: `true`
- `pullBeforePush`: `true`
- `basePath`: `shared`
- `syncMethod`: `merge`

#### Коммиты

**Шаблон сообщения коммита**:
```
vault backup: {{date}}
```

#### Рабочий процесс

1. Создать репозиторий Git в папке `Shared/`
2. Настроить remote repository
3. Автоматические коммиты каждые 5 секунд после изменений
4. Автоматический pull при запуске Obsidian
5. Ручной push через панель Git

#### Основные команды

- `Ctrl+P` → `Git: Commit` → Создать коммит
- `Ctrl+P` → `Git: Push` → Отправить на remote
- `Ctrl+P` → `Git: Pull` → Получить изменения
- `Ctrl+P` → `Git: View file history` → История изменений

---

### 5. Obsidian Jira Issue - Интеграция с Jira

**Конфигурация**:
- **Host**: `https://jira.beauit.com`
- **Authentication**: Bearer Token
- **Inline issue prefix**: `JIRA:`
- **Account alias**: "Maks L"

#### Использование

**Вставка задачи Jira**:
```markdown
JIRA:PROJ-123
```

**Автоматическое отображение**:
- Ключ задачи (PROJ-123)
- Заголовок (Summary)
- Статус (Status)
- Приоритет (Priority)
- Назначенный (Assignee)
- Ссылка на Jira

#### Custom Fields (отображаются)

- `Разработка` (id: 10000)
- `Рейтинг` (id: 10100)
- `Спринт` (id: 10101)
- `Ссылка на эпик` (id: 10102)
- `Статус эпика` (id: 10103)
- `Имя эпика` (id: 10104)
- `Цвет эпика` (id: 10105)
- `Team` (id: 10600)
- `Parent Link` (id: 10601)

---

### 6. Obsidian Kanban - Канбан доски

**Настройки**:
- `new-note-folder`: `Work/Jira`
- `show-add-list`: `true`

#### Создание канбан доски

Создать файл с содержанием:

```markdown
---
kanban-plugin: basic
---

## To Do
- [ ] Задача 1
- [ ] Задача 2

## In Progress
- [x] Задача 3
- [ ] Задача 4

## Done
- [x] Задача 5
```

#### Функции

- Перетаскивание задач между колонками
- Фильтрация по тегам и датам
- Сортировка по приоритету
- Интеграция с TaskNotes (если задачи из TaskNotes)

---

### 7. Calendar - Календарь

**Настройки**:
- `weekStart`: `monday`
- `wordsPerDot`: `250`
- `localeOverride`: `system-default`

#### Функции

- Визуальный календарь в сайдбаре
- Создание ежедневных заметок по клику на дату
- Показывает даты с существующими заметками
- Интеграция с Daily Notes plugin

#### Создание заметки через календарь

1. Открыть панель Calendar
2. Кликнуть на нужную дату
3. Автоматически создается заметка в `Journal/01 - Daily/`
4. Применяется шаблон `Templates/Daily Note Template.md`

---

### 8. Journals - Управление дневниками

**Настройки**:
- Управление ежедневными и еженедельными записями
- Интеграция с Calendar plugin

#### Функции

- Автоматическое создание записей по расписанию
- Навигация между записями
- Статистика по записям

---

### 9. QuickAdd - Быстрое создание заметок

**Настройки**:
- `enableRibbonIcon`: `false`
- `showCaptureNotification`: `true`

#### Настроенные choices

**New Idea**:
- **Type**: Template
- **Template**: `Templates/Idea Template.md`
- **Folder**: `Ideas`
- **Open in**: Split (vertical)

#### Использование

1. `Ctrl+P` → `QuickAdd: Run Choice` → `New idea`
2. Ввести заголовок идеи
3. Автоматически создается файл в `Ideas/` с шаблоном

#### Idea Template (`Templates/Idea Template.md`)

```markdown
---
type: idea
date: <% tp.date.now("YYYY-MM-DD") %>
---

# <% tp.file.title %>

## Суть идеи

## Контекст

## Детали

## Связи

## Следующие шаги
```

---

### 10. Omnisearch - Мощный поиск

**Настройки**:
- `useCache`: `true`
- `fuzziness`: `1`
- `ribbonIcon`: `true`
- `showExcerpt`: `true`
- `maxEmbeds`: `5`

#### Функции

- Индексирует все содержимое vault
- Поддерживает нечеткий поиск
- Показывает отрывки текста с подсветкой
- Быстрее и точнее, чем стандартный поиск

#### Использование

1. `Ctrl+P` → `Omnisearch: Open Omnisearch`
2. Ввести запрос для поиска
3. Навигация по результатам с клавиатурой

#### Весовые коэффициенты (важность)

- Basename (имя файла): `10`
- Directory (папка): `7`
- H1 (заголовки 1 уровня): `6`
- H2 (заголовки 2 уровня): `5`
- H3 (заголовки 3 уровня): `4`
- Unmarked tags: `2`

---

### 11. Callout Manager - Кастомные callouts

**Настроенные callouts**:

**Calendar**:
- Icon: `lucide-calendar-days`
- Dark theme color: Green (#27aa4e)

**Time**:
- Icon: `lucide-calendar-clock`
- Color: Green (#20b166)

#### Использование

```markdown
> [!calendar]
> Событие: 2026-01-14

> [!time]
> Время: 14:30
```

---

### 12. Obsidian Icon Folder - Иконки для папок

**Функции**:

- Добавление иконок к папкам в File Explorer
- Визуальная навигация по структуре vault

#### Использование

1. Правый клик на папку в File Explorer
2. `Set icon` → Выбрать иконку
3. Иконка отображается рядом с названием папки

---

### 13. Beautitab - Кастомная главная вкладка

**Функции**:

- Кастомизация домашней страницы
- Отображение статистики vault
- Быстрый доступ к часто используемым заметкам

---

### 14. Leader Hotkeys Obsidian - Vim-style горячие клавиши

**Функции**:

- Режим leader (`Space` по умолчанию)
- Быстрые команды через сочетания клавиш
- Поведение похоже на Vim leader keys

#### Примеры

- `Space f` → Найти файл
- `Space s` → Поиск
- `Space t` → Открыть task view

---

## Common Workflows

### Создание ежедневной заметки

1. Открыть панель Calendar
2. Кликнуть на дату
3. Автоматически создается файл в `Journal/01 - Daily/YYYY-MM-DD.md`
4. Применяется шаблон `Templates/Daily Note Template.md`

### Создание задачи

#### Через TaskNotes Plugin

1. `Ctrl+P` → `TaskNotes: Create Task`
2. Заполнить поля (title, status, priority, scheduled, etc.)
3. Сохранить

#### Через API (для автоматизации)

Использовать Python скрипт с UTF-8 кодировкой (см. раздел TaskNotes)

### Создание идеи

1. `Ctrl+P` → `QuickAdd: Run Choice` → `New idea`
2. Ввести заголовок идеи
3. Файл создается в `Ideas/` с шаблоном `Templates/Idea Template.md`

### Поиск

- **Omnisearch**: `Ctrl+P` → `Omnisearch: Open Omnisearch` (глобальный поиск)
- **Graph**: Открыть Graph View для визуального поиска связей
- **Tags**: Использовать Tag Pane для навигации по тегам

### Работа с задачами

#### Просмотр задач

- `Ctrl+P` → `TaskNotes: Open Calendar View`
- `Ctrl+P` → `TaskNotes: Open Kanban View`
- `Ctrl+P` → `TaskNotes: Open Tasks View`
- `Ctrl+P` → `TaskNotes: Open Agenda View`

#### Обновление статуса задачи

Нажать на статус в карточке задачи или использовать natural language:
- `*open` → Установить статус "open"
- `*in-progress` → Установить статус "in-progress"
- `*done` → Установить статус "done"

#### Добавление свойств через NLP

```
Задача #work @home +project !high *in-progress
```

### Синхронизация с Git

1. Проверить изменения: `Ctrl+P` → `Git: View file history`
2. Создать коммит: `Ctrl+P` → `Git: Commit`
3. Отправить на remote: `Ctrl+P` → `Git: Push`

Автоматический pull при запуске Obsidian включен.

---

## Additional Resources

### Файлы конфигурации

- `.obsidian/core-plugins.json` - Список активированных core plugins
- `.obsidian/community-plugins.json` - Список установленных community plugins
- `.obsidian/plugins/{plugin-name}/data.json` - Конфигурация конкретного плагина

### Шаблоны

- `Templates/Daily Note Template.md` - Шаблон ежедневной заметки
- `Templates/Weekly Note Template.md` - Шаблон еженедельной заметки
- `Templates/Idea Template.md` - Шаблон идеи

### TaskNotes Views

- `TaskNotes/Views/mini-calendar-default.base` - Мини календарь
- `TaskNotes/Views/kanban-default.base` - Канбан доска
- `TaskNotes/Views/tasks-default.base` - Список задач
- `TaskNotes/Views/agenda-default.base` - Повестка дня
