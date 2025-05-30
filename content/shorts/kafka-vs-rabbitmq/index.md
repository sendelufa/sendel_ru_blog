---
title: "Выжимка Kafka vs Rabbitmq"
date: 2025-05-05T09:42:43+05:00
description: Сравнение и выделение особенностей каждого из брокеров с примерами и пояснениями.
author: "Konstantin Shibkov"
tags: ["dev", "kafka", "rabbitmq"]
categories: ["Разработка"]
draft: false
---
# Общие понятия и сравнение Kafka и RabbitMQ
Очередь сообщений — компонент асинхронной архитектуры, где отправитель (producer) и получатель (consumer) не взаимодействуют напрямую.

Позволяет буферизовать данные, повысить отказоустойчивость, обеспечить масштабирование и decoupling между сервисами.

## Kafka и RabbitMQ — общие черты

| Характеристика      | Kafka                                                                                                                                                                                                                            | RabbitMQ                                                                                                                                                                                          |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Подход              | Log-based, pub/sub                                                                                                                                                                                                               | Classic message queue (AMQP)                                                                                                                                                                      |
| Доставка            | Pull — потребители _читают_ из журнала сами.                                                                                                                                                                                     | Push — сообщения _отправляются_ потребителям.                                                                                                                                                     |
| Архитектура         | Сообщения сохраняются в топике, потребители могут читать их независимо.                                                                                                                                                          | Брокер получает сообщения и доставляет их одному или нескольким потребителям.                                                                                                                     |
| Сохранение          | Длительное хранение (log)                                                                                                                                                                                                        | До подтверждения/TTL/DLQ                                                                                                                                                                          |
| Распределенность    | Из коробки (брокеры, ZooKeeper/KRaft)                                                                                                                                                                                            | Менее масштабируем без кластеризации                                                                                                                                                              |
| Повторная обработка | Да, возможен повторный pull                                                                                                                                                                                                      | Зависит от ack/nack и DLQ                                                                                                                                                                         |
| Идемпотентность         | Требует ручного обеспечения                                                                                                                                                                                                      | Требует ручного обеспечения                                                                                                                                                                       |
| Use-case            | Высоконагруженных систем, стриминга событий, аналитики, масштабируемого логирования.<br><br>- Высокая пропускная способность<br><br>- Историчность событий (event sourcing)<br><br>- Data pipeline (ETL, аналитика, clickstream) | Классической асинхронной обработки задач, микросервисов, где важно подтверждение доставки (ack).<br><br>- Webhooks, RPC<br>    <br>- Отправка email/SMS<br>    <br>- Task-ориентированные сервисы |
| Особенность         | Сообщения остаются в Kafka долго, и потребители могут их перечитывать.                                                                                                                                                           | Поддерживает сложные маршруты через Exchange/Binding.                                                                                                                                             |

Представь магазин:

- **RabbitMQ** будет как курьер — доставляет каждый заказ (сообщение) нужному отделу (потребителю) и убеждается, что его приняли.

- **Kafka** — это как огромная лента заказов, к которой разные отделы могут подключаться, читать старые заказы и анализировать поведение покупателей.

# Устройство и архитектура RabbitMQ

## Основные понятия
- **Producer** — отправитель сообщений. Отправляет сообщение **в Exchange**, а не прямо в очередь. (⛔ Прямой путь в очередь невозможен без участия Exchange.)
- **Exchange** (обменник) — получает сообщение и **на основании типа и правил (Binding)** решает, в какую **очередь или очереди** отправить это сообщение.
- **Queue** — хранилище сообщений, из которого читают консьюмеры. Место, где **буферизуются** сообщения. Один или несколько **Consumers (потребителей)** могут читать из неё.
- **Binding** — это как **маршрутное правило**, связывающее Exchange с очередью. В нём указывается, **при каких условиях сообщение из Exchange попадёт в очередь.**
- **Routing Key** - это строка, которая используется Exchange'ом для принятия решения, в какую очередь (или очереди) направить сообщение. Она задаётся продюсером при отправке сообщения и анализируется Exchange'ом в зависимости от его типа.

