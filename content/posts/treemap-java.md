---
title: "Класс TreeMap в Java - устройство и использование"
date: 2022-11-04T00:46:36+05:00
description: "Класс TreeMap входит в состав Java Collection Framework и
представляет собой структуру данных в виде дерева. Разберемся, как устроен и когда стоит применять." 
author: "Konstantin Shibkov"
tags: ["java", "TreeMap", "Map"]
categories: ["java"]
draft: true
---

Класс TreeMap входит в состав Java Collection Framework и
представляет собой структуру данных в виде дерева. Разберемся, как устроен и когда стоит применять. Особенность
TreeMap - ключи хранятся в отсортированном порядке и сложность
поиска ключа составляет O(log n).

{{< callout type="info" >}}Используется в примерах Java версии 17{{< /callout >}}

## Иерархия наследования TreeMap

Класс <a href="https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/TreeMap.html" target="_blank">TreeMap</a>
наследует интерфейс
<a href="https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/NavigableMap.html" target="_blank">NavigableMap</a>
и расширяет абстрактный класс <a href="https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/AbstractMap.html" target="_blank">AbstractMap</a>.

![Иерархия класса TreeMap](treemap-class-diagram.png)

`NavigableMap` обязывает класс реализовать методы, часть из которых:

* `Map.Entry<K,V> firstEntry()` - получение записи Entry с наименьшим ключом.
* `Map.Entry<K,V> lastEntry()` - получение записи Entry с наибольшим ключом.
* `Map.Entry<K,V> pollFirstEntry()` - возвращает запись Entry с наименьшим ключом и удаляет из TreeMap.
* `Map.Entry<K,V> pollLastEntry()` - возвращает запись Entry с наибольшим ключом и удаляет из TreeMap.
* `NavigableMap<K,V> descendingMap()` - возвращает Map в обратном направлении, изменения в возвращаемом Map напрямую влияет на исходную TreeMap
* `K lowerKey(K key)` - получение первого ключа, точно меньше по значению чем переданный ключ или null, если такого ключа не найдено
* `K higherKey(K key)` - получение первого ключа, точно больше по значению чем переданный ключ или null, если такого ключа не найдено

Остальные методы похожи на представленные в списке или имеют вариации.

Класс `AbstractMap` уже имеет часть реализаций интерфейса `Map`, например `int size()`, `boolean isEmpty()`,
`boolean containsValue(Object value)`, `boolean containsKey(Object key)` и другие. Причем, часть из них `TreeMap`
переопределяет для реализации своей структуры.

## Особенности TreeMap

Перед погружением в структуру класса, рассмотрим - "Что делает TreeMap особенным и когда стоит его использовать?"

Стоит использовать TreeMap, в случае если вам требуется быстро извлекать данные по ключу, при этом требуется сортировка по ключу.
Например, если вам необходимо получать минимальный или максимальный ключ или получать набор данных меньше или больше определенного ключа.

Если обратиться к более конкретным примерам:

* необходимо хранить строки-ключи в алфавитном порядке, ключ String.
* расположить объекты по приоритету, но получать по ключу. *Первую часть у нас может решить TreeSet, но получать по ключу мы не можем.*
* хранить объекты в разных TreeMap, с разной сортировкой.

{{< callout type="success" >}}Воспользоваться всеми преимуществами TreeMap возможно, если объявлять тип не просто Map, а
NavigableMap<K,V> или TreeMap<K,V>{{< /callout >}}

🧑‍💻 **Пример:**

Записать слова Zeta, Asteroid, back, Art, Bow, zen  в списки по первой букве:

```java
{a=[Asteroid, Art], b=[back, Bow], z=[Zeta, zen]}
```

или по количеству букв:

```java
{3=[Art, Bow, zen], 4=[Zeta, back], 8=[Asteroid]}
```

Таким образом, мы можем быстро получить все слова на букву B или все слова,
в которых количество букв больше 3.

Код реализующий пример выше:

```java
String[] words = { "Zeta", "Asteroid", "back", "Art", "Bow", "zen" };

        // создаем TreeMap и заполняем словами, ключ для каждого - первая буква
        // используем в объявлении типа именно NavigableMap, так как нам нужны будут
        // его уникальные методы
        NavigableMap<Character, List<String>> wordsByFirstLetter = new TreeMap<>();

        for (String s : words) {
            wordsByFirstLetter
                    .computeIfAbsent(s.toLowerCase().charAt(0), ArrayList::new)
                    .add(s);
        }

        System.out.println(wordsByFirstLetter);
        
        // Все слова на букву b
        System.out.println(wordsByFirstLetter.get('b'));

        // создаем TreeMap и заполняем словами, ключ для каждого - длина слова
        // используем в объявлении типа именно NavigableMap, так как нам нужны будут
        // его уникальные методы
        NavigableMap<Integer, List<String>> stringsBySize = new TreeMap<>();

        for (String s : words) {
            stringsBySize.computeIfAbsent(s.length(), ArrayList::new).add(s);
        }

        System.out.println(stringsBySize);
        
        // Получим все слова длиной больше или равные 4 и сведем в один список
        List<String> wordsLengthGtThree = stringsBySize.tailMap(4)
                .values()
                .stream()
                .flatMap(Collection::stream)
                .toList();

        System.out.println(wordsLengthGtThree);
```

