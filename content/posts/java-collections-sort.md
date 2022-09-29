---
title: "Cортировка списков в Java"
date: 2022-09-29T09:05:12+03:00
description: 
author: "Konstantin Shibkov"
tags: ["java", "collection"]
categories: ["java"]
---

Для возможности сортировки объектов в коллекциях наследниках
List в Java существует статический метод класса `java.util.Collections`.

Это значит вы можете сортировать элементы таких классов как ArrayList,
LinkedList, CopyOnWriteArrayList и других классов, имплементирующих
интерфейс `List`.

В общем виде, если у вас есть список из строк:

```text
[z, b, c, a, k, z]
```

то после сортировки получите в списке порядок:

```text
[a, b, c, k, z, z]
```

## Простое использование метода sort()

Если у нас в списке находятся объекты классов, которые известно
как сравнить, то достаточно просто вызвать метод sort() и передать туда
список. Таким образом в списке элементы поменяют порядок и будут
отсортированы в **порядке возрастания**

```java
//создание списка на основе массива
var stringList = Arrays.asList("z", "b", "c", "a", "k", "z");
System.out.println(stringList);

//сортировка списка в порядке возрастания
Collections.sort(stringList);

System.out.println(stringList);
```

Вывод в консоль:

```text
[z, b, c, a, k, z]
[a, b, c, k, z, z]
```

Так мы можем сортировать множество стандартных классов,
таких как String, Integer, Double, Character и множество других.

Если более точно выразиться, то без дополнительных параметров
возможно отсортировать список из любых элементов, классы которых имплементируют
интерфейс сравнения `Comparable<T>`.

## Сортировка в обратном порядке

Если мы хотим сортировать элементы в обратном порядке. То для этого
передадим дополнительный аргумент в метод сортировки:

```java
//создание списка на основе массива
var stringList = Arrays.asList("z", "b", "c", "a", "k", "z");
System.out.println(stringList);

//сортировка списка в обратном направлении
Collections.sort(stringList, Collections.reverseOrder());

System.out.println(stringList);
```

Вывод:

```text
[z, b, c, a, k, z]
[z, z, k, c, b, a]
```

## Добавляем возможность сортировки своих классов

Если стандартные классы уже готовы к сортировке, то если мы напишем
свой класс, то Java не знает как есть сравнивать с объектами этого же
класса.

Чтобы научить сравнивать объекты есть два варианта:

* создать класс на основе `Comparator<T>` и там прописать правила сравнения
в методе `int compare(T o1, T o2)`.
Полученный объект из класса использовать всегда, когда нам надо сортировать объекты. Такой
вариант отлично подходит, когда нам надо сортировать объекты по разным правилам и
можем использовать нужный нам класс Comparator.
* добавить в класс (являющимся, элементом списка) имплементацию интерфейса
`Comparable<T>` и прописать правила сравнения в методе `int compareTo(T o)`. Тогда не потребуется указывать каждый раз компаратор, данное правило сравнение будет
по-умолчанию для этого объекта.

Оба метода возвращают целое число, которое обычно интерпретируется так:

* число больше 0 -> объект с которым сравнивают больше текущего
* число равно 0 -> объекты одинаковые
* число меньше 0 -> объект с которым сравнивают меньше текущего

К примерам!

Создадим свой класс, например для студента:

```java
class Student {
    private final String name;
    private final double avgMark;

    public Student(String name, double avgMark) {
        this.name = name;
        this.avgMark = avgMark;
    }

    @Override
    public String toString() {
        return "{n='" + name + '\'' + ", m=" + avgMark + '}';
    }
}
```

Класс специально минимально простой: все параметры задаются в конструкторе,
и используются значения только для печати данных при вызове toString, что
поможет нам в визуализации результата.

Для начала, посмотрим, что будет если мы попробуем отсортировать список из
студентов:

```java
var ivan = new Student("Иван", 4.3);
var olga = new Student("Ольга", 3.8);
var eugene = new Student("Женя", 4.9);
var studentList = Arrays.asList(ivan, olga, eugene);

System.out.println(studentList);
//сортировка списка
Collections.sort(studentList);
System.out.println(studentList);
```

Такой код не скомпилируется, так как метод sort() не просто ожидает список,
но еще важно, чтобы элемент списка был наследником Comparable:

```java
public static <T extends Comparable<? super T>> void sort(List<T> list) {
 list.sort(null);
}
```

### Использование Comparable\<T\>

Для создания возможности сортировки, нам необходимо научить сравнить
объекты с другими такого-же типа. И такая реализация будет использоваться
по-умолчанию при сравнении объектов одного класса.

Имплементируем `Comparable` интерфейс, и реализуем метод compareTo:

```java
class Student implements Comparable<Student> {
    private final String name;
    private final double avgMark;

    public Student(String name, double avgMark) {
        this.name = name;
        this.avgMark = avgMark;
    }

    @Override
    public String toString() {
        return "{n='" + name + '\'' + ", m=" + avgMark + '}';
    }

    @Override
    public int compareTo(Student o) {
        return name.compareTo(o.name);
    }
}
```