### Простой аналог: 📦📬

- Producer — как человек, отправляющий посылку на почте.
- Exchange — как почтовый сортировочный центр.
- Binding — как маршрутная наклейка: "в какой город/отделение отправить".
- Queue — как отделение почты, где хранятся посылки до получения.
- Consumer — как человек, пришедший забирать посылку.

## Типы маршрутизации Exchange
| Тип     | Как используется routing key                                                                                                                                                                                |
| ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Direct  | Должен точно совпадать с ключом в binding'е очереди                                                                                                                                                         |
| Topic   | Использует шаблоны (`*`, `#`) для сопоставления (например, `user.*`, `*.order`)                                                                                                                             |
| Fanout  | Игнорируется. Сообщеие отправляется всем очередям, связанным с exchange, без условий                                                                                                                        |
| Headers | Не используется. По заголовкам сообщения, не по ключу. Хорошо подходит для сложной маршрутизации, основанной на мета-информации. Часто используется для фильтрации по типу документа, региону, языку и т.д. |

👉 Routing key настраивается в момент Binding — при связи очереди и обменника. Можно сказать, что **очередь "подписывается" на сообщения с определёнными ключами** при помощи binding'а.

### 🔍 Примеры:

Direct:

```
Routing Key: order.created
Binding Key: order.created
→ Сообщение попадёт в очередь
```

Topic:

```
Routing Key: user.profile.updated
Binding Key: user.*
→ Сообщение попадёт в очередь

Routing Key: user.profile.updated
Binding Key: user.#
→ Тоже попадёт, так как # — подстановочный шаблон
```

### Разница между `*` и `#` в routing key шаблонах

| Символ | Значение                              |
| ------ | ------------------------------------- |
| `*`    | Заменяет **ровно одно** слово (токен) |
| `#`    | Заменяет **ноль или более** слов      |


**Примеры: Какие routing keys попадут в очередь?**

У нас есть очередь с привязкой `Binding key: user.*.created`

| Routing Key                  | Совпадение | Объяснение                                         |
| ---------------------------- | ---------- | -------------------------------------------------- |
| `user.profile.created`       | ✅          | `*` = `profile` (1 токен между `user` и `created`) |
| `user.a.created`             | ✅          | `*` = `a`                                          |
| `user.created`               | ❌          | нет промежуточного токена                          |
| `user.profile.extra.created` | ❌          | слишком много токенов                              |

Если очередь с привязкой: `Binding key: user.#.created`

| Routing Key                     | Совпадение | Объяснение               |
| ------------------------------- | ---------- | ------------------------ |
| `user.created`                  | ✅          | `#` = 0 слов             |
| `user.profile.created`          | ✅          | `#` = `profile`          |
| `user.profile.extra.created`    | ✅          | `#` = `profile.extra`    |
| `user.profile.extra.v1.created` | ✅          | `#` = `profile.extra.v1` |
## 📤 Публикация сообщений (Producer → Exchange → Queue)

1. **Producer** открывает соединение (TCP) и создает **Channel**.
2. Producer публикует сообщение с указанием:
    - Exchange (куда отправить)
    - Routing key (для маршрутизации)
    - Headers (если нужен headers exchange)
    - Body (полезная нагрузка)
3. **Exchange** принимает сообщение и сопоставляет его с очередями по правилам:
    - Direct / Topic / Fanout / Headers
4. Сообщение помещается в подходящие **очереди** (одну или несколько).

### 🔁 Publisher Confirms vs Transactions

А теперь интересная часть — как понять, что **сообщение действительно было доставлено**?

1. Transactions
	- Producer может начать транзакцию, опубликовать сообщение, а потом её зафиксировать (commit).
	- Это **надежно**, но **медленно** — каждое сообщение требует полной транзакции. И следующее сообщение отправляется только после подтверждения предыдущего.
