# Servlet API. Знакомство с web.xml

В данной статье мы познакомимся с основным файлом конфигурации сервлетных приложений - web.xml. В процессе 
придется затронуть большую часть управляющих и вспомогательных конструкций Servlet API, которые так
или иначе были упомянуты в статье, посвященной жизненному циклу. Мы пока не будем разбирать особенности использования
самих этих конструкций, но подготовим почву для их эффективной конфигурации к моменту изучения API. Это кажется
возможным, поскольку их роль в системе была рассмотрена ранее, что дает базовое понимание сферы применения.

## Назначение

`web.xml` (deployment descriptor, дескриптор развертывания) - ключевой файл конфигурации в сервлетных приложениях. Он 
позволяет определить в одном месте практически все необходимые для веб-приложения настройки:

- Название и описание приложения. Сюда же можно отнести и ряд других общих параметров - от прикладных до
  информационных, вплоть до установки favicon. Но специфика обработки некоторых параметров может отличаться от
  используемого контейнера - в т.ч. не поддерживаться именно формат установки через `web.xml`;
- Описание других параметров контекста. Таким образом можно передавать различные параметры конфигурации приложения,
  которые должны быть доступны на глобальном уровне. В этом пункте может быть множество оговорок относительно
  корректности такого подхода и зоны применения этих параметров в каждом конкретном случае, но они выходят за пределы
  обзора возможностей `web.xml`
- Декларация сервлетов и маппингов для них. В этом случае описание в `web.xml` заменяет собой использование
  `@WebServlet`. Также можно передать и какие-либо параметры, специфичные для конкретных сервлетов;
- Фильтры, слушатели и их параметры. От правил применения (например, фильтры только для конкретных путей) до порядка
  применения;
- Поведение по умолчанию. Например, установка сервлета, который обрабатывает запросы, если нет подходящего маппинга.
  Или страницу, которую сервер будет отдавать при возникновении ошибок;
- Настройки авторизации и безопасности. От ограничения доступа к конкретным ресурсам сервера до конфигурации сессии -
  например, ее времени жизни.

Описание всех описанных параметров в одном месте упрощает поддержку глобальной конфигурации системы - разработчик
всегда знает, где найти определенный набор информации о приложении. С другой стороны, в больших проектах и `web.xml`
может сильно разрастись, что в итоге усложнит поддержку такой конфигурации. Но последнее является бичом почти любых
больших файлов - от конфигурации и скриптов сборки (например, `build.gradle`) до Java-классов.

## Расположение

`web.xml` располагается в проекте (в случае с Gradle) по пути `${projectHome}/src/main/webapp/WEB-INF`. Первые
несколько директорий нам хорошо известны, на оставшихся остановимся подробнее:

- `webapp`. Отвечает за хранение ресурсов, в первую очередь статических, которые необходимы сервлетному приложению:
  HTML-страницы*, CSS (стили для HTML), файлы JavaScript, картинки и другая мультимедиа. В современной разработке эти
  данные принято организовывать иначе, но для чисто сервлетного приложения использовался бы именно такой подход;
- `WEB-INF`. Кроме `web.xml` здесь могут храниться и другие данные, в т.ч. конфигурация для подключенных в проект
  библиотек. Еще несколько кандидатов на помещение в `WEB-INF` мы рассмотрим в следующих статьях текущего раздела.

> *Сюда же относится и JSP - некое подобнее HTML с интеграцией Java-кода. В текущем разделе мы поверхностно
> познакомимся с этой технологией - она практически не используется в современных проектах, но является важной вехой
> в развитии веб-разработки на Java.

При рассмотрении `WEB-INF` очень важно различать данную директорию в проекте - то, что описано выше, и ее тезку в
собранном сервлетном приложении. Так, если зайти в `${tomcatHome}/webapps/${someApp}/WEB-INF` можно обнаружить там
много интересного:

- Все тот же `web.xml` и другие файлы, помещенные в `WEB-INF` проекта;
- Директорию, в которой хранятся скомпилированные Java-классы для данного приложения. В ту же директорию будут помещены
  файлы, расположенные в `${projectHome}/src/main/webapp/resources`;
- Директорию, в которой хранятся все библиотеки, используемые в проекте в виде jar-файлов. Кроме тех, которые
  поставляются контейнером сервлетов. Т.е. именно в `WEB-INF` можно будет найти Jackson, HikariCP и любые иные
  библиотеки, которые были подключены в проект. И именно оттуда они будут загружаться контейнером сервлетов;
- Ряд файлов связанных с JSP, если он используется в приложении.

## Теги

Как и другие XML-файлы, `web.xml` конфигурируется с помощью набора тегов, характерных именно для этого вида
документов. Для конкретного контейнера сервлетов набор используемых тегов и атрибутов тегов может несколько
отличаться, но основной перечень совпадает.

