+++
title = "Установка WSL2 в Windows 10/11"
date = 2022-01-05T00:33:53+05:00
draft = true
author = "Konstantin Shibkov"
tags = ["windows", "wsl", "linux"]
categories = ["howto", "windows"]
description = "Краткий список действия для установки WSL2 на Windows 10/11"
+++

Зачем нужен WSL2? Самый часты ответ - простое использованое Linux программ в среде Windows. Docker самая распространенная причина для установки WSL2.

## Что надо для WSL2

- убедиться что в BIOS включена виртуализация, ищите пункт меню: 
  - процессор **Intel**: VT-x, Intel Virtualization Technology, VTx
  - процессор **AMD**: SVM Mode, AMD-V, Virtualization Technology
- версия Windows 11 или 10 не ниже версии 2004 (сборка 19041)

