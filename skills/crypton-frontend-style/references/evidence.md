# Основания и границы профиля

Исследован локальный снимок коллекции проектов Crypton от 18 октября 2026 года. Пути ниже относительны корню исходной коллекции проектов; они нужны для сверки первоисточника, а не являются обязательными зависимостями skill. Правила и примеры включены в сам skill.

Приоритет задан пользователем:

1. Nutsfarm, id-frontend, x-landing, events-front, nutcracker-front-main — основной материал.
2. Academy, academy-tma — вторичные варианты.
3. Остальные проекты — обзор для выявления исключений, без переноса их отличий в основной профиль.

Это профиль наблюдаемого кода, не доказательство личного авторства каждого файла: история авторов/коммитов не анализировалась. Полный аудит поведения приложений и выполнение их сборок в задачу исследования стиля не входили. Подробно прочитаны характерные компоненты, API, состояние, формы, UI-примитивы и конфигурации; остальные исходники использованы для структуры и механической сверки экспортов.

## Сверка распространённости экспортов

Обход `src` всех семи основных/вторичных проектов. Колонки экспортов — число TSX-файлов с текстовыми признаками `export default` и `export const/function/{`; это не AST-анализ, категории могут пересекаться. Общий счёт TS/TSX включает декларации, стили — только `*.module.scss/css`.

| Проект | TS/TSX + module styles | TSX | С default export | С named value export |
| --- | ---: | ---: | ---: | ---: |
| nutsfarm | 402 | 308 | 277 | 37 |
| id-frontend | 229 | 171 | 151 | 20 |
| x-landing | 62 | 32 | 26 | 5 |
| events-front | 103 | 74 | 67 | 7 |
| nutcracker-front-main/nutcracker-front-main | 32 | 16 | 11 | 4 |
| academy | 574 | 423 | 287 | 129 |
| academy-tma | 413 | 256 | 24 | 226 |

Вывод: default export — полезное умолчание для обычного компонента основных проектов; именованные экспорты academy-tma — выраженное локальное правило. Эти числа не доказывают частоту других практик.

## Основные источники

### Nutsfarm

- `nutsfarm/package.json`, `.prettierrc`, `tsconfig.app.json`, `tailwind.config.js`: React 18, Vite, TypeScript, Tailwind 3, `@/*`, `semi: false`, Tailwind formatter; специфичные mobile/height breakpoints и шрифты.
- `src/modules/Tasks/index.tsx`: предметный модуль, inline типы аргументов, именованные типы для сложных данных, RTK Query hooks, алиасы данных, `useMemo`, локальные таблицы групп.
- `src/modules/Tasks/components/Task.tsx`, `PartnerGroup.tsx`: inline-пропсы/пересечения типов, обычные условия и обработчики, конкретные UI-блоки, Tailwind и местные ассеты.
- `src/modules/Friends/components/ReferralLinks.tsx`: допустимое исключение `export default function`, controlled inputs, отдельные loading-алиасы, `.unwrap()`, sonner, Motion.
- `src/modules/Tasks/TasksLoadingScreen.tsx`: отдельный loading-компонент с повторением геометрии карточек.
- `src/shared/ui/Button.tsx`, `LargeButton.tsx`: `forwardRef`, нативные пропсы, условный `appearance`, внешние className, декоративные слои; именованные exports.
- `src/shared/ui/DialogModal.tsx`: Radix-композиция, `displayName`, portal в `#dialogs`, Telegram safe area. Контейнер и safe area — локальная интеграция, не общий шаблон каждого диалога.
- `src/redux/api/task.ts`, `src/types/task.ts`: `injectEndpoints`, payload/response, строковые unions статусов, tags и hooks.
- `src/redux/store.ts`, `src/redux/hooks.ts`: configureStore/combineReducers, typed selector/dispatch.
- `src/lib/cn.ts`, `src/lib/i18n.ts`, `src/components/Providers.tsx`: cn, RU/EN ресурсы, инфраструктура в provider-композиции.

### id-frontend

