# Блог sendel.ru

Пишу статьи в markdown, Hugo собирает в статический
сайт, а читают все вокруг на [sendel.ru](https://sendel.ru)

## Разработка

### Репозиторий

При клонировании репозитория желательно сразу получить
все подмодули:

```git clone --recurse-submodules -j8 git@github.com:sendelufa/sendel_ru_blog.git```

ИЛИ

после клонирования репозитория, необходимо обновить
все подмодули:

`git submodule update --init --recursive`

### Hugo

Блог использует движок Hugo для статической генерации сайта.

- [Установка Hugo](https://gohugo.io/getting-started/quick-start/)

Запуск сайта локально:

```bash
hugo server
```

Запуск локально с черновиками:

```bash
hugo server -D
```