Результат работы:

```text
{a=[Asteroid, Art], b=[back, Bow], z=[Zeta, zen]}
[back, Bow]
{3=[Art, Bow, zen], 4=[Zeta, back], 8=[Asteroid]}
[Zeta, back, Asteroid]
```

## Что внутри TreeMap?

Для этого зайдем в исходный код самого класса и найдем,
что он хранит в полях:

* Компаратор - неудивительно, ведь нам нужны правила для сравнения ключей и расположения их
в нужном порядке.

```java
private final Comparator<? super K> comparator
```

* Корневая запись, содержащая ключи K и значение V.

```java
private transient Entry<K,V> root
```

* счетчик размера элементов size, при добавлении +1, при удалении -1

```java
private transient int size = 0
```

* счетчик изменений коллекции, используется для определения попытки изменить
TreeMap несколькими потоками. Если такое будет замечено - получим `ConcurrentModificationException`

```java
private transient int modCount = 0
```

Больше ничего и нет в TreeMap, если не учитывать `size` и `modCount` - ведь они не несут полезной информации.
То в TreeMap есть только компаратор и ссылка на корневой элемент дерева.

И скорее вы уже слышали, что в TreeMap используется красно-черное дерево. Давай посмотрим наглядно, что
происходит, когда мы создаем TreeMap и наполняем его элементами. Будем использовать наш пример со словами.

### Конструкторы

Первое что нам нужно - создать новый TreeMap. Какие у нас есть конструкторы для этого?

* Конструктор по умолчанию
  
```java
var wordsByLength = new TreeMap<Integer, String>();
```

После его создания внутри класса comparator=null, root=null, size=0, modCount=0. Полностью пустой класс.
Пустым конструктором можно пользоваться, если класс ключа реализует интерфейс `Comparable`, который
делаем возможным сравнить два объекта класса и расставить их по порядку. Класс Integer наследует Comparable
и мы можем его использовать как ключ без дополнительных настроек.

* Конструктор c компаратором

```java
record Planet(String name, long radius) {}

var populationByPlanet = new TreeMap<Planet, Long>(
    Comparator.comparing(Planet::radius));
```

После создания, снова заглянем внутрь TreeMap и увидим: comparator=Comparator$lambda@716, root=null,
size=0, modCount=0. Мы передали компаратор в виде лямбды и видим - класс знает в каком порядку
расположить планеты. Можно заполнять планетами и они будут расположены в порядке роста радиуса.

🌟 Больше про компараторы и варианты сортировки -
[в моей статье "Сортировка в Java"]({{< ref "/posts/java-collections-sort" >}})

* Конструктор принимающий другую Map

```java
var treeMap = new TreeMap<>(Map.of(8, "Oxygen", 75, "Rhenium"));
```

Integer известно как сравнивать - компаратор не понадобится. И в итоге в объекте
comparator=null, root=TreeMap$Entry, size=2, modCount=2.

В конструкторе произошло добавление элементов, поэтому видим - корневой элемент уже не пустой,
было 2 модификации и количество элементов равно 2. Пока достаточно того, что у нас данные из
Map перенеслись в TreeMap. Дальше мы будем смотреть как они располагаются.

* Конструктор принимающий SortedMap

Фактически это конструктор принимающий другую TreeMap, так как TreeMap это наследник SortedMap.
При этом копируется значение comparator и все элементы переносятся во вновь созданный
TreeMap.

```java
var treeMap = new TreeMap<>(Map.of(8, "Oxygen", 75, "Rhenium"));
var subTreeMap = new TreeMap<>(treeMap.subMap(10, 100));
```

В subTreeMap будет один элемент `{75 => "Rhenium"}`. Если мы просто сохранили результат
метода `treeMap.subMap(10, 100)`, то получили SortedMap, а не TreeMap.

### Внутреннее строение

Самое время посмотреть, что скрывается за `Entry<K,V> root` и как работает поиск
по ключу.

