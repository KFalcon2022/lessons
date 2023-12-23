![](../../commonmedia/header.png)

***

   

Базы данных. SQL и NoSQL
========================

Сегодняшний урок открывает следующий объемный раздел нашего курса – **базы данных** (**БД**, **database**, **DB**). В рамках этого раздела мы сконцентрируемся на **реляционных базах данных** (что это такое – разберемся ниже), но в ознакомительном формате рассмотрим и их альтернативы. Но начнем мы не с этого.

Вероятно, каждый из вас так или иначе сталкивался со словосочетанием «база данных». Основная задача текущей статьи – разобраться, что оно значит и почему ему уделяется так много внимания – вы могли заметить по Road Map, что этот раздел по числу уроков больше любого из уже пройденных, не считая Java Core.

### Что такое база данных

База данных – набор данных, хранящихся и обрабатываемых в соответствии с определенными правилами.

Строго говоря, под подобное определение можно подвести как Java-коллекцию, так и обычный текстовый файл, при определенных условиях (например, мы использовали файлы для хранения информации о машинах в ряде практических заданий).

Проблема усугубляется еще и тем, что в обиходе под словосочетанием «база данных» в зависимости от контекста понимают два совершенно разных термина:

· **СУБД** – система управления базами данных. Это ПО, которое обеспечивает хранение данных в заданном виде и обработку операций как над способом организации данных (в каком виде они хранятся, какие ограничения и т.д.), так и над самими данными (их добавление, изменение, удаление, поиск и чтение);

> Когда у разработчика спрашивают, с какими базами данных ему приходилось работать – имеют ввиду, СУБД.

· База данных – структурированный набор данных в рамках конкретной СУБД. С определенным допущением можно сказать, что база данных – это наибольшая единица структуры данных, с которой работает СУБД. Более мелкие единицы зависят от конкретной СУБД. В рамках одной СУБД может быть множество баз данных.

> Когда кто бежит по коридору с заявлением на увольнение и истерично кричит, что удалил базу данных на проде – речь идет о базе данных как совокупности данных.

На данном этапе остановимся в текущих терминах – они, вероятно, слишком абстракты, но более-менее объясняют суть. В некоторые нюансы того, как это все соотносится с реальным миром, углубимся в следующих пунктах.

 Пока же, вероятно, возникает вопрос: а зачем городить огород, если есть коллекции в Java или, на худой конец, текстовые файлы.

### Зачем нужны базы данных

Данный вопрос можно освещать с разных сторон и растянуть на целый цикл статей, потому что причин для использования БД много и ряд из них зависит от конкретной базы данных (читай, СУБД). Но в рамках данной статьи постараемся разобрать наиболее общие причины, которые в той или иной степени актуальны для всех (или большинства) СУБД.

Первый и самый актуальный вопрос, который возникает: зачем нужны базы данных, если мы изучали переменные, коллекции, стримы и массу других инструментов, необходимых для хранения и обработки данных в Java?

Ответов можно найти несколько:

1\. Сохранение информации после завершения (или на время приостановки) работы программы. Переменные в Java хранятся в оперативной памяти, соответственно, они будут потеряны, как только программы завершится или компьютер (или сервер) будет выключен. В свою очередь базы данных (не все, но об этом ниже) хранят данные на диске, как, например, и файлы;

2\. Объемы информации. Одна база данных может содержать терабайты информации. Хранить такие объемы данных на диске можно. Оперативная память даже современных серверов к такому не готова;

3\. Общее хранилище. Одна система может быть разбита на множество маленьких программ (сервисов, микросервисов, функций – зависит от используемой архитектуры и/или инфраструктуры). Или одна и та же программа, запущенная в виде нескольких независимых экземпляров, для повышения производительности.  
Такие программы будут работать в разных процессах (или даже физически на разных серверах), соответственно, не имеют общих ресурсов в оперативной памяти, но вполне могут иметь необходимость работать с одними и теми же данными. В таком случае база данных может выступать общим хранилищем данных, с которым будут общаться все программы (экземпляры программ) в рамках системы. Это едва ли можно назвать причиной существования БД, но вполне можно использовать в качестве ответа «зачем нужна БД?».

Приведенные выше пункты, полагаю, исчерпывающе отвечают на вопрос, почему данные имеет смысл хранить на диске, а не в оперативной памяти. Но не отвечают на вопрос «почему бы вместо этого не использовать текстовые файлы?»

Постараемся разобраться:

· Способ хранения данных. Используя текстовые файлы, нам приходится заботиться о том, как представить каждый конкретный объект (если говорить в терминах ООП) в файле, предусматривать какие-то разделители между полями и объектами. К тому же, объект может быть не один – здесь одним файлом обойтись уже тяжело, придется использовать уже набор файлов. СУБД берет размещение информации на себя, позволяя декларативно описывать схему хранения данных (конкретные сущности и поля, другие нюансы, в зависимости от конкретной СУБД);