2. Publisher Confirms
	- Более эффективный способ: producer получает **асинхронное подтверждение** от брокера, что сообщение получено и сохранено.
	- Это **намного быстрее**, особенно в высоконагруженных системах. Отправляем сообщения одно за другим и получаем `ack (Acknowledgement (подтверждение))` или `nack (Negative Acknowledgement (отказ))` и реагируем на это. Например, если сообщение не обработано publisher, то его можно еще раз попробовать отправить или логировать ошибку.

## 📥 Вычитка сообщений (Consumer ← Queue)

1. Consumer подключается к RabbitMQ и открывает **Channel**.
2. Отправляет команду `basicConsume`, подписываясь на очередь.
3. RabbitMQ **пушит сообщения** в Consumer по TCP (Push-модель).
4. Consumer:
    - Обрабатывает сообщение
    - Отправляет `ACK` (если авто-подтверждение выключено)

### Под капотом при вычитке

| Механизм           | Назначение                                                  |
| ------------------ | ----------------------------------------------------------- |
| **Prefetch (QoS)** | Ограничивает число неподтвержденных сообщений               |
| **Manual ack**     | Позволяет точно контролировать подтверждение                |
| **Auto-ack**       | Сообщение считается принятым сразу (риск потери при сбое)   |
| **Nack / Reject**  | Явный отказ от обработки, можно вернуть в очередь или в DLQ |

## 🎯 Сценарий "из жизни"

> Один сервис публикует события в `event.exchange` с routing key `order.created`, которые доставляются в очередь `billing.queue`. Консьюмер из `billing-service` подписан на `billing.queue`, обрабатывает события и подтверждает их вручную. При ошибке сообщение отправляется в DLQ.

### Что происходит если консьюмер не обработал сообщение?

- Сообщения от продюсера идут в основную очередь через основной обменник.
- Если потребитель не может обработать сообщение, он отправляет его в **retry exchange**, который направляет в **retry очередь** с задержкой (TTL = 60 секунд).
- После истечения TTL, сообщение автоматически возвращается обратно в основную очередь через DLX.
- Если количество попыток превышено, сообщение можно направить в **DLQ** (dead-letter queue) для последующего анализа.

```css
[Producer]
     |
     v
[Main Exchange]
     |
     v
[Main Queue] ---> [Consumer]
                     |
                (nack or fail)
                     |
                     v
             [Retry Exchange]
                     |
                     v
          [Retry Queue (TTL=60s)]
                     |
         (message expires via TTL)
                     |
                     v
             [Main Exchange]  <---+
                                |
                     +------------+
                     |
               [DLQ] (если макс. попытки исчерпаны)

```

## 📤 Push vs Pull: модель доставки в RabbitMQ

👉 **RabbitMQ использует push-модель доставки**.

- **Push** — брокер сам **отправляет** сообщения потребителям, как только они появляются в очереди.
- В отличие от Kafka, где **потребитель сам запрашивает** (`pull`) данные, в RabbitMQ они приходят "сами".


🤔 Почему это важно?
Потому что push-модель:
- быстрее для большинства приложений
- но может перегружать медленных потребителей (и тут важно использовать `prefetch`)

### 🔄 Round-Robin и Prefetch

 1. 🌀 Round-Robin (по умолчанию)
RabbitMQ отправляет сообщения **по очереди** между всеми активными потребителями.
Например, если у тебя 3 потребителя и 6 сообщений, то каждый получит по 2.
🧠 Но это только **если prefetch не ограничен**.

2. 🎛️ Prefetch
Параметр `basic.qos(prefetch_count=N)` задаёт, **сколько сообщений может быть отправлено потребителю без подтверждения** (`ack`).

Например:
- `prefetch = 1`: RabbitMQ подождёт, пока ты обработаешь одно сообщение, прежде чем дать тебе следующее.
- Это предотвращает "перекос", когда один быстрый потребитель получает слишком много, а другие простаивают.
📌 **Prefetch помогает сбалансировать нагрузку** между медленными и быстрыми потребителями.

### 🔗 Channel vs Connection

 **🔌 Connection**
