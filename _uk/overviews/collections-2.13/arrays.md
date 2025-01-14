---
layout: multipage-overview
title: Масиви
partof: collections-213
overview-name: Collections
num: 10
language: uk
---

[Масиви](https://www.scala-lang.org/api/{{ site.scala-version }}/scala/Array.html) є спеціальним типом колекцій в Scala. З однієї сторони, Scala масиви один в один відповідають масивам в Java. Наприклад, Scala масив `Array[Int]` представлений в Java, як `int[]`, а `Array[Double]` в Java виглядатиме, як `double[]` і `Array[String]` відповідатиме Java `String[]`. Але з іншої сторони, Scala масиви пропонують набагато більше, ніж Java аналоги. По-перше, Scala масиви можуть бути _узагальнені_. Наприклад ви можете мати `Array[T]`, де `T` - це параметр типу або ж абстрактний тип. По-друге, Scala масиви сумісні зі Scala послідовностями (`Seq`), тобто ви можете передати на вхід `Array[T]` де вимагається `Seq[T]`. Врешті-решт, Scala масиви також підтримують усі операції, які можна зробити з послідовностями (`Seq`). Ось приклад цього на ділі:

    scala> val a1 = Array(1, 2, 3)
    a1: Array[Int] = Array(1, 2, 3)
    scala> val a2 = a1 map (_ * 3)
    a2: Array[Int] = Array(3, 6, 9)
    scala> val a3 = a2 filter (_ % 2 != 0)
    a3: Array[Int] = Array(3, 9)
    scala> a3.reverse
    res0: Array[Int] = Array(9, 3)

Враховуючи те, що Scala масиви представлені, так само як в Java, яким чином ці додаткові можливості підтримуються в Scala? Реалізація Scala масивів систематично використовує неявні перетворення. В Scala, масив не намагається _вдавати_ послідовність. Насправді він і не може, тому що тип даних представлений у чистому масиві не є підтипом `Seq`. Замість цього відбувається неявне "обгортання", перетворення між масивами та екземпляром класу `scala.collection.mutable.ArraySeq`, який у свою чергу є підтипом `Seq`. Ось, як це працює:

    scala> val seq: collection.Seq[Int] = a1
    seq: scala.collection.Seq[Int] = ArraySeq(1, 2, 3)
    scala> val a4: Array[Int] = seq.toArray
    a4: Array[Int] = Array(1, 2, 3)
    scala> a1 eq a4
    res1: Boolean = false

Приклад вище показує, що масиви сумісні з послідовностями, тому що відбувається неявне перетворення з масивів в `ArraySeq`. Щоб зробити зворотнє перетворення, з `ArraySeq` в `Array`, ви можете використати `toArray` метод, який визначений в `Iterable`. Останній REPL рядок вище демонструє, що обгортання, а потім розгортання за допомогою `toArray` створює копію оригінального масиву.

Існує й інше неявне перетворення, яке застосовується до масивів. Це перетворення просто "додає" усі методи послідовностей до масивів, але не перетворює самі масиви в послідовності. "Додання" означає, що масив є обгорнутий в інший об'єкт типу `ArrayOps`, який підтримує усі методи послідовностей. Зазвичай, цей `ArrayOps` об'єкт є недовговічний; він, як правило, буде недоступним після виклику метода послідовності і його зберігання може бути утилізоване. Новітні віртуальні машини часто уникають створення цього об'єкта повністю.

Різниця між двома неявними перетвореннями масивів показана в наступному прикладі:

    scala> val seq: collection.Seq[Int] = a1
    seq: scala.collection.Seq[Int] = ArraySeq(1, 2, 3)
    scala> seq.reverse
    res2: scala.collection.Seq[Int] = ArraySeq(3, 2, 1)
    scala> val ops: collection.ArrayOps[Int] = a1
    ops: scala.collection.ArrayOps[Int] = scala.collection.ArrayOps@2d7df55
    scala> ops.reverse
    res3: Array[Int] = Array(3, 2, 1)

Ви можете побачити, що виклик метода `reverse` на `seq`, який має тип `ArraySeq`, знову верне тип `ArraySeq`. Це логічно, тому що `ArraySeq` є `Seq`, і виклик метода `reverse` на інших `Seq` знову вертатиме `Seq`. З іншої сторони, виклик `reverse` на об'єкті класу `ArrayOps` верне тип `Array`, а не `Seq`.

Приклад `ArrayOps` вище був дещо штучний, лише для того, щоб показати різницю з `ArraySeq`. Зазвичай ви ніколи не будете створювати самі об'єкт класу `ArrayOps`. Ви будете просто викликати `Seq` метод на масиві:

    scala> a1.reverse
    res4: Array[Int] = Array(3, 2, 1)

Об'єкт `ArrayOps` вставляється автоматично шляхом неявного перетворення. То ж приклад вище еквівалентний наступному:

    scala> intArrayOps(a1).reverse
    res5: Array[Int] = Array(3, 2, 1)

де `intArrayOps` є неявним перетворенням, що було вставлене перед цим. Це викликає питання, як компілятор вибрав `intArrayOps` серед інших неявних перетворень `ArraySeq` в рядку вище. Зрештою, обидва перетворення змінять масив в тип, що підтримує метод `reverse`, що й вказано у вхідних даних. Відповідь на це питання полягає в тому, що два неявних перетворення мають пріоритет. Перетворення `ArrayOps` має більший пріоритет, ніж `ArraySeq`. Перший визначений в `Predef` об'єкті, а другий в класі `scala.LowPriorityImplicits`, який наслідується `Predef`. Неявне перетворення в підкласах та підоб'єктах мають перевагу над неявним перетворенням в базовому класі. То ж, якщо обидва перетворення застосовані, вибирається, той що в `Predef`. Дуже схожий підхід використовується в рядках (`String`).

То ж тепер ви знаєте, як масиви можуть бути сумісні з послідовностями та як вони можуть підтримувати усі операції послідовностей. А як же ж узагальнення? В Java, ви не можете писати `T[]` де `T` є типом параметра. Як в такому випадку представлений Scala `Array[T]`? Фактично узагальнений масив, як `Array[T]` може бути під час виконання будь-яким із восьми примітивних типів масивів Java `byte[]`, `short[]`, `char[]`, `int[]`, `long[]`, `float[]`, `double[]`, `boolean[]`, або ж це може бути масив об'єктів. Єдиний спільний тип під час виконання, який охоплює всі ці типи є `AnyRef` (або еквівалентний `java.lang.Object`), тож це той тип, яким компілятор Scala відображатиме `Array[T]`. Під час виконання, при виклику або оновленню елемента масиву типу `Array[T]` існує послідовність тестів типу, які визначають фактичний тип масиву, після чого виконується правильна операція з масивом Java. Ці тести типу дещо сповільнюють операції з масивами. Ви можете очікувати, що доступ до узагальнених масивів буде в три-чотири рази повільніший, ніж доступ до примітивних або об’єктних масивів. Це означає, що якщо вам потрібна максимальна продуктивність, вам слід віддати перевагу конкретним, а не узагальненим масивам. Представлення узагальненого типу масиву недостатньо, також повинен існувати спосіб створення узагальнених масивів. Це ще складніша проблема, яка вимагає від вас трохи допомоги. Щоб проілюструвати цю проблему, розглянемо наступну спробу написати узагальнений метод, який створює масив.

    // це неправильно!
    def evenElems[T](xs: Vector[T]): Array[T] = {
      val arr = new Array[T]((xs.length + 1) / 2)
      for (i <- 0 until xs.length by 2)
        arr(i / 2) = xs(i)
      arr
    }

Метод `evenElems` повертає новий масив який складається з усіх елементів аргументу вектора `xs` які знаходяться на парних позиціях у векторі. Перший рядок тіла метода `evenElems` створює новий масив результатів, який має той самий тип елемента, що й аргумент. Отже, залежно від фактичного параметра типу `T`, це може бути `Array[Int]`, або `Array[Boolean]`, або масив інших примітивних типів в Java, або ж масив певного посилального типу. Але всі ці типи мають різне представлення під час виконання, тож як середовище виконання Scala вибере правильне? Насправді воно не може цього зробити на основі наданої інформації, оскільки фактичний тип, який відповідає параметру типу `T` стирається під час виконання. Ось чому ви отримаєте таке повідомлення про помилку, якщо скомпілюєте наведений вище код:

    error: cannot find class manifest for element type T
      val arr = new Array[T]((arr.length + 1) / 2)
                ^

Тут потрібно, щоб ви допомогли компілятору, надавши деяку підказку під час виконання, яким є фактичний параметр типу `evenElems`. Ця підказка під час виконання має форму маніфесту типу класу `scala.reflect.ClassTag`. Маніфест класу - це об'єкт дескриптор типа, що описує, який тип у класу верхнього рівня. Як альтернатива маніфестам класу існують також повні маніфести типу `scala.reflect.Manifest`, які описують усі аспекти типу. Але для створення масиву потрібні лише маніфести класу.

Компілятор Scala створюватиме маніфести класів автоматично, якщо ви дасте йому вказівку. «Вказівка» означає, що ви вимагаєте маніфест класу як неявний параметр, наприклад:

    def evenElems[T](xs: Vector[T])(implicit m: ClassTag[T]): Array[T] = ...

Використовуючи альтернативний і коротший синтаксис, ви також можете вимагати, щоб тип постачався з маніфестом класу використовуючи прив'язання до контексту (`context bound`). Це означає встановити зв'язок з типом `ClassTag`, який слідує після двокрапки в описі типу, як в цьому прикладі:

    import scala.reflect.ClassTag
    // це працює
    def evenElems[T: ClassTag](xs: Vector[T]): Array[T] = {
      val arr = new Array[T]((xs.length + 1) / 2)
      for (i <- 0 until xs.length by 2)
        arr(i / 2) = xs(i)
      arr
    }

Дві переглянуті версії `evenElems` означають одне і теж. У будь-якому випадку відбувається те, що під час створення `Array[T]` компілятор шукатиме маніфест класу для параметра типу `T`, тобто шукатиме неявне значення типу `ClassTag[T]`. Якщо таке значення знайдено, то цей маніфест буде використовуватись для побудови потрібного виду масиву. В іншому випадку ви побачите повідомлення про помилку, таке як було показано вище.

Ось деякі приклади використання метода `evenElems` в REPL:

    scala> evenElems(Vector(1, 2, 3, 4, 5))
    res6: Array[Int] = Array(1, 3, 5)
    scala> evenElems(Vector("this", "is", "a", "test", "run"))
    res7: Array[java.lang.String] = Array(this, a, run)

В обох випадках, Scala компілятор автоматично побудував маніфест класу для типу елемента (спершу, `Int`, потім `String`) і вставив його, як неявний параметр метода `evenElems`. Компілятор може зробити це для всіх конкретних типів, але не тоді, коли аргумент сам по собі є іншим параметром типу без свого маніфесту класу. Для прикладу, наступний код не скомпілюється:

    scala> def wrap[U](xs: Vector[U]) = evenElems(xs)
    <console>:6: error: No ClassTag available for U.
         def wrap[U](xs: Vector[U]) = evenElems(xs)
                                               ^

Що сталося тут, так це те, що `evenElems` вимагає маніфест класу для параметра типу `U`, але його не було знайдено. Рішення в цьому випадку, звичайно, полягає в тому, щоб вимагати ще один неявний маніфест класу для `U`. То ж наступний приклад буде працювати:

    scala> def wrap[U: ClassTag](xs: Vector[U]) = evenElems(xs)
    wrap: [U](xs: Vector[U])(implicit evidence$1: scala.reflect.ClassTag[U])Array[U]

Цей приклад також показує, що прив'язання до контексту з `U` є лише скороченням для неявного параметра, названого тут `evidence$1` типа `ClassTag[U]`.

Таким чином, створення узагальненого масиву вимагає маніфестів класів. Тому щоразу, створюючи масив параметра типу `T`, вам також потрібно надати неявний маніфест класу для `T`. Найпростіший спосіб зробити це — оголосити параметр типу з прив'язанням до контексту `ClassTag`, як `[T: ClassTag]`.

