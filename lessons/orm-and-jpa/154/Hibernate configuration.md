# Подключение Hibernate к проекту. Конфигурация

Сегодня мы разберемся, как подключить Hibernate к проекту. Данная статья все еще остается теоретической - 
поверхностно познакомимся с файлами конфигурации, посмотрим, какие требуются зависимости.

Также тезисно разберем разницу подходов к работе с Hibernate напрямую и через JPA.

Как и в случае с Servlet API и `web.xml`, начнем со знакомства с файлами конфигурации, а затем постепенно будем 
обогащать их полезной нагрузкой.

## Gradle-зависимости

Несмотря на то, что мы приступаем к изучению высокоуровневого инструмента, база никуда не исчезает - нам все так же 
необходимо указать драйвер БД, который будет использован JDBC для работы с БД. Лишь само взаимодействие с JDBC 
теперь будет происходить неявно, во внутренней логике ORM:

```groovy
    runtimeOnly 'org.postgresql:postgresql:42.7.1'
```

Также нам необходим указать конкретную ORM, которую мы будем использовать. Как было сказано ранее, мы будем 
использовать Hibernate:

```groovy
    implementation 'org.hibernate.orm:hibernate-core:6.6.2.Final'
```

В принципе, на этом можно остановиться. Hibernate самодостаточен и предоставляет все необходимое публичное API.

Но так как наша основная цель - знакомство с JPA, также нам необходимо добавить зависимость для нее:

```groovy
    implementation 'jakarta.persistence:jakarta.persistence-api:3.1.0'
```

Данная зависимость не принесет нам никакой новой функциональности сама по себе, но позволит условно-неявно 
использовать сам Hibernate - вместо обращения к его интерфейсам и классам, мы будем взаимодействовать с JPA, которая 
внутри себя будет обращаться к Hibernate.

У такого подхода можно выделить следующие преимущества:

- Единообразие API. Несмотря на огромную популярность Hibernate, он является не единственной реализацией JPA. И 
  гораздо проще изучить одну спецификацию, а затем использовать ее вне зависимости от конечной реализации ORM 
  (Hibernate, EclipseLink и т.д.), нежели изучать отдельный API для каждой из них;
- Дешевая замена реализации. Теоретически подход к работе через API спецификации позволяет достаточно быстро - или 
  даже бесшовно - заменить одну ORM на другую. Практическая ценность этой возможности околонулевая - обычно решение 
  принимается тогда же, когда выбирается и СУБД, поскольку различные реализации имеют разную производительность с 
  разными базами данных и не меняется в дальнейшем. Но в качестве формального плюса имеет право на жизнь.

Принципиальные минусы подхода найти сложно. Даже если нам по какой-то причине понадобятся специфические возможности 
Hibernate, недоступные через JPA - мы практически всегда сможем их использовать, не нарушив общую работоспособность 
системы - ведь API Hibernate все еще доступен в пределах проекта.

В демонстрационных целях ниже приводится конфигурация приложения как для подхода с использованием JPA, так и для 
прямого взаимодействия с Hibernate.

Также при работе при работе с Hibernate, независимо от подхода, есть некоторая специфика конфигурации HikariCP. 
Поэтому для демонстрации сразу добавим и его:

```groovy
    implementation 'com.zaxxer:HikariCP:5.1.0'
```

## Файлы конфигурации

### Прямое подключение

При подключении Hibernate напрямую должен быть использован его собственный файл конфигурации - `hibernate.cfg.xml`, 
который определяется в директории `./src/main/resources`.

В нем необходимо указать параметры подключения к БД и внутренние настройки Hibernate для его основной управляющей 
сущности - `SessionFactory`. С самой сущностью мы будем знакомиться в следующих статьях, пока можем ограничиться тем,
что она являет собой глобальный контекст для работы с БД.