Создадим список городов и на основе списка сформируем TreeMap, ключом будет расстояние города от
Москвы. Для расчета расстояния созданы дополнительные методы `calcDistance`, `deg2rad`, `rad2deg` -
в их суть вникать не требуется, нам важен код в `main`: 

```java
public class CitiesExample {
    public static void main(String[] args) {
        City moscow = new City("Moscow", 55.45, 37.37);
        var cities = List.of(
                new City("Ufa", 54.44, 55.58),
                new City("Saint Petersburg", 59.57, 30.19),
                new City("Kazan", 55.47, 49.06),
                new City("Minsk", 53.54, 27.34),
                new City("Kyiv", 50.27, 30.31),
                new City("Astana", 51.10, 71.26));

        // упаковываем список городов в Map, где ключ это расстояние от Москвы,
        // а значение сам город
        TreeMap<Double, City> citiesByDistance = new TreeMap<>();
        for (City city: cities){
            citiesByDistance.put(calcDistance(moscow, city), city);
        }

        //вывод городов с расстоянием от москвы от 0км до 1000км
        System.out.println(citiesByDistance.subMap(0d, 1000d));
        //вывод городов с расстоянием от москвы от 1500км до 5000км
        System.out.println(citiesByDistance.subMap(1500d, 5000d));
    }

    record City(String name, double latitude, double longitude) {
    }

    private static double calcDistance(City cityFrom, City cityTo) {
        double lon1 = cityFrom.longitude();
        double lon2 = cityTo.longitude();
        double lat1 = cityFrom.latitude();
        double lat2 = cityTo.latitude();

        double theta = lon1 - lon2;
        double dist = Math.sin(deg2rad(lat1)) * Math.sin(deg2rad(lat2))
                + Math.cos(deg2rad(lat1)) * Math.cos(deg2rad(lat2))
                * Math.cos(deg2rad(theta));
        dist = Math.acos(dist);
        dist = rad2deg(dist);
        return dist * 60 * 1.1515 * 1.609344;
    }

    private static double deg2rad(double deg) {
        return (deg * Math.PI / 180.0);
    }

    private static double rad2deg(double rad) {
        return (rad * 180.0 / Math.PI);
    }
}
```

Теперь посмотрим на структуру внутри TreeMap по мере наполнения
новыми элементами и как будет строится сбалансированное красно-черное дерево:

1. **При создании пустого TreeMap, поле `root` равен `null`**

    ![Иерархия класса TreeMap](structure-0.svg)

2. **Добавляем в TreeMap первый город Ufa**

   Для ключа рассчитываем расстояние от Москвы на основе координат. Так как
   в TreeMap ничего нет, первый элемент становится корневым, то есть заполняет root.

    ![Иерархия класса TreeMap](structure-1.svg)

    Каждый узел (элемент) дерева является классом `Entry<K,V>` и имеет три ссылки: на `Entry` родителя,
    `Entry` правого дочернего узла, `Entry` левого дочернего узла и еще свойство `color` - это цвет узла,
    может быть BLACK или RED. Причем храниться в boolean, где BLACK = true.

    У первого элемента нет ни родителя, ни дочерних узлов и `root` всегда черного цвета.

    В последующих схемах, right или left дочерние элементы будут показаны только если они не null.

3. **Добавляем в TreeMap второй город Saint Petersburg**

    Значение ключа = 626.83, это меньше чем у корневого элемента, поэтому мы создаем Entry и
    записываем ссылку на него в `Entry<Double, City> left`. То есть, если ключ меньше чему у узла
    на котором мы находимся - мы идем по левой ветке, а если больше - то по правой. Это справедливо
    и для поиска, в чем мы еще успеем убедиться.

    В итоге получаем такую структуру:

    ![Иерархия класса TreeMap](structure-2.svg)

    Дерево в данном случае сбалансировано, если по простому, то на каждом ярусе дерева должны быть заполнены все
    узлы перед тем как создавать узлы нового яруса. В нашем случае перебалансировка не требуется.

4. **Добавляем в TreeMap третий город Kazan**

    Значение ключа = 736.10, это меньше чем у корневого элемента, идем по левой ветке. У Saint Petersburg
    ключ меньше чем 736.10, значит нам надо записать Kazan в правый дочерний узел. Так и сделаем:

    ![Иерархия класса TreeMap](structure-3-nb.svg)

    И дерево уже несбалансированно, так как мы создали третий ярус, но второй еще не заполнен, ведь у
    root нет правого элемента. В таком случае требуется перебалансировка.

    В данной статье не будем разбираться как она реализована. Мы можем мысленно попробовать перестроить
    структуру, так чтобы был root и два его дочерних узла были заполнены. Казань станет root, справа
    Уфа и слева Санкт-Петербург.

    ![Иерархия класса TreeMap](structure-3-b.svg)







