# Организация состояния и поведения компонента

Содержание: состояние шага и поля формы; группы внутри формы; отдельные сценарии; callbacks переходов; общий сброс; группы запросов и мутаций.

Фрагменты показывают части тела React-компонента. Для примеров с `useState` используется `import { useState } from "react"`.

## UI-сценарий отдельно от полей формы

Состояние шага отделено от двух полей. Пара обработчиков управляет только переходом; введённые значения не очищаются при возвращении назад.

```tsx
const [isSecondStep, setIsSecondStep] = useState(false)

const [oldEmailInput, setOldEmailInput] = useState("")
const [newEmailInput, setNewEmailInput] = useState("")

const openSecondStep = () => setIsSecondStep(true)
const closeSecondStep = () => setIsSecondStep(false)
```

Поля остаются в родителе, а дочерние шаги получают нужные значения и callbacks. Не поднимай их в глобальное хранилище ради того, что форму составляют несколько компонентов.

## Внутри формы тоже есть смысловые группы

Название и описание относятся к содержанию. Часы и минуты — к длительности. Все четыре значения являются полями формы, но читать их удобнее двумя блоками.

```tsx
const [moduleName, setModuleName] = useState("")
const [moduleDescription, setModuleDescription] = useState("")

const [moduleHours, setModuleHours] = useState("00")
const [moduleMinutes, setModuleMinutes] = useState("00")
```

Строковое представление длительности относится к её вводу; не воспринимай его как правило хранения любого времени.

## Разделение по сценарию важнее разделения по типу

В первом блоке поле и ошибка входа через email. Во втором — подпись и ошибка другого способа входа. Здесь boolean стоит вместе с данными своего сценария.

```tsx
const [emailInput, setEmailInput] = useState("")
const [isUserNotFoundByMailLogin, setIsUserNotFoundByMailLogin] =
  useState(false)

const [signature, setSignature] = useState("")
const [isUserNotFoundByWalletAuth, setIsUserNotFoundByWalletAuth] =
  useState(false)
```

Паттерн переносится на параллельные сценарии: создание и редактирование, поиск и фильтры, загрузку и предпросмотр.

Не переставляй обе ошибки в конец списка только ради общего типа `boolean`. Не добавляй объект `state` с десятком несвязанных полей вместо визуально сгруппированных независимых hooks.

## Общий сброс и предметные переходы

Если при нескольких переходах повторяется один набор операций, дай ему имя:

```tsx
const resetState = () => {
  setSelectedVariant(null)
  setSelectedVariants([])
  setIsSucceeded(false)
}

const onPrevClick = () => {
  resetState()
  goToPreviousLesson()
}

const onNextClick = () => {
  resetState()
  goToNextLesson()
}
```

`goToPreviousLesson/goToNextLesson` — callbacks родителя. Два конкретных намерения используют один повторяемый сброс.

Не превращай это в правило «сбрасывать абсолютно всё»: набор значений определяется переходом. При длинном сбросе визуально отделяй, например, флаги предпросмотра от текущего шага и данных. При закрытии формы с query-mutation может потребоваться сброс самой формы и статуса запроса:

```tsx
const onDialogClose = () => {
  form.reset()
  resetMutation()
  close()
}
```

Этот обработчик предполагает `form` из формы и `resetMutation` из mutation-hook. Он выражает завершение сценария, не заменяет поведение закрытия любого диалога. При необходимости сохранить черновик его не следует копировать буквально.

## Запросы и мутации читаются группами

После локальных состояний может идти общий state, затем запросы, затем мутации по действиям. Порядок не обязан быть одинаковым в каждом компоненте, но связанные строки не должны перемешиваться случайно.

```tsx
const { data: categories, isLoading: isCategoriesLoading } =
  useGetCategoriesQuery()
const { data: authorCategories, isLoading: isAuthorCategoriesLoading } =
  useGetAuthorCategoriesQuery()

const [createAuthorCategory, { isLoading: isCreateAuthorCategoryLoading }] =
  useCreateAuthorCategoryMutation()
const [updateAuthorCategory, { isLoading: isUpdateAuthorCategoryLoading }] =
  useUpdateAuthorCategoryMutation()
const [deleteCategory, { isLoading: isDeleteCategoryLoading }] =
  useDeleteCategoryMutation()

const [createAdminCategory, { isLoading: isCreateAdminCategoryLoading }] =
  useCreateAdminCategoryMutation()
const [updateAdminCategory, { isLoading: isUpdateAdminCategoryLoading }] =
  useUpdateAdminCategoryMutation()
```

Queries сгруппированы как чтение данных, mutations — по двум сценариям управления. Алиасы объясняют, к какому действию относится pending-состояние.

## Эффекты и JSX продолжают тот же принцип

Один эффект отвечает за один связанный внешний процесс: таймер со своим cleanup, отдельная подписка со своим cleanup, синхронизация с внешним источником. Это не требование создавать effect для каждого вычисления. Действия пользователя остаются в handlers, а значения, вычисляемые из props/state, не требуют effect только ради своей группировки.

В разметке оставляй пустые строки между заголовком, основным содержимым и действиями. Label, input и сообщение об ошибке одного поля остаются рядом. Для коротких групп достаточно пробелов и хороших имён; комментарий нужен только там, где он объясняет неочевидное назначение или ограничение.
