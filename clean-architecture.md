# Clean Architecture Principles

## 1. Назначение

Clean Architecture — подход к организации программного обеспечения, направленный на разделение ответственности и снижение зависимости бизнес-логики от технических деталей.

В рамках frontend-архитектурного стандарта компании Clean Architecture используется как **обязательный набор принципов проектирования кода**, а не как требование создавать определённую структуру директорий.

Основная цель — обеспечить независимость бизнес-правил от:

- UI-фреймворков;
- способов управления состоянием;
- сетевых протоколов и HTTP-клиентов;
- форматов внешних API;
- браузерных API;
- конкретных реализаций внешних сервисов.

**Бизнес-правила не должны зависеть от деталей того, как приложение отображает, получает и сохраняет данные.**

Принципы применяются на проектах любой сложности, но количество файлов, абстракций и архитектурных слоёв определяется реальными потребностями функциональности.

## 2. Логические слои Clean Architecture

В классической Clean Architecture выделяются четыре основных слоя:

```text
Frameworks & Drivers
        │
Interface Adapters
        │
    Use Cases
        │
     Entities
```

Зависимости исходного кода направлены к внутренним слоям.

Для frontend-проектов удобно использовать следующую логическую модель:

```text
Presentation
Application
Domain
Infrastructure
```

Это не обязательные каталоги, а области ответственности.

### 2.1. Domain

Domain описывает бизнес-понятия и правила предметной области.

Может содержать:

- Entities;
- Value Objects;
- бизнес-правила;
- инварианты;
- доменную валидацию;
- Domain Services.

Пример:

```ts
export type Order = {
  id: string;
  total: number;
  status: 'draft' | 'paid';
};

export function canPayOrder(order: Order): boolean {
  return order.status === 'draft' && order.total > 0;
}
```

Domain не должен импортировать React, Vue, API-клиенты, stores или другие внешние технические механизмы.

### 2.2. Application

Application отвечает за выполнение сценариев приложения.

Примеры:

- создание заказа;
- оформление покупки;
- активация промокода;
- выполнение сложного расчёта;
- координация нескольких операций.

Application использует доменные правила и определяет последовательность действий.

Если сценарию нужны внешние возможности, он может обращаться к ним через абстракции — ports.

Application не должен знать, какой конкретный HTTP-клиент или механизм хранения реализует эти возможности.

### 2.3. Infrastructure

Infrastructure реализует взаимодействие с внешним миром.

Например:

- REST / GraphQL;
- localStorage / IndexedDB;
- WebSocket;
- сторонние SDK;
- API-клиенты;
- repository adapters;
- DTO mappers.

Infrastructure знает о конкретных технологиях и внешних контрактах.

При использовании Dependency Inversion инфраструктурные реализации зависят от контрактов, определённых внутренней логикой.

### 2.4. Presentation

Presentation отвечает за взаимодействие с пользователем.

Содержит:

- UI-компоненты;
- presentation state;
- обработку пользовательского ввода;
- hooks и composables;
- ViewModels и presentation adapters.

Presentation может вызывать application-сценарии и использовать доменные модели.

При этом бизнес-правила не должны существовать исключительно внутри компонентов или зависеть от UI-фреймворка.

## 3. Dependency Rule

Ключевой принцип Clean Architecture:

**Dependencies point inward.**

Внутренняя логика не должна зависеть от внешних механизмов.

Упрощённая схема зависимостей:

```text
             Domain
                ▲
                │
           Application
            ▲       ▲
            │       │
     Presentation  Infrastructure
```

Типичные зависимости:

```text
Presentation → Application
Application  → Domain

Infrastructure → Application contracts
Infrastructure → Domain models
```

Domain не зависит от внешних архитектурных областей.

Application не зависит от конкретной инфраструктурной реализации.

### Важное различие

Необходимо отличать:

1. Архитектурные зависимости исходного кода.
2. Последовательность выполнения операций (Runtime Flow).

Например, выполнение может выглядеть так:

```text
UI
 ↓
Use Case
 ↓
Repository Port
 ↓
HTTP Repository
 ↓
REST API
```

Но Use Case при этом зависит не от конкретного HTTP Repository, а от его интерфейса.

Конкретная реализация зависит от интерфейса и передаётся Use Case извне.

Таким образом, Runtime Flow может обращаться к внешнему миру, не нарушая Dependency Rule.

## 4. Clean Architecture внутри FSD

FSD и Clean Architecture решают разные задачи.

FSD определяет, как разделять проект по функциональности.