Обратите внимание, внутри метод мы решили сравнить две строки, а так как
у `String` есть реализация `Comparable` - мы можем ее использовать.

В данном коде опущены части, с проверкой на null объектов `o` и полей класса.

Давайте проверим, как это будет работать:

```java
var ivan = new Student("Иван", 4.3);
var olga = new Student("Ольга", 3.8);
var eugene = new Student("Женя", 4.9);

var studentList = Arrays.asList(ivan, olga, eugene);
System.out.println(studentList);
//сортировка списка
Collections.sort(studentList);
System.out.println(studentList);
```

Вывод:

```text
[{n='Иван', m=4.3}, {n='Ольга', m=3.8}, {n='Женя', m=4.9}]
[{n='Женя', m=4.9}, {n='Иван', m=4.3}, {n='Ольга', m=3.8}]
```

Все отлично, список отсортирован по полю `name`.

Вы можете делать более сложные условия сравнения, только не забывайте
учитывать требование для успешной сортировки - два объекта, сколько бы
мы их не сравнивали - должны всегда давать одинаковый результат.

### Использование Comparator\<T\>

А что если нам надо сортировать студентов не по имени, а по средней оценке?
И при этом оставить возможность сортировать по имени, которое должна использоваться
по умолчанию для создания различных документов.

Нам на помощь придет отдельный класс `Comparator`, которые хранит в себе логику
сравнения объектов и при сортировке, мы можем использовать нужное правило, то есть
нужный объект класса `Comparator`.

Для начала добавим в класс Student геттеры, так как нам уже необходимо использовать
данные класса в классе компаратора.

Создадим класс:

```java
class Student implements Comparable<Student> {
    private final String name;
    private final double avgMark;

    public Student(String name, double avgMark) {
        this.name = name;
        this.avgMark = avgMark;
    }

    @Override
    public String toString() {
        return "{n='" + name + '\'' + ", m=" + avgMark + '}';
    }

    @Override
    public int compareTo(Student o) {
        return name.compareTo(o.name);
    }

    public String getName() {
        return name;
    }

    public double getAvgMark() {
        return avgMark;
    }
}
```

и теперь создадим класс `Comparator`, тип для сравнения `Student`:

```java
class ComparatorByAvgMark implements Comparator<Student> {
    @Override
    public int compare(Student o1, Student o2) {
        return Double.compare(o1.getAvgMark(), o2.getAvgMark());
    }
}
```

Мы снова использовали готовый метод для сравнения стандартного класса `Double`,
это помогает не выдумывать свои реализации, а использовать уже существующие.

Также снова опущены проверки на null объектов o1, o2.

Теперь можно использовать данный класс, и в этот раз нам пригодится перегруженный
метод `Collections.sort()`, который принимает компаратор:

```java
var ivan = new Student("Иван", 4.3);
var olga = new Student("Ольга", 3.8);
var eugene = new Student("Женя", 4.9);

var studentList = Arrays.asList(ivan, olga, eugene);
System.out.println(studentList);

//сортировка списка c использованием компаратора
Collections.sort(studentList, new ComparatorByAvgMark());

System.out.println(studentList);
```

Вывод:

```text
[{n='Иван', m=4.3}, {n='Ольга', m=3.8}, {n='Женя', m=4.9}]
[{n='Ольга', m=3.8}, {n='Иван', m=4.3}, {n='Женя', m=4.9}]
```

И мы видим - сортировка по возрастанию средней оценки студента.

Хорошо, давайте сделаем обратную сортировку, высокие оценки должны быть
в начале списка. Для этого нам потребуется изменить поведение компаратора,
и для этого у компаратора есть метод `reversed()`:

```java
Collections.sort(studentList, new ComparatorByAvgMark().reversed());
```

и в итоге получим нужный результат:

```text
[{n='Женя', m=4.9}, {n='Иван', m=4.3}, {n='Ольга', m=3.8}]
```

Но это еще не все что может компаратор, можно создавать цепочки. Например,
сначала сортируем по оценкам, а если оценки одинаковые, то по имени.

Это можно реализовать не создавая отдельного класса, а воспользоваться
функцией:

```java
Collections.sort(studentList,
        new ComparatorByAvgMark().reversed()
                .thenComparing(Student::getName));
```

При такой сортировки, оценки будут в порядке убывания, а внутри одной
средней оценки, студенты будут по имени в порядке возрастания.

## Метод sort() у самого списка

Кроме использования метода `Collections.sort()`, можно вызывать похожий
метод у самого списка `List.sort()`.
Метод принимает один аргумент - компаратор.

На примере списка студентов:

```java
studentList.sort(new ComparatorByAvgMark());
```

🌿 Счастливой сортировки!
