# Секционная вёрстка и адаптивность

Пример ниже выделен из работающего блока с анимацией появления. Удалены иллюстрации и рекламные утверждения, название секции и текст сделаны нейтральными. Сохранены отдельный компонент секции, `useInView`, Motion props и стили в SCSS Module. SCSS показан в сокращённой форме: вложенность и близость responsive-переопределений к блоку важнее конкретных чисел.

## Композиция страницы

```tsx
import Hero from "./sections/Hero/Hero"
import Benefits from "./sections/Benefits/Benefits"
import Footer from "./sections/Footer/Footer"

const App = () => {
  return (
    <>
      <Hero />
      <Benefits />
      <Footer />
    </>
  )
}

export default App
```

Здесь Hero/Footer — другие секции страницы. Не создавай конфигурационную модель или registry компонентов для трёх явных JSX-элементов.

## Секция

`Benefits.tsx` использует React, `framer-motion`, `react-intersection-observer` и поддержку SCSS Modules. Аналогичный API доступен через `motion/react`, если выбран этот пакет; импорт соответствует используемой зависимости.

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
    <section className={styles.section} id="benefits">
      <h2>Возможности для участников</h2>
      <div className={styles.container} ref={ref}>
        <motion.div
          initial={{ opacity: 0, y: 50 }}
          animate={{ opacity: inView ? 1 : 0, y: inView ? 0 : 50 }}
          transition={{ duration: 0.8 }}
          className={styles.card}
        >
          <h3>Обменивайтесь опытом</h3>
          <p>Обсуждайте идеи и находите ответы вместе с участниками.</p>
        </motion.div>
      </div>
    </section>
  )
}

export default Benefits
```

## Стили

`Benefits.module.scss`:

```scss
@use "../../mixins";

.section {
  display: flex;
  flex-direction: column;
  align-items: center;
  color: black;

  h2 {
    padding-top: 100px;
    padding-bottom: 80px;
    text-align: center;
    width: 900px;
    font-size: 48px;
    font-weight: 400;

    @include mixins.desktop {
      padding-top: 5.2083vw;
      padding-bottom: 4.1667vw;
      width: 46.875vw;
      font-size: 2.5vw;
    }

    @include mixins.tablet {
      padding-top: 48px;
      padding-bottom: 32px;
      width: auto;
      font-size: 28px;
    }
  }
}

.container {
  width: 100%;
  max-width: 1160px;
  padding: 0 24px 64px;

  @include mixins.phone {
    padding: 0 16px 32px;
  }
}

.card {
  padding: 24px;
  border: 2px solid black;
  border-radius: 24px;

  h3 {
    margin-bottom: 12px;
  }
}
```

`_mixins.scss`, размещённый в `src`, содержит определения используемых имён:

```scss
@mixin desktop {
  @media (min-width: 1921px) {
    @content;
  }
}

@mixin tablet {
  @media (max-width: 768px) {
    @content;
  }
}

@mixin phone {
  @media (max-width: 480px) {
    @content;
  }
}
```

В этом варианте базовые размеры заданы в px, для широкого экрана — в vw относительно 1920px, для узких экранов — конкретные переопределения. Значения иллюстрируют один макет и не становятся глобальными константами skill. Меняй их под задачу. Для обычного прикладного блока с Tailwind не требуется добавлять SCSS и Motion ради этого примера.
