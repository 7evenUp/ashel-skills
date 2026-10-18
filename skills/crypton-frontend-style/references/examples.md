# Примеры формы кода

Это сокращённые/адаптированные примеры по прочитанным исходникам, а не готовые файлы для слепого копирования. Перед использованием проверь реальные пути, exports, prop API, переводы и серверные контракты. Дизайн и доменные поля принадлежат конкретным проектам.

## Обычный компонент: default export, inline-пропсы, cn

Основа: `nutsfarm/src/modules/Tasks/components/PartnerGroup.tsx`, `events-front/src/pages/Quest/components/Inventory/InventoryItem.tsx`. Здесь показана только визуальная часть: обработчик принадлежит родителю.

```tsx
import { useTranslation } from "react-i18next"

import { cn } from "@/lib/cn"

const TaskCard = ({
  title,
  reward,
  isClaimed,
  isLoading,
  onClaim,
  className,
}: {
  title: string
  reward: number
  isClaimed: boolean
  isLoading: boolean
  onClaim: () => void
  className?: string
}) => {
  const { t } = useTranslation()

  return (
    <div
      className={cn(
        "flex items-center gap-3 rounded-2xl bg-[#17202D] p-3",
        isClaimed && "opacity-75",
        className
      )}
    >
      <div className="flex flex-col gap-1">
        <h3 className="text-sm font-bold">{title}</h3>
        <span className="text-xs text-white/50">{reward}</span>
      </div>
      <button
        type="button"
        className="ml-auto rounded-xl bg-white px-3 py-2 text-sm font-bold text-black disabled:opacity-50"
        disabled={isClaimed || isLoading}
        aria-busy={isLoading}
        onClick={onClaim}
      >
        {isClaimed ? t("Получено") : isLoading ? t("Загрузка") : t("Забрать")}
      </button>
    </div>
  )
}

export default TaskCard
```

В реальном модуле предпочти существующий Button, если его API и оформление подходят. Не добавляй этот элемент рядом с уже существующим эквивалентом. Для Nutsfarm конкретная готовая кнопка может требовать `appearance`, для другого приложения она будет выглядеть иначе.

## RTK Query: доменные endpoints и сгенерированные hooks

Сокращённый фрагмент `events-front/src/redux/api/inventory.ts`:

```ts
import { api } from "./core"

import { Inventory } from "@/types/inventory"

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
        url: "api/me/inventory",
        method: "GET",
      }),
      providesTags: ["Inventory"],
    }),
    claim: build.mutation<ClaimResponse, { inventory_prize_id: number }>({
      query: ({ inventory_prize_id }) => ({
        url: `api/me/inventory/${inventory_prize_id}/claim`,
        method: "POST",
      }),
      invalidatesTags: ["Inventory"],
    }),
  }),
  overrideExisting: false,
})

export const { useInventoryQuery, useClaimMutation } = inventoryApi
```

Схема обработчика внутри ClaimDialog, адаптированная без подавления типов:

```tsx
const [claim, { isLoading }] = useClaimMutation()

const onSubmit = async () => {
  if (isLoading) return

  try {
    const responseData = await claim({ inventory_prize_id: id }).unwrap()

    if (type === "nft") {
      setResponseData(responseData.claim_url)
    } else if (type === "crypton_promo_code") {
      setResponseData(responseData.prize_code)
    }
  } catch (error) {
    onOpenChange(false)

    const code =
      typeof error === "object" && error !== null && "data" in error &&
      typeof error.data === "object" && error.data !== null &&
      "code" in error.data && typeof error.data.code === "string"
        ? error.data.code
        : undefined

    if (code === "inventory_prize_already_used") {
      toast.error("Уже использовано")
    } else {
      toast.error("Произошла ошибка. Пожалуйста, попробуйте позже")
    }
  }
}
```

Это фрагмент тела компонента; `id`, `type`, `setResponseData`, `onOpenChange` объявлены в нём. Проверка `data.code` относится только к данному API. Если есть helper ошибок с подходящим контрактом — используй его. Повторяющуюся проверку формы ответа вынеси в локальный helper; другой API может возвращать `data.detail.code`.

## Вариант academy-tma: named export и public API

Сокращённый `widgets/CoursesCards/ui/CoursesCards.tsx`:

```tsx
import { useTranslation } from "react-i18next"
import { Link } from "@tanstack/react-router"

import type { IMeCourse } from "@/entities/course"

import { ROUTES_PATHS } from "@/shared"

interface CoursesCardProps {
  courses: IMeCourse[]
}

export const CoursesCards = ({ courses }: CoursesCardProps) => {
  const { t } = useTranslation("courses")

  return (
    <div className="mt-2">
      {courses.map((course) => (
        <Link
          className="flex items-center gap-3 border-t border-grey-1 py-3 first:border-0"
          key={course.id}
          to={ROUTES_PATHS.library.coursePreview(course.id, course.type)}
        >
          <div className="flex flex-col gap-2">
            <h3 className="text-sm/[100%]">{course.name}</h3>
            <span className="text-[12px]/[100%] text-grey-4">
              {t("lessons", { count: course.lessonsCount })}
            </span>
          </div>
        </Link>
      ))}
    </div>
  )
}
```

Public API `widgets/CoursesCards/index.ts`: `export { CoursesCards } from "./ui/CoursesCards"`. Не переносить этот стиль экспорта автоматически в старые `modules`-проекты.

## Вариант x-landing: секция и SCSS Module

Форма по `sections/About` и `sections/Team`; текст и композиция ниже условные.

```tsx
import { motion } from "framer-motion"
import { useInView } from "react-intersection-observer"

import styles from "./Benefits.module.scss"

const Benefits = () => {
  const { ref, inView } = useInView({
    threshold: 0.4,
    triggerOnce: true,
  })

  return (
    <section className={styles.section} id="benefits" ref={ref}>
      <h2>Возможности сообщества</h2>
      <motion.div
        className={styles.card}
        initial={{ opacity: 0, y: 40 }}
        animate={{ opacity: inView ? 1 : 0, y: inView ? 0 : 40 }}
        transition={{ duration: 0.8 }}
      >
        <h3>Обменивайтесь опытом</h3>
        <p>Знакомьтесь с участниками и обсуждайте новые идеи.</p>
      </motion.div>
    </section>
  )
}

export default Benefits
```

```scss
@use "../../mixins";

.section {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 96px 24px;

  @include mixins.desktop {
    padding: 5vw 1.25vw;
  }

  @include mixins.tablet {
    padding: 48px 20px;
  }

  h2 {
    margin-bottom: 48px;
    font-family: "Unbounded", cursive;
    font-size: 48px;

    @include mixins.desktop {
      margin-bottom: 2.5vw;
      font-size: 2.5vw;
    }

    @include mixins.tablet {
      margin-bottom: 24px;
      font-size: 26px;
    }
  }
}

.card {
  width: 100%;
  max-width: 664px;
  padding: 32px;
  border: 2px solid black;
  border-radius: 24px;

  @include mixins.desktop {
    max-width: 34.5833vw;
    padding: 1.6667vw;
    border-radius: 1.25vw;
  }

  @include mixins.phone {
    padding: 20px;
  }
}
```

Шрифт, mixins и библиотеки должны существовать в выбранном проекте. Не подключай этот шаблон к приложению с Tailwind лишь ради одинакового внешнего вида.