- Это **TCP-соединение** между приложением и RabbitMQ сервером.
- ❗ Дорогое в создании. Часто открывается один раз и живёт долго.

**🔄 Channel**
- Это **логическое соединение внутри TCP-соединения**.
- Лёгкий, быстрый.
- На одном соединении можно открыть **много каналов** — по одному на каждый поток, например.
- Рекомендуется: **один канал на один поток** (в многопоточной среде).
📌 Channels — это то, где ты публикуешь, подписываешься, подтверждаешь, и т.п.

🚧 Очень желательно использовать принцип один поток приложения = один channel. Это позволяет избежать race condition. Риски когда несколько потоков используются один channel это смешение команд (сообщение ушло, но не туда или подтверждение было отправлено не тому сообщению)

✅ Идеальный подход:
> Один поток = Один channel (в рамках одного общего connection)

```bash
[Connection] ← открывается один раз, шарится
     ├── [Channel 1] ← используется Потоком 1
     ├── [Channel 2] ← используется Потоком 2
     └── [Channel N] ← используется Потоком N

```

🔧 В Java это обычно делается через:
- Пул соединений (например, `CachingConnectionFactory` в Spring AMQP)
- Потокобезопасные фабрики каналов
- Обёртки, которые создают **channel на поток**, и закрывают его корректно при завершении

 Долгоживущие соединения (реальный сценарий):
- Создаёшь **один connection** при старте приложения
- На каждый поток создаёшь канал:
```java
Channel channel = connection.createChannel();
// регистрируешь shutdown hook или ловишь исключения
channel.close();
```

## Полезные советы:

- При завершении приложения — **всегда закрывай connection**, иначе соединение останется висеть на сервере.
- При исключениях (например, потребитель упал) — **лови и закрывай канал**.
- В Spring AMQP — можно использовать `@PreDestroy` или `DisposableBean` для очистки.

## 🧵 Параллельная вычитка и масштабирование

RabbitMQ масштабируется по горизонтали через:

 ✅ Несколько потребителей на одну очередь
- Несколько консьюмеров (в разных потоках/инстансах) читают **одну очередь**
- RabbitMQ распределяет сообщения **round-robin** (если нет prefetch!)
📌 Это способ масштабировать **параллельную обработку**
---
 ✅ Несколько очередей (sharding)
- Каждая очередь обслуживает свою часть сообщений (например, по ID пользователя)
- Каждый consumer читает свою очередь
📌 Это уже **шардинг**, даёт ещё больший контроль

## ⚖️ Prefetch count — баланс между скоростью и справедливостью

 `prefetch=1`
- Каждому консьюмеру даётся **одно сообщение за раз**
- Отлично для медленных задач, чтобы не перегружать никого
- 📉 Низкая производительность, но 📈 равномерная загрузка

 `prefetch=N`
- RabbitMQ может отправить сразу **N сообщений** консьюмеру
- Увеличивает throughput (черезput)
- Но! Быстрые потребители могут "захватить" больше, а медленные простаивать
🎯 Надо подбирать `prefetch` по типу нагрузки и числу потоков

## ❌ Проблема дублирующихся сообщений и идемпотентность

Вот реальная боль. В RabbitMQ **гарантия — at least once**, значит:
> Сообщение может быть доставлено **повторно** (например, после сбоя до `ack`)

---

🔄 Что с этим делать?

👉 **Идемпотентная обработка** — обработка, которая не изменяет результат при повторном выполнении.

**Примеры:**
- Проверка: “обрабатывал ли я это сообщение с таким ID?”
- Использование базы с `unique constraint` по `message_id`
- Использование Redis для хранения `processed_message_ids`
---
📌 Важно:
- RabbitMQ **не гарантирует**, что сообщение придёт **только один раз**
- Это должен гарантировать **твой код**

# Устройство и архитектура Kafka

 📦 **Topic**
- Это как “категория событий” — поток сообщений на одну тему.
- Пример: `user-activity`, `orders`, `logs`, `payment-failures`.
Ты как бы говоришь: “Вот сюда я буду складывать все события типа Х”.

 🔀 **Partition**
 Партиции — это **физическое деление топика** на несколько потоков данных. Каждая партиция — это **упорядоченный, непрерывный лог** (append-only).
- Каждый топик делится на **партиции (разделы)** — это **ключ к масштабированию и параллельной обработке**.
- Сообщения в партиции **упорядочены**, но **между партициями порядок не гарантируется**.

Партиции нужны, чтобы:
- распределить данные по брокерам
- давать каждому консьюмеру свою "дорожку"
- параллельно обрабатывать данные

📌 Итого:
> Один `topic` = N `partitions`
> В каждой партиции — **свой лог** (append-only), читаемый независимо

**Как формируются партиции?**

Когда ты создаёшь топик, ты указываешь, сколько у него будет партиций:

```bash
kafka-topics.sh --create --topic my-topic --partitions 3 ...
```

Тогда Kafka создаст:
- `my-topic-0`
- `my-topic-1`
- `my-topic-2`

И будет распределять сообщения по этим частям.

Producer отправляет сообщение → Kafka решает:
- Если указан **ключ (key)** → Kafka применяет `hash(key) % partitions` → однотипные сообщения пойдут в одну и ту же партицию (например, по `userId`)
- Если ключ не указан → Kafka может распределить **равномерно**

👉 Пример:
У тебя есть топик `user-actions` с 3 партициями:

| Partition 0 | Partition 1 | Partition 2 |
| ----------- | ----------- | ----------- |
| user 1      | user 2      | user 3      |
| user 4      | user 5      | user 6      |
Пишем с ключами — `userId`
→ пользователь 1 всегда идёт в Partition 0
→ порядок сохраняется **внутри одной партиции**

⚠️ Kafka гарантирует порядок сообщений только внутри одной партиции.

Так что если у тебя события:
1. `user.registered`
2. `user.emailConfirmed`
3. `user.passwordChanged`
и они **распределятся по разным партициям**, то:
- Один consumer может получить `passwordChanged` раньше `emailConfirmed` 😱
- Это приведёт к **логической ошибке** в бизнес-процессе
## Producers, Brokers, Offset, Consumers

 🛠 **Producer**
- Отправляет события в Kafka.
- Может сам решать, **в какую партицию писать**:
    - По ключу (`key`) — например, по `userId` для стикерованного порядка
    - Случайно — для равномерной нагрузки

 🧱 **Broker**
- Сервер Kafka. Один Kafka-брокер — это узел в кластере.
- Хранит партиции и обслуживает запросы от продюсеров и консьюмеров.

📌 Несколько брокеров = кластер Kafka
→ Партиции распределяются по ним

### 🔢 Offset
📌 Offset — это просто **номер сообщения в партиции**.

Kafka хранит все сообщения в каждой партиции **в строгом порядке**. И каждое сообщение получает свой **порядковый номер**, начиная с 0:
```bash
Partition 0:
[0] User A зарегистрировался
[1] User A подтвердил email
[2] User A сменил пароль
```

### 📦 Пример:
Ты читаешь Partition 0, и обработал offset 1
→ Kafka теперь знает: “Ты остановился на 1, следующее будет 2”

Если ты перезапустишь потребителя:
- Он продолжит **с offset = 2**, если offset **сохранён**
- Или начнёт сначала / с конца — если offset **не сохранён**
### 🧾 **Consumer**
- Читает сообщения из одной или нескольких партиций.
- Подписан на один или несколько топиков.

## Offset **хранится отдельно для каждой consumer group**

То есть:
- Группа `analytics` может читать `orders` и быть на offset 100
- Группа `billing` читает тот же `orders`, но на offset 70

🔁 Kafka не удаляет сообщение после чтения — ты сам контролируешь, где ты **находишься в журнале событий**

### 👥 Consumer group
Это **группа потребителей, которые читают один и тот же топик ВМЕСТЕ**.
Но каждый из них читает **разные партиции**, чтобы:
> 📌 Обрабатывать сообщения **параллельно**, но **не дублируя** их.

### 🎯 Пример:
Допустим, у тебя есть топик `orders` с 3 партициями: `P0`, `P1`, `P2`.
Ты запускаешь **3 консьюмера**, и даёшь им **одно и то же имя группы**, например `order-processors`.

→ Kafka говорит:
> "Ага, вы все из одной группы. Тогда я **поделю партиции между вами**."

```bash
P0 → Consumer 1  ↰
P1 → Consumer 2   ↣ одна группа
P2 → Consumer 3  ↲
```

**Главное правило:**
В одной consumer group, одна партиция → одному потребителю → Чтобы не было дублирования, но вся группа обработала все данные

❓Что будет если добавить `Consumer 4`?
- Consumer 4 не будет получать сообщения, так как ему не достанется партиции

❓Полезен ли избыток консьюмеров?
- Да, потому что если кто-то из активных потребителей **вдруг упадёт**, Kafka скажет: “О, у меня есть свободный участник!” → И сделает **rebalance**, назначив ему партицию, которую обрабатывал упавший.
📌 То есть: **дополнительные консьюмеры = резерв**

## Перепаспределение консьюмеров при сбое

Допустим:
- У тебя есть топик с 3 партициями: `P0`, `P1`, `P2`
- Есть consumer group `order-processors` с 3 потребителями: `C1`, `C2`, `C3`
- Каждому назначена своя партиция:
```
P0 → C1
P1 → C2
P2 → C3
```

**❌ Что происходит, если `C2` (обрабатывающий `P1`) упал?**
Kafka не может “удерживать” партицию за упавшим консьюмером, потому что:
> В consumer group партиции не закреплены жёстко за консьюмерами навсегда.

**Kafka запускает rebalance:**
1. Все оставшиеся живые консьюмеры (например, `C1` и `C3`) **отключаются от своих партиций временно**
2. Kafka **перераспределяет все партиции заново** среди доступных
```
P0 → C1
P1 → C3
P2 → C1
```
→ Теперь `C3` обрабатывает `P1`, которую раньше читал `C2`

Важно:
- **Kafka не привязывает партиции к конкретному консьюмеру навсегда**
- Перераспределение происходит **каждый раз**, когда:
    - консьюмер **уходит или появляется**
    - **число партиций** изменяется
    - **число участников группы** меняется

🟡 **Кто отвечает за отслеживание состояния группы?**
Kafka использует **Group Coordinator** — это один из брокеров, который:
- следит за **пульсом** (heartbeat) от каждого консьюмера
- сохраняет offset'ы
- запускает **rebalance**, когда нужно

🟡 **Как Kafka определяет, что consumer “пропал”?**
Когда ты запускаешь `KafkaConsumer`, он:
1. Присоединяется к группе (через `group.id`)
2. Регулярно отправляет **heartbeat** брокеру (Group Coordinator)
3. Если heartbeat **не приходит вовремя**, брокер считает его **“мертвым”**

🕒 Важные параметры:

| Параметр                | Значение                                                                           |
| ----------------------- | ---------------------------------------------------------------------------------- |
| `session.timeout.ms`    | Сколько времени Kafka ждёт heartbeat перед тем как считать потребителя "пропавшим" |
| `heartbeat.interval.ms` | Как часто потребитель шлёт heartbeat                                               |
| `max.poll.interval.ms`  | Макс. время между `poll()` — защита от зависания                                   |

**🧨 Если heartbeat не пришёл → ребаланс**
Когда Kafka решает, что потребитель умер:
- **вся группа переходит в режим ребаланса**
- Партиции перераспределяются между оставшимися
- Консьюмеры получают коллбек: `onPartitionsRevoked` → `onPartitionsAssigned`

**🧨 Если консьюмер не пришел за новым сообщение за `max.poll.interval.ms`→ ребаланс**
Когда Kafka решает, что потребитель завис, даже если отправляет heartbeat:
- **вся группа переходит в режим ребаланса**
- Партиции перераспределяются между оставшимися
- Консьюмеры получают коллбек: `onPartitionsRevoked` → `onPartitionsAssigned`