Clean Architecture определяет, как организовывать зависимости между обязанностями внутри функциональности.

Пример:

```text
features/submit-order/
├── ui/
├── model/
├── api/
└── index.ts
```

Примерное соответствие:

| FSD                             | Clean Architecture                  |
| ------------------------------- | ----------------------------------- |
| `ui`                            | Presentation                        |
| Чистые бизнес-правила в `model` | Domain                              |
| Сценарии в `model`              | Application                         |
| Технические адаптеры в `api`    | Infrastructure / Interface Adapters |

Соответствие не является строгим один к одному.

Например, FSD-сегмент `model` может содержать React hook или Zustand store, которые архитектурно относятся к Presentation.

Поэтому необходимо различать **расположение файла** и **его логическую ответственность**.

### Основное внутреннее правило

Чистый Domain и Application не должны импортировать конкретные технические адаптеры.

Например:

```text
model/domain.ts  -X→  api/httpClient.ts
model/useCase.ts -X→  api/repository.ts
```

При этом:

```text
api/repository.ts → model/ports.ts
api/mapper.ts     → model/domain.ts
```

являются допустимыми направлениями зависимостей.

Presentation может использовать чистую логику, а внешняя композиция связывает конкретные реализации с абстракциями.

## 5. Разделение ответственности без лишних файлов

**Чистая архитектура определяется зависимостями, а не количеством каталогов.**

Одна и та же архитектурная идея может быть реализована несколькими способами.

### 5.1. Простая функциональность

Например, калькулятор стоимости на лендинге.

```text
price-calculator/
├── model.ts
├── ui/
│   └── PriceCalculator.tsx
└── index.ts
```

`model.ts` содержит чистые бизнес-правила:

```ts
export function calculatePrice(
  basePrice: number,
  discountPercent: number,
): number {
  if (
    discountPercent < 0 ||
    discountPercent > 100
  ) {
    throw new Error('Invalid discount');
  }

  return basePrice * (1 - discountPercent / 100);
}
```

UI использует эту функцию, но сама функция ничего не знает о React.

```text
PriceCalculator.tsx
         ↓
      model.ts
```

В данном случае отсутствует необходимость создавать:

```text
domain/
application/
infrastructure/
```

Отсутствие этих директорий не нарушает Clean Architecture.

Физическое разделение на UI и чистые бизнес-правила уже обеспечивает нужную границу ответственности.

Если функциональность не содержит бизнес-правил вообще, отдельный `model.ts` создавать не требуется.

### 5.2. Функциональность с API

При появлении взаимодействия с сервером структура может расшириться:

```text
features/price-estimation/
├── ui/
│   └── PriceEstimator.tsx
├── model.ts
├── api.ts
└── index.ts
```

Ответственность:

- `model.ts` — чистые расчёты и бизнес-правила;
- `api.ts` — запросы и преобразование внешних данных;
- `ui` — отображение и управление взаимодействием.

При простой загрузке данных UI-adapter или hook может координировать вызов `api.ts` и отображение результата.

Необязательно создавать отдельный Use Case, если самостоятельной application-логики нет.

Главное условие:

**`model.ts` не должен импортировать `api.ts`.**

Если сценарий становится сложнее, его следует выделить из UI-координации.

### 5.3. Сложная функциональность

При появлении многошаговых сценариев, внешних зависимостей и сложных правил структура может выглядеть так:

```text
features/submit-order/
├── ui/
│   └── SubmitOrderButton.tsx
│
├── model/
│   ├── submitOrder.ts
│   └── ports.ts
│
├── api/
│   ├── httpOrderGateway.ts
│   ├── dto.ts
│   └── mapper.ts
│
└── index.ts
```

В данном случае:

- `submitOrder.ts` — Application / Use Case;
- `ports.ts` — контракты необходимых внешних возможностей;
- `api/` — Infrastructure;
- `ui/` — Presentation.

Domain-модель заказа может размещаться в `entities/order/model`.

Это позволяет соблюдать Clean Architecture, сохраняя структуру FSD.

## 6. Практический пример: оформление заказа

Рассмотрим сценарий, в котором пользователь отправляет заказ на сервер.

### 6.1. Domain

Модель и правило находятся в `entities/order/model/order.ts`.

```ts
export type Order = {
  id: string;
  items: Array<{
    productId: string;
    quantity: number;
  }>;
};

export function validateOrder(order: Order): void {
  if (order.items.length === 0) {
    throw new Error('Order is empty');
  }

  if (order.items.some(item => item.quantity <= 0)) {
    throw new Error('Invalid quantity');
  }
}
```

