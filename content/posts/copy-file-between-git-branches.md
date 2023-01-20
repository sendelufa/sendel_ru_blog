---
title: "Как скопировать файл из другой Git ветки"
date: 2023-01-20T17:28:25+05:00
description: 
author: "Konstantin Shibkov"
tags: ["git"]
categories: ["git"]
draft: false
---

Иногда требуется скопировать файл из одной ветки или из другого коммита. Зачем?
Ну например обновили конфиг и надо у себя в ветке тоже его использовать без
слияний и прочих лишних действий.

## Кратко

```bash
git checkout <from-branch-name> <path-to-file-or-dir>
```

## Как это работает

Если хотите повторить все действия, сделайте простой репозиторий
с двумя ветками и в ветке `second`  будет файл в директории. Порядок команд для этого:

```bash
git init -b master&& \
echo "this is txt file" > main.txt && \
git add main.txt && \
git commit -m "Initial commit" && \
git checkout -b second && \
mkdir myDir && \
echo "must be copied" > myDir/second.txt && \
git add myDir/second.txt && \
git commit -m "added second.txt"
```

Вы можете скопировать и выполнить все команды за один раз.

Допустим, нам требуется скопировать файл `myDir/second.txt` из ветки `second`
в `master`. Для копирования файла, перейдите в ветку, в которую хотите скопировать файл или директорию из другой ветки.

```bash
git checkout master
```

После можно выполнить команду копирования:

```bash
git checkout second ./myDir/second.txt
```

После этого из ветки `second` будет скопирован файл, при этом сохранится
путь до файла. То есть в `master` будет лежать файл в папке `mydDir`.

{{< callout type="warning" >}}
Если файл уже есть в текущей ветке - он будет перезаписан!
{{< /callout >}}

Давайте посмотрим и убедимся - файл у нас скопировался:

```bash
git status

On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	new file:   myDir/second.txt
```

При этом, файл уже подготовлен к коммиту.

{{< callout type="info" >}}
Если вам нужно из конкретного коммита скопировать файл, просто замените
название ветки на хэш коммита.
{{< /callout >}}

## А что еще есть?

А если мы хотим скопировать файл по другому пути? Так тоже можно, для
этого можно использовать другую команду `git show`.

Чтобы повторить действия - удалите директорию и снова создайте репозиторий
с двумя ветками, использую код выше.

И давайте перейдем в `master` и создадим папку `config` и в нее позже
скопируем файл `second.txt`.

```bash
git checkout master
mkdir config
```

И самое время выполнить команду вида:

```bash
git show <branch-name-or-hash>:<path-to-copy> > <path-to-paste>
```

указываем ветку или хэш коммита, далее источник файла для копирования
и путь до нового файла в текущей ветке.

{{< callout type="info" >}}
Это немного хак, так как команда show "показывает" файл из
другой ветки. А мы этот вывод пишем в новый файл у себя, для этого и указываем
вывод через >
{{< /callout >}}

В нашем случае будет так:

```bash
git show second:myDir/second.txt > config/config.txt
```

Убедимся что файл скопирован:

```bash
git status -uall

On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
	config/config.txt

nothing added to commit but untracked files present (use "git add" to track)
```

`-uall` нам покажет файлы в новых директориях, без этого параметра не увидим
содержимое `config`

Файл на месте. Обратите внимание, в этом случае он просто скопирован в
рабочую директорию и к коммиту не подготовлен и не отслеживается гитом.
