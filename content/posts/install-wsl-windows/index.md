+++
title = "Установка WSL2 в Windows 10/11"
date = 2022-01-05T00:33:53+05:00
draft = false
author = "Konstantin Shibkov"
tags = ["windows", "wsl", "linux"]
categories = ["howto", "windows"]
description = "Краткий список действия для установки WSL2 на Windows 10 и 11"
+++

Зачем нужен **Windows Subsystem for Linux 2** (WSL2)? Самый частый ответ - простое использованое Linux программ в среде Windows. Docker самая распространенная причина для установки WSL2.

> <i class="fas fa-info-circle"></i> Данный гайд подходит как шпаргалка по командам. <a href="https://docs.microsoft.com/ru-ru/windows/wsl/install" target="_blank">Полный официальный гайд на странице Microsoft.</a>

## Что надо для WSL2

- убедиться что в BIOS включена виртуализация, ищите пункт меню: 
  - процессор **Intel**: VT-x, Intel Virtualization Technology, VTx
  - процессор **AMD**: SVM Mode, AMD-V, Virtualization Technology
- версия Windows 11 или 10 не ниже версии 2004 (сборка 19041 *~декабрь 2019*)
- права администратора

## Всего несколько команд

Установим само ядро wsl и запустим дистрибутив Ubuntu для работы в консоли.

Открывайте PowerShell с правами Администратора. Самый простой вариант по мне -> WIN+X и выбрать **Терминал Windows (Администратор)**.

Выполняйте команду:

```shell
wsl --install
```

По-умолчанию, установится WSL2 с GUI и Ubuntu . Установка займет определенное время, подождите.

### <i class="fas fa-bomb"></i> Если ошибка 0x80072eff

 В процессе, можете увидеть ошибку `Во время установки произошла ошибка, но установка может быть продолжена. Компонент: 'Ядро WSL' код ошибки:  0x80072eff`.

{{< figure src="error_0x80072eff.png" alt="0x80072eff">}}

А значит, все автоматически не сработало, не беда. Перезагружайтесь и снова запустите PowerShell от Администратора.

- Запустите обновление wsl

```shell
wsl --update
```

- Завершите работу wsl

```shell
wsl --shutdown
```

- Запустите заново wsl

```shell
wsl
```

- Запустите установку Ubuntu (если надо более специфический дистр, посмотрите все варианты `wsl -l -o`)

```shell
wsl --install -d Ubuntu
```

{{< figure src="ubuntu_success.png" alt="ubuntu success install">}}

### Успешная установка Ubuntu

После установки, введите имя пользователя и пароль для Ubuntu, и вам будет доступна среда Linux.

{{< figure src="login_pass.png" alt="ubuntu login pass">}}

Рекомендуется сделать одну команду, обновить список пакетов:

```bash
sudo apt-get update
```

и наслаждаться :)

### Как зайти в WSL консоль

- запускайте команду `wsl`. (например, в настройках среды разработки указать wsl это для терминала по-умолчанию)
- можете вынести из Пуска, ярлык Ubuntu на панель задач.
- или поставить обновленный <a href="https://github.com/microsoft/terminal" target="_blank">Microsoft Terminal</a>, и в нем настроить при запуске сразу открывать консоль wsl, рекомендую.

{{< figure src="start_wsl.png" alt="start wsl">}}

{{< figure src="ubuntu_info.png" alt="ubuntu info">}}