· Объемы данных. Если мы хотим более-менее адекватные по скорости чтение и запись данных из/в файл, вряд ли вы станете хранить свои текстовые файлы заархивированными. Соответственно, они могут быть очень большими по размеру. СУБД, в свою очередь, более эффективно хранит данные. Вряд ли это можно назвать критической причиной, к тому же, далеко не все СУБД заботятся об эффективном сжатии данных и, в конце концов, тратят память на хранение побочной информации, не только данных как таковых, но все же. При прочих равных, обычный текстовый файл вряд ли будет более эффективным хранилищем информации, чем БД;

· И снова объемы информации. Когда данных много, их не просто нужно уметь хранить. Их еще нужно эффективно (по времени) изменять и, что еще важнее, эффективно в этом наборе данных искать.  
Использование текстового файла, так или иначе, заставляет вас считывать из него данные в Java и уже там отбирать необходимые. Здесь есть что оптимизировать, но проблематика очевидна: текстовый файл плохо подходит для поиска в нем подмножеств записей. Обычный поиск записей в файле по значению определенного поля (в терминах Java) может стать целым приключением.  
В это время СУБД имеют встроенные механизмы для поиска данных и их изменения. А также, как правило, механизмы для более тонкой настройки, позволяющих сделать более эффективными конкретные операции (скажем, эффективный поиск по значению конкретного поля, если вы обычно ищете именно по нему). Тут есть оговорки и зависимость от СУБД, но любая из них в рамках своих задач выиграет у текстового файла по эффективности поиска и изменения данных.  
**Итого**: СУБД ищут информацию внутри себя, не заставляя тянуть лишнее на уровень Java, имеют эффективные механизмы взаимодействия с хранимыми данными;

· Защищенность данных. Очевидно, что данные имеют ценность. И требуют определенных ограничений по доступу к ним. СУБД, в большинстве своем, предоставляют готовые механизмы ролей, прав и прочих механик, позволяющих разграничивать доступ к данным, права на чтение и запись и пр. Конечно, подобно можно сделать и для файлов – практически любая операционная система позволяет настроить права доступа. Но это куда более громоздкий механизм, к тому же, завязанный на конкретную операционную систему и настройки в рамках конкретного сервера/компьютера. СУБД в этом вопросе будет куда более дружелюбна;

· Понятное API. Текстовый файл, грубо говоря, не имеет вообще никакого API – мы можем изменять его каким угодно образом (если имеем доступ): от некорректных записей в условные «поля» до записи вообще чего-то, не относящегося к хранимым данным и их структуре. Достаточно заменить разделители между объектами-записями и файл станет непригоден для использования. СУБД же предоставляет как конкретное API по работе с данными (операции вставки, изменения, удаления, поиска данных и пр.), так и имеет массу механизмов валидации, которые не позволят добавить конкретной записи что-то, что в ней быть не должно – от проверки типов до уникальных значений и даже больше – все зависит от СУБД, нюансов настройки и ваших потребностей.

  

Вполне вероятно, что преимуществ можно найти еще много даже в выбранной парадигме сравнения. Но, полагаю, уже на данном этапе примерно понятно, почему базам данных уделяется столько внимания (как в разработке, так и в рамках курса) – они являются основным хранилищем информации в практически любой системе.

### Классификации баз данных

Баз данных (СУБД) достаточно много. Вероятно, наиболее используемые можно ограничить парой десятков, но это все равно целая плеяда инструментов, предназначенная под свои задачи (широкие или узкие), имеющая свои преимущества, недостатки и просто особенности.

Классифицировать СУБД можно по-разному, но нас будут интересовать лишь две классификации, обе достаточно простые.

**Базы данных по «месту» хранения данных**:

· **Disk-based БД**. К этой группе относится абсолютное большинство СУБД. Хранение данных происходит на диске. Таким образом, чтение и запись у таких БД менее эффективны, чем в рамках оперативной памяти, но они позволяют хранить огромные объемы информации и устойчивы к различного рода отказам – в целом, можно сюда поместить все достоинства хранения данных на диске против хранения в java-переменных;

· **In-memory БД**. Такие СУБД живут в рамках оперативной памяти. Плюсом является то, что чтение и запись в них намного быстрее, чем у disk-based. Минусы – очевидно, объемы и отказоустойчивость. Как правило, такие БД используются для относительно узких задач – хранение кэшей, обеспечение доступа к данным разным программам и т.д. Все то, что не требует долговременного хранения данных и не имеет ценности, если работа системы будет остановлена. С последним есть нюансы, но на текущем уровне понимания это более-менее корректно.

