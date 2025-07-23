---
title: "Сравнение MacBook M1 Pro и M4 Pro глазами разработчика"
date: 2025-07-22T22:56:27+03:00
description: "Сравнил MacBook M1 Pro и M4 Pro с упором на задачи разработчика: сборка проектов, тесты, AI и рендеринг. Цифры, реальные проекты и вывод — стоит ли обновляться?"
author: "Konstantin Shibkov"
tags: ["review", "apple", "macbook"]
categories: ["review"]
draft: false
image: cover.webp
---

При переходе на новое поколение MacBook я решил провести серию тестов,
чтобы оценить изменения именно с точки зрения повседневной работы разработчика.

Надеюсь, это поможет понять, стоит ли обновляться, и насколько четвёртое
поколение действительно лучше самого первого чипа Apple Silicon — M1.

## Различие в железе

Начнем с простого перечисления технических характеристик, которые изменились:

| Свойство             | MacBook M1 Pro                   | MacBook M4 Pro                  |
| -------------------- | -------------------------------- | ------------------------------- |
| Процессор            | 6 произв + 2 энергоэфф           | 10 произв + 4 энергоэфф         |
| GPU                  | 14 ядер                          | 20 ядер                         |
| Шина памяти          | 200 ГБ/с                         | 273 ГБ/с                        |
| Neural Engine        | до 11 TOPS                       | до 38 TOPS                      |
| Аппаратное ускорение | H.264, HEVC, ProRes и ProRes RAW | Все, что в M1 + AV1 + x2 ProRes |
| Thunderbolt          | версия 4, 40 Гб/c                | версия 5, 120 Гб/c              |
| Bluetooth            | 5.0                              | 5.3                             |
| Wi-Fi                | 6, до 1,146 Гб/c                 | 6E, до 1,788 Гб/c               |
| Камера               | ~2,1 МП                          | 12 МП Center Stage              |
| SSD, чтение          | 5109 Мб/c                        | 7715 Мб/c                       |
| SSD, запись          | 3570 Мб/c                        | 5107 Мб/с                       |
| Картридер, чтение    | 148,4 Мб/с                       | 152,2 Мб/с                      |
| Картридер, запись    | 89,3 Мб/с                        | 90,2 Мб/c                       |

> Тест SSD и Картридера производился через BlackMagic Disk Speed Test. Карточка SD Transcend UHS-I(SD 3.0)
> с заявленным 125 Мб/c на запись и 160 Мб/с на чтение. Выглядит так, что картридер не изменился, но это
> неточно.

**Что действительно оказалось заметным и важным для меня:**

- Разница в производительности CPU и GPU
- Улучшенная камера с более качественным изображением и режимом «В центре кадра» (Center Stage)
- Поддержка кодека AV1 и большее количество аппаратных кодеров/декодеров
ProRes — при рендере видео прирост производительности хорошо ощущается.
Хотя это, скорее, часть общего прироста CPU+GPU

🧐 **Thunderbolt 5** — это круто, но пока таких SSD почти нет или они стоят заметно дороже.
Реальная польза — в будущем.

🧐 Wi‑Fi 6E может быть полезен, если у вас дома интернет свыше 1 Гб/с.
У меня канал 500 Мб/с, и разницы по факту нет. Пока простого Wi-Fi 6 хватает.

🧐 Bluetooth 5.3 якобы надёжнее и энергоэффективнее, но на практике разницу заметить сложно.
С наушниками с Bluetooth 5.0/5.3 всё работало стабильно и раньше.

## Как проводились тесты

Каждый тест представлял собой компиляцию проекта и запуск тестов.
Для повышения точности каждый запуск выполнялся 5 раз, после чего вычислялось
среднее время выполнения.

Заранее для каждого проекта были загружены зависимости в кэш (Gradle/Maven/npm),
чтобы исключить влияние скачивания на результат.

