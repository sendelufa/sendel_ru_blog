---
title: "Сборка JAR со всеми зависимостями Gradle"
date: 2021-12-13T23:50:43+05:00
tags: ["gradle", "java"]
categories: ["howto"]
description: "Сборка Gradle JAR в два приёма."
toc: false
---

Ситуация - есть проект с Gradle надо создать jar со всеми зависимостями и запускать командой `java -jar file`. Еще называют такие сборки **Fat Jar**.

{{< callout type="info" >}}
Используется Gradle 7.1
{{< /callout>}}

- Подключаем библиотеку Shadow

В файле `build.gradle` добавляйте в список плагинов <a href="https://github.com/johnrengelman/shadow" taget="_blank">Gradle Shadow</a>. В readme в гитхабе уточните нужную вам версию.

```groovy
 id("com.github.johnrengelman.shadow") version "7.1.0"
```

Получится так:

```groovy
plugins {
    id 'java'
    id("com.github.johnrengelman.shadow") version "7.1.0"
}
```

- Добавляем конфигурацию запуска

Минимально, в значении `Main-Class` необходимо указать, какой класс содержит main метод для запуска:

```groovy
jar {
    manifest {
        attributes(
                'Main-Class': 'ru.sendel.Main'
        )
    }
}
```

- Собираем jar

Осталось выполнить команду:

```bash
gradle clean shadowJar
```

и в директории `build/libs/` появится fat jar.

Profit! ✨
