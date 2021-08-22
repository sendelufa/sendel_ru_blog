+++
title = "Настройка миграции Liquibase на Spring Boot"
date = 2021-08-23T00:33:53+05:00
draft = true
author = "Konstantin Shibkov"
tags = ["liquibase", "spring boot", "spring", "java"]
categories = ["guide", "howto", "spring boot", "java"]
description = "Настройка и добавление миграций версий  базы данных, используя Liquibase и Spring Boot. Добавляем зависимости, пишем код миграций. "
+++

## Кратко о миграциях версий баз данных

Самое простое объяснение - это аналог git, только версионированию подвергнута структура базы данных, в некоторых случаях ее базовое наполнение.

Использование миграций делает возможным:

- всей команде одного проекта иметь одинаковую структуру БД, обновления которой поставляются вместе с новой версией приложения и автоматически обновляют структуру БД.

- создавать первоначальную структуру БД

- заполнить БД минимальным обязательным набором данных.

- отказаться от ручной работы с базой данных.

Миграции не зря похожи на git, они слой за слоем последовательно "накатываются "в базу данных в известном порядке. Если в базе уже есть часть выполненных миграций, то будут использоваться только новые.

Информация, какие миграции были применены к базе данных, по умолчанию находится в этой же БД, в специальной таблице.

## Начальные условия

В этой гайде будет вы можете использовать [готовый проект Spring Boot со сборщиком Gradle](/dddd)

Или создать пустой проект Spring Boot с такими зависимостями и версиями:

```bash

plugins {
    id 'org.springframework.boot' version '2.5.4'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.liquibase:liquibase-core'
    runtimeOnly 'com.h2database:h2'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

В качестве БД будем использовать h2 - база данных SQL в оперативке и через веб интерфейс наблюдать за изменениями.