Большинство тестов вы можете воспроизвести самостоятельно — большниство репозиториев публичные.
Я указываю конкретные коммиты, на которых стоит запускать тесты,
чтобы обеспечить повторяемость условий.

Все тесты проводились при включённом режиме «Максимальная производительность»
в настройках питания, устройства были подключены к сети и полностью заряжены (100%).

## 1 - spring-boot-realworld-example-app

> https://github.com/gothinkster/spring-boot-realworld-example-app

Небольшой Spring Boot проект. Требует Java 11 версии, для тестов использовалась версия
openjdk 11.0.26.

Получить репозиторий и нужный коммит:

```bash
git clone https://github.com/gothinkster/spring-boot-realworld-example-app.git project1
cd project1
git checkout ee17e31
```

Перед запуском сборки надо инициализировать проект

```bash
./gradlew :spotlessApply
```

Запуск сборки с учетом времени выполнения команды.
Первый раз будет долгий запуск, так будут скачиваться все зависимости:

```bash
time ./gradlew clean build
```

Запуск приложения, сборка проекта уже выполнена.

```bash
time ./gradlew bootRun
```

### Результаты

| Команда        | MacBook M1 Pro | MacBook M4 Pro | Разница (%) |
| -------------- | -------------- | -------------- | --|
| clean build, с | 15,54          | 7,91           | −49,1% |
| bootRun, с     | 5,05           | 3,40           | −32,7% |

{{< chart >}}
{
    type: 'bar',
    data: {
        labels: [
          'M1 Pro build',
          'M4 Pro build',
          'M1 Pro bootRun',
          'M4 Pro bootRun'
        ],
        datasets: [{
            label: 'ceкунды',
            data: [15.54, 7.91, 5.05, 3.40],
            backgroundColor: [
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                        ]
        }]
    },
    options: {
        indexAxis: 'y',
        plugins: {
            legend: {
                display: false
            },
            title: {
                display: true,
                text: 'spring-boot-realworld-example-app'
            }
        }
    }
}
{{< /chart >}}

## 2 - Traccar - OS GPS Tracking

> https://github.com/traccar/traccar

Приложение для GPS трекинга с открытым исходным кодом.

```bash
git clone --recursive https://github.com/traccar/traccar.git
git checkout 5d829b46a
```

Сборка JAR серверной части и тесты, использовалась openjdk 24+37

```bash
./gradlew clean assemble
```

---

```bash
./gradlew clean test
```

Сборка Web UI:

```bash
npm run build
```

### Результаты

| Команда     | MacBook M1 Pro | MacBook M4 Pro | Разница (%) |
| ----------- | -------------- | -------------- | --|
| assemble, с      | 8,95           | 5,63           | -37% |
| test, с          | 12,60          | 8,33           | -33% |
| npm run build, с | 7,47           | 4,45           | -40% |


{{< chart >}}
{
    type: 'bar',
    data: {
        labels: [
          'M1 Pro assemble',
          'M4 Pro assemble',
          'M1 Pro test',
          'M4 Pro test',
          'M1 Pro npm run build',
          'M4 Pro npm run build'
        ],
        datasets: [{
            label: 'ceкунды',
            data: [8.95, 5.63, 12.60, 8.33, 7.47, 4.45],
            backgroundColor: [
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                        ]
        }]
    },
    options: {
        indexAxis: 'y',
        plugins: {
            legend: {
                display: false
            },
            title: {
                display: true,
                text: 'Traccar - OS GPS Tracking'
            }
        }
    }
}
{{< /chart >}}


## 5 - Рабочий проект

Один из микросервисов, над которым я сейчас работаю, к сожалению, не могу показать — исходный код закрыт.
Поэтому для тестов я выбрал не слишком большой открытый проект, чтобы можно было сделать больше прогонов.

Общие характеристики:

- Java 17
- Spring Boot 3.2
- Maven
- Количество строк кода без комментов и пустых строк:  43 967
- Количество тестов: 923

Запуск компиляции:

```bash
time ./mvnw clean compile
```

