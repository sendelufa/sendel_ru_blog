+++
title = "Рекомендуемые книги"
showReadingTime = false
tableOfContents = false
toc = false
show_comments = false
Comments = "false"
+++

## <i class="fas fa-journal-whills"></i> Художественная литература

- Братья Стругацкие

  - «Далёкая Радуга» 1963
  - «Понедельник начинается в субботу» 1965
  - «Отель „У Погибшего Альпиниста“» 1970
  - «За миллиард лет до конца света» 1977

- Лю Цысинь

  - {{< newtabref  href="https://ru.wikipedia.org/wiki/%D0%97%D0%B0%D0%B4%D0%B0%D1%87%D0%B0_%D1%82%D1%80%D1%91%D1%85_%D1%82%D0%B5%D0%BB_(%D1%80%D0%BE%D0%BC%D0%B0%D0%BD)" title="«Задача трёх тел»" >}} /
    {{< newtabref  href="https://en.wikipedia.org/wiki/The_Three-Body_Problem_(novel)" title="«The Three-Body Problem»" >}}
    2007
  - {{< newtabref  href="https://ru.wikipedia.org/wiki/%D0%A2%D1%91%D0%BC%D0%BD%D1%8B%D0%B9_%D0%BB%D0%B5%D1%81_(%D1%80%D0%BE%D0%BC%D0%B0%D0%BD)" title="«Тёмный лес»" >}} /
    {{< newtabref  href="https://en.wikipedia.org/wiki/The_Dark_Forest" title="«The Dark Forest»" >}} 2008
  - {{< newtabref  href="https://ru.wikipedia.org/wiki/%D0%92%D0%B5%D1%87%D0%BD%D0%B0%D1%8F_%D0%B6%D0%B8%D0%B7%D0%BD%D1%8C_%D0%A1%D0%BC%D0%B5%D1%80%D1%82%D0%B8" title="«Вечная жизнь Смерти»" >}} /
    {{< newtabref  href="https://en.wikipedia.org/wiki/Death%27s_End" title="«Death's End»" >}} 2010

- Питер Уоттс «Ложная слепота» 2006

## <i class="fas fa-atlas"></i> Техническая литература

- Чарльз Петцольд «Код: тайный язык информатики»
- Егор Бугаенко «Элегантные Java объекты»
- Владстон Феррейра Фило «Теоретический минимум по Computer Science»
- Роберт Мартин «Чистая архитектура»
- Роберт Лафоре «Структуры данных и алгоритмы в Java»
- Филипе Гутьеррес «Spring Boot 2: лучшие практики для профессионалов»

## <i class="fas fa-book"></i> Настройка окружения и себя

- Максим Ильяхов «Новые правила деловой переписки»
- Максим Дорофеев «Путь джедая»

{{< code language="lisp" title="Really cool snippet" id="1" expand="Show" collapse="Hide" isCollapsed="false" >}}
pre {
; list functions

(cons E L) ; returns new list with E inserted at front of L
(car L) ; returns front element of L
(cdr L) ; returns a list of everything except front element of L
(length L) ; returns length of L
(listp L) ; true iff L is a list
(nth N L) ; returns nth element of L
(copy-list L) ; returns a new list whose contents are duplicates of those in L
(last L) ; returns last element of L
(butlast L) ; returns list of everything except last element of L
(append L1 L2) ; returns list of all elements of L1 followed by all elements of L2
(member E L) ; true iff E is an element of L
(remove E L) ; returns copy of L with all elements matching E removed
(reverse L) ; returns reverse of L
(null L) ; true iff L is empty
(position E L) ; returns first position of E in L, or nil if not found
}
{{< /code >}}
