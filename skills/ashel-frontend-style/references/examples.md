# Примеры кода

Содержание: группировка импортов; cn; Skeleton; нативная кнопка; debounce; типизированные запросы; submit формы.

Ниже приведены фрагменты рабочего кода. Изолированные утилиты и примитивы сохранены целиком; в предметных примерах убраны специфичные имена и оставлена относящаяся к правилу логика. Небольшие дополнения, нужные для автономности примера, обозначены явно. Это образцы стиля, а не требование устанавливать все упомянутые библиотеки.

## Группировка импортов

Этот блок сохраняет смысловую группировку формы с запросом: библиотеки, UI, API, состояние, утилиты. Пути здесь обозначают обычные соседние модули, не внешний репозиторий.

```tsx
import { useState } from "react"
import { toast } from "sonner"
import { useTranslation } from "react-i18next"

import Input from "./Input"
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
} from "./Dialog"

import { useCreateModuleMutation } from "./moduleApi"

import { useAppDispatch, useAppSelector } from "./store"
import { addModule, setActiveModuleId } from "./moduleSlice"

import { getApiErrorCode } from "./apiErrors"
import { cn } from "./cn"
```

Не создавай эти модули лишь для повторения списка. Пример показывает визуальные границы групп; реальные импорты определяются компонентом.

## cn: условные классы и разрешение конфликтов

Полная функция. Единственное изменение оформления — явный type-import. Зависимости: `clsx`, `tailwind-merge`.

```ts
import { type ClassValue, clsx } from "clsx"
import { twMerge } from "tailwind-merge"

export const cn = (...inputs: ClassValue[]) => {
  return twMerge(clsx(inputs))
}
```

`clsx` собирает условные классы, `twMerge` разрешает конфликтующие Tailwind-значения. Например, `cn("h-8", "h-12")` оставляет `h-12`; поэтому внешний className ставится последним.

## Skeleton: небольшой UI-примитив целиком

В исходную форму добавлен type-import, чтобы пример не полагался на глобальный namespace React. `./cn` — функция выше.

```tsx
import type { HTMLAttributes } from "react"

import { cn } from "./cn"

const Skeleton = ({
  className,
  ...props
}: HTMLAttributes<HTMLDivElement>) => {
  return (
    <div
      className={cn("animate-pulse rounded-md bg-white/10", className)}
      {...props}
    />
  )
}

export { Skeleton }
```

Примитив отвечает за анимацию и поверхность. Геометрию загрузки задаёт отдельный компонент экрана:

```tsx
import { Skeleton } from "./Skeleton"

const TasksLoadingScreen = () => {
  return (
    <div className="flex w-full flex-col gap-2">
      <Skeleton className="flex h-[62px] w-full items-center gap-3 rounded-2xl p-1.5 pr-3">
        <Skeleton className="h-12 w-12 shrink-0 rounded-xl" />
        <div className="flex flex-col items-start gap-1">
          <Skeleton className="h-6 w-[120px]" />
          <Skeleton className="h-5 w-[90px]" />
        </div>
        <Skeleton className="ml-auto h-9 w-20" />
      </Skeleton>
    </div>
  )
}

export default TasksLoadingScreen
```

Это сокращённый блок реального loading-компонента: оставлена одна строка и удалены нестандартные responsive-префиксы. Цвет поверхности выбирается под фон интерфейса.

## Нативная кнопка с ref

Полная обёртка для React 18. Из строки классов удалён только пользовательский шрифт, добавлен displayName. Остальные свойства, проброс props/ref и оформление сохранены. Код требует React, Tailwind и `./cn` из примера выше.

```tsx
import React from "react"

import { cn } from "./cn"

const OutlinedButton = React.forwardRef<
  HTMLButtonElement,
  React.ComponentProps<"button">
>(({ className, children, ...props }, forwardedRef) => {
  return (
    <button
      className={cn(
        "flex items-center justify-center gap-2 text-[10px]/none px-5 2xl:px-6 h-10 md:h-[45px] 2xl:h-[52px] rounded-lg md:text-sm/none md:rounded-[10px] 2xl:text-base/none border border-black bg-transparent hover:bg-black/10 active:bg-black/20 text-black transition-all",
        className
      )}
      {...props}
      ref={forwardedRef}
    >
      {children}
    </button>
  )
})
OutlinedButton.displayName = "OutlinedButton"

export default OutlinedButton
```

Свойства `disabled`, `type`, события и aria-атрибуты доступны через нативные props. Этот компонент не объявляет собственную матрицу вариантов: одной обёртки здесь достаточно.

## Debounce: переиспользуемая логика без UI

Полный hook с generic-значением и очисткой таймера. Зависимость — только React.

```ts
import { useState, useEffect } from "react"

function useDebounce<TData>(data: TData, interval: number) {
  const [liveData, setLiveData] = useState<TData>(data)

  useEffect(() => {
    if (typeof window !== "undefined") {
      const handler = setTimeout(() => {
        setLiveData(data)
      }, interval)
      return () => {
        clearTimeout(handler)
      }
    }
  }, [data, interval])

  return liveData
}

export default useDebounce
```

Обычная функция допустима для helper/hook; предпочтение стрелочных компонентов не запрещает такую форму. Возвращаемое значение имеет тот же тип, что входной `data`.

## RTK Query: типы, endpoints и hooks

Из рабочего API сохранены `injectEndpoints`, типизированные query/mutation, tags и exports. Названия предметной области обобщены до inventory, модель сокращена. Добавлен минимальный `core.ts`, чтобы вся используемая инфраструктура была видна в примере. URL — иллюстративный контракт, не существующий публичный сервис.

`core.ts`:

```ts
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query/react"

export const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: "/api/" }),
  tagTypes: ["Inventory"],
  endpoints: () => ({}),
})
```

`inventory.ts`:

```ts
import { api } from "./core"

interface InventoryItem {
  id: number
  title: string | null
  is_used: boolean
}

interface Inventory {
  items: InventoryItem[]
  items_count: number
}

interface ClaimResponse {
  instruction: string | null
  prize_code: string | null
  help_text: string | null
  claim_url: string | null
}

const inventoryApi = api.injectEndpoints({
  endpoints: (build) => ({
    inventory: build.query<Inventory, void>({
      query: () => ({
        url: "me/inventory",
        method: "GET",
      }),
      providesTags: ["Inventory"],
    }),
    claim: build.mutation<ClaimResponse, { inventory_prize_id: number }>({
      query: ({ inventory_prize_id }) => ({
        url: `me/inventory/${inventory_prize_id}/claim`,
        method: "POST",
      }),
      invalidatesTags: ["Inventory"],
    }),
  }),
  overrideExisting: false,
})

export const { useInventoryQuery, useClaimMutation } = inventoryApi
```

Для использования hooks базовый API подключается к Redux store: reducer под `api.reducerPath`, `api.middleware` в middleware, Provider над приложением. Здесь показано определение API, а не bootstrap приложения. Типы ответа сохраняют nullability и серверные snake_case-поля.

## Submit: прямой поток действий

Это фрагмент тела формы создания модуля. Сохранены guard, trim, payload, unwrap, обновление общего состояния и очистка после успеха. Предметные ветки ошибок заменены общим fallback; непрозрачный helper ошибок исключён.

Значения `courseId`, `moduleName`, `moduleDescription`, `moduleHours`, `moduleMinutes`, `modules` принадлежат форме; actions и hooks — её модулю состояния. Это намеренно пример обработчика, а не отдельный исполняемый файл.

```tsx
const [createModule, { isLoading: isCreateModuleLoading }] =
  useCreateModuleMutation()

const submitModuleCreation = async () => {
  if (isCreateModuleLoading) return

  const cleanName = moduleName.trim()
  const cleanDescription = moduleDescription.trim()

  if (cleanName.length === 0 || cleanDescription.length === 0) return

  try {
    const data = await createModule({
      course_id: courseId,
      name: cleanName,
      description: cleanDescription,
      position:
        modules.length === 0 ? 0 : modules[modules.length - 1].position + 1,
      timeToLearn: `${moduleHours}:${moduleMinutes}:00`,
    }).unwrap()

    dispatch(addModule(data))
    dispatch(setActiveModuleId(data.id))
    dispatch(setActiveLessonId(null))

    clearState()
  } catch {
    toast.error(t("Произошла ошибка при создании модуля"))
  }
}
```

Кнопка этой формы получает `disabled={isCreateModuleLoading}`. Если API возвращает известные коды, добавь их явную обработку перед fallback. Не требуется выносить короткий submit в отдельный hook или класс сервиса.