Доменная модель не зависит от HTTP или React.

### 6.2. Application Port

```ts
// features/submit-order/model/ports.ts

import type { Order } from '@/entities/order';

export type OrderReceipt = {
  orderId: string;
  status: 'accepted';
};

export interface OrderGateway {
  submit(order: Order): Promise<OrderReceipt>;
}
```

Application определяет, какая внешняя возможность ему необходима.

Он не описывает, каким способом будет отправляться заказ.

### 6.3. Application Use Case

```ts
// features/submit-order/model/submitOrder.ts

import {
  type Order,
  validateOrder,
} from '@/entities/order';

import type { OrderGateway } from './ports';

export async function submitOrder(
  order: Order,
  gateway: OrderGateway,
) {
  validateOrder(order);

  return gateway.submit(order);
}
```

Use Case:

1. Получает заказ.
2. Проверяет доменные правила.
3. Передаёт заказ внешнему сервису через контракт.
4. Возвращает результат.

При этом он не знает о `fetch`, REST и конкретном endpoint.

### 6.4. Infrastructure Adapter

```ts
// features/submit-order/api/httpOrderGateway.ts

import type { OrderGateway } from '../model/ports';

export const httpOrderGateway: OrderGateway = {
  async submit(order) {
    const response = await fetch('/api/orders', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(order),
    });

    if (!response.ok) {
      throw new Error('Failed to submit order');
    }

    const data: { id: string } = await response.json();

    return {
      orderId: data.id,
      status: 'accepted',
    };
  },
};
```

Адаптер знает:

- endpoint;
- HTTP-метод;
- формат ответа;
- способ выполнения запроса.

В реальном приложении здесь также могут находиться DTO mapping, обработка ошибок интеграции и другие технические детали.

### 6.5. Presentation

UI-компонент получает обработчик через props.

```tsx
// features/submit-order/ui/SubmitOrderButton.tsx

import type { Order } from '@/entities/order';

type Props = {
  order: Order;
  onSubmit: (order: Order) => Promise<unknown>;
};

export function SubmitOrderButton({
  order,
  onSubmit,
}: Props) {
  return (
    <button onClick={() => void onSubmit(order)}>
      Оформить заказ
    </button>
  );
}
```

Компонент не знает, каким образом отправляется заказ.

В реальном приложении он также может управлять состоянием отправки и отображать ошибки.

### 6.6. Composition Root

Конкретный gateway связывается с Use Case на внешнем уровне.

Например, в странице:

```tsx
// pages/checkout/ui/CheckoutPage.tsx

import {
  submitOrder,
  httpOrderGateway,
  SubmitOrderButton,
} from '@/features/submit-order';

import type { Order } from '@/entities/order';

export function CheckoutPage({ order }: { order: Order }) {
  return (
    <SubmitOrderButton
      order={order}
      onSubmit={(currentOrder) =>
        submitOrder(currentOrder, httpOrderGateway)
      }
    />
  );
}
```

Предполагается, что перечисленные символы экспортированы через Public API feature.

Если конкретный адаптер предназначен только для сервера, его экспорт должен быть отделён от клиентского API.

Результирующее направление зависимостей:

```text
                Domain
                  ▲
                  │
              Application
               ▲       ▲
               │       │
         Presentation  Infrastructure
```

В этом примере страница выступает локальной точкой композиции.

Для глобальных зависимостей Composition Root может находиться в `app`.

## 7. Dependency Inversion

Dependency Inversion Principle применяется для отделения внутренних сценариев от конкретных внешних реализаций.

Вместо прямой зависимости:

```text
SubmitOrder
    ↓
HttpOrderGateway
    ↓
fetch
```

создаётся контракт:

```text
         OrderGateway
          ▲       ▲
          │       │
    SubmitOrder  HttpOrderGateway
```

Use Case зависит от интерфейса, а инфраструктурный адаптер этот интерфейс реализует.

### Когда применять

Рекомендуется использовать Dependency Inversion, если:

- сценарий содержит существенную бизнес-логику;
- необходимо независимо тестировать выполнение сценария;
- внешний источник данных может измениться;
- существует несколько реализаций внешнего сервиса;
- необходимо подменять внешние зависимости;
- сценарий координирует несколько внешних операций.

### Когда можно упростить

Если функциональность сводится к обычному API-запросу и отображению результата:

```text
Component
    ↓
TanStack Query
    ↓
API function
```

создание отдельного Use Case и Repository Interface обычно не требуется.

Однако API-вызов должен оставаться локализованным в инфраструктурном модуле, а бизнес-правила — отдельно от технических механизмов.