---

```bash
time ./mvnw clean test
```

### Результаты

| Команда     | MacBook M1 Pro | MacBook M4 Pro | Разница (%) |
| ----------- | -------------- | -------------- | --|
| compile, с  | 12,53          | 7,56           | -39% |
| test, с     | 29,79          | 23,62          | -21% |

{{< chart >}}
{
    type: 'bar',
    data: {
        labels: [
          'M1 Pro compile',
          'M4 Pro compile',
          'M1 Pro test',
          'M4 Pro test'
        ],
        datasets: [{
            label: 'ceкунды',
            data: [12.53, 7.56, 29.79, 23.62],
            backgroundColor: [
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                        ]
        }]
    },
    options: {
        indexAxis: 'y',
        plugins: {
            legend: {
                display: false
            },
            title: {
                display: true,
                text: 'Рабочий проект Spring Boot'
            }
        }
    }
}
{{< /chart >}}

> В тестах много интеграционных проверок с использованием Testcontainers, и, похоже, работа с Docker и сетевое взаимодействие становятся бутылочным горлышком — прирост производительности уже не столь значителен. Для пробы я взял ещё один проект с ~1000 тестами — ситуация повторилась:
на M1 Pro — 2 мин 11 сек, на M4 Pro — 1 мин 30 сек, разница составила –31%.
Причём в этом проекте большинство тестов — юнит-тесты.

## 6 - Просто JS проект

Создал пустой проект

```bash
npx create-react-app test-build-speed
cd test-build-speed
npm install ajv@8 ajv-keywords@5
time npm run build
```

### Результаты

| Команда     | MacBook M1 Pro | MacBook M4 Pro | Разница (%) |
| ----------- | -------------- | -------------- | --|
| build, с | 4,80           | 1,95           | -59% |

{{< chart >}}
{
    type: 'bar',
    data: {
        labels: [
          'M1 Pro build',
          'M4 Pro build',
        ],
        datasets: [{
            label: 'ceкунды',
            data: [4.80, 1.95],
            backgroundColor: [
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                        ]
        }]
    },
    options: {
        indexAxis: 'y',
        plugins: {
            legend: {
                display: false
            },
            title: {
                display: true,
                text: 'npm run build'
            }
        }
    }
}
{{< /chart >}}

## 7 - Локальный AI

Использовал:

- LM Studio 0.3.18
- Модель Gemma 3 12B 4bit

Модель выбрана так, чтобы поместится в память M1 Pro с 16Гб ОЗУ.

### 1. Запрос генерации простого кода

> Напиши Pojo класс являющимся Value Object. Напиши классы для веса, валюты. Используй Java 17.

**M1 Pro**
* 16.23 токенов/сек
* 1.30 сек. до первого токена

**M4 Pro**
* 32.03 токенов/сек
* 0.33s сек. до первого токена

{{< chart >}}
{
    type: 'bar',
    data: {
        labels: [
          'M1 Pro',
          'M4 Pro',
        ],
        datasets: [{
            label: 'токенов/сек',
            data: [16.23, 32.03],
            backgroundColor: [
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                        ]
        },
        {
            label: 'sec',
            data: [1.30, 0.33],
            backgroundColor: [
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                        ]
        }]
    },
    options: {
        indexAxis: 'y',
        plugins: {
            legend: {
                display: false
            },
            title: {
                display: true,
                text: 'Текстовый запрос – генерация кода'
            }
        }
    }
}
{{< /chart >}}

### 1.2 Добавлено уточнение

> А можешь использовать record

**M1 Pro**
* 17.07 токенов/сек
* 21.40 сек. до первого токена

**M4 Pro**
* 31.59 токенов/сек
* 5.90 сек. до первого токена

