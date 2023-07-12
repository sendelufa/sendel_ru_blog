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

## 🚨 Возможные ошибки

Если возникает ошибка:

```txt
FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':xxxxx:shadowJar'.
> org.apache.tools.zip.Zip64RequiredException: archive contains more than 65535 entries.

  To build this archive, please enable the zip64 extension.
  See: http://gradle.org/docs/2.1/dsl/org.gradle.api.tasks.bundling.Zip.html#org.gradle.api.tasks.bundling.Zip:zip64
```

Причина этого - использование zip32 и у него ограничение на 65535 записей внутри архива.
Необходимо в `build.gradle` добавить параметр для shadowJar включающий zip64:

```gradle
shadowJar {
  zip64 true
}
```