**Не нужно создавать интерфейсы исключительно ради формального соблюдения Dependency Inversion.**

## 8. Domain Models и DTO

Внешняя модель данных не всегда совпадает с внутренней бизнес-моделью.

Пример ответа API:

```ts
type ProductDto = {
  product_id: number;
  product_name: string;
  price_cents: number;
};
```

Внутренняя модель:

```ts
type Product = {
  id: string;
  name: string;
  price: number;
};
```

Преобразование выполняется на границе интеграции:

```ts
function mapProduct(dto: ProductDto): Product {
  return {
    id: String(dto.product_id),
    name: dto.product_name,
    price: dto.price_cents / 100,
  };
}
```

### Правила

- Domain не должен знать о деталях внешнего DTO.
- Mapping располагается рядом с инфраструктурной границей.
- Не требуется создавать отдельный mapper, если данные уже соответствуют внутренней модели и преобразование не нужно.
- Внутренняя модель должна отражать потребности приложения, а не автоматически копировать внешний контракт.

Отдельная доменная модель особенно полезна, когда backend DTO нестабилен, данные поступают из нескольких источников или приложение содержит собственные бизнес-правила.

## 9. Clean Architecture и управление состоянием

Redux, Zustand, Pinia, TanStack Query и другие инструменты управления состоянием являются техническими механизмами.

Они могут использоваться в Presentation и адаптерах, но не должны определять внутреннюю форму бизнес-правил.

Пример:

```text
React Component
      ↓
useOrder / OrderStore
      ↓
Application Use Case
      ↓
Domain Rules
```

Store или hook может отвечать за:

- загрузку и обновление UI;
- loading/error состояния;
- координацию вызовов;
- сохранение presentation state.

Однако расчёты, ограничения и другие бизнес-правила желательно размещать в независимых функциях.

### Особенность FSD

FSD допускает хранение stores, hooks и бизнес-логики в сегменте `model`.

Это не означает, что весь `model` должен быть framework-independent.

Например:

```text
features/checkout/
└── model/
    ├── rules.ts
    ├── submitOrder.ts
    └── useCheckout.ts
```

Ответственность:

- `rules.ts` — чистая бизнес-логика;
- `submitOrder.ts` — Application;
- `useCheckout.ts` — React adapter / presentation state.

Допустимая зависимость:

```text
useCheckout.ts → submitOrder.ts → rules.ts
```

Недопустимая зависимость:

```text
rules.ts → useCheckout.ts
```

Таким образом, Clean Architecture соблюдается внутри `model` на уровне зависимостей между файлами.

Если структура усложняется, файлы можно разделить на подкаталоги по логической ответственности.

## 10. Разделение валидации

Необходимо отличать валидацию пользовательского ввода от бизнес-правил.

### Presentation Validation

Отвечает за удобство ввода и отображение ошибок:

- обязательные поля;
- формат email;
- длина строки;
- отображение подсказок;
- состояние ошибок формы.

Может выполняться через React Hook Form, Zod и другие инструменты.

### Domain Validation

Отвечает за бизнес-инварианты:

- заказ не может быть пустым;
- скидка не может превышать установленный предел;
- запрещён недопустимый переход статуса;
- определённую операцию нельзя выполнить при конкретном состоянии.

Такие правила должны быть независимы от UI.

**Проверка в форме не заменяет проверку бизнес-инварианта в соответствующем сценарии.**

Frontend-проверки также не заменяют серверную валидацию и авторизацию.

## 11. Ошибки и внешние исключения

Разные архитектурные области могут использовать собственные представления ошибок.

Например:

- Domain — ошибка нарушения бизнес-правила.
- Application — ошибка выполнения сценария.
- Infrastructure — HTTP error, timeout, недоступность внешнего сервиса.
- Presentation — пользовательское сообщение об ошибке.

Рекомендуется преобразовывать технические ошибки в понятные приложению результаты на соответствующей границе.

Например, UI не должен анализировать специфический формат ошибки Axios, чтобы определить, можно ли повторить бизнес-операцию.

Для простых сценариев допускается обычная обработка исключений без отдельной иерархии классов ошибок.

## 12. Тестирование

Одно из основных преимуществ Clean Architecture — возможность тестировать внутренние правила без UI и внешних сервисов.

### Domain

Unit-тесты бизнес-правил:

```ts
expect(
  calculatePrice(1000, 20)
).toBe(800);
```

### Application

Тестирование с подставными зависимостями:

```ts
const gateway: OrderGateway = {
  submit: async () => ({
    orderId: 'test-order',
    status: 'accepted',
  }),
};

const result = await submitOrder(order, gateway);

expect(result.status).toBe('accepted');
```

### Infrastructure

Проверка:

- формирования запросов;
- DTO mapping;
- обработки ответов и ошибок;
- взаимодействия с внешними сервисами.

### Presentation

Проверка:

- пользовательских сценариев;
- компонентов;
- UI-состояний;
- доступности;
- интеграций с приложением.

Тип тестов определяется ответственностью модуля.

## 13. Прогрессивная организация Clean Architecture

Степень физического разделения кода определяется сложностью функциональности.

| Уровень организации | Пример |
|---|---|
| Минимальный | `model.ts` + UI |
| С интеграциями | `model.ts` + `api.ts` + UI |
| Сложный | Domain rules + Use Case + Ports + Adapters + UI |

Это не отдельные архитектуры, а разные способы реализации одних и тех же принципов.

### Когда разделять код на дополнительные файлы

Разделение оправдано, когда:

- появляются самостоятельные причины изменения;
- необходимо обеспечить независимое тестирование;
- файл содержит несколько несвязанных обязанностей;
- сценарии требуют собственных контрактов;
- инфраструктурные детали начинают проникать в бизнес-логику;
- текущая структура затрудняет навигацию и сопровождение.

Не требуется выделять отдельный файл для каждого класса, типа или функции.

При этом сокращение числа файлов не должно приводить к смешению несовместимых обязанностей.

## 14. Частые архитектурные ошибки

### Бизнес-логика внутри UI

```text
Component
  ├── HTTP
  ├── DTO mapping
  ├── Business rules
  └── UI rendering
```

Решение: выделить чистые правила, а при необходимости — application-сценарий и инфраструктурный адаптер.

### Domain зависит от Store

```ts
// domain/rules.ts

import { useCartStore } from '../store';
```

Решение: передавать необходимые данные в доменную функцию, не импортируя state manager.

### Use Case напрямую использует HTTP

```ts
async function submitOrder() {
  return fetch('/api/orders', { method: 'POST' });
}
```

Если самостоятельная application-логика отсутствует, такую функцию лучше рассматривать как API-адаптер, а не Use Case.

Если сценарий сложный, необходимо отделить его от транспорта.

### Избыточные абстракции

```text
GetProductsUseCase
ProductRepositoryInterface
ProductRepositoryFactory
HttpProductRepository
ProductService
```

Если вся реализация сводится к одному `GET /products`, такая структура может быть необоснованной.

Нужно оценивать архитектурную ценность каждого дополнительного элемента.

### Формальное наличие слоёв

Создание папок:

```text
domain/
application/
infrastructure/
presentation/
```

не гарантирует Clean Architecture.

Если Domain напрямую импортирует API-клиент, принцип Dependency Rule уже нарушен.

## 15. Архитектурный чек-лист

Перед добавлением или изменением функциональности:

- [ ] Бизнес-правила не зависят от UI и инфраструктуры.
- [ ] Application-сценарии не зависят от конкретных внешних адаптеров.
- [ ] HTTP, storage и другие интеграции локализованы.
- [ ] Доменная модель не зависит от формата внешнего API.
- [ ] Сложные сценарии можно тестировать независимо от внешних сервисов.
- [ ] Stores и hooks не становятся единственным местом хранения бизнес-правил.
- [ ] Зависимости направлены к более стабильной внутренней логике.
- [ ] Используемые абстракции имеют конкретное назначение.
- [ ] Не создано лишних архитектурных файлов и директорий.
- [ ] Структура соответствует правилам FSD.

Чек-лист применяется с учётом реальной ответственности модуля. Отсутствующие обязанности не требуют искусственного создания соответствующих слоёв.

## 16. Источники

- Robert C. Martin — Clean Architecture: A Craftsman's Guide to Software Structure and Design
- [Clean Architecture on Frontend — Alex Bespoyasov](https://bespoyasov.me/blog/clean-architecture-on-frontend/)
- [Feature-Sliced Design](https://feature-sliced.design/)
- [FSD — Layers](https://feature-sliced.design/docs/reference/layers)
- [FSD — Slices and Segments](https://feature-sliced.design/docs/reference/slices-segments)

---

**Итог:** Clean Architecture во frontend — это прежде всего разделение ответственности и направление зависимостей. Бизнес-логика должна оставаться независимой от UI и инфраструктуры независимо от того, реализован модуль в двух файлах или в нескольких отдельных архитектурных слоях.