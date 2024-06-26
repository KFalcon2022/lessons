# Методы. Первое знакомство. Параметры. Возвращение значений

## Первое знакомство

**Метод** — функция (или процедура), принадлежащая какому-либо классу или объекту. Поскольку мы еще не знакомы с ООП, на
текущем этапе примем, что метод — это **функция** в Java или другом объектно-ориентированном языке(C#, например).

### Функция

В программировании **функцией** называют фрагмент кода (**подпрограмму**, набор операций) с собственным именем, к
которому можно обратиться из другой части программы. Обращение происходит по имени функции (на самом деле не совсем,
уточним этот момент на одном из ближайших уроков).

Таким образом, функция (или метод) — это блок кода, который можно использовать множество раз, просто вызывая в нужном
месте.

Примерами могут быть уже знакомые нам `Math.round()`, `Math.pow()`. `System.out.ptintln()`. Здесь `round()` и `pow()` -
методы класса `Math`, `println()` - метод поля `out` класса `System`. `main()` — также является методом. Не стоит 
пугаться незнакомых слов, мы все их разберем уже в ближайших уроках.

Рассмотрим [статью](https://metanit.com/java/tutorial/2.7.php).

Оговорюсь сразу, она не самая удачная. Поэтому постараемся разобрать подробно.

Итак, в общем виде синтаксис метода выглядит так:

```
    [модификаторы] тип_возвращаемого_значения название_метода([параметры]) {
        // тело метода
    }
```

**Модификаторы** нас пока мало интересуют, эту тему мы будем разбирать уже после знакомства с ООП. На данном этапе
предлагаю использовать такой вариант:

```
    static тип_возвращаемого_значения название_метода([параметры]) {
        // тело метода
    }
```

В рамках данного подраздела ограничимся разбором названия и тела метода. **Параметры** и **возвращаемые значения**
разберем в двух подразделах ниже.

Итак, что же остается сейчас:

```java
    static void название_метода() {
        // тело метода
    }
```

**void** — это примитивный тип данных, который ничего не хранит. Ранее мы его не касались, потому что использовать ни в
каком виде не могли.

**название_метода** — по аналогии с переменной, название может быть практически любым. Ограничения в наименовании ровно
те же, что и для переменных: можно использовать буквы, символы `$` и `_`, а также цифры. Однако цифра не может быть 
первым символом наименования:

* `something1()` - можно;
* `1something()` - недопустимо.

Кроме синтаксических ограничений, есть правила, принятые в Java-сообществе:

* Первое (даже единственное) слово в названии метода — глагол. Т.е. название метода `something()` - некорректно.
  Правильным названием было бы `doSomething()`;
* Название метода должно начинаться со строчной буквы: не `DoSomething()`, а `doSomething()`;
* В Java принят **camelCase**. Т.е. слова в названиях разделяются посредством написания каждого следующего слова с
  прописной буквы: `do_something_interesting()`, `dosomethinginteresting()` — некорректно. `doSomethingInteresting()` -
  корректно.

Данные правила не всегда выполняются в базовых классах Java, однако обязательны для исполнения в любом клиентском (том,
который пишут разработчики ПО) коде. Исключения в этих правилах, как всегда, есть, но они вряд ли станут для вас
проблемой к моменту, когда вы их встретите.

### Тело метода

Тело метода, как и тело любой подпрограммы (цикла, if-блока и т.д.) - это набор операций, инструкций для JVM. Т.е. та
полезная нагрузка, которую несет метод.

В качестве примера рассмотрим метод:

```java
    static void printGreeting() {
        System.out.println("Hello!");
        System.out.println("I'm glad to see you!");
    }
```

В данном случае код

```java
    System.out.println("Hello!");
    System.out.println("I'm glad to see you!");
```

Является телом метода. Метод может содержать в теле любое количество строк, в некоторых случаях даже нулевое. В теле
метода могут объявляться переменные, использоваться условные конструкции и циклы, вызываться другие методы. Все, что мы
писали ранее — мы писали в теле метода `main()`.

Вызвать метод `printGreeting()` в `main()` достаточно просто:

```java
    public static void main(String[] args) {
        printGreeting();
    }
```

Также важно отметить, что переменные, объявленные внутри метода, существуют только в этом методе. Рассмотрим на примере:

```java
    static void method1() {
        int a = 5;
        //do sth
    }

    static void method2() {
        System.out.println(a); //Недоступно, переменная "a" не существует в методе method2()
    }
```

Кроме того, из последнего примера следует, что мы можем в соседних методах использовать переменные с одинаковым
названием - это все равно будут разные переменные с точки зрения Java. Но каждую одноименную переменную нужно будет
объявить в каждом методе.

### Параметры метода

Абсолютное большинство методов принимают в себя **параметры**.

Параметры метода — это те переменные, которые передаются в метод для обработки.

Ознакомимся со [статьей](https://metanit.com/java/tutorial/2.16.php).

Теперь общий синтаксис метода для нас выглядит так:

```java
    static void название_метода([параметры]) {
        // тело метода
    }
```

При описании параметров метода мы должны указать тип каждого параметра и его название в рамках этого метода. В теле
метода будет нельзя создать переменную с тем же именем, что у параметра. Несколько параметров указываются через запятую:

```java
    static void method1(int a, int b, double c, String s) {
        //do sth
    }
```

Вызов такого метода может выглядеть так:

```java
    public static void main(String[] args) {
        method1(1, 1, 1.0, "какая-то строка");
    }
```

Или так:

```java
    public static void main(String[] args) {
        int a = 1;
        int b = 1;
        double d = 1.0;
        Strign str = "какая-то строка";

        method1(a, b, d, str);
    }
```

Или даже так:

```java
    public static void main(String[] args) {
        int a = 1;
        double d = 1.0;
        Strign str = "какая-то строка";

        method1(a, 1, d, str);
    }
```

> **!NB**: НЕТ связи между названием параметра при объявлении метода и названием переменной, которая на
> место этого параметра передается. Важен только порядок следования параметров. При объявлении и при вызове он должен
> совпадать.

Еще один важный момент: мы не можем изменить значение переменной, переданной как параметр. Похожую ситуацию мы
видели в цикле `foreach`. Например:

```java
    public static void main(String[] args) {
        int a = 1;
        method1(a);
        System.out.println(a);
    }
    
    static void method1(int a) {
        a = 2;
        System.out.println(a);
    }
```

Такой код будет иметь следующий вывод в консоль:

```
    2
    1
```

В рамках метода значение переменной перезаписалось, однако после выхода из него — не сохранилось и используется то,
которое было до вызова метода.

### Возвращение значений из метода

В этом подразделе мы ознакомимся с синтаксисом, позволяющим возвращать из метода значение, которое можно обрабатывать в
дальнейшем коде.

Для начала, [статья](https://metanit.com/java/tutorial/2.17.php).

В отличие от предыдущих пунктов, данная статья достаточно хорошо раскрывает функциональность оператора `return`.

Подчеркнем лишь несколько моментов:

* Мы можем использовать `return` для void-методов. Синтаксис немного отличается от вызова return в методах, возвращающих
  другие типы: `return someVar;` → `return`;
* Использование `return` для void-методов необязательно, но часто используется для предварительного завершения метода,
  например:

```java
    static void method() {}
        if (someCondition) {
            return;
        }

        // do sth
    }
```

Обратите внимание, что оборачивать логику, находящуюся ниже такого `return`, в `else` нет необходимости: код и так не
выполнится, если условие `if` будет `true`. Метод просто завершится раньше, чем обработает код, находящийся на месте 
`// do sth`.

На этом первое знакомство с методами можно закончить.

## Немного о декомпозиции или зачем вообще нужны методы

Казалось бы, почему бы не писать все в `main()` и жизнь была бы прекрасна.

Но люди придумали красивое слово **декомпозиция** и начали активно им пользоваться.

Декомпозиция — прием разбиения чего-либо на более мелкие части. Проблемы, задачи, куска кода — неважно.

Если говорить о коде, то мы, конечно, можем все писать в `main()`. Но есть нюанс. Даже два:

1. Большие куски кода становятся нечитабельны. Представьте, что `main()` содержит сотни тысяч строк кода. Насколько
   удобно будет поддерживать такую программу?
2. Код может повторяться. Зачем каждый раз переписывать что-либо в нескольких местах одновременно, если можно вынести
   повторяющуюся логику в метод и использовать уже его? Даже если придется что-то поправить, достаточно будет исправить
   это в одном месте.

На самом деле, декомпозиция в программировании — очень обширная тема, которая затрагивает не только выделение частей
кода в методы, но и более глобальные вопросы: разделение кода на классы, проектов — на сервисы (или микросервисы),
отчуждение каких-либо частей кода в отдельные библиотеки или проекты и т.д.

К вопросу декомпозиции мы будем возвращаться еще не раз. Качественная декомпозиция — один из краеугольных камней чистого
кода.

Частый вопрос — до какого уровня стоит декомпозировать код. Абсолютно истинный ответ дать сложно. Но есть масса правил,
позволяющие понять, когда стоит начать, а когда — остановиться. Сегодня познакомимся с двумя из них:

1. Если после декомпозиции понять код будет тяжелее, чем до — декомпозицию стоит отложить;
2. Если ваш метод не влазит в один экран (поэтому некоторые его поворачивают на 90 градусов:)) - метод стоит
   декомпозировать. В целом, если метод получается больше 30 строк — стоит как минимум задуматься о выделении части
   логики в отдельные методы. Не всегда этот подход работает, но является неплохим маркером.

#### С теорией на сегодня все!

![img.png](../../../commonmedia/defaultFooter.jpg)

Переходим к практике:

## Задача 1:

Декомпозировать [задачу](https://github.com/KFalcon2022/practical-tasks/blob/master/src/com/walking/lesson3_casts_conditional_constructions/Task2SwitchCase.java).
Вынести в отдельный метод использование switch-case. Также вынести в отдельный метод использование `System.out.println`

## Задача 2:

Декомпозировать [задачу](https://github.com/KFalcon2022/practical-tasks/blob/master/src/com/walking/lesson3_casts_conditional_constructions/Task4.java).
Вынести в отдельный метод логику, которая отрабатывает, когда первое число кратно и двум, и трем.

## Задача 3:

Написать программу, которая принимает длину и ширину прямоугольника (2 целых числа). Нарисовать в консоли заданный
прямоугольник, используя `-` и `|`. Углы прямоугольника обозначить символом ` `. Каждая единица длины должна
обозначаться одним символом `-`, каждая единица ширины – символом `|`.

Произвести декомпозицию по своему усмотрению. Рекомендую скинуть на проверку. Контакт ниже.

## Также:

Рекомендую попробовать декомпозировать и другие задачи на ваш выбор. Делитесь результатами, посмотрим, что у вас вышло!

**Разбор практики для этого урока**:
[ссылка](https://github.com/KFalcon2022/practical-tasks/tree/master/src/com/walking/lesson6_methods)

> Если что-то непонятно или не получается – welcome в комменты к посту или в лс:)
>
> Канал: https://t.me/ViamSupervadetVadens
>
> Мой тг: https://t.me/ironicMotherfucker
>
> **Дорогу осилит идущий!**
