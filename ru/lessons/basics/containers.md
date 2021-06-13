---
version: 1.3.1
title: Контейнеры
---

Кортежи, списки, списки ассоциаций, множества, отображения / хэш-таблицы, векторы.

{% include toc.html %}

## Когда какой контейнер брать

Вначале дадим кратчайшее (одно предложение на контейнер) руководство о том,
когда использовать каждый контейнер. Всё непонятное здесь должно проясниться
после прочтения более глубоких объяснений каждого контейнера далее.

- Быстро сгруппировать несколько фрагментов данных? Смотрим на [кортежи](#кортежи)
- Быстрое добавление в начало? Смотрим на [списки](#списки)
- Уникальные и отсортированные элементы? Смотрим на [множества](#множества)
- Искать ключ в сопоставлении "ключ-значение"? Смотрим на [отображения](#отображения)
- Быстрая индексация? Смотрим на [вектора](#вектора)

## Использование контейнерных модулей

Приятная особенность экосистемы контейнеров в Haskell заключается в том, что все
модули предоставляют очень схожий интерфейс. Это позволяет развить интуицию и
быстро освоить широкий набор инструментов. Это также означает неизбежность
конфликтов имён. На помощью приходит практика квалифицированного импорта.

Если вы собираетесь запускать примеры, приведённые в этой главе, на своём
компьютере в GHCi, вначале введите следующие квалифицированные импорты, чтобы
все контейнерные модули оказались области видимости.

```haskell
ghci> import Data.Set as Set
ghci> import Data.Map as Map
ghci> import Data.List.NonEmpty as NE
ghci> :t Map.empty
Map.empty :: Map k a
ghci> :t Set.empty
Set.empty :: Set.Set a
ghci> :t NE.head
NE.head :: NE.NonEmpty a -> a
```

На всякий случай приведём пример ошибке, которая возникает
когда вы используете несколько неквалифицированных контейнерных модулей.

```haskell
ghci> import Data.List
ghci> import Data.Set
ghci> import Data.Map
ghci> lookup

<interactive>:4:1: error:
    Ambiguous occurrence ‘lookup’
    It could refer to
       either ‘Data.Map.lookup’,
              imported from ‘Data.Map’
              (and originally defined in ‘Data.Map.Internal’)
           or ‘Prelude.lookup’,
              imported from ‘Prelude’ (and originally defined in ‘GHC.List’)
ghci> empty

<interactive>:5:1: error:
    Ambiguous occurrence ‘empty’
    It could refer to
       either ‘Data.Map.empty’,
              imported from ‘Data.Map’
              (and originally defined in ‘Data.Map.Internal’)
           or ‘Data.Set.empty’,
              imported from ‘Data.Set’
              (and originally defined in ‘Data.Set.Internal’)
```

## Кортежи

Изучение контейнеров в Haskell начнём с кортежей. Они просты, встроены в язык и
имеют лаконичный синтаксис. Способ обращения к полю кортежа зависит от позиции
поля в этой структуре. Хотя теоретически кортежи могут содержать любое
количество элементов, на практике Haskell Report требует от реализации языка не
менее 15 полей, а GHC поддерживает максимум 62 поля. Мы сосредоточимся на
двухэлементных кортежах, поскольку Haskell имеет хорошую встроенную поддержку
для них. Однако для примера покажем кортеж с 8 полями.

```haskell
ghci> :t ('0', '1', '2', '3', '4', '5', '6', "8-tuple")
('0', '1', '2', '3', '4', '5', '6', "8-tuple")
  :: (Char, Char, Char, Char, Char, Char, Char, [Char])
```
__Замечание__: команда `:t` в GHCi показывает тип значения. Она распечатает
сообщение в форме `выражение :: Тип`.

### Когда использовать

Кортежи полезны для компоновки данных; они позволяют сказать: «Мне нужно и то, и
другое сразу». Кортежи подчёркивают *структуру* данных, а не их
*предназначение*, поэтому иногда кортежи избегают. Если данные используются в
нескольких местах, зачастую лучше определить запись. Однако если случай
использования локален (например, в пределах области видимости одной функции),
тогда кортеж может пригодиться!

### Создание кортежей

Кортежи создают, используя круглые скобки с элементами, разделенными запятой
`(a, b)`.

```haskell
ghci> myTuple = (True, "hello")
ghci> :t myTuple
myTuple :: (Bool, [Char])
```

Также можно оставить поле кортежа пустым, и это превратит его в функцию. Такая
техника называется сечением кортежей. Она требует расширение языка
`TupleSections`.

```haskell
ghci> :set -XTupleSections
ghci> :t (True,)
(True,) :: t -> (Bool, t)
ghci> :t (,"hello")
(,"hello") :: t -> (t, String)
ghci> :t (,)
(,) :: a -> b -> (a, b)
ghci> (,) True "hello"
(True,"hello")
```
__Замечание__: синтаксис для включения языкового расширения внутри GHCi выглядит
так: ключевое слово `:set` позволяет GHCi узнать что мы устанавливаем флаг
компилятора; префикс `-X` сообщает GHCi, что мы обращаемся за расширением языка;
в конце следует имя расширения.

### Использование кортежей

Для использования кортежей их разбивают на части. Самый распространенный подход
это сопоставление с образцом по структуре кортежа с последующим доступом к его
компонентам. Такая техника хороша тем, что она обобщается на кортежи любого
размера.

```haskell
ghci> (\(a, b) -> not a) myTuple
False
ghci> (\(a, b) -> b <> " world") myTuple
"hello world"

ghci> (\(a, b, c) -> a <> " " <> b <> " " <> c) ("my", "name", "is")
"my name is"
```

Для кортежа из двух элементов есть функции `fst` и `snd` из стандартной
библиотеки. Двухэлементные кортежи это единственная разновидность кортежа, для
которого имеется такого рода встроенная поддержка, поэтому, если вам нужен
кортеж побольше, будьте готовы к написанию своих собственных функций доступа.

```haskell
ghci> :t fst
fst :: (a, b) -> a
ghci> fst myTuple
True
ghci> :t snd
snd :: (a, b) -> b
ghci> snd myTuple
"hello"
```

### Ограничения кортежей

Основное ограничение кортежей в Haskell состоит в том, что кортежи разных длин
это различные типы данных. Таким образом, нет общей функции для добавления
нового элемента к произвольному кортежу. Более того, такие функции должны
определяться для каждого типа отдельно.

Вот пример увеличения длины двухэлементного кортежа.

```haskell
ghci> twoTupleToThreeTuple c (a, b) = (a, b, c)
ghci> twoTupleToThreeTuple () myTuple
(1, "world", ())
```

Следующая попытка вызвать нашу функцию для кортежа неправильной длины приводит к
ошибке типов, сообщающей нам прямо: функция ожидала 2-кортеж, а мы передали ей
3-кортеж.

```haskell
ghci> twoTupleToThreeTuple True (1, 2, 3)

<interactive>:19:27: error:
    • Couldn't match expected type: (a, b)
                  with actual type: (a0, b0, c0)
                  ...
```

## Списки

С точки зрения удобства использования списки решают проблему изменения длины
контейнера, которая характерна для кортежей, но списки могут содержать только
один тип (другими словами, они однородны). Списки также имеют специальный
встроенный синтаксис.

```haskell
ghci> [1,2,3,4]
[1,2,3,4]
```

### Индуктивные типы

Списки начинают наше знакомство с «индуктивными» или «рекурсивными» типами.
Посмотрим на код, который идентичен встроенной реализации Haskell, но без
синтаксического сахара.

```haskell
data List a = Nil | Cons a (List a)
```

Заметим, что этот тип рекурсивный: `Nil` это базовый случай, а конструктор
`Cons` объединяет элемент `a` и рекурсивный вызов `List a`. Также понятно,
почему списки могут содержать только 1 тип: элементы одного типа `a` наполняют
всю структуру данных на каждом шаге рекурсии. Если в данном определении `Nil`
заменить пустым списком `[]`, а `Cons` заменить `:`, то мы вернёмся к
встроенному синтаксису.

Проведём демонстрацию списков с помощью встроенного синтаксиса, а также
конструктора `:` и, наконец, нашего ручного определения. Все три примера
эквивалентны:

```haskell
ghci> [1,2,3,4]
[1,2,3,4]
ghci> 1 : 2 : 3 : 4 : []
[1,2,3,4]
ghci> Cons 1 (Cons 2 (Cons 3 (Cons 4 Nil)))
Cons 1 (Cons 2 (Cons 3 (Cons 4 Nil)))
```

### Когда использовать

Связные списки — чрезвычайно распространенная структура данных в функциональном
программировании, а значит вы встретите их повсюду в Haskell. Обычно списки это
первый контейнер, к которому тянется программист на Haskell. Из-за медленной
конкатенации и относительно медленного индексирования (𝛰 (n), где n это
индекс), они используются в ситуациях, когда известно, что вам придется
пройтись по всему набору данных, или необходимо сохранить порядок элементов.

Отличный пример абстрактной структуры данных, для которой связный список хорошо
подходит в качестве конкретной реализации, это стеки. Всё потому что `push` и
`pop` имеют в таком случае сложность 𝛰 (1).

Плохим вариантом использования списка будет очередь, где либо поставить в
очередь, либо извлечь из очереди займёт 𝛰 (n) (в зависимости от того, с какой
стороны связного списка вы решили добавлять элементы).

Распространенный реальный пример использования списков это запросы к базе
данных в памяти. Запрос к такой базе может не вернуть результата `[]` или
вернуть несколько потенциально упорядоченных результатов `[entity ..]`. Такая
библиотека для баз данных не будет реализовывать доступ по индексу и оставит
это на усмотрение клиента.

### Конкатенация списков

Для конкатенации списков используется операция `++`:

```haskell
ghci> [1, 2] ++ [3, 4, 1]
[1, 2, 3, 4, 1]
```

### Голова и хвост

При использовании списков обычно обрабатывают голову и хвост списка. Голова это
первый элемент списка, а хвост это список всех остальных элементов.

Haskell предоставляет две полезные функции для работы с этими частями, `head` и
`tail`:

```haskell
ghci> head ["Orange", "Banana", "Apple"]
"Orange"
ghci> tail ["Orange", "Banana", "Apple"]
["Banana","Apple"]
```

К сожалению, эти функции раскрывают неприглядную сторону стандартной библиотеки
языка (`base`): голова и хвост могут вызвать исключение, даже если им передан
аргумент нужного типа. Причина в том, что эти функции не покрывают всё
множество возможных входов.

```haskell
ghci> head []
*** Exception: Prelude.head: empty list
ghci> tail []
*** Exception: Prelude.tail: empty list
```

В Haskell распространена идиома для безопасной работы с частичными функциями —
тип `Maybe`. Он позволяет сказать, что необработанные входные данные приводят к
значению `Nothing` на выходе. При таком подходе вызывающая сторона обязана
обработать случай `Nothing`, но взамен мы получаем защиту от неприятной
исключительной ситуации во время выполнения.

```haskell
ghci> :i Maybe
data Maybe a = Nothing | Just a   -- Defined in ‘GHC.Maybe’
...
```
__Замечание__: `:i` в GHCi предоставляет информацию о типе; первая строка это реализация.

Покажем как определить тотальную функцию головы и хвоста при помощи
сопоставления с образцом и `Maybe`.

```haskell
ghci> :{
| safeHead :: [a] -> Maybe a
| safeHead [] = Nothing
| safeHead (x:xs) = Just x
|
| safeTail :: [a] -> Maybe [a]
| safeTail [] = Nothing
| safeTail (x:xs) = Just xs
| :}
ghci> safeHead ["Orange", "Banana", "Apple"]
Just "Orange"
ghci> safeHead []
Nothing
ghci> safeTail ["Orange", "Banana", "Apple"]
Just ["Banana","Apple"]
ghci> safeTail []
Nothing
```
__Замечание__: `:{` и `:}` позволяют писать многострочные определения в GHCi.

Больше никаких исключений!

Другой способ обеспечить безопасность `head` и `tail` это тип непустого списка:

```haskell
ghci> import Data.List.NonEmpty
ghci> :i NonEmpty
data NonEmpty a = a .:| [a]
```

Из определения видно, что `NonEmpty` требует наличия первого элемента. Символ
`:|` является конструктором, аналогично `:`; фактически определение `NonEmpty`
эквивалентно спискам но исключает случай пустого списка.

Такое решение проблемы частичных функций прямо противоположно решению с
`Maybe`. Вместо того, чтобы заставлять клиента функции защищаться от
нежелательного случая при обработке её результата, мы заставляем предоставить
изначально корректный вход (при вызове функции).

```haskell
ghci> import Data.List.NonEmpty as NE
ghci> :t NE.head
NE.head :: NonEmpty a -> a
ghci> head (1 :| [])
1
ghci> NE.head []
<interactive>:11:9: error:
    • Couldn't match expected type: NonEmpty a
                  with actual type: [a]
    ...
```

Обратите внимание, что на этот раз ошибка это не исключение времени выполнения,
а ошибка типов; компилятор сообщает, что мы пытались использовать
(потенциально пустой) список, а не требуемый тип `NonEmpty`.

### List Performance

Haskell implements lists as linked lists. The cons cells (the operator `:` is
called cons, short for constructor) act as the links. This dictates which
operations can be done quickly and which can be slow:

Prepending a value to a list is easy and fast - all we have to do is create a
new cons cell with the element we want to prepend and point it to the existing
list.

```haskell
prepend value list = value : list
```

On the other hand, since the list data type (as shown above) can be either
empty (`[]` or `Nil`) or a cons cell that will point to the rest of the list,
it does not contain information about the length of the list, or a reference to
the end of the list.

Because of that, in order to retrieve the length of a list we must walk
each cons cell and count until we reach the end of the list. To find the
value at a specific index we need to traverse the list until we reach it.

Similarly, in order to append a list to an existing list, we need to go to the
end of the existing list, and add a cons cell that points to the new list:

```haskell
append originalList newList =
    case originalList of
        [] -> newList
        x : xs -> x : append xs newList
```

The append function defined here is really the same as the `++` operator, as you
might have deduced we need to be careful when using list append. Particularly
`++` inside of loops has quadratic performance!

By virtue of the linked list data structure, many list operations run in linear
time (`𝛰(n)`). In many cases the same operation is significantly slower for
lists than for other containers, this is a great reason to be familiar with each
and their tradeoffs!

## Assoc lists

So far we have only been able to access values via position indexed lookup, or
pattern matching. However, one of the most common use cases for containers is
acting as a key value store. Assoc(iation) lists provide this by combining
2-tuples and regular lists. Since assoc lists are really just the combination
of two existing data types, the only thing we need is a lookup function, which
is provided in the `Data.List` module in base.

```haskell
ghci> import Data.List as List
ghci> assoc = [("foo", True), ("bar", False)]
ghci> :t assoc
assoc :: [(String, Bool)]
ghci> :t List.lookup
List.lookup :: Eq a => a -> [(a, b)] -> Maybe b
ghci> List.lookup "foo" assoc
Just True
ghci> List.lookup "bar" assoc
Just False
ghci> List.lookup "baz" assoc
Nothing
```

We can see the pattern, again, where no result is encoded using the `Maybe`
type. This is because it is always *possible* that the key are looking up isn't
present in our assoc list.

It is also interesting to note the `Eq a` constraint on the "key" which allows
the lookup function to do an equality comparison on them.

While assoc lists are a nice introduction to key value containers, and they
build on the previous types we learned about, they are not particularly useful.
A list simply isn't a very good data structure for lookup, as it provides worst
case `𝛰(n)` asymptotics. Assoc lists are usually an intermediate data structure
which Haskell programmers will convert into a `Map`. Although this conversion
is itself an `𝛰(n*log n)` operation `Map` provides an `𝛰(log n)` lookup, so the
conversion cost is amortised very quickly.

## Sets

Sets are a very interesting container, the core concept of a set is membership.
It is common to build up sets, and then test to see if an element exists in that
set.

A set can be constructed exclusively by inserting elements into the empty set

```haskell
ghci> import Data.Set as Set
ghci> :t Set.empty
Set.empty :: Set a
ghci> Set.empty
fromList []
ghci> Set.insert 1 (Set.insert 2 (Set.insert 3 Set.empty))
fromList [1,2,3]
```

Or by creating a set from a list

```haskell
ghci> Set.fromList [4,3,2,1]
fromList [1,2,3,4]
```

You might notice that the elements are sorted after turning them into a `Set`.
Internally the `Set` data type depends on its contents being orderable. The
concrete implementation of sets in Haskell are binary trees, which depend on an
ordering. We can see the ordering requirement from the constraints on the
`insert` and `fromList` (and other) functions.

```haskell
ghci> :t Set.insert
Set.insert :: Ord a => a -> Set a -> Set a
ghci> :t Set.fromList
Set.fromList :: Ord a => [a] -> Set a
ghci> :t Set.member
Set.member :: Ord a => a -> Set a -> Bool
```

Sets have a very useful property, they cannot contain duplicates.

```haskell
ghci> insert1 = Set.insert 1
ghci> insert1 Set.empty
fromList [1]
ghci> insert1 (insert1 Set.empty)
fromList [1]
ghci> insert1 (insert1 (insert1 Set.empty))
fromList [1]
```

Alright, let's see an almost-practical use case for sets.

```haskell
ghci> evens = Set.fromList [0,2..1000000]
ghci> Set.member 7 evens
False
ghci> Set.member 200012 evens
True
ghci> isEven n = Set.member n evens
ghci> isEven 7
False
ghci> isEven 8
True
```

You might say "hmmm a `1000000` limit for even numbers seems incorrect", and you
would be right! This highlights a property of sets in haskell, they are finite
due to the strictness in the underlying data structure. This contrasts lists,
which are lazy and therefore potentially infinite.

### Set Operations

Set difference is a good way to break apart a set based on the membership of its
elements in another set. It will return a set with all the elements of the first
argument less the elements of its second argument.

```haskell
ghci> set1 = Set.fromList ["a", "b", "c", "1", "2", "3"]
ghci> letters = Set.fromList ["a", "b", "c"]
ghci> nums = Set.fromList ["1", "2", "3"]
ghci> Set.difference set1 letters
fromList ["1","2","3"]
ghci> Set.difference set1 nums
fromList ["a","b","c"]
```

Union's are useful for when we want to build up a new set from the combination
of two other sets.

```haskell
ghci> nums
fromList ["1","2","3"]
ghci> letters
fromList ["a","b","c"]
ghci> Set.union nums letters
fromList ["1","2","3","a","b","c"]
ghci> set1
fromList ["1","2","3","a","b","c"]
ghci> Set.union nums set1
fromList ["1","2","3","a","b","c"]
```

Intersection allows us to find elements that sets have in common.

```haskell
ghci> Set.intersection nums letters
fromList []
ghci> Set.intersection nums set1
fromList ["1","2","3"]
ghci> Set.intersection letters set1
fromList ["a","b","c"]
ghci> Set.intersection (fromList [1, 2]) (fromList [2, 3])
fromList [2]
```

Subsets allow us to detect if all the elements of a set are contained within
another set.

```haskell
ghci> Set.isSubsetOf letters set1
True
ghci> Set.isSubsetOf nums set1
True
ghci> Set.isSubsetOf nums letters
False
ghci> Set.isSubsetOf set1 nums
False
```

Every set is a subset of itself.

```haskell
ghci> Set.isSubsetOf nums nums
True
ghci> Set.isSubsetOf set1 set1
True
ghci> Set.isSubsetOf letters letters
True
```

## Maps

In Haskell, maps are the "go-to" key-value store, sometimes referred to as a
dictionary.

A `Map` requires that its key has an ordering (`Ord k`), in the same way that
`Set` required an ordering on the value. This is because the internal
implementation of the `Map` type in Haskell is a size balanced binary tree. This
is actually the same data structure as is used for `Set`.

The `Map` data type and the functions for interacting with it are exported from
`Data.Map` which is a module in the `containers` package. Don't worry about
dependencies though, containers is a core library and ships with ghci.

Like sets, a map can be constructed by inserting key value pairs into an empty map

```haskell
ghci> import Data.Map (Map)
ghci> import qualifide Data.Map as Map
ghci> :t Map.empty
Map.empty :: Map k a
ghci> Map.empty
fromList []
ghci> Map.insert "a" 'a' (Map.insert "b" 'b' (Map.insert "c" 'c' Map.empty))
fromList [("a",'a'),("b",'b'),("c",'c')]
```

Or by creating it from a list

```haskell
ghci> Map.fromList [(4, '4'), (3, '3'), (2, '2'), (1,'1')]
fromList [(1,'1'),(2,'2'),(3,'3'),(4,'4')]
```

## Updating Values to a Map

A useful function to be aware of is `adjust`. This lets us update a value at a
specified key, only if it exists, if not the old map is returned.

```haskell
ghci> Map.adjust (+2) "first" oneItem
fromList [("first",3)]
ghci> :t Map.adjust
Map.adjust :: Ord k => (a -> a) -> k -> Map k a -> Map k a
ghci> Map.adjust (+2) "second" oneItem
fromList [("first",1)]
```

The observant reader will notice that `adjust` doesn't actually update the map.
The second invocation of adjust returns the original `oneItem` map, this makes
sense when you consider that all data in Haskell is immutable!

## When to use Maps

Maps are really great for in memory persistence of state that will need to be
retrieved by a key of some arbitrary type. This is is because maps have great
lookup asymptotics (`𝛰(log n)`) due to the ordering on the key values. The
example that immediately springs to mind is session storage on the server in a
web application. The session state can be indexed by an id that is stored in
the cookie, and the session state can be retrieved from an in memory `Map` on
every request. This solution doesn't scale infinitely, but you would be
surprised how well it works!

## HashMaps

There are some cases where we do not have an ordering on our type, but still
want to use it as a key to index a map. In this case we probably want to reach
for a hashmap. A hashmap simply hashes the key, et voilà, we have an ordering!
This does require that the key type is hashable though.

The module that exports the `HashMap` data type and functionality is called
`Data.HashMap.Strict`, it lives in the `unordered-containers` package. The api
is identical to `Map` aside from a `Hashable k` constraint instead of an `Ord k`
on the key.

## Vectors

Sometimes we want a list like container where we can have performant indexed
access. In fact arrays are often the primitive sequential data structure in
lots of popular programming langauges. Haskell has this kind of data structures
too, and the most popular implementation is the `Vector` type from the `vector`
library.

One of the coolest things about the `Vector` type is that it offers `𝛰(1)`
indexed access. The `vector` library offers both a safe and unsafe index
accessor operator.

Here is an example where we want to create a container that allows us to
retrieve ascii chars by their character code.

__Note__: You can fetch the `vector` library using `cabal repl --build-depends "vector"`
or `stack exec --package vector -- ghci`

```haskell
ghci> import Data.Vector as V
ghci> asciiChars = V.fromList ['\NUL'..'\DEL']
ghci> asciiChars ! 48
'0'
ghci> asciiChars ! 97
'a'
ghci> asciiChars ! 65
'A'
ghci> asciiChars !? 65
Just 'A'
ghci> asciiChars !? 128
Nothing
ghci> asciiChars !? 127
Just '\DEL'
```

Vectors also offer efficient slicing `𝛰(1)`, so they are really good for
creating sub-vectors. The slice function takes the index to start at, the
length of the slice, and the vector to slice, and returns a new vector. Slicing
is unsafe, if the starting index plus the length of the slice exceeds the vector
you will get a runtime error.

```haskell
ghci> :t V.slice
V.slice :: Int -> Int -> Vector a -> Vector a
ghci> lowerCase = V.slice 97 26 asciiChars
ghci> lowerCase
"abcdefghijklmnopqrstuvwxyz"
ghci> upperCase = V.slice 65 26 asciiChars
ghci> upperCase
"ABCDEFGHIJKLMNOPQRSTUVWXYZ"
ghci> nums = V.slice 48 10 asciiChars
ghci> V.length nums
128
ghci> nums
"0123456789"
-- Error case, the asciiChars vectors has length 128, and our slice supposes a
-- length of 97 + 92 (189)
ghci> V.slice 97 92 asciiChars
"*** Exception: ./Data/Vector/Generic.hs:408 (slice): invalid slice (97,92,128)
CallStack (from HasCallStack):
  error, called at ./Data/Vector/Internal/Check.hs:87:5 in vector-0.12.3.0-8cc976946fcdbc43a65d82e2ca0ef40a7bb90b17e6cc65c288a8b694f5ac3127:Data.Vector.Internal.Check
```

## Overloaded Lists

You may have noticed that we frequently use the `fromList` function to
construct our containers. There is a language extension that allows us to
simply use the list syntax to construct these containers. The drawback of
overloading the list syntax is that we can get confusing type errors if we are
not explicit about our types.

```haskell
ghci> :set -XOverloadedLists -- This is the language extension
ghci> [1,2,3] :: Set Int
fromList [1,2,3]
ghci> [1,2,3] :: Vector Int
[1,2,3]
ghci> [('a', 1),('b', 2),('c', 3)] :: Map Char Int
fromList [('a',1),('b',2),('c',3)]
```

However, if we aren't explicit about the type...

```haskell
ghci> ["1", "2", "3"]

<interactive>:11:1: error:
    • Illegal equational constraint GHC.Exts.Item l ~ [Char]
      (Use GADTs or TypeFamilies to permit this)
    • When checking the inferred type
        it :: forall {l}.
              (GHC.Exts.IsList l, GHC.Exts.Item l ~ [Char]) =>
              l
```

This error is a bit of a doozey, which is why this language extension is
typically not enabled by default. It is still good to be aware of it!
