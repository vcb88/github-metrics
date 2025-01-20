# Подробное руководство по настройке GitHub Metrics

## Содержание
1. [Основные параметры настройки](#основные-параметры-настройки)
2. [Настройка количества анализируемых коммитов](#настройка-количества-анализируемых-коммитов)
3. [Управление блоками визуализации](#управление-блоками-визуализации)
4. [Настройка расписания обновления метрик](#настройка-расписания-обновления-метрик)
5. [Полный список параметров конфигурации](#полный-список-параметров-конфигурации)
6. [Работа с плагинами](#работа-с-плагинами)

## Основные параметры настройки

GitHub Metrics можно настроить несколькими способами:
- Через файл конфигурации `.github/workflows/metrics.yml`
- Через параметры в URL при использовании веб-версии
- Через переменные окружения при запуске в Docker

## Настройка количества анализируемых коммитов

Для настройки количества анализируемых коммитов используются следующие параметры:

```yaml
- name: Metrics embed
  uses: lowlighter/metrics@latest
  with:
    # Максимальное количество коммитов для анализа
    commits_limit: 100
    
    # Глубина анализа (в днях)
    commits_days: 90
```

## Управление блоками визуализации

Каждый блок в метриках можно включить или отключить. Основные блоки:

```yaml
- name: Metrics embed
  uses: lowlighter/metrics@latest
  with:
    # Базовые блоки
    base: header, activity, repositories
    
    # Дополнительные блоки
    plugin_languages: yes
    plugin_achievements: yes
    plugin_isocalendar: yes
    plugin_habits: yes
    plugin_contributors: yes
```

Чтобы отключить блок, установите параметр в `no` или удалите его из конфигурации.

## Настройка расписания обновления метрик

Расписание обновления настраивается в файле workflow через синтаксис cron:

```yaml
name: Metrics
on:
  # Расписание по cron
  schedule:
    - cron: "0 0 * * *"    # Каждый день в полночь
    # - cron: "0 */6 * * *"  # Каждые 6 часов
    # - cron: "0 */12 * * *" # Каждые 12 часов
  
  # Другие триггеры
  workflow_dispatch:  # Позволяет запускать вручную
  push:              # При push в определенные ветки
    branches: [ "main" ]
```

## Примеры конфигураций

### Базовая конфигурация
```yaml
name: Metrics
on:
  schedule:
    - cron: "0 0 * * *"
  workflow_dispatch:
jobs:
  github-metrics:
    runs-on: ubuntu-latest
    steps:
      - name: Metrics embed
        uses: lowlighter/metrics@latest
        with:
          token: ${{ secrets.METRICS_TOKEN }}
          base: header, activity, repositories
          plugin_languages: yes
          commits_limit: 100
```

### Расширенная конфигурация
```yaml
name: Metrics
on:
  schedule:
    - cron: "0 */12 * * *"
  workflow_dispatch:
jobs:
  github-metrics:
    runs-on: ubuntu-latest
    steps:
      - name: Metrics embed
        uses: lowlighter/metrics@latest
        with:
          token: ${{ secrets.METRICS_TOKEN }}
          base: header, activity, repositories
          plugin_languages: yes
          plugin_languages_ignored: html, css
          plugin_languages_skipped: my-test-repo
          plugin_achievements: yes
          plugin_achievements_only: >-
            polyglot, stargazer, sponsor, deployer, member, maintainer
          plugin_isocalendar: yes
          plugin_isocalendar_duration: full-year
          plugin_habits: yes
          plugin_habits_from: 200
          plugin_habits_days: 14
          plugin_habits_facts: yes
          plugin_habits_charts: yes
          commits_limit: 100
          commits_days: 90
```

## Часто задаваемые вопросы

### Как изменить количество отображаемых языков программирования?
Используйте параметр `plugin_languages_limit`:
```yaml
plugin_languages: yes
plugin_languages_limit: 8  # Показывать только top 8 языков
```

### Как исключить определенные репозитории из анализа?
Используйте параметр `plugin_languages_skipped`:
```yaml
plugin_languages_skipped: repo1, repo2, test-repo
```

### Как изменить период анализа активности?
Комбинируйте параметры `commits_days` и `commits_limit`:
```yaml
commits_days: 90    # Анализировать последние 90 дней
commits_limit: 100  # Максимум 100 коммитов
```

## Полный список параметров конфигурации

### Общие параметры
```yaml
- name: Metrics embed
  uses: lowlighter/metrics@latest
  with:
    # Токен GitHub (обязательный)
    token: ${{ secrets.METRICS_TOKEN }}
    
    # Пользователь для анализа (по умолчанию - владелец токена)
    user: username
    
    # Шаблон (по умолчанию - classic)
    template: classic
    
    # Цветовая схема (по умолчанию - light)
    config_display: large
    config_padding: 10%
    config_timezone: Europe/Moscow
    config_order: base.header, isocalendar, languages
    
    # Настройки вывода
    output_action: commit       # commit/gist/none
    output_condition: always    # always/data-changed
    filename: github-metrics.*  # Шаблон имени файла
```

### Параметры для базовых метрик
```yaml
    # Базовые секции
    base: ""                   # header, activity, community, repositories, metadata
    base_indepth: no          # Подробный анализ
    repositories: 100         # Количество анализируемых репозиториев
    repositories_forks: no    # Включать форки
    repositories_affiliations: owner # Типы репозиториев (owner, collaborator, organization_member)
```

### Параметры плагина Languages
```yaml
    plugin_languages: yes
    plugin_languages_analysis_timeout: 15    # Таймаут анализа в секундах
    plugin_languages_categories: markup, programming, prose
    plugin_languages_colors: github          # github/custom
    plugin_languages_limit: 8                # Лимит показываемых языков
    plugin_languages_recent_categories: markup, programming, prose
    plugin_languages_recent_days: 14         # Анализ за последние N дней
    plugin_languages_recent_load: 300        # Количество загружаемых коммитов
    plugin_languages_sections: most-used, recently-used
    plugin_languages_threshold: 0%           # Минимальный % для отображения языка
```

### Параметры плагина Notable
```yaml
    plugin_notable: yes
    plugin_notable_filter: stars:>500        # Фильтр для определения notable
    plugin_notable_from: organization        # Источник (organization, repository)
    plugin_notable_repositories: yes         # Включать репозитории
    plugin_notable_types: commit            # Типы вкладов (commit, issue, pr)
```

### Параметры плагина Habits
```yaml
    plugin_habits: yes
    plugin_habits_charts: yes               # Включить графики
    plugin_habits_charts_type: classic      # classic/graph
    plugin_habits_days: 14                  # Период анализа
    plugin_habits_facts: yes                # Включить факты
    plugin_habits_from: 200                 # Количество событий для анализа
    plugin_habits_languages_limit: 8        # Лимит языков
    plugin_habits_languages_threshold: 0%   # Порог для языков
```

## Работа с плагинами

### Встроенные плагины
GitHub Metrics поставляется с большим набором встроенных плагинов. Каждый плагин можно активировать, добавив соответствующий параметр в конфигурацию:

```yaml
plugin_имя_плагина: yes
```

Основные встроенные плагины:
- 🈷️ Languages (`plugin_languages`)
- 📅 Isocalendar (`plugin_isocalendar`)
- ⏱️ PageSpeed (`plugin_pagespeed`)
- 🏆 Achievements (`plugin_achievements`)
- 🎟️ Discussions (`plugin_discussions`)
- 💡 Habits (`plugin_habits`)
- 👨‍💻 Lines (`plugin_lines`)
- 🎭 People (`plugin_people`)
- 📓 Projects (`plugin_projects`)
- 🎼 Music (`plugin_music`)
- 📌 Topics (`plugin_topics`)
- 🌟 Stars (`plugin_stars`)

### Создание собственных плагинов

Вы можете создать собственный плагин для GitHub Metrics. Для этого:

1. Создайте новую директорию в папке `source/plugins/`
2. Создайте основной файл плагина `index.mjs`
3. Создайте файл метаданных `metadata.yml`

Пример структуры простого плагина:

```js
// source/plugins/myplugin/index.mjs
export default async function(instance, {login, q}) {
  // Логика плагина
  return {
    type: "custom",
    data: {
      // Данные плагина
    }
  }
}
```

```yaml
# source/plugins/myplugin/metadata.yml
name: "My Plugin"
description: "Description of my plugin"
supports:
  - user
  - organization
scopes: []
inputs:
  plugin_myplugin:
    description: Enable my plugin
    type: boolean
    default: no
```

### Использование сторонних плагинов

Для использования стороннего плагина:

1. Добавьте его как подмодуль в вашу копию репозитория:
```bash
git submodule add https://github.com/user/metrics-plugin source/plugins/custom_plugin
```

2. Активируйте плагин в конфигурации:
```yaml
- name: Metrics embed
  uses: lowlighter/metrics@latest
  with:
    plugin_custom_plugin: yes
```

### Советы по работе с плагинами

- Каждый плагин может иметь свои собственные требования к токенам и разрешениям
- Некоторые плагины могут требовать дополнительных токенов (например, для работы с API других сервисов)
- Проверяйте документацию конкретного плагина для понимания всех доступных опций
- При создании собственных плагинов следуйте структуре существующих плагинов для совместимости