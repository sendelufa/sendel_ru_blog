---
title: "Переходим с HTTPS на SSH доступ в GitHub"
date: 2021-08-16T00:12:13+05:00
tags: ["git", "github", "ssh"]
categories: ["guide", "howto"]
description: "GitHub отказался от работы по HTTPS, как использовать SSH? в этой статье необходимые действия для перехода от HTTPS к SSH для доступа к репозиториям на Github. А также, как перевести существующие локальныые репозитории на SSH."
---

C 13 августа 2021 года Github блокирует доступ по HTTPS протоколу к репозиториям, то есть теперь вы не сможете, получить или отправить изменений в удаленный репозиторий без SSH ключа.

В этом гайде покажу как создать ключ, загрузить на сервер и перенастроить локальный репозиторий на SSH доступ. В качестве примера используется Windows 10 и GitBash.

## Что необходимо

- Gitbash или установленный git для Linux и macOS.
- Репозиторий на GitHub.

## Создание пары ключей SSH

SSH (Secure SHell) - это протокол, который позволяет безопасно авторизоваться в различные сервисы, подключаться к удаленным терминалам, передавать по шифрованным каналам информацию. Очень распрастранен при работе с репозиториями. Использует пару ключей - публичный и приватный.

Открывайте GitBash или терминал, вводите:

```bash
cd ~/.ssh
```

💡 Если у вас будет ошибка: `No such file or directory` - создайте папку, выполнив команду:

```bash
mkdir ~/.ssh
```

После заходите в папку.

Для генерации ключа используется программа `ssh-keygen`, она обычно установлена, в Windows встроена в GitBash.

Github рекомендует использовать ключ типа **ed25519**, так как этот алгоритм на данный момент самый безопасный, с коротким открытым ключом (68 символов, против 544 у RSA) и что важно - быстро работает. За тип ключа отвечает параметр `-t`.

Длина ключа рекомендуется 4096 бит, при создании это параметр `-b`.

💡 При генерации ключей , ключи по-умолчанию будут сохранены в папке `.ssh` текущего пользователя.

В итоге для запуска генерации ключа, выполните:

```bash
ssh-keygen -t ed25519 -b 4096
```

Вам будут заданы несколько вопросов:

- Куда сохранить файл (`Enter file in which to save...`) - нажмите **Enter** и по умолчанию ключ будет назван `id_ed25519` и сохранится в .ssh папке профиля текущего пользователя. (в Windows папки пользователя в C:/Users, в macOs/Linux папка пользователя в /home)

- Введите кодовую фразу (`Enter passphrase...`) - опционально, кодовая фраза это элемент безопасности. Если ваш приватный ключ попадет в чужие руки, им не смогут воспользоваться пока не подберут кодовую фразу. Это даст вам больше времени для замены ключей и отказа от скомпрометированного ключа. Предлагаю в данный момент отказаться от ключевой фразы и просто нажать **Enter**.
- Подтвердить кодовую фразу или ее отсутсвие, тоже нажав **Enter**.

В результате вам покажут рисунок вашего ключа:

```bash
+--[ED25519 256]--+
|        o + .    |
|       . * *     |
|      o   * .    |
|     o   . .     |
|.   .   S      . |
|o. .    . . + . .|
|o..o   o =.+ = o |
|. = . o o E+=.B o|
|   o .   ++o.*=X+|
+----[SHA256]-----+
```

Проверьте, на месте ли ключи, выведите список файлов в папке `.ssh`:

```bash
ls ~/.ssh
```

Вывод должен быть таким:

```bash
id_ed25519 id_ed25519.pub
```

первый файл это приватный ключ, а второй с .pub это публичный.

## Активация ключа

Для того чтобы ключ использовался системой, необходимо добавить ключ в ssh-agent.

Запустите ssh-agent:

```bash
eval "$(ssh-agent -s)"
```

результат, номер процесса может отличаться:

```bash
Agent pid 595
```

 Добавьте ранее созданный ключ:

 ```bash
 ssh-add ~/.ssh/id_ed25519
 ```

 При успехе получите ответ:

 ```bash
 Identity added: /c/Users/JohnDoe/.ssh/id_ed25519
 ```

 Ключи SSH готовы к использованию!

## Добавление публичного ключа в профиль на GitHub

ℹ Кратко про ключи. С помощью публичного ключа, каждый может зашифровать информацию, расшифровать может только владелец приватного ключа, поэтому приватный ключ нельзя передавать, отправлять или хранить в открытом виде. Желательно пару ключей дополнительно скопировать на отдельный носитель, на случай потери ключа на компьютере.

В GitHub для работы с репозиториями скопируйте публичный ключ, одним из способов:

- в GitBash выполните команду:

 ```bash
 clip < ~/.ssh/id_ed25519.pub
 ```

 это скопирует публичный ключ в буфер обмена.

- если данная команда не работает, откройте файл `id_ed25519.pub` в любом текстовом редакторе, и скопируйте все содержимое файла, публичный ключ выглядит так:

```bash
ssh-ed25519 AAAAC3NzaC1lZDI1NME5AAAAIMNTCjcl4AXofA/lBMPT7iLAdoPyWJab9N9enEj2iXOb JohnDoe@DESKTOP-IKJ2EOH
```

- Переходите на страницу управления ключами:

<https://github.com/settings/keys>

- Нажимайте не кнопку `New SSH Key`

- В поле `Key` вставьте скопированный ключ:

![Поле добавления ключа SSH](2021-08-16-01-26-49.png)

- В поле `Title` можете вставить название ключа, пригодится если у вас будет в профиле более одного ключа. Поможет их различать.

- Нажимайте кнопку `Add SSH Key`.

Теперь можно использовать SSH доступ к вашим репозиторияем!

## Получение репозитория по SSH

Откройте репозиторий и скопируйте ссылку для SSH доступа:

![Выбор ссылки для доступа к репозиторию](2021-08-17-00-54-23.png)

И как обычно используйте команду git clone:

```bash
git clone git@github.com:sendelufa/sendel_ru_blogging.git
```

## Как сменить работу с HTTPS на SSH 

Если у вас есть локальный (на вашем рабочем компьютере) репозиторий полученный по https, очень просто сменить доступ на SSH.

Для этого убедитесь что доступ по HTTPS, для этого выведите список remote:

```bash
git remote -v

origin  https://github.com/sendelufa/tutorial-ssh.git (fetch)
origin  https://github.com/sendelufa/tutorial-ssh.git (push)
```

ссылки начинаются c https, а значит вы не можете ни загрузить ни получить обновления удаленного репозитория.

Зайдите в репозиторий и скопируйте SSH ссылку доступа, перейдите в локальный репозиторий и удалите текущий remote origin:

```bash
git remote rm origin
```

и добавьте новый, последняя строка в команде это ссылка доступа SSH:

```bash
git remote add origin git@github.com:sendelufa/tutorial-ssh.git
```

проверьте список удаленных репозиториев:

```bash
git remote -v

origin  git@github.com:sendelufa/tutorial-ssh.git (fetch)
origin  git@github.com:sendelufa/tutorial-ssh.git (push)
```

и если у вас формат без https в начале ссылки, то все выполнено верно, можно работать с репозиторием и проверить командой `git fetch`

На этом вопросы с доступом к вашим репозиториям на гитхабе закрыт!

Чистого кода и красивых коммитов!
