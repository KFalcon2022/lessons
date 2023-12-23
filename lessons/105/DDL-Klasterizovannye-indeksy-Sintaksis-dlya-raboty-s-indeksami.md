![](../../commonmedia/header.png)

***

   

DDL. Кластеризованные индексы. Синтаксис для работы с индексами
===============================================================

### Кластеризованные индексы

Сегодня вернемся к тебе индексов. Начнем с классификации, которую в прошлый раз упустили - **кластеризованные** и **некластеризованные** индексы.

Как говорилось в предыдущем уроке, индекс представляет собой структуру данных, которая содержит в себе ссылки на конкретные записи таблиц. Собственно, это справедливо для обычных индексов - их также можно назвать не кластеризованными в данной классификации.

Однако в некоторых СУБД (но не в PostgreSQL) существует и другой тип - кластеризованные. В целом, можно было бы опустить эту тему, если бы не любовь к ней на собеседованиях.

Итак. Кластеризованный индекс характеризуется тем, что он не является отдельной от таблицы структурой - данный индекс “хранится” вместе с таблицей и определяет порядок, в котором хранятся сами записи в рамках таблицы.

Таким образом, при использовании кластеризованного индекса можно утверждать, что записи в таблице хранятся в отсортированном виде - на основании колонок или выражений, заданных таким индексом.

> **!NB**: Это вовсе не означает, что в выборке без указания сортировки будет применен этот же порядок. Лишь указывает на то, что фильтрация и сортировка по ключу индекса будет быстрее.

Исходя из того, что кластеризованный индекс определяет порядок записей в таблице, несложно догадаться, что такой индекс может быть всего один на таблицу.

Как правило, кластеризованные индексы строятся на базе B-tree (по крайней мере, я не знаю альтернативных примеров).

И, наконец, немного об использовании.

В общем-то, кластеризованный индекс обладает теми же рекомендациями по определению колонок, что и любой другой B-tree индекс. Но есть нюанс. Обычно индексы призваны ускорить поиск по таблице, что характерно и для кластеризованного индекса. Но поиск - это не только про WHERE-условия, но еще и про _JOIN_. А _JOIN_, в свою очередь, чаще всего происходит по Primary Key. В силу этого исторически сложилась популярность кластеризованного индекса по PK.

В целом, нельзя сказать, что кластеризованный индекс является необходимым атрибутом таблицы - PostgreSQL, не имеющий такого механизма тому доказательство. Однако у такого индекса есть как минимум один плюс: он не требует дискового пространства для своего хранения:)

  

### Синтаксис работы с индексами

Теперь предлагаю перейти к самой простой части в теме индексов - синтаксису. Как и в других случаях, будем опираться на PostgreSQL. В случае с индексами это особенно актуально - спецификация SQL вообще не оперирует понятием индекса. Поэтому хоть общий синтаксис их использования в различных СУБД и похож (читай, продиктован духом DDL), общего регламента для синтаксиса нет.

Итак, рассмотрим простейший пример запроса для создания составного индекса:

```java
create index i_name on t1 (a, b);
```

В данном случае мы создали индекс с именем _i\_name_ на основании колонок _a_ и _b_ таблицы _t1_. Полагаю, очевидно, что колонки и выражения указываются в скобках через запятую. Соответственно, если бы нам потребовался не составной индекс - в скобках была бы указано одна колонка (или выражение).

К слову, пример создания индекса на базе выражения - для более удобного поиска по _LIKE_ для фамилии пассажира:

```java
create index i_passenger_last_name_lower 
  on passenger (lower(last_name));
```

Советую давать индексам осмысленные имена, которые позволяют определить таблицу, колонки и нюансы конкретного индекса - это облегчит дальнейшую поддержку.

Возможно, вы уже задались вопросом, какой тип индекса будет использован - ведь мы уже познакомились с _BTREE_, _HASH_, _GIN_ и другими.

Собственно, данный синтаксис создает индекс с типом по умолчанию - _BTREE_. Если мы хотим указать тип индекса явно - мы должны добавить предложение **_USING_**:

```java
create index i_passenger_last_purchase_hash 
  on passenger using hash (last_purchase);
```

Кроме этого существуют и некоторые другие предложения запроса, которые можно использовать при создании индекса (например, _WHERE_ для указания условий **частичного индекса**). Но, на мой взгляд, это не актуально на данном этапе (и вообще не актуально для основной массы разработчиков).

Также для индексов существуют и опции с оператором _ALTER_. Правда, на данном этапе нам актуально только переименование индекса, что не является частой операцией. Но хорошая новость в том, что основные опции _ALTER_ по отношению к индексу разработчику бывают нужны еще реже.

Итак, переименование:

```java
alter index i_passenger_last_purchase_hash 
  rename to ix_passenger_last_purchase_hash;
```

Как видите, общая структура запросов в рамках DDL мало меняется от того, с каким структурным элементом мы работаем в данный момент.

И, наконец, удаление индекса:

```java
drop index ix_passenger_last_purchase_hash;
```

Здесь тоже без сюрпризов.

  

### Вместо заключения

Индексы - крайне обширная тема, которой можно посвятить отдельный полноценный курс. В рамках текущего курса мы лишь знакомимся с основными терминами и концепцией индексов. Это более, чем достаточно для джуниоров (и, пожалуй, мидлов). Но, вероятно, настанет момент, когда вам придется углубиться в эту тему, выйдя за пределы курса. Хорошая новость заключается в том, что к этому моменту вы, вероятно, будете иметь более обширную теоретическую базу, чем сейчас, а также практический опыт, который кратно облегчит дальнейшее погружение в тему.

  

С теорией на сегодня все!

![](../../commonmedia/footer.png)

Переходим к практике:

### Задача

Проанализируйте SELECT-запросы, представленные в практике к предыдущим урокам. Добавьте индексы на колонки, которые наиболее часто использовались для фильтрации и сортировки данных.

Везде ли типом индекса стоит выбрать _BTREE_? Требуются ли составные индексы в рамках этой задачи?

  

Если что-то непонятно или не получается – welcome в комменты к посту или в лс:)

Канал: [https://t.me/ViamSupervadetVadens](https://t.me/ViamSupervadetVadens)

Мой тг: [https://t.me/ironicMotherfucker](https://t.me/ironicMotherfucker)

_Дорогу осилит идущий!_