- `id-frontend/.prettierrc`: `semi: false`, `trailingComma: es5`, `quoteProps: as-needed`, сортировка `cn/clsx/cva`.
- `src/modules/auth/AuthContainer/components/EmailLogin/index.tsx`: локальная форма, RHF/Zod, `z.infer`, TanStack `useMutation`, AxiosError, inline-пропсы, ранний выход при pending, несколько состояний после отправки.
- `src/modules/auth/AuthContainer/api/index.ts`: отдельный Axios API в изолированном auth-модуле. Обоснование не навязывать RTK Query абсолютно всем запросам.
- `src/shared/ui/Button.tsx`: CVA, VariantProps, `appearence`, `isLoading`, `forwardRef`, default export, условный scale.
- `src/modules/header/components/ThemeToggle.tsx`: Radix Switch, Redux, `dark:`, `data-state`, условные vw-классы.
- `src/routes/Main.tsx`: композиция modules, SkeletonLayout, query flags, cn и условный scale.
- `src/redux/api/points.ts`, `src/redux/root-tabs/slice.ts`, `src/redux/hooks.ts`: доменные endpoints, invalidation tags, простой slice, typed hooks.
- `src/lib/classNames.ts`: тот же cn с другим путём; не нужно создавать `lib/cn.ts` поверх существующего.

### x-landing

- `x-landing/package.json`, `src/App.tsx`, `src/sections/index.ts`: React/Vite, явная композиция секций, re-export секций.
- `src/sections/Hero/Hero.tsx`: default export стрелочного компонента, clsx, SCSS module, inline SVG-хелперы в том же файле, старый SVGR import.
- `src/sections/About/About.tsx`, `About.module.scss`: `useInView`, явные Motion props, вложенный SCSS, px/vw и локальные breakpoint overrides.
- `src/sections/Team/Team.tsx`, `Team.module.scss`: локальные компоненты, данные из `teamData.ts`, модалки вокруг карточек, CSS keyframes, responsive layouts.
- `src/_mixins.scss`: точные media-границы из patterns.md.
- `src/hooks/useMedia.tsx`: собственный предметный hook. Его конкретная подписка — пример реализации, а не предписание копировать работу с resize.

### events-front

- `events-front/src/pages/Main/index.tsx`: явная композиция локальных секций, Motion scroll/transform для декора.
- `src/pages/Quest/components/Inventory/InventoryItem.tsx`: inline-пропсы, локальный open state, cn, статическая карта редкости, композиция InfoDialog/ClaimDialog.
- `src/pages/Quest/components/Inventory/ClaimDialog.tsx`: controlled Radix dialog, `.unwrap()`, response state, конкретные сообщения по кодам ошибок, loader в кнопке.
- `src/redux/api/core.ts`, `inventory.ts`: общий RTK API и инъекция endpoints, credentials, публичные endpoints, query/mutation tags.
- `tailwind.config.js`: собственные брейкпоинты, цвета, шрифты и keyframes. В этом проекте строки классов не всегда отсортированы; не следует объявлять один ручной порядок авторским законом.

### nutcracker-front-main

Реальный корень: `nutcracker-front-main/nutcracker-front-main`, не соседний Vue-проект `nutcrackers`.

- `package.json`: React 18/Vite/Tailwind 3, RTK Query, i18next, TON.
- `src/components/Tabs/Tab2.tsx`: локальные шаги, query flags, early returns для подключения/загрузки/ошибки, Skeleton, предметная транзакционная логика.
- `src/shared/OutlinedButton.tsx`: компактная `forwardRef`-обёртка, `ComponentProps`, cn и className последним.
- Структура `components`, `shared`, `redux/api`, `types`: основание не добавлять тяжёлую модульную архитектуру в небольшой проект.

## Вторичные источники

### Academy

- `academy/.prettierrc`, `package.json`: сохранение без semicolon, es5 trailing commas, Tailwind 4 через CSS, Vitest и отдельный build-check.
- `src/pages/AuthorWorkshop/Subpages/CreateNew/TabsContent/Tab2/modals/CreateModule.tsx`: inline-пропсы, controlled поля, конкретный async submit, trim, pending guard, обновление slice, диалоговые/типографические примитивы.
- `src/utils/api.ts`, `src/constants/apiErrors.ts`: helpers/коды ошибок; наличие helper не доказывает строгую runtime-валидацию, его типы надо проверять по задаче.
- `src/utils/hooks/useDebounce.tsx`: небольшой generic-hook с очисткой timeout.

