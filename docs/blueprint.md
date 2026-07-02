# СхемоВиз — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Telegram-бот для генерации визуализаций электрических схем микросхем на основе структурированных текстовых описаний или netlist-файлов. Поддерживает валидацию входных данных, выбор стиля визуализации и экспорт в PNG/SVG/PDF.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- professional engineers
- electronics developers

## Success criteria

- Генерация точных визуализаций схем по netlist/DSL-входу
- Обработка ошибок валидации с понятными сообщениями
- Экспорт векторных форматов для профессиональной документации

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Открыть главное меню с опциями загрузки netlist/ввода описания
- **Загрузить netlist** (button, actor: user, callback: upload:netlist) — Загрузить файл netlist в формате CSV/DSL
  - inputs: netlist_file
  - outputs: parser_report
- **Ввести описание** (button, actor: user, callback: input:description) — Ввести структурированный текстовый формат описания
  - inputs: text_description
  - outputs: parser_report
- **Настройки визуализации** (button, actor: user, callback: settings:visualization) — Выбрать стиль, масштаб и опции отображения
  - inputs: style, scale
  - outputs: visualization_preview
- **История проектов** (button, actor: user, callback: history:projects) — Просмотреть прошлые визуализации и проекты
  - inputs: project_id
  - outputs: visualization_files

## Flows

### Создание визуализации
_Trigger:_ /start

1. Загрузка netlist/ввод описания
2. Парсинг и валидация
3. Выбор настроек визуализации
4. Генерация PNG/SVG/PDF
5. Отправка результатов

_Data touched:_ project, component, netlist, visualization

### Управление проектами
_Trigger:_ history:projects

1. Показать список проектов
2. Выбрать проект
3. Показать детали проекта
4. Опции: повторная генерация/удаление

_Data touched:_ project, visualization

### Валидация входных данных
_Trigger:_ upload:netlist

1. Парсинг файла
2. Проверка целостности (контакты, соединения)
3. Отчет об ошибках/предупреждениях

_Data touched:_ netlist, component

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **project** _(retention: persistent)_ — Проект пользователя с входными данными и историей визуализаций
  - fields: id, user_id, netlist, components, visualizations, created_at
- **component** _(retention: session)_ — Микросхема с определением контактов и функций
  - fields: id, pin_numbers, pin_names, voltage_levels
- **netlist** _(retention: session)_ — Список соединений между контактами
  - fields: id, connections, validation_report
- **visualization** _(retention: persistent)_ — Сгенерированный файл визуализации и метаданные
  - fields: id, project_id, format, file_url, validation_errors
- **user** _(retention: persistent)_ — Telegram-аккаунт инженера с тарифным планом
  - fields: id, telegram_id, subscription_type, usage_count

## Integrations

- **Telegram** (required) — Базовое взаимодействие: команды, файлы, уведомления
- **Telegram Payments** (required) — Обработка подписок и платежей
- **Email/Webhook** (optional) — Опциональная доставка результатов (платный тариф)
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Управление проектами (удаление, повторная генерация)
- Настройка стилей визуализации
- Выбор тарифного плана
- Управление уведомлениями (вебхук/email)

## Notifications

- Уведомление о готовности визуализации в чат
- Оповещение о превышении лимитов бесплатного тарифа

## Permissions & privacy

- Данные проектов хранятся 90 дней по умолчанию
- Пользователь может запросить удаление проекта в любое время
- Доступ к данным ограничен владельцем проекта

## Edge cases

- Неправильный формат netlist/DSL
- Превышение лимита генераций в бесплатном тарифе
- Сложные короткие замыкания, требующие ручной проверки

## Required tests

- Проверка корректности генерации схемы из netlist-файла
- Тестирование валидации с ошибочными входными данными
- Проверка экспорта в SVG/PDF с высоким разрешением

## Assumptions

- Формат входных данных: структурированный netlist/DSL
- Базовая валидация топологии, без детального электрического анализа
- Telegram-аккаунт используется для аутентификации