Другой важно и, вероятно, даже более популярной классификацией СУБД является разделение по способу хранения данных (используемым единицам информации).

#### Реляционные базы данных

Именно этому виду СУБД будет посвящен весь остальной курс.

С точки зрения структуры такие БД представляют собой **таблицы**. Можно провести аналогию с классами: одна таблица – один класс. Таблицы состоят из **колонок** (можно ассоциировать с полями класса). **Записи** (строки) в таких таблицах в нашей системе координат соответствуют объектам.

На самом деле, к аналогиям из мира ООП стоит относиться осторожно, когда речь идет о БД. Не все можно подвести под эти аналогии, не везде они будут полностью эквивалентны. Но они могут помочь освоиться на первых этапах.

Возвращаясь к **реляционным СУБД** (РСУБД, relational database management system, **RDBMS**), свое название они получили не за хранение информации в таблицах. Их основная особенность – реализация «**отношений**» между записями в таблицах. Так, реляционные базы данных предоставляют механизмы, позволяющие однозначно связать запись в одной таблице с записью (записями) в другой таблице. Что позволяет строить целые цепочки связей (отношений) и иметь гарантию их корректности – СУБД регламентирует, не может быть связи с несуществующим значением или других эксцессов, связанных с некорректной связью.

На первый взгляд может звучать не так серьезно, но пока рекомендую поверить: этот механизм крайне упрощает жизнь и его тяжело недооценить. Как правило, именно реляционные БД являются основными хранилищами данных. На текущем этапе можно считать реляционную базу данных решением по умолчанию.

Справедливости ради, реляционные базы данных заняли свою нишу далеко не только из-за механизма отношений, но подробнее будем разбираться в следующих уроках, уже в привязке к конкретным инструментам реляционных БД.

Следующим важным моментом, касающимся реляционных СУБД, является то, что все они (по крайней мере, широко известные) имеют одинаковый интерфейс для работы – SQL.

**SQL** (**Structured Query Language**) – язык запросов, позвляющий работать с реляционными базами данных. Он покрывает практически все – от создания самой БД (как единицы в рамках СУБД) и таблиц в ней до вставки и поиска по конкретным таблицам и многое другое.

На самом деле, каждая реляционная СУБД имеет собственный диалект SQL, который может иметь свои особенности, фишки и нюансы. Но все эти диалекты строятся на базе общей спецификации SQL. Поэтому научившись работать с одной реляционной базой данных, можно работать со всеми. Конечно, такой переход не будет бесшовным – привыкать и изучать отличающиеся возможности придется. Но это несоизмеримо проще, чем, например, перейти с одного языка программирования на другой.

Именно изучению SQL преимущественно и будет посвящен текущий раздел. На данном этапе стоит отметить, что SQL плотно ассоциируется именно с реляционными БД.

#### NoSQL

Возвращаясь к вопросу классификации, альтернатива реляционным базам данных – **NoSQL** БД. Он объединяет все СУБД, которые не вошли в первую категорию

NoSQL – НЕ означает, что эти СУБД не работают с SQL. Некоторые из них его вполне используют. Термин имеет больше маркетинговый смысл (SQL очень плотно ассоциируется с реляционной моделью данных).

NoSQL объединяет под собой нереляционные БД. Они бывают разными и основные виды мы тезисно разберем чуть ниже.

Как правило, NoSQL БД используют для достаточно широкого спектра конкретных задач. Если реляционные БД являются хранилищем по умолчанию, то каждая NoSQL БД будет выбрана под конкретные сценарии использования. Поэтому вполне ожидаемо, что на многих проектах есть реляционная база данных, но нет NoSQL – просто не возникло задач, которые требовали бы применения этих инструментов.

Рассмотрим конкретные виды NoSQL БД. Способ классификации тот же – по способу хранения данных. Каждый из них имеет свои реализации.

> **!NB**: каждая реализация может иметь свой собственный интерфейс – здесь нет единого стандарта, как в случае с SQL. Поэтому при использовании вам могут встретиться совершенно разные вариации – от привычного (уже скоро он станет таким) SQL до специфических наборов команд или общения с базой данных через JSON-файлы.

  

#### БД вида «ключ-значение»

Вероятно, самый простой для восприятия вид NoSQL БД. По сути, каждая база данных будет представлять собой ассоциативный массивов – то, что в Java мы называем «_Map_».

Классический пример применения – хранение кэшей. В целом, позволяет хранить любую информацию, которая укладывается в структуру ключ-значение.

#### Колоночные БД

Можно также встретить название «**столбцовые**» БД и прочи вариации, основанные на английском «**column**».

