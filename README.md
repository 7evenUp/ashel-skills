# ashel-skills

Личные skills для AI-агентов: повторяемые рабочие подходы и стиль кода из моих проектов.

## Skills

| Skill | Назначение |
| --- | --- |
| [crypton-frontend-style](skills/crypton-frontend-style/SKILL.md) | React/TypeScript frontend в стиле проектов Nutsfarm, id-frontend, x-landing, events-front и Nutcracker, с учётом вариантов Academy и academy-tma |

Skill описывает организацию компонентов, типизацию пропсов, экспорты, обработчики, Tailwind/SCSS Modules, UI-примитивы, формы и API. Он выбирает подходящий вариант по текущему проекту и не требует доступа к исходным рабочим репозиториям.

## Установка

После публикации этого репозитория на GitHub:

```sh
npx skills add 7evenUp/ashel-skills --skill crypton-frontend-style
```

Установить глобально для Codex:

```sh
npx skills add 7evenUp/ashel-skills --skill crypton-frontend-style --agent codex --global
```

Посмотреть доступные skills без установки:

```sh
npx skills add 7evenUp/ashel-skills --list
```

Для проверки до публикации запустите из корня локального репозитория:

```sh
npx skills add . --list
```

Команда установки требует Node.js/npm. При установке через CLI выберите нужного агента и область установки, если они не указаны флагами.

## Использование

Пример запроса после установки:

```text
Используй $crypton-frontend-style и реализуй страницу профиля в моём стиле.
```

Можно уточнить вариант:

```text
Используй $crypton-frontend-style. Сделай лендинг в духе x-landing:
React, секции и SCSS Modules, без внедрения Redux ради статического контента.
```

Правила находятся в `SKILL.md`. Подробности загружаются по необходимости из `references/patterns.md`, примеры — из `references/examples.md`, основания выводов — из `references/evidence.md`. Файл `agents/openai.yaml` содержит метаданные для Codex.

## Структура

```text
ashel-skills/
  .github/workflows/validate.yml
  .gitattributes
  .gitignore
  README.md
  skills/
    crypton-frontend-style/
      SKILL.md
      agents/openai.yaml
      references/
        patterns.md
        examples.md
        evidence.md
```

Новые skills добавляются в `skills/<skill-name>/SKILL.md`. Поля `name` и `description` в YAML обязательны. Название каталога должно совпадать с `name`. Не размещайте общий `SKILL.md` в корне репозитория: коллекция должна обнаруживаться как отдельные skills.

## Публикация на GitHub

1. Создайте пустой публичный репозиторий `7evenUp/ashel-skills`, без автоматически добавленных README, .gitignore и лицензии.
2. Откройте терминал в корне этой папки. Локальный Git-репозиторий уже инициализирован с веткой `main`.
3. Создайте первый коммит и отправьте его:

```sh
git add .
git commit -m "Add crypton frontend style skill"
git remote add origin https://github.com/7evenUp/ashel-skills.git
git push -u origin main
```

Если `origin` уже настроен, проверьте его через `git remote -v` и не добавляйте повторно. Выбор лицензии оставлен владельцу; файл LICENSE пока не добавлен.

## skills.sh

Отдельно загружать архив на skills.sh не требуется. Источник skill — GitHub-репозиторий. После публикации проверьте удалённое обнаружение и установите skill:

```sh
npx skills add 7evenUp/ashel-skills --list
npx skills add 7evenUp/ashel-skills --skill crypton-frontend-style
```

Согласно [FAQ skills.sh](https://skills.sh/docs/faq), каталог учитывает установки через CLI и добавляет skills в рейтинг на основе анонимной телеметрии. Локальная проверка `--list` не означает, что skill уже опубликован или появился в каталоге. Срок появления на сайте не гарантируется.

Правила формата и обнаружения: [официальный репозиторий skills CLI](https://github.com/vercel-labs/skills#creating-skills).

## Проверка изменений

```sh
git diff --check
npx skills add . --list
```

GitHub Actions выполняет проверку обнаружения skills при push и pull request. Эта проверка подтверждает доступность формата для CLI, но не качество генерируемого frontend-кода. Примеры и инструкции дополнительно сверяйте с реальными задачами.
