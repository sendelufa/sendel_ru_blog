---
title: "Пишем Java код в VSCode"
date: 2024-03-17T10:57:26+05:00
description: Как перейти на VSCode для разработки на Java и Spring Boot
author: "Konstantin Shibkov"
tags: ["java", "VSCode", "spring boot"]
categories: ["soft", "java"]
draft: true
---

Давайте признаемся, что подавляющее большинство пишет Java код используя
JetBrains IDEA. Да, это отличная IDE. Для нее есть большое количество расширений,
среда очень тесно интегрируется и знает особенности Spring Framework.

А еще есть Community версия, так что можно даже не платить за основной функционал.

Тогда зачем пробовать что-то другое? 

В какой-то мере все java разработчики стали заложниками, так как IDEA так
обволакивает и нет желания пробовать другое, ведь все такое удобное, знакомое.

Но при каких-то изменениях, сложностей использования - испытываешь неприятные
ощущения когда тебя начинают ограничивать, да еще и за деньги.

Поэтому в этой статье делюсь опытом изучения вопроса "А можно ли перейти на VSCode?".

Какие боли, а может и радости при этом испытываешь.

## Точка старта

Начнем с того, какие базовые условия разработки имеются сейчас. И чем пользуюсь часто,
для чего есть интеграции в IDEA:

- Java 17/21 + Kotlin
- Spring Boot 3
- использование spring initializr (публичный и корпоративный)
- многомодульные Maven/Gradle проекты
- Postgres
- MyBatis для работы с БД
- Junit 5 + AssertJ
- Copilot
- Документация в MD + AsciiDoc + Plantuml
- Kubernetes

Попробуем найти аналоги или возможно для VSСode есть точно такие же расширения.

## Установка VSCode и расширений

Тут еще отмечу, решил попробовать испытывать в связке Windows 11 + WSL 2 c Ubuntu.
Код лежит в WSL пространстве,
<a href="https://code.visualstudio.com/" target="_blank">VSCode установлен в Windows</a>
и подключается к коду в WSL.

![Голый VScode](vscode_after_install.png)

> Если у вас Linux или macOS, то все что касается WSL просто надо игнорировать. У
> вас будет запускаться все локально. 

Особенность работы через WSL заключается к том, что VSCode у вас будет как бы
дублирован. Одна версия VSCode будет установлена в Windows, а другая в WSL.
Это позволяет работать в среде Linux не выходя из Windows и не используя GUI Linux.

По простому - в Windows у вас будет клиент, а все действия будут выполняться в WSL,
то есть в Linux. Наборы расширений ставятся и настраиваются отдельно
для WSL VSCode и для Windows VSCode. Вам необходимо ставить расширения после подключения
к WSL.

В Windows откройте VSCode и подключитесь к WSL, есть разные способы:

- нажать на значок в нижнем левом углу, с иконкой `><`
- вызвать палитру команд <kbd>CTRL</kbd> + <kbd>SHIFT</kbd> + <kbd>P</kbd>

и выбрать <mark>Connect to WSL using Distro...</mark>,
после выбрать дистрибутив WSL.

Скачается серверная часть VSCode, которая будет работать в WSL, а в Windows останется только тонкий клиент

### Extension Pack for Java

Ставим минимальный набор расширений для Java. Это позволит
проверять синтаксис, подсвечивать код,
использовать автокомплит, включит инструменты рефакторинга, режим дебага,
запускать JUnit тесты.

Устанавливать расширения можно перейдя по ссылке или через поиск в разделе EXTENSIONS.

<a href="https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack"
target="_blank">Extension Pack for Java</a>

![Базовый набор для работы с Java](java_base_ext.png)

### SonarLint

Анализ кода, куда же без него. Также можно и подключить
удаленный SonarQube.

<a href="https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarlint-vscode"
target="_blank">SonarLint</a>

![SonarLint](sonarlint_ext.png)

Возможно потребуется установка Node.js. Найдете подробности на странице https://github.com/nodesource/distributions

### 🧪 Промежуточная проверка

А теперь можно проверить как запускает Java код на этом этапе. Сделаем простой Hello, world!

У вас уже должен быть установлен JDK. Для Windows - JDK необходимо установить в WSL среде.
Самый простой способ это использовать утилиту <a href="https://sdkman.io/" target="_blank">SdkMan</a>.

Создайте файл `HelloWorld.java`

```bash
touch HelloWorld.java
```

и запишите в него код:

```java
class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```

> В Linux / macOS достаточно открыть файл `HelloWorld.java`,
дополнительных действий не требуется

В Windows откройте VSCode и подключитесь к WSL, есть разные способы:

- нажать на значок в нижнем левом углу, с иконкой `><`
- вызвать палитру команд <kbd>CTRL</kbd> + <kbd>SHIFT</kbd> + <kbd>P</kbd>

и выбрать <mark>Connect to WSL using Distro...</mark>,
после выбрать дистрибутив WSL.

Скачается серверная часть VSCode, которая будет работать в WSL,
а в Windows останется только тонкий клиент

Открывайте файл `HelloWorld.java` и запускайте через:

- Run ▸ Run Without Debugging...
- <kbd>CTRL</kbd> + <kbd>F5</kbd> / <kbd>⌃</kbd> + <kbd>F5</kbd>
- Нажать на символ Play, отмечен на скрине

![HelloWorld.java](run_java_file.png)

Если JDK установлен - то файл исполнится. Также можете проверить debug, установка
breakpoint также левее от номера строки и в подменю запускаете в режиме debug.

Пора двигаться дальше.