### academy-tma

- `academy-tma/package.json`, `.prettierrc`, `eslint.config.js`: React 19, Tailwind 4, TanStack Router; `import type`, simple-import-sort по слоям, pnpm, Vitest.
- `src/widgets/CoursesCards/ui/CoursesCards.tsx`: named export, Props interface, сущности через public API, Icon namespace, переводы с namespace, route constants.
- `src/features/ReviewCourseDialog/ui/ReviewCourseDialog.tsx`: именованный компонент, Dialog primitives из shared, конкретная логика оценки, типы API.
- `src/entities/user/api/user.ts`: baseApi injection в сущности, типизированные query/mutation/infiniteQuery, optimistic patch с undo при ошибке.
- `src/entities/course/index.ts`: public exports типов, hooks и actions; `entities/user/api/user.ts` использует межсущностный `@x/user`.
- `src/shared/ui/Button/Button.tsx`: ComponentProps без forwardRef, локальный buttonVariants, обязательные appearance/size.
- `src/shared/api/getApiErrorCode.ts`: локальный helper ошибок. Не считать cast внутри helper полноценным сужением неизвестного значения.
- `src/app/routes/library.tsx`: маршруты через `createRoute`, а не предположенную file-based генерацию.

## Обзор остальных проектов

Просмотрены manifests/структура и выборочные исходники, без того же уровня подробности:

| Проект | Наблюдение и вес |
| --- | --- |
| battleroyal-frontend | React 19/Tailwind 4, слои, прямой Radix; `src/features/manage-settings/LanguageSwitcher.tsx` подтверждает близкий UI-почерк. Не меняет приоритет основных проектов. |
| nutsfarm-admin | React/Vite, Redux, RHF/Zod, Radix; `src/main.tsx` с Providers/Toaster. Подтверждает экосистему, не диктует архитектуру продукта. |
| calendar-frontend | React, отдельные api/reducers/services, SCSS; `src/helpers/Helpers.tsx` использует semicolon и `I`-типы. Отличия не стали основным стилем. |
| frontend и tools/frontend | Next 15, TanStack Query, Zustand, отдельные http/client/server; `src/client/PreloadResources.tsx` с одинарными кавычками и semicolon. Сходные деревья не считались двумя независимыми подтверждениями. |
| prime-landing | Next Pages, SCSS, существующий provider/API-стек; просмотрен `pages/_document.tsx`. Не основание внедрять Next во все новые лендинги. |
| in-prime | Next/SCSS, собственные hooks; `hooks/useSwiperReactive.tsx`. Архитектурный вариант, не общий default. |
| graniverse-landing | Next/Tailwind; `components/TeamMemberCard.tsx` с обычными функциями, Next Image/Link и локальными типами. Подтверждает допустимость вариантов объявления компонента. |
| nutcrackers | Vue 3/Pinia/composables, `src/App.vue`. Исключён из React-профиля; не путать с приоритетным Nutcracker. |
| crypton-docs | Astro/Starlight: документация, учитывались manifest/структура, а не авторские React-паттерны. |
| analytics | Небольшая TS-библиотека; просмотрен `src/index.ts`. Вспомогательный код, не источник структуры frontend-приложения. |

## Что намеренно не стало стилевым требованием

- Подавления TypeScript в ClaimDialog/ReferralLinks и неподтверждённые casts в error helpers.
- Закомментированный старый код, повторные литералы в union, случайные опечатки новых идентификаторов.
- Пропущенная очистка timeout/listener/effect-dependencies: корректность важнее воспроизведения дефекта.
- Одинаковый UI/палитра, Web3, Telegram, звук, аналитика и конкретные URL/идентификаторы на любом новом сайте.
- Обязательная FSD, именованные экспорты или `I`-префиксы во всех репозиториях.
- Слепой перенос React 18, Tailwind 3, старого SVGR и библиотечных версий в новый проект.

При существующем контракте вроде `appearence` в Button совместимость важнее исправления имени; для новых независимых API опечатку не воспроизводить. Если пользователь позже уточнит личное предпочтение или авторство, его прямое указание важнее выводов по этому снимку.