Пример `hibernate.cfg.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <property name="hibernate.connection.driver_class">org.postgresql.Driver</property>
        <property name="hibernate.connection.url">jdbc:postgresql://localhost:5432/test_db</property>
        <property name="hibernate.connection.username">postgres</property>
        <property name="hibernate.connection.password">postgres</property>

        <property name="hibernate.connection.provider_class">com.zaxxer.hikari.hibernate.HikariConnectionProvider</property>
        <property name="hibernate.hikari.minimumIdle">10</property>
        <property name="hibernate.hikari.maximumPoolSize">10</property>
        <property name="hibernate.hikari.idleTimeout">600000</property>
        <property name="hibernate.hikari.connectionTimeout">30000</property>
        <property name="hibernate.hikari.maxLifetime">1800000</property>
    </session-factory>
</hibernate-configuration>
```

В данном случае мы указываем в конфигурации лишь параметры для базы данных и параметры для HikariCP, не прописывая 
ничего из специфичных настроек самой ORM.

Можете обратить внимание, что исходя из имени параметров, Hibernate имеет внутренний модуль для интеграции с 
HikariCP. Также по `HikariConnectionProvider` и его пакету несложно догадаться, что и сам Hikari из коробки содержит 
API для интеграции с Hibernate. Это не всегда было так и ранее приходилось использовать для интеграции отдельную 
зависимость, но это уже в прошлом.

В базовом виде конфигурация готова - при минимальных навыках работы с XML проблем с ней не возникнет. Единственный 
вопрос - откуда брать имена параметров.

Вариантов ответов может быть несколько:

- Документация. Самый очевидный и правильный ответ. Начать можно
  [отсюда](https://docs.jboss.org/hibernate/orm/6.6/introduction/html_single/Hibernate_Introduction.html#configuration).
  Но есть нюанс. В документации конфигурация может быть размазана тонким слоем по разделам с привязкой под то, за что 
  отвечает конкретный ключ;
- Интерфейсы с ключами. Например, `AvailableSettings` и другие *Settings-интерфейсы в пакете `org.hibernate.cfg`. 
  Ключи будут сгруппированы также по зоне ответственности, но хотя бы лежат в одном месте.

Указанные выше интерфейсы появились не просто так. Кроме XML-конфигурации Hibernate также предлагает конфигурацию 
через properties-файлы и через Java-код. Вот для последнего и требуются заранее определенные константы ключей. 
Java-конфигурация доступна через класс `Configuration`. Мы с ним еще столкнемся, но именно для Java-конфигурации 
использовать не будем - этот подход имеет право на жизнь в целях изучения или маленьких пет-проектах, но для любой 
серьезной разработки он неудобен и традиционно используются файлы конфигурации в том или ином виде.

### Подключение через JPA

JPA декларирует один общий файл конфигурации - `persistence.xml`, расположенный в пути `./src/main/resources/META-INF`.

Глобально подход к наполнению очень похож. Но если Hibernate-конфигурация оперирует конфигурацией `SessionFactory`, 
то JPA смотрит на мир шире - через конфигурацию Persistence Unit.

Фактически, Persistence Unit - это надстройка, которая отвечает за один конкретный контекст для работы с конкретной БД.
Из-за того, что под контролем JPA в одном приложении может быть коммуникация с разными БД, в том числе (теоретически) 
через разные реализации JPA, приходится вводить дополнительный уровень абстракции. Внутри конфигурации Persistence 
Unit фактически располагается конфигурация `EntityManagerFactory` - интерфейса, по зоне ответственности аналогичного
`SessionFactory` в Hibernate*.

При этом для классического приложения с одной БД и одним контекстом для нее, основная разница в конфигурации 
сводится к названию тега - и в `persistence.xml`, и в `hibernate.cfg.xml` для такой ситуации будет определен 
примерно один и тот же набор параметров с небольшими различиями в названиях ключей и синтаксисе тегов.

> *В предыдущей статье было указано, что по историческим причинам JPA и Hibernate называют одни и те же абстракции 
> разными именами. Это один из примеров такой ситуации.
> 
> Если мы откроем исходный код `SessionFactory` - увидим, что этот интерфейс наследует JPA-интерфейс
> `EntityManagerFactory`. Поэтому в теоретическом материале между ними зачастую можно ставить знак равенства.
> 
> С тем, какова роль данных интерфейсов в приложении, будем разбираться в одной из ближайших статей.

Пример конфигурации Hibernate через `pesistence.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="https://jakarta.ee/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="https://jakarta.ee/xml/ns/persistence
                                 https://jakarta.ee/xml/ns/persistence/persistence_3_0.xsd"
             version="3.0">
    <persistence-unit name="Hibernate" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>

        <properties>
            <property name="jakarta.persistence.jdbc.driver" value="org.postgresql.Driver"/>
            <property name="jakarta.persistence.jdbc.url" value="jdbc:postgresql://localhost:5432/test_db"/>
            <property name="jakarta.persistence.jdbc.user" value="postgres"/>
            <property name="jakarta.persistence.jdbc.password" value="postgres"/>

            <property name="hibernate.connection.provider_class" value="com.zaxxer.hikari.hibernate.HikariConnectionProvider"/>
            <property name="hibernate.hikari.minimumIdle" value="10"/>
            <property name="hibernate.hikari.maximumPoolSize" value="10"/>
            <property name="hibernate.hikari.idleTimeout" value="600000"/>
            <property name="hibernate.hikari.connectionTimeout" value="30000"/>
            <property name="hibernate.hikari.maxLifetime" value="1800000"/>
        </properties>
    </persistence-unit>
</persistence>
```

Важные моменты, которые стоит отметить, сравнивая данную конфигурацию с `hibernate.cfg.xml`:

- Несмотря на то, что JPA декларирует единый подход для любой реализации, внутри часто можно увидеть параметры, 
  характерные для конкретного Persistence Provider'а - в данном случае мы видим параметры из Hibernate для работы с 
  HikariCP;
- Отличается синтаксис тега `<property>` - в `hibernate.cfg.xml` значение передается как значение тега, в
  `pesistence.xml` - как атрибут `value`, сам тег при этом не имеет значения в привычном для XML понимании;
- Нам необходимо определять Persistence Provider для конкретного Persistence Unit - в данном случае указан класс 
  `HibernatePersistenceProvider`. Сам механизм работы провайдеров чем-то похож на то, как работает подгрузка 
  драйверов в JDBC. Чуть подробнее остановимся на этом моменте при детальном знакомстве с `EntityManagerFactory`;
- Атрибуты `<persistence-unit>`. В них определяется имя конкретного Unit'a - в следующем пункте мы разберемся, для 
  чего это нужно. Также есть малопонятный атрибут `transaction-type`. О нем тоже поговорим, но в следующих статьях.

В остальном полезная нагрузка конфигураций не отличается.

> Сейчас, видя много непонятных терминов и атрибутов, может показаться, что конфигурация ORM - нечто очень сложное.
> 
> На самом деле это не так, по крайней мере, в большинстве случаев. Практически каждый непонятный момент будет разобран
> в ближайших статьях - сейчас основной интерес представляют сами файлы конфигурации их общая структура. Потому что без 
> них тяжело показывать работу с ORM, а не выделяя их в отдельную статью - тяжело сформировать у читателя целостное 
> понимание их содержимого 

## Запуск приложения и проверка подключения к БД

В данном разделе демонстрации будут проходить на примере консольного приложения, чтобы не нагружать незнакомый для
читателя контекст еще и не так давно изученным Servlet API. Кардинальных различий в пределах курса это не принесет -
в практике и для самостоятельных экспериментов вы можете смело использовать сервлетные приложения, если хотите.

В качестве демонстрации успешного запуска попробуем создать и закоммитить транзакцию без изменений на уровне БД.

### Hibernate

```java
public class HibernateApplication {
//    Для лаконичности в примере опущена работа с closeable-ресурсами
    public static void main(String[] args) {
//        Создаем объект конфигурации и динамически загружаем hibernate.cfg.xml
        Configuration configuration = new Configuration()
                .configure();

//        На основе загруженной конфигурации создаем SessionFactory
        SessionFactory sessionFactory = configuration.buildSessionFactory();

//        Открываем сессию - высокоуровневую надстройку над транзакцией на уровне БД.
//        Этот интерфейс отвечает за наиболее значимую логику ORM.
        Session session = sessionFactory.openSession();

//        Получаем объект транзакции. Отражение реальной транзакции на уровне БД в Hibernate.
        Transaction transaction = session.getTransaction();

//        Открытие транзакции
        transaction.begin();
//        Коммит транзакции
        transaction.commit();
    }
}
```

Код все еще содержит много незнакомых интерфейсов, но знакомство с ними (точнее, их JPA-аналогов) - цель ближайших 
статей.

### JPA

```java
public class JpaApplication {
//    Для лаконичности в примере опущена работа с closeable-ресурсами
    public static void main(String[] args) {
//        Динамически загружаем persistence.xml,
//        по значению параметра (persistenceUnitName) определяем, кто выступает в роли PersistenceProvider
//        и создаем EntityManagerFactory через реализацию в этом PersistenceProvider.
//        В данном случае будет создан объект SessionFactoryImpl - основной реализации SessionFactory из Hibernate
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("Hibernate");

//        Получаем объект EntityManager. Именно этот объект контролирует взаимодействие
//        с реальной транзакцией на уровне БД и реализует наиболее значимую логику ORM.
//        В данном случае будет создан объект SessionImpl - основной реализации Session из Hibernate
        EntityManager em = emf.createEntityManager();

//        Получаем объект транзакции. Отражение реальной транзакции на уровне БД в JPA.
//        Простой интерфейс, понятный даже сейчас - открытие, коммит и откат транзакции.
//        Подобный API нам знаком еще по JDBC. Есть небольшие дополнения, но о них позже
        EntityTransaction transaction = em.getTransaction();

//        Открытие транзакции
        transaction.begin();
//        Коммит транзакции
        transaction.commit();
    }
}
```

Обратите внимание, что для создания объекта `EntityManagerFactory` мы используем имя Persistence Unit - то же, что 
указывали в `persistence.xml` в атрибуте `name` тега `<persistence-unit>`.

Само имя не регламентировано - мы можем указать его в свободной форме. Оно нужно лишь для маппинга с конкретным 
Persistence Provider'ом. В данном случае оно позволяет увидеть в конфигурации, что для получения 
`EntityManagerFactory` нужно использовать `HibernatePersistenceProvider`.

В качестве эксперимента можете указать иное имя или определить в конфигурации несколько Persistence Unit, связанных 
с различными БД (или даже СУБД). И попробовать найти через дебаггер, к Connection'ам каких драйверов (или к каким БД)
будут привязаны объекты `EntityManager`.

Еще одно небольшое уточнение относительно кода выше. Вы можете заметить, что в коде используются сокращения - `emf`, 
`em`. Если вы читаете эту статью, вероятно знаете, что различные аббревиатуры в именах переменных не приветствуются. 
Данные случаи можно считать одними из немногих массово используемых заключений.

Демонстрационное приложение можно найти здесь: [ссылка](https://github.com/KFalcon2022/jpa-sample).

#### На сегодня все!

![img.png](../../../commonmedia/justTheoryFooter.png)

> Если что-то непонятно или не получается – welcome в комменты к посту или в лс:)
>
> Канал: https://t.me/ViamSupervadetVadens
>
> Мой тг: https://t.me/ironicMotherfucker
>
> **Дорогу осилит идущий!**
