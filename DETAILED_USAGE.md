# Подробное руководство по настройке GitHub Metrics

## Содержание
1. [Основные параметры настройки](#основные-параметры-настройки)
2. [Настройка количества анализируемых коммитов](#настройка-количества-анализируемых-коммитов)
3. [Управление блоками визуализации](#управление-блоками-визуализации)
4. [Настройка расписания обновления метрик](#настройка-расписания-обновления-метрик)

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