{{< chart >}}
{
    type: 'bar',
    data: {
        labels: [
          'M1 Pro',
          'M4 Pro',
        ],
        datasets: [{
            label: 'tok/sec',
            data: [17.07, 31.59],
            backgroundColor: [
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                        ]
        },
        {
            label: 'sec',
            data: [21.40, 5.90],
            backgroundColor: [
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                        ]
        }]
    },
    options: {
        indexAxis: 'y',
        plugins: {
            legend: {
                display: false
            },
            title: {
                display: true,
                text: 'Текстовый запрос – генерация кода'
            }
        }
    }
}
{{< /chart >}}

### 2. Проанализируй изображение

![Изображения для анализа](lamp.webp)

> Что изображено на изображении?

**M1 Pro**
* 18.24 токенов/сек
* 6.61 сек. до первого токена

**M4 Pro**
* 33.10 токенов/сек
* 2.21 сек. до первого токена

{{< chart >}}
{
    type: 'bar',
    data: {
        labels: [
          'M1 Pro build',
          'M4 Pro build',
        ],
        datasets: [{
            label: 'tok/sec',
            data: [18.24, 33.10],
            backgroundColor: [
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                        ]
        },
        {
            label: 'sec',
            data: [6.61, 2.21],
            backgroundColor: [
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                        ]
        }]
    },
    options: {
        indexAxis: 'y',
        plugins: {
            legend: {
                display: false
            },
            title: {
                display: true,
                text: 'Анализ изображения'
            }
        }
    }
}
{{< /chart >}}

### 3. Анализ PDF файла

Был выбран файл и запросы суммаризации по нему:

- 5 основых идей
- Опиши общую концепцию предложенную в файле

Использовался файл-спецификация JSR-133.

**M1 Pro**
* 17.35 токенов/сек
* 9.73 сек. до первого токена

**M4 Pro**
* 31.95 токенов/сек
* 2.78 сек. до первого токена

{{< chart >}}
{
    type: 'bar',
    data: {
        labels: [
          'M1 Pro build',
          'M4 Pro build',
        ],
        datasets: [{
            label: 'tok/sec',
            data: [17.35, 31.95],
            backgroundColor: [
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                        ]
        },
        {
            label: 'sec',
            data: [9.73, 2.78],
            backgroundColor: [
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                        ]
        }]
    },
    options: {
        indexAxis: 'y',
        plugins: {
            legend: {
                display: false
            },
            title: {
                display: true,
                text: 'Анализ PDF файла'
            }
        }
    }
}
{{< /chart >}}


## 8 – Конвертация видео

Сценарий: была записана встреча в Контур Толке, после чего видео конвертировалось для хранения или публикации.

Исходник — видео в h.264 с продолжительностью 1:15:56, записанное во время созвона.
Цель — подготовить ролик для загрузки на YouTube в формате 1080p30.

Для конвертации использовался HandBrake версии 1.9.2.

| MacBook M1 Pro | MacBook M4 Pro | Разница (%) |
| -------------- | -------------- | --|
| 7м 40с          | 3м 15с          | -57% |

{{< chart >}}
{
    type: 'bar',
    data: {
        labels: [
          'M1 Pro',
          'M4 Pro',
        ],
        datasets: [{
            label: 'ceкунды',
            data: [7.67, 3.25],
            backgroundColor: [
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                        ]
        }]
    },
    options: {
        indexAxis: 'y',
        plugins: {
            legend: {
                display: false
            },
            title: {
                display: true,
                text: 'Конвертация видео'
            }
        }
    }
}
{{< /chart >}}

> Увеличение количество аппаратных декодера с 1 до 2 дает о себе знать.
Обработка более чем в 2 раза быстрее.

## 9 – Синтетика Cinebench 2024.1.0

Прогнал синтетические тесты, которые максимально могут показать
разницу в потенциале процессоров и GPU.

| Тест     | MacBook M1 Pro | MacBook M4 Pro | Разница (%) |
| ----------- | -------------- | -------------- | --|
| GPU, pts         | 2 256      | 9 147           | +305% |
| CPU Single, pts  | 112       | 172            | +54%  |
| CPU Multi, pts   | 630       | 1501           | +138% |