❓Если один консьюмер в группе долго обрабаывает сообщения — блокирует ли он других на прочтение новых сообщений?
**Нет, не блокирует.**
→ Потому что:
> Каждая партиция обрабатывается независимо, и offset сохраняется отдельно для каждой партиции и consumer group.

 🎯 Ключевые принципы:
1. **Offset хранится по схеме:**

```
<consumer group ID> + <partition ID> → offset
```

→ То есть:
- Не один offset на всю группу!
- У каждой партиции свой offset

📌 Поэтому, даже если один consumer завис:
- Он блокирует **только свои партиции**
- Остальные консьюмеры продолжают обрабатывать свои

2. **Консьюмеры не зависят друг от друга**
Если:
- `C1` обрабатывает `P0` и завис
- `C2` обрабатывает `P1` — он продолжает работать!
- Kafka никак не блокирует `C2`

2. **Подтверждение обработки = commit offset**
Консьюмер:
- читает сообщение (например, offset = 101)
- обрабатывает его
- затем делает **commit(offset = 102)**
    → это говорит Kafka: "я обработал всё до 101 включительно"

📌 Commit может быть:
- автоматическим (`enable.auto.commit=true`)
- ручным (`commitSync()` или `commitAsync()`)

⚠️ Если consumer **обработал сообщение**, но **не сделал commit offset**:
→ Kafka **не знает**, что сообщение было успешно обработано
→ Значит, **offset не сдвигается**
→ При следующем запуске consumer:
- Он получит **то же самое сообщение снова**
- Потому что Kafka думает: “ты его ещё не читал!”

📌 Поэтому всегда:
- Обработка должна быть **идемпотентной**
- Offset'ы нужно **коммитить вручную**, если ты хочешь контролировать надёжность
# Итог

## ✅ Когда использовать **Kafka**:

- 🔁 **Важен порядок событий** внутри ключа (например, `userId`)
- ⏳ **Хочешь хранить события долго** (например, 7 дней или больше)
- 📊 **Нужно читать один и тот же поток разными сервисами**, независимо (consumer groups)
- 🔄 **Нужно перечитывать события** (replay), перемещая offset назад
- 🚀 **Обрабатываешь большие объемы данных в реальном времени** — логи, аналитика, мониторинг

📌 Kafka = поток данных (stream), ориентированный на **лог событий** и **масштаб**

## ✅ Когда использовать **RabbitMQ**:
- 📮 **Нужна быстрая и надёжная доставка задач между микросервисами**
- 🧾 **Каждое сообщение нужно обработать один раз и забыть**
- ⏱ **Хочешь гибко контролировать маршруты сообщений** (`direct`, `fanout`, `topic`)
- 💥 **Важно обрабатывать события немедленно** (push-модель)
- 📦 **Нужна DLQ и повторные попытки** с контролем времени

📌 RabbitMQ = очередь задач (task queue), ориентированная на **событийную обработку**

## 🔄 Когда используют Kafka + RabbitMQ вместе
### 🎯 Сценарий: у тебя микросервисная система
1. **Kafka** используется для:
    - Сбор событий из всех сервисов (`user.created`, `order.placed`, `payment.failed`)
    - Хранение этих событий как логов
    - Передача данных в системы аналитики, отчётности и логирования

2. **RabbitMQ** используется для:
    - Реального исполнения задач: отправка email, генерация PDF, дебет с карты
    - Асинхронная, быстрая доставка в очередь задач
    - Повторы, DLQ, маршрутизация и контроль доставки

- **События** в Kafka:
    - Нельзя терять
    - Нельзя перепутать порядок (`user.created` → `user.verified`)
    - Нужно уметь "перемотать назад" (replay)

- **Задачи** в RabbitMQ:
    - Можно выполнять в любом порядке
    - Можно повторять (через DLQ)
    - Главное — чтобы кто-то это **выполнил**
