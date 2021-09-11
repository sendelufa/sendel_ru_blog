+++
title = "Настройка и добавление миграции Liquibase на Spring Boot"
date = 2021-09-03T00:33:53+05:00
draft = false
author = "Konstantin Shibkov"
tags = ["liquibase", "spring boot", "spring", "java"]
categories = ["guide", "howto", "spring boot", "java"]
description = "Настройка и добавление миграций версий  базы данных, используя Liquibase и Spring Boot. Добавляем зависимости, пишем код миграций. "
+++

## Кратко о миграциях версий баз данных

Самое простое объяснение - это аналог git, только версионированию
подвергнута структура базы данных, в некоторых случаях ее базовое наполнение.

Использование миграций делает возможным:

- всей команде одного проекта иметь одинаковую структуру БД,
обновления которой поставляются вместе с новой версией приложения
и автоматически обновляют структуру БД.

- создавать первоначальную структуру БД

- заполнить БД минимальным обязательным набором данных.

- отказаться от ручной работы с базой данных.

Миграции не зря похожи на git, они слой за слоем последовательно "накатываются"
в базу данных в известном порядке. Если в базе уже есть часть выполненных миграций,
то будут использоваться только новые.

Информация, какие миграции были применены к базе данных,
по умолчанию находится в этой же БД, в специальной таблице.

## Начальные условия

В этом гайде вы можете использовать
<a href="https://github.com/sendelufa/lesson-liquibase-start" target="_blank">готовый  проект Spring Boot со сборщиком Gradle</a>
(ветка master) или создать с нуля по рекомендациям ниже.

### Создание проекта с нуля

Или создать пустой проект Spring Boot с зависимостями и версиями:

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

В качестве БД будем использовать h2 - база данных SQL будет в отдельном файле и через веб интерфейс будем наблюдать за изменениями.

Для старта отключите liquibase, установив параметр в `application.yml`:

```yml
spring:
  liquibase:
    enabled: false
```

{{< code language="properties" title="Для application.properties" id="2" isCollapsed="true" >}}spring.liquibase.enabled = false
{{< /code >}}

Если это не сделать, то наше приложение не запустится, так как не созданы конфигурации миграций.

{{< code language="log" title="Ошибка запуска" isCollapsed="true">}}
***************************
APPLICATION FAILED TO START
***************************

Description:

Liquibase failed to start because no changelog could be found at 'classpath:/db/changelog/db.changelog-master.yaml'.

Action:

Make sure a Liquibase changelog is present at the configured path.
{{< /code >}}

### Настройка и подключение h2

Для включения и доступа к h2 через веб интерфейс добавьте настройки в `application.yml`:

```yml
spring:
  liquibase:
    enabled: false
  datasource:
    url: jdbc:h2:file:./db # в корне проект файл бд db.mv.db
    username: u # имя пользователя консоли
    password: 1 # пароль консоли 
  jpa:
    database-platform: org.hibernate.dialect.H2Dialect
  h2:
    console:
      enabled: true
```

Для `application.properties`:

{{< code language="properties" title="Для application.properties" id="3" isCollapsed="true" >}}
spring.liquibase.enabled=false
spring.datasource.url=jdbc:h2:file:./db
spring.datasource.username=u
spring.datasource.password=1
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.h2.console.enabled=true
{{< /code >}}

После запуска приложения перейдите по ссылке <a href="http://localhost:8080/h2-console" target="_blank">http://localhost:8080/h2-console</a>
и попадете в консоль управления базой данных.
Введите данные URL, логин и пароль из настройки и после нажатия
Connect попадете в упраление базой данных.

![h2 console](2021-08-24-23-31-57.png)

### Создание Entity классов

Создайте два класса со связями, пусть это будет статьи и их авторы.
У автора может быть множество статей, а у статьи только один автор.

![tables Authors and Articles in databases](2021-08-24-23-37-11.png)

Код каждого класса (геттеры и сеттеры не показаны):

{{< code language="java" title="class Article" id="4" isCollapsed="true" >}}
@Entity
public class Article {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  long id;