{{< chart >}}
{
    type: 'bar',
    data: {
        labels: [
          'M1 Pro',
          'M4 Pro',
        ],
        datasets: [{
            label: 'pts',
            data: [2256, 9147],
            backgroundColor: [
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                        ]
        }]
    },
    options: {
        indexAxis: 'y',
        plugins: {
            legend: {
                display: false
            },
            title: {
                display: true,
                text: 'GPU'
            }
        }
    }
}
{{< /chart >}}

{{< chart >}}
{
    type: 'bar',
    data: {
        labels: [
          'M1 Pro',
          'M4 Pro',
        ],
        datasets: [{
            label: 'pts',
            data: [112, 172],
            backgroundColor: [
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                        ]
        }]
    },
    options: {
        indexAxis: 'y',
        plugins: {
            legend: {
                display: false
            },
            title: {
                display: true,
                text: 'CPU Single'
            }
        }
    }
}
{{< /chart >}}

{{< chart >}}
{
    type: 'bar',
    data: {
        labels: [
          'M1 Pro',
          'M4 Pro',
        ],
        datasets: [{
            label: 'pts',
            data: [630, 1501],
            backgroundColor: [
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                        ]
        }]
    },
    options: {
        indexAxis: 'y',
        plugins: {
            legend: {
                display: false
            },
            title: {
                display: true,
                text: 'CPU Multi'
            }
        }
    }
}
{{< /chart >}}

## 10 – Синтетика Geekbench

Использовалась версия 6.4.0

| Тест     | MacBook M1 Pro | MacBook M4 Pro | Разница (%) |
| ----------- | -------------- | -------------- | --|
| Single-Core Score | 2 306       | 3 891            | +68% |
| Multi-Core Score  | 9 371       | 22 495           | +140%  |

- {{< newtabref  href="https://browser.geekbench.com/v6/cpu/12811848" title="Результаты тестов M1 Pro на geekbench.com" >}}
- {{< newtabref  href="https://browser.geekbench.com/v6/cpu/12819483" title="Результаты тестов M4 Pro на geekbench.com" >}}

{{< chart >}}
{
    type: 'bar',
    data: {
        labels: [
          'M1 Pro',
          'M4 Pro',
        ],
        datasets: [{
            label: 'Single',
            data: [2306, 3891],
            backgroundColor: [
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                        ]
        },
        {
            label: 'Multi',
            data: [9371, 22495],
            backgroundColor: [
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                        ]
        }]
    },
    options: {
        indexAxis: 'y',
        plugins: {
            legend: {
                display: false
            },
            title: {
                display: true,
                text: 'Geekbench'
            }
        }
    }
}
{{< /chart >}}


## Вывод

Если вы разработчик и у вас появляются мысли: «А пора ли обновляться?»

✅ Обновляйтесь, если:

- Вы работаете с действительно большими проектами, часто собираете их локально, запускаете и прогоняете тесты.
В этом случае прирост производительности будет ощутим: сборка может ускориться на 30–50%.
- Вы хотите регулярно использовать локальные LLM.
- Вы не только пишете код, но и работаете с видео или фото — тут прирост особенно заметен.

🛑 M1 и выше всё ещё остаётся отличным выбором, если:

- Вы в основном работаете в IDE, а сборки и тесты чаще запускаются в CI/CD. Например, если локальный запуск — это боль или требует особой подготовки.
- Вы не работаете с медиаконтентом, AI и редко собираете контейнеры.
- У вас нет долгих и ресурсоёмких задач. На коротких дистанциях разницу между поколениями процессоров ощутить непросто.

💡На мой взгляд, вопрос об обновлении MacBook у разработчика чаще стоит не как «Мне не хватает мощности»,
а как «Мне не хватает оперативной памяти».
Вот мои 16 ГБ из 2022 года уже не хватало в 2025-м, так что надеюсь, 48 ГБ хватит надолго.