### Spring Boot Extension Pack

В него входит Spring Tools для работы с кодом, который использует Spring.
Spring Initializr для упрощения создания нового проекта. Spring Dashboard
для управления Spring Boot проектами: запуск, работа, список ендпоинтов, дебаг.

<a href="https://marketplace.visualstudio.com/items?itemName=vmware.vscode-boot-dev-pack"
target="_blank">Spring Boot Extension Pack</a>

![Spring Boot Extension Pack](spring_ext.png)

После установке будет предложено создать пример проекта, для тестирования
расширения. Будет склонирован проект PetClinic в выбранную директорию.

Предлагаю так и сделать: создать проект, запустить, убедиться
что все работает. После клонирования, необходимо будет подждать
пока проект проиндексируются и будут скачены все зависимости.

Чтобы создать новый Java проект, перейдите в EXPLORER вкладку и найдите
кнопку создать новый проект. А дальше в интерактивном меню вы можете выбрать
тип проекта и какими средствами его создать, включая Spring Initializr.

![Создание нового проекта](create_new_project.png)

#### Как добавить корпоративный Spring Initializr

Возможно настроить использование инициализации проекта с указанием
сервера сборки шаблонного проекта. 

Перейдите в настройки расширения Spring Initializr. Мне удобно это делать
через палитру действий, кликнув на шестеренку:

![Настройка Spring Initializr](initializr_config.png)

Находите пункт Service Url, заходите в редактирование `settings.json` и прописывайте
свое значение URL в параметр `"spring.initializr.serviceUrl"`. Сохраняйте файл и теперь создавайте проекты используя
свой сервис сборки проектов.

### Рекомендуемые расширения

- <a href="https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-gradle"
target="_blank">Gradle for Java</a>
- <a href="https://marketplace.visualstudio.com/items?itemName=GoogleCloudTools.cloudcode"
target="_blank">Google Cloud Code (для k8s)</a>
- <a href="https://marketplace.visualstudio.com/items?itemName=jebbs.plantuml"
target="_blank">PlantUML</a>
- <a href="https://marketplace.visualstudio.com/items?itemName=asciidoctor.asciidoctor-vscode"
target="_blank">AsciiDoc</a>
- <a href="https://marketplace.visualstudio.com/items?itemName=GitLab.gitlab-workflow"
target="_blank">GitLab Workflow - CR/CI/MR</a>
- <a href="https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools"
target="_blank">Kubernetes</a>
- <a href="https://marketplace.visualstudio.com/items?itemName=Arjun.swagger-viewer"
target="_blank">Swagger Viewer</a>

💡 Чтобы PlantUML отрисовывался в AsciiDoc, необходимо активировать kroki.

Открыть настройки пользователя
<kbd>CTRL</kbd> + <kbd>,</kbd> / <kbd>⌘</kbd> + <kbd>,</kbd>.
Открыть JSON файл для редактирования.

![Настройки](json_settings.png)

И добавить параметр:

```json
{
    "asciidoc.extensions.enableKroki": true
}
```


### Как повторить опыт использования IDEA

#### Полезные расширения

Перенесите хоткеи, к которыми привыкли:
- <a href="https://marketplace.visualstudio.com/items?itemName=k--kato.intellij-idea-keybindings"
target="_blank">IntelliJ IDEA Keybindings</a>

Если вам поможет тема из IDEA, тут она тоже есть:
- <a href="https://marketplace.visualstudio.com/items?itemName=trinm1709.dracula-theme-from-intellij"
target="_blank">Darcula IntelliJ Theme</a>

Вы смотрите на файлы и не понимаете что иконки означают - верните привычный вид:
- <a href="https://marketplace.visualstudio.com/items?itemName=lijialin.IDEA-Icons-To-Maven-For-VSCode"
target="_blank">IDEA Icons in VSCode</a>

#### Включить автосохранение файлов

- откройте настройки <kbd>CTRL</kbd> + <kbd>,</kbd> / <kbd>⌘</kbd> + <kbd>,</kbd>
- найдите пункт <mark>Auto Save</mark>
- установите значение в <mark>OnFocusChange</mark> или в другое, подходящее.

### Подключиться к приложению в режиме Debug

Мне например удобно всегда запускать приложение с возможностью подключиться в режиме Debug.
Чтобы сделать это в VSCode создайте конфигурацию запуска launch.json

![Создание нового конфига для запуска](create_launch_json.png)

Вставьте конфиг, при необходимости измените хост и порт.

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "java",
            "name": "Attach to Remote Java Application",
            "request": "attach",
            "hostName": "127.0.0.1",
            "port": 5005
        }
    ]
}
```

- hostName - адрес хоста, где развернуто приложение
- port - это порт для подлючения удаленного дебага

Запускайте дебаг сессию после запуска приложения.

### Активировать внешние правила оформления кода

Скачайте файл с правилами по оформлению кода

```bash
wget https://raw.githubusercontent.com/google/styleguide/gh-pages/eclipse-java-google-style.xml
```

Укажите в настройках параметр:

```json
{
    "java.format.settings.url": "/home/shibkov/dev/eclipse-java-google-style.xml"
}
```

### Использование Copilot

Установите расширение <a href="https://marketplace.visualstudio.com/items?itemName=GitHub.copilot"
target="_blank">GitHub Copilot</a>

Нажмите на иконку Copilot в статусной строке и выберите Sign In

![Войти через аккаунт GitHub](copilot_icon.png)

Для выхода из GitHub аккаунта:

![Выйти из GitHub](copilot-sign-out.png)