  @Column(nullable = false, length = 100)
  String title;

  @Column(nullable = false)
  String text;

  @ManyToOne
  Author author;

  // getters & setters

}
{{< /code >}}

{{< code language="java" title="class Author" id="5" isCollapsed="true" >}}
@Entity
public class Author {

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  Long id;

  @Column(nullable = false)
  String name;

  // getters & setters
  
}
{{< /code >}}

## Работа Hibernate

{{< callout type="info" >}}
Пропустите этот раздел, если вы знаете как Hibernate обновляет и изменяет структуру БД.
{{< /callout >}}

Как можно управлять структурой базы данных через Hibernate?
Есть параметр, который указывает что делает Hibernate при запуске приложения:

yml config:

```yml
spring:
  jpa:
    hibernate:
      ddl-auto: none;
```

properties config:

```properties
spring.jpa.hibernate.ddl-auto = none
```

По-умолчанию стоит значение `none`, то есть не делать ничего.
Кроме этого есть такие параметры как:

- `create` - создавать структуру базы данных на основе java классов @Entity.
Уничтожает структуру базу данных со всеми данными перед созданием.
- `update` - обновить при возможности структуру базы данные на основе java
классов @Entity. При возможности сохраняются данные.
- `validate` - проверить, соответсвует ли существующая структура базы данных
написанным классам @Entity. Если не соответсвует, приложение не запустится.
- `none` - не делать ничего. Даже если структуры базы данных нет, то приложения
запустится и завершится с ошибкой при первом доступе к бд.

Остальные параметры вы можете посмотреть
<a href="https://vladmihalcea.com/hibernate-hbm2ddl-auto-schema/" target="_blank">в статье Vlad Mihalcea</a>

С одной стороны, это открывает возможности всегда поддерживать
структуру бд в соответсвии с написанным кодом. И да,
это удобно на самом старте проекта, когда и бд небольшая и работаете вы одни над проектом.
Данных никаких нет, можно каждый раз обнулять БД.

При этом нам придется мириться с тем:

- как Hibernate будет создавать структуру
- какие ключи прописывать
- как строить связи
- какие значения по умолчанию использовать

Для наглядности, включите параметр `create`, запустите программу,
после успешного старта завершите работу программы и зайдите в бд.

![data scheme h2](2021-08-25-00-34-46.png)

Все на месте, как и написали. Есть связи, пусть и Hibernate
назвал их на свой лад, есть все поля, первичные ключи.
Продолжаем работать, переключаем параметр в `update` чтобы не
удалить данные при следующем запуске и поддерживать структуру
в актуальном состоянии.

Теперь поменяйте класс `Author`, нам понадобилось не
одно имя хранить, а полное имя автора:

```java
@Entity
public class Author {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    Long id;

    @Column(nullable = false)
    String fullname;

    // getters & setters

}
```

{{< callout type="info" >}}Тоже самое будет, если использовать
@Column(name = "fullname")
{{< /callout >}}

запускайте приложение снова и заходите в консоль. Мы ожидаем, что поле name будет переименовано в fullname.
Но Hibernate просто создал еще одну колонку, старая осталась.

![name column still exist](2021-08-25-00-46-12.png)

А если у нас были бы данные в name, то нам бы хотелось чтобы и данные остались, а колонка была бы переименована.
И снова, на старте разработки, меняем на `create` и получаем чистую структуру. Никаких проблем!

А если у нас есть база, с данными, и нам надо ее изменить, при этом сохранить данные, а также чтобы все действия
были повторены не только локально, но и у каждого участника команды проекта? Тут и приходит на помощь миграция версий
баз данных. Даже на таком простом примере, уже виден профит.

{{< callout type="info" >}}
Перед следующей главой, вернитесь к исходной версии кода, где у автора было поле `name`.
{{< /callout >}}

## Пишем первую миграцию для Liquibase

В структуре миграций два основых типа файлов:

- Файл `changelog` (мастер файл) - содержит список миграций (changeset),
которые применяются к базе данных. Миграции выполняются последовательно,
в том порядке в котором записаны. Возможные форматы файла: SQL, XML, JSON, YAML.
Имя по умолчанию: `db.changelog-master`. 
<a href="https://vladmihalcea.com/hibernate-hbm2ddl-auto-schema/" target="_blank">в статье Vlad Mihalcea</a>

- Файлы `changeset` (файлы миграций) - каждый из файлов содержит набор изменений,
которвые применяются к базе данных. На эти файлы ссылается `chagelog`.
Если при выполнении миграций, файл был уже применен к базе данных - прописанные
изменения в нем игнорируются.
Возможные форматы файла: SQL, XML, JSON, YAML. 
<a href="https://docs.liquibase.com/concepts/basic/changeset.html" target="_blank">Документация</a>

Liquibase при старте работы ищет файл с названием`db.changelog-master` в 
директории `src/main/resources/db/changelog`.

{{< callout type="info" >}}
Если вы используете IDEA, то убедитесь что вы создали папки db и в ней папку changelog,
а не одну папку с названием db.changelog
{{< /callout >}}

Создайте файл `db.changelog-master.yaml`. 

Полный путь до файла будет 
`src/main/resources/db/changelog/db.changelog-master.yaml`

Добавьте в него запись о первом файле миграции:

```yaml
databaseChangeLog: #параметр в котором находятся миграции
  - include: # список активных миграций
      file: db/changelog/changeset/create-author-table.yaml # путь к файлу миграции
```

Файлы с содержимым миграций `changeset` могут располагаться как в той же папке что и `changelog`,
так и в отдельной директории. Рекомендуется отдельная директория, создайте
для них директорию `changeset` в `src/main/resources/db/changelog`.

Создайте указанный файл миграции `create-author-table.yaml` в директории `src/main/resources/db/changelog/changeset`, полный путь до файла будет 
`src/main/resources/db/changelog/changeset/create-author-table.yaml`

```yaml
databaseChangeLog:
  - changeSet:
      id: create-author #текстовый идентификатор (Обязателен)
      author: Konstantin Shibkov # автор (Обязателен)
      changes:
        - createTable: # создаем новую таблицу
            tableName: author
            columns: # объявления колонок
              - column:
                  name: id
                  type: bigint
                  autoIncrement: true
                  constraints:
                    primaryKey: true
                    nullable: false
              - column:
                  name: name
                  type: varchar(200)
                  constraints:
                    nullable: false
```

[скачать файл create-author-table.yaml](create-author-table.yaml)

Измените параметр `liquibase.enabled` в настройках проекта application на true:

```yaml
  liquibase:
    enabled: true
```

Если у вас осталась база данных db.mv.db - удалите ее.
При использовании H2 базы данных, пустой файл базы данных будет создан вновь.

Запустите проект, посмотрите в логи, при успехе у вас должна быть такая строка:

```log
2021-08-29 00:26:38.133  INFO 853778 --- [main]
liquibase.changelog
: ChangeSet db/changelog/changeset/create-author-table.yaml
::create-author::Konstantin Shibkov ran successfully in 5ms
```

а значит миграция накатилась успешно. В противном случае,
проверьте названия файлов, синтаксис файлов конфигураций.

Зайдите в консоль h2 <http://localhost:8080/h2-console> и откройте список таблиц.

В базе будет три таблицы:

- `author` - таблица для хранения авторов
- `databasechangelog` - хранится информация о примененых миграциях
- `databasechangeloglock` - таблица блокировка, которая гарантирует
работу одного экземпляра Liquibase при применение миграций.

![таблицы бд после применений миграции](2021-08-29-00-37-07.png)

Посмотрите структуру таблицы `author` - она соотвествует прописанному в файле.

## Вторая миграция и валидация Hibernate

Предлагаю вам теперь поменять параметр Hibernate ddl-auto на validate:

```yaml
    hibernate:
      ddl-auto: validate
```

и снова запустите проект. Проект не запуститься и посмотрим на логи, показаны избранные строки:

```log
2021-08-29 01:26:13.441 ERROR 915296 --- [main]
o.s.boot.SpringApplication: Application run failed

...

nested exception is 
org.hibernate.tool.schema.spi.SchemaManagementException:
Schema-validation: missing table [article]
```

Причина отказа в запуске явно прописана: в базе данных не найдена таблица `article`.

Добавьте новый файл миграции `create-article-table.yaml`, с описанием новой таблицы `article`:

```yaml
databaseChangeLog:
  - changeSet:
      id: create-article
      author: Konstantin Shibkov
      changes:
        - createTable:
            tableName: article
            columns:
              - column:
                  name: id
                  type: bigint
                  autoIncrement: true
                  constraints:
                    primaryKey: true
                    nullable: false
              - column:
                  name: title
                  type: varchar(100)
                  constraints:
                    nullable: false
              - column:
                  name: text
                  type: varchar(255)
                  constraints:
                    nullable: false
              - column:
                  name: author_id
                  type: bigint
                  constraints:
                    foreignKeyName: author_article_fk
                    referencedTableName: author
                    referencedColumnNames: id
```

и впишите новую строку с этой миграцией в файл `changeset`:

```yaml
  - include:
      file: db/changelog/changeset/create-article-table.yaml
```

Запускайте проект, если не запустился и ругается на миграции. 
Удалите из корня проекта базу данных.

После запуска, переходите [в консоль h2](http://localhost:8080/h2-console) и там
вас ожидают две таблицы, со внешним ключом. А если посмотреть в таблицу DATABASECHANGELOG

```sql
SELECT * FROM DATABASECHANGELOG 
```

в ней две записи, о каждой миграции которые были применены к базе данных.

Базовая работа с миграциями завершена, если вам надо создать новую таблицу
или поменять текущие - пишите миграции, новые миграции версий бд буду 
исполняться при запуске приложения.

## Внесение изменений в таблицу с помощью миграций

А теперь повторите ситуацию, когда надо изменить имя столбца,
для этого вам потребуется написать новую миграцию, поправить Entity
класс и снова запустить проект:

- новая миграция

```yaml
databaseChangeLog:
  - changeSet:
      id: change-column-author
      author: Konstantin Shibkov
      changes:
      - renameColumn:
          tableName: author
          newColumnName: fullname
          oldColumnName: name
```

- в классе Author укажите новое название поля, можно указать это в анотации

```java
    @Column(name = "fullname", nullable = false)
    String name;
```

- обязательно добавьте миграцию в `db.changelog-master.yaml`:

```yaml
  - include:
      file: db/changelog/changeset/rename-author-name-to-fullname.yaml
```

Стартуйте и в консоли h2 увидите измененную таблицу:

![обновленная таблица author](2021-09-04-00-34-56.png)

Дублирования нет, произошло переименование таблицы, если там были данные,
они не пострадали.

## Готовый код

После всех предложенных действий код найдете в репозитории,
в ветке result <a href="https://github.com/sendelufa/lesson-liquibase-start/tree/result" target="_blank">готовый проект по рекомендациям из статьи</a>

## Типичные шаги разработчика при работе с миграцией

- пишите новый changeset

- запускайте миграцию и проверяете, что она работает

- вносите изменения в код Java приложения

- проверяете на соответcтвие кода и миграции

- в одном коммите фиксируете код и миграцию вместе и одновременно

> Вольный перевод блока статьи
<a href="https://www.liquibase.org/get-started/best-practices" target="_blank">Some best practices to keep in mind when using Liquibase - Typical procedure for the developer</a>

## Что в итоге?

- миграции баз данных позволяют контролировать состояние бд,
и распространять изменения среди команды

- каждый файл миграции исполняется в базе данных один раз.

- файлы миграций можно писать на SQL, XML, YAML, JSON

- контроль над структурой базы данных должен остатьтся только
у liquibase, ORM только могуть проверять соотвествие своего
кода и структуры бд

- миграции можно откатывать, писать rollback скрипты

**Да будут ваши структуры базы данных целостными!**