Этот вид СУБД, на первый взгляд, очень похож на реляционный. Данные хранятся в таблицах. Различия с реляционными БД, на самом деле, колоссально, но на данном этапе ограничимся тем, что колоночные БД не предоставляют механизма отношений. Безусловно, ничто не мешает его эмулировать, но это не всегда возможно и такая эмуляция не дает тех же гарантий и возможностей, что и реляционная база данных.

Как правило, используются для аналитических приложений – данные БД традиционно обеспечивают быстрое чтение из столбцов, что позволяет эффективно манипулировать данными в разрезе их фильтрации и агрегации.

#### Документные (документоориентированные) БД

Данные БД хранят данные в «**документах**» - более гибких структурах, чем таблицы. Как правило, речь идет об использовании **JSON** или **XML**. Стоит загуглить и посмотреть формат хранения данных в виде JSON, в этом нет ничего сложного и это удобно ложится в объектную модель, привычную в Java. XML в этом отношении тяжелее воспринимать внешне, в остальном он имеет схожую структуру.

Такие «документы» являются менее жесткой структурой, чем таблица. Но главное их преимущество – они позволяют хранить иерархические структуры. Условно, поле документа (можно ассоциировать с полем у объекта класса) может содержать другой объект. Вложенные объекты могут содержать свои вложенные объекты и т.д.

Проводя аналогии с реляционными БД, документ обычно аналогичен записи в таблице.

Документы могут быть сгруппированы в аналоги таблиц – такие аналоги называются **коллекциями**.

Сценарии использования документоориентированных СУБД достаточно широкие – по сути, они покрывают большинство сценариев, в которых нужно хранить и обрабатывать относительно структурированные наборы данных со сложной – не укладывающейся в таблицу – структурой. Строго говоря, документ можно разложить в несколько таблиц, если потребуется его представить в типичном для РСУБД виде. Но далеко не всегда в этом есть необходимость.

В качестве условного примера – в документоориентированной БД можно было бы хранить статьи данного курса. Каждая статья могла бы быть разбита на разделы, которые разбивались бы, в свою очередь, на абзацы. В итоге хранимый документ представлял бы собой не полотно текста, а иерархическую структуру из определенных блоков. При этом каждый раздел мог бы хранить в себе различное число абзацев, а статья – различное число разделов. Чем-то это может напомнить практическую с разбором текста на элементы, предложенную в уроке по regex. Но в данном случае вовсе не обязательно было бы хранить элементы в массиве – вполне можно было бы выделить собственное поле каждому разделу/абзацу/другой единице текста на своем уровне иерархии.

#### Графовые БД

Данный вид БД подходит для хранения элементов со сложными системами отношений между собой. Скажем, в графовой БД можно было бы хранить списки друзей в социальной сети, если бы было необходимо искать друзей друзей или строить цепочки «рукопожатий» – в данном случае, речь про сценарии использования связей между данными, а не самих атомарных данных как таковых.

Под подобные задачи не подойдут, например, документоориентированные БД – там могут быть иерархии, но множественные связи в таких БД строить тяжело и, как правило, некорректно.

Реляционные БД под решение задачи множественных связей подходят – виды отношений мы будем разбирать в свое время, пока нюансы не так важны. Но построение сложных цепочек связей обычно является дорогой задачей у этого вида БД.

Графовые же направлены именно на быстрое и дешевое построение цепочек подобных связей и эффективны в своей (относительно узкой) зоне ответственности.

### В качестве заключения

Базы данных многообразны. Это огромный пласт знаний, который придется поднимать годами, углубляясь в конкретные ответвления по мере необходимости. Задача данной статьи – дать общее представление о том, что такое БД, зачем они нужны (в общем случае) и какие бывают.

Задача раздела – научить основам работы с реляционными БД. В отличии от NoSQL, без реляционных БД никуда. NoSQL же хоть и является куда более многообразным по формам, но в среднем он куда проще в плане освоения конкретной реализации. К тому же, он не так актуален для начинающих специалистов – по крайней мере, в контексте собеседования.

Полагаю, очевидно, что при описанном подходе нереляционные БД останутся за бортом. На данном этапе это, вероятно, не критично. Но, надеюсь, именно данная статья поможет легче зайти в мир NoSQL, когда это потребуется.

С теорией на сегодня все!

Статья вводная, практики нет:)

![](../../commonmedia/footer.png)

Если что-то непонятно или не получается – welcome в комменты к посту или в лс:)

Канал: [https://t.me/ViamSupervadetVadens](https://t.me/ViamSupervadetVadens)

Мой тг: [https://t.me/ironicMotherfucker](https://t.me/ironicMotherfucker)

_Дорогу осилит идущий!_