# Примеры кода

Содержание: `cn`; Skeleton; типизированный RTK Query API. Используй примеры, когда соответствующие библиотеки уже выбраны для проекта.

## cn: условные классы и разрешение конфликтов

Зависимости: `clsx` и версия `tailwind-merge`, совместимая с Tailwind проекта.

```ts
import { type ClassValue, clsx } from "clsx"
import { twMerge } from "tailwind-merge"

export const cn = (...inputs: ClassValue[]) => {
  return twMerge(clsx(inputs))
}
```

`clsx` собирает условные классы, `twMerge` разрешает конфликтующие Tailwind-значения. Например, `cn("h-8", "h-12")` оставляет `h-12`; поэтому внешний className ставится последним.

## Skeleton: небольшой UI-примитив целиком

Зависимости: React, Tailwind и `./cn` из примера выше. Декоративный skeleton скрыт от скринридера.

```tsx
import type { HTMLAttributes } from "react"

import { cn } from "./cn"

const Skeleton = ({
  className,
  ...props
}: HTMLAttributes<HTMLDivElement>) => {
  return (
    <div
      aria-hidden="true"
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
    <div
      role="status"
      aria-label="Загрузка задач"
      className="flex w-full flex-col gap-2"
    >
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

Цвет поверхности выбирается под фон интерфейса. Геометрия соответствует конкретному экрану и меняется вместе с ним.

## RTK Query: типы, endpoints и hooks

Пример требует `@reduxjs/toolkit` и настроенного Redux store. URL и модели иллюстративные. Мутация инвалидирует query-кэш; серверные сущности не копируются в отдельный slice.

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