К сожалению, полного списка допустимых тегов в удобном для чтения виде я не нашел, поэтому для справки прилагаю
несколько источников, которые можно использовать для справки. Также ниже будет разбор наиболее востребованных тегов:

1. [Ссылка](https://docs.oracle.com/cd/E17904_01/web.1111/e13712/web_xml.htm). Документация для Oracle WebLogic
   Server. Наиболее удобный ресурс с точки зрения формата, на мой взгляд. Проблема в том, что это документация под
   конкретный сервер приложений, поэтому некоторые атрибуты специфичны именно для этого сервера, а для части тегов нет
   описания, лишь ремарка, об отсутствии поддержки со стороны этого конкретного сервера;
2. [Ссылка](https://jakarta.ee/specifications/servlet/6.1/jakarta-servlet-spec-6.1). Спецификация Jakarta Servlet
   последней версии. Хорошая подача с точки зрения изучения, но нет именно формата справочника. Но здесь слишком много
   упора на специфику в виде `web-fragment.xml`. И нет описания тегов именно в формате справочника - лишь описание 
   использования в контексте использования конкретного инструментария. Но самое важное - это первоисточник. Почти
   все, что можно найти сверх - лишь производные от этой или другой версии спецификации; 
3. Также можно опираться на схемы, которые указаны в шапке внутри самого `web.xml` - они точно содержат набор
   допустимых тегов и документацию для них. Но не слишком удобны с точки зрения человекочитаемости и зачастую 
   требуют перехода по ссылкам внутри одного документа. Например: 
   [ссылка](https://jakarta.ee/xml/ns/jakartaee/web-common_6_1.xsd).

## Пример

Чтобы иметь перед глазами практический пример, предлагаю рассмотреть следующий проект: 
[ссылка](https://github.com/KFalcon2022/servlet-sample/tree/web-xml).

Он не является показательным и рекомендуемым с позиции использованных подходов в части Java-кода, но позволяет 
продемонстрировать базовые возможности `web.xml`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--Корневой тег. Именно в нем через указание схемы определяется набор допустимых тегов-->
<web-app
        xmlns = "https://jakarta.ee/xml/ns/jakartaee"
        xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
<!--        Используемая схема. В ней можно найти перечень тегов, их атрибутов и документацию. Не очень удобно, но 
исчерпывающе и гарантированно актульно для конкретной версии спецификации-->
        xsi:schemaLocation = "https://jakarta.ee/xml/ns/jakartaee
        https://jakarta.ee/xml/ns/jakartaee/web-app_6_1.xsd"
        version = "6.1">
<!--Название и описание приложения. Как правило несет информационную функцию-->
    <display-name>My sample app</display-name>
    <description>The description for my sample app</description>
    
<!--Регистрация слушателей. Сводится к указанию класса каждого конкретного слушателя. Порядок, в котором указаны 
слушатели в файле определяет порядок их загрузки при старте приложения. Это может иметь значение, если работа 
определенных слушателей зависит от результата работы других слушателей-->
    <listener>
        <listener-class>com.walking.sample.listener.InitParamsListener</listener-class>
    </listener>

<!--Фильтры.-->
    <filter>
<!--    Декларация фильтра происходит с указанием имени фильтра - оно еще потребуется для идентификации ниже-->
        <filter-name>CommonRequestCounterFilter</filter-name>
<!--Класс фильтра. В первую очередь мы указываем, какие классы фильтров есть и даем им внутренние имена 
- они могут быть использованы как в web.xml, так, при необходимости, и в Java-коде-->
        <filter-class>com.walking.sample.filter.CommonRequestCounterFilter</filter-class>
<!--При необходимости можно задать параметры инициализации фильтров - данные, которые будут доступны во внутренней 
конфигурации саомго фильтра. Можно указать любые строковые пары ключ-значение-->
        <init-param>
            <param-name>someParam</param-name>
            <param-value>paramValue</param-value>
        </init-param>
    </filter>
    <filter>
        <filter-name>HelloWorldRequestCounterFilter</filter-name>
        <filter-class>com.walking.sample.filter.HelloWorldRequestCounterFilter</filter-class>
    </filter>
    <filter>
        <filter-name>InvalidPathRequestCounterFilter</filter-name>
        <filter-class>com.walking.sample.filter.InvalidPathRequestCounterFilter</filter-class>
    </filter>

<!--Сервлеты-->
    <servlet>
    <!--Декларация сервлета почти идентична декларации фильтра - имя, класс, параметры инициализации и ряд других 
    второстепенных атрибутов при необходимости. Как и для фильтра, обязательными являются лишь имя и класс-->
        <servlet-name>HelloWorldServlet</servlet-name>
        <servlet-class>com.walking.sample.servlet.HelloWorldServlet</servlet-class>
    </servlet>
    <servlet>
        <servlet-name>StartPageServlet</servlet-name>
        <servlet-class>com.walking.sample.servlet.StartPageServlet</servlet-class>
        <init-param>
            <param-name>knownPath</param-name>
            <param-value>/hello-world</param-value>
        </init-param>
    </servlet>

<!--Маппинги фильтров - описание, какие фильтры должны применяться для каких путей запросов и/или сервлетов.
Порядок описанных маппингов в файле имеет значение для порядка обработки, как и у слушателей. Но алгоритм определения 
порядка сложнее, поэтому будет разобран отдельно, в соответствующей статье-->
    <filter-mapping>
        <filter-name>CommonRequestCounterFilter</filter-name>
<!--        Данный атрибут может описывать конкретный путь или маску (шаблон) пути. При соответствии пути запроса 
тому, что указано в url-pattern, фильтр будет применен до и после обработки запроса сервлетом-->
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <filter-mapping>
        <filter-name>HelloWorldRequestCounterFilter</filter-name>
<!--        Альтернативно можно привязать фильтр к конкретному сервлету, вместо привязки по url-pattern-->
        <servlet-name>HelloWorldServlet</servlet-name>
    </filter-mapping>
    <filter-mapping>
        <filter-name>InvalidPathRequestCounterFilter</filter-name>
<!--        При желании можно внутри маппинга фильтра указать несколько url-pattern и/или servlet-name. Тогда фильтр 
будет привязан ко всем указанным маскам путей и сервлетам-->
        <servlet-name>StartPageServlet</servlet-name>
<!--    Стоит отметить, что несколько фильтров могут быть привязаны к одним и тем же путям или сервлетам, это 
нормальная практика.-->
    </filter-mapping>

<!--Маппинг сервлетов - альтернатива путям, которые были ранее указаны через @WebServlet-->
    <servlet-mapping>
        <servlet-name>HelloWorldServlet</servlet-name>
        <url-pattern>/hello-world</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>StartPageServlet</servlet-name>
<!--        В отличие в фильтров, на один путь может быть замаплен только один сервлет, что логично. Если есть 
несоклько масок, которые подходят под конкретный путь, будет использоваться наиболее близкая. Таким образом, 
подходящая под все пути маска /* будет иметь наименьший приоритет и использоваться только если не подошла никакая 
иная-->
        <url-pattern>/*</url-pattern>
    </servlet-mapping>

<!--Исключительно демонстрационный блок, показывающий, что в случае ошибок можно отдавать не результат работы 
сервлета, а специальные страницы (HTML или JSP). Или отдельные классы-обработчики. В рассматриваемом приложении 
описанные ниже страницы недостижимы по ряду причин. Подробнее эту тему рассмотрим ближе к завершению раздела-->
    <error-page>
        <error-code>404</error-code>
        <location>/404.html</location>
    </error-page>
    <error-page>
        <error-code>500</error-code>
        <location>/500.html</location>
    </error-page>

</web-app>
```

## Заключение

Рассмотренная сегодня тема - краеугольная для сервлетных приложений. Тем не менее несложно заметить, что ее 
рассмотрение больше подходит под обзор, без разбора практик использования, типовых ошибок и подобного.

Связано это в первую очередь с тем, что сервлетные приложения, в их привычном понимании, ушли в прошлое.

Но изучение сервлетов - важный шаг к пониманию того, как работают современные серверные приложения на Java. 
Большинство из них не используют сервлеты на прямую, но все еще строятся на инфраструктуре, которую предоставляет 
Servlet API и контейнеры сервлетов. Кроме того, даже в современных приложениях используются некоторые инструменты 
Servlet API - например, фильтры. И, безусловно, остаются актуальны многие концепции, которые были заложены именно 
сервлетными приложениями.

Поэтому эта и следующие статьи будут несколько различны по фокусам. В тех случаях, когда сам механизм имеет 
прикладное значение, будет больше деталей о сценариях использования и особенностях работы. Там, где важна сама 
концепция - фокус будет на ней, а API и технические аспекты будут рассматриваться фоном.

Так или иначе, основная цель изучения Servlet API - знакомство с базисом, на котором строится работа современных 
фреймворков для разработки серверных приложений вроде Spring MVC. Именно из спецификации сервлетов и проблем, 
которые она несет, вытекает актуальность современных решений. И из этих же проблем или попыток их решить рождается 
набор инструментов, который используется сегодня.  

#### На сегодня все!

![img.png](../../../commonmedia/justTheoryFooter.png)

> Если что-то непонятно или не получается – welcome в комменты к посту или в лс:)
>
> Канал: https://t.me/ViamSupervadetVadens
>
> Мой тг: https://t.me/ironicMotherfucker
>
> **Дорогу осилит идущий!**
