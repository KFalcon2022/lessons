# Servlet API. Cookies. Сессия. Аутентификация в сервлетном приложении

В данной статье познакомимся с API для работы с cookies и сессиями в сервлетных приложениях. Также рассмотрим, как
может быть реализована аутентификация на базе сессий с точки зрения кода. В том числе постараемся разобраться, на
каких уровнях приложения стоит располагать ту или иную функциональность.

## Cookie API

Работа с Cookies в сервлетных приложениях достаточно проста. Есть методы для чтения cookies из запросов, добавления
cookies в ответах и, наконец, сам класс `Cookie` для взаимодействия с одним конкретно взятым cookie.

Технически, можно обойтись и без отдельного API данных методов - в конце концов, взаимодействие с cookies в
HTTP-коммуникации сводится к добавлению соответствующих заголовков в запросе и ответе. Но такой формат очевидно не
удобен - придется вручную получать данные заголовка `Cookie` (или добавлять заголовки `Set-Cookie`), парсить
строковые данные во что-то более удобное для работы и т.д. Чтобы избежать этих однообразных операций и выделен более
дружелюбный API.

Метод `HttpServletRequest`:

- `getCookies()`. Возвращает `Cookie[]`, переданные в текущий запрос. Каждый элемент - конкретная `cookie`, со своим
  именем и значением. В этом же массиве будет лежать и идентификатор сессии, если таковая есть.

Метод `HttpServletResponse`:

- `addCookie(Cookie cookie)`. Добавляет в ответ объект `Cookie`, переданный параметром. В конечном итоге каждый
  такой объект будет представлен в HTTP-ответе в виде отдельного заголовка `Set-Cookie`, содержащем имя и значение
  cookie из объекта, ее атрибуты (при наличии).

Класс `Cookie`, выборочно:

- `Cookie(String name, String value)` (Конструктор). Очевидно, параметрами передается имя cookie и ее значение;
- `getName()`, `getValue()`, `setValue(String newValue)` - геттеры и сеттер для имени и значения. Имя cookie
  ниезменяемо, что логично;
- `setAttribute(String name, String value)`. Добавляет для cookie атрибут, который будет передан в заголовке
  `Set-Cookie`. Ранее мы упоминали лишь атрибуты  `Secure` и `HttpOnly`, связанные с безопасностью хранения и
  передачи конкретной cookie. Существуют и другие атрибуты, также есть возможность добавлять свои собственные при
  необходимости;
- `getAttribute(String name)`, `getAttributes()`. Позволяет получить строковые значения атрибутов cookie. Либо по
  имени, либо все сразу в виде `Map`;
- `getMaxAge()`, `setMaxAge(int expiry)`, `setSecure(boolean flag)`, `getSecure()` и другие. Для наиболее
  получения значений и установки распространенных атрибутов выделены отдельные методы. Это позволяет не заниматься
  ручным приведением к строке и не углубляться в тонкости установки атрибутов без значений. Отдельные методы
  выделены для атрибутов  `Domain`, `Max-Age`, `Path`, `Secure`, `HttpOnly`. Подробнее об их назначении можно
  почитать самостоятельно или отложить до момента, когда это сможет пригодиться на практике. В общем случае, эти
  атрибуты позволяют настроить область видимости, время жизни, правила передачи и использования cookie.

В целом, взаимодействие с Cookie достаточно примитивно - сама механика очень проста. Место чтения или установки
cookie всегда зависит от бизнес-смысла. Чаще всего это будет где-то в логике фильтров, но в отдельных случаях,
особенно, если конкретная cookie специфична для определенного пути (определяется атрибутом `Path`) - вполне возможна
ситуация, когда взаимодействие с ней будет происходить внутри логики конкретного сервлета.

## Session API

Работа с сессиями немного более разнообразна, хоть и остается достаточно легкой в техническом плане.

Разнообразие диктуется назначением сессий. По сути, это единственный предоставляемый Servlet API механизм,
позволяющий хранить данные, необходимые для авторизации пользователей. То есть лишь в атрибутах сессии можно хранить
данные, которые позволят, например, определить идентификатор, роль пользователя или иную информацию, необходимую для
определения прав пользователя на конкретный ресурс. Любой иной подход потребует подключать сторонние инструменты для
этих целей или изобретать собственный велосипед. Таким образом, работа с сессиями - это не только обработка
соответствующей cookie и прочие действия вокруг HTTP, но и API для конфигурации логики аутентификации, и API для
управления авторизационными данными.

### Java API

С точки зрения Java API нас интересуют методы `HttpServletRequest`, интерфейсы `HttpSession` и `SessionCookieConfig`.

#### HttpServletRequest

`HttpServletRequest` предоставляет как методы для получения объекта сессии, так и информационных методов:

- `HttpSession getSession(boolean create)`. Метод для получения объекта текущей сессии. В случае, если сессия по
  переданному в запросе id существует - возвращает ее, если нет - смотрит на значение параметра. При `true` -
  создает новый объект сессии, при `false` - возвращает `null`;
- `HttpSession getSession()`. Перегрузка, аналогичная `getSession(true)`;
- `String changeSessionId()`. Меняет id текущей сессии на новый. При этом сам объект сессии остается тем же, в том
  числе сохраняются все его атрибуты. Данный метод предоставляет дополнительный слой защиты от злоумышленников, которые
  могут перехватить идентификатор сессии. Технически, при использовании данного метода id сессии можно менять с
  заданной частотой или даже на каждый запрос пользователя;
- `String getRequestedSessionId()`. Возвращает id сессии, переданный в запросе;
- `boolean isRequestedSessionIdValid()`. Проверяет, валиден ли id сессии, переданный в запросе. Иными словами,
  проверяет, существует ли сессия по указанному id. Ведь технически никто не мешает передать с клиента случайный
  идентификатор. Если аутентификация будет происходить по факту наличия идентификатора, а не сессии - такой ход
  приведет к успешному взлому. Данный метод позволяет защитить систему от подобного;
- `boolean isRequestedSessionIdFromCookie()`. Проверяет, был ли id сессии передан через cookie. Хоть такой подход и
  является стандартным, он не единственный возможный;
- `boolean isRequestedSessionIdFromURL()`. Проверяет, были ли id сессии передан через параметры запроса. Этот способ
  считается менее безопасным, но он может быть полезен, если пользователь заблокировал cookies на сайте.

Важным моментом является то, что любые манипуляции с сессией - ее создание, обновление id - не требуют ручного
добавления cookie в ответе. Это произойдет автоматически. В целом, при стандартном подходе разработчику вообще не
приходится работать с cookie `JSESSIONID` напрямую.

#### HttpSession

Следующий на очереди - `HttpSession`. Это интерфейс, являющийся репрезентацией фактической сессии. Большая часть его
API - достаточно типовая, с прослеживающимися аналогиями относительно других ключевых объектов Servlet API. Мы
рассмотрим практически все методы, исключив лишь те (фактически, тот), которые бесполезны с точки зрения понимания
логики взаимодействия:

- `String getId()`. Получения идентификатора сессии;
- `ServletContext getServletContext()`. Полагаю, комментарии излишни.
- `getAttribute(String name)`, `getAttributeNames()`, `setAttribute(String name, Object value)`,
  `removeAttribute(String name)`. Стандартный набор для работы с атрибутами. Атрибуты доступны в пределах сессии,
  именно на этом можно строить модель авторизации, сохраняя в атрибуты данные о пользователе;
- `void setMaxInactiveInterval(int interval)`, `int getMaxInactiveInterval()`. Установка и получение информации о
  времени жизни сессии. Установка имеет смысл при необходимости установить нестандартное значение, отличающееся от
  принятого в рамках приложения. Для определения общей конфигурации есть более удобные способы - о них ниже;
- `boolean isNew()`. Возвращает `true`, если текущая сессия была создана в рамках данного запроса. Т.е. клиент еще
  не знает id данной сессии;
- `void invalidate()`. Инвалидация сессии. В результате вызова этого метода будут удалены все атрибуты сессии. Можно
  рассматривать как механизм удаления самой сессии - технически это не так, но для пользователя будет иметь именно
  такой эффект. Обычно используется для логаута;
- `long getCreationTime()`. Возвращает время создания сессии в виде количества миллисекунд от начала UNIX-эпохи.
  Практическую ценность несет, в первую очередь, для построения метрик;
- `long getLastAccessedTime()`. Возвращает время, когда был произведен последний запрос в рамках текущей сессии.
  Фактически, по этому признаку можно отсчитывать переход пользователя в оффлайн-режим. Как и предыдущий метод,
  может быть полезен для построения метрик.

#### SessionCookieConfig

Интерфейс `SessionCookieConfig` представляет API для настройки cookie, через которую будет передаваться id сессии
клиенту. По большей части аналогичен методам работы с атрибутами в классе `Cookie`, единственное интересное отличие -
метод `setName(String name)`, позволяющий определить собственное название cookie для передачи сессии, взамен
`JSESSIONID`. Но практической ценности это не несет.

Большее значение имеет время использования данного интерфейса. Фактически, любые set-методы в нем допустимо вызывать
лишь до того, как завершится инициализация `ServletContext`. В остальное время `SessionCookieConfig` доступен только
для чтения, любая попытка изменения приведет к исключению. В целом, вполне логичное ограничение - ведь сами сессии
находятся под управлением контейнера сервлетов, изменение настроек в момент, когда приложение способно обрабатывать
запросы, чревато ошибками обработки или не очевидным для клиента поведением.

Вторая важная особенность `SessionCookieConfig` - его можно конфигурировать в `web.xml`.

#### Слушатели событий

Servlet API предоставляет ряд интерфейсов-слушателей, завязанных на действия с сессиями. Но данные интерфейсы не
несут пользы в контексте знакомства с сессиями, а шанс применить их на практике достаточно мал. Поэтому
останавливаться на них мы не будем.

### web.xml

В дескрипторе развертывания можно сконфигурировать работу с сессиями в декларативной форме, не разбрасывая
конфигурацию по различным Java-классам. Вся конфигурация будет инкапсулирована внутри атрибута `<session-config>`,
расположенного внутри `<web-app>`:

```xml

<session-config>
    <!--  Время жизни сессии в минутах. По истечении срока сессия будет удалени Значение 0 означает бессрочную сессию-->
    <session-timeout>30</session-timeout>
    <!--  Атрибуты конфига для cookie сессии. Представление SessionCookieConfig в xml-->
    <cookie-config>
        <http-only>true</http-only>
        <secure>true</secure>
        <!--          Пример добавления собственного атрибута-->
        <attribute>
            <attribute-name>myAttribute</attribute-name>
            <attribute-value>myAttributeValue</attribute-value>
        </attribute>
    </cookie-config>
    <!--  Способ передачи id сессии. Можно указать одновременно несколько вариантов из предложенных: COOKIE, ERL, SSL-->
    <tracking-mode>COOKIE</tracking-mode>
</session-config>
```

Фактически, настройки сессии обычно достаточно стандартны, на практике зачастую ограничиваются лишь указанием
`<session-timeout>`, в зависимости от требований к конкретному проекту.

## Аутентификация и авторизация по сессии

На данный момент у нас хватает информации, которая позволит реализовать аутентификацию на базе сессий. В базовом виде,
все можно свести к сервлетам, ответственным за логин и логаут, и фильтру, который будет обеспечивать авторизацию -
проверку прав на доступ. Примеры ниже упрощены для лаконичности.

### Логин

```java

@WebServlet("/login")
public class LoginServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        // Находим пользователя по username, переданному в параметрах запроса.
        //  На практике стоит использовать тело запроса
        String username = request.getParameter("username");
        String user = userService.getByUsername(username);

        // Если пользователя с указанным юзернеймом не существует - выбрасываем ошибку. Способ пробрасывания ошибки 
        //  зависит от используемых средств обработки ошибок
        if (user == null) {
            response.sendError(401, "Неверный логин или пароль");
            return;
        }

        // Хешируем переданный пользователем пароль и сравниваем с хешем настоящего пароля. Если не совпадают - 
        //  выбрасываем ошибку
        String password = request.getParameter("password");
        String encodedPassword = passwordEncoder.encode(password);

        if (!encodedPassword.equals(user.getPassword())) {
            response.sendError(401, "Неверный логин или пароль");
            return;
        }

        // По сути, на этом этапе аутентификация прошла успешно. Создаем сессию, устанавливаем необходимые для 
        //  дальнейшем работы атрибуты

        HttpSession session = request.getSession();
        session.setAttribute("userId", user.getId());

        // В упрощенном примере использовать не будем, но в более реальной системе - имеет право на жизнь
        session.setAttribute("role", user.getRole());
    }
}
```

### Логаут

```java

@WebServlet("/logout")
public class LogoutServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {

        // Находим сессию. Если таковая существует, инвалидируем ее 
        HttpSession session = request.getSession(false);

        if (session != null) {
            session.invalidate();
        }

        // Здесь могут быть дополнительные действия - какие-то сайд-эффекты в серверной логике или формирование 
        //  сообщения пользователю. Или редирект на страницу авторизации, если наше приложение управляет HTML или JSP 
        //  страницами. Если для этого используется отдельное клиентское приложение или графический интерфейс 
        //  отсутствует - скорее всего хватит успешного кода ответа.  
    }
}
```

### Авторизация

```java

@WebFilter
public class AuthorizationFilter extends HttpFilter {
    @Override
    protected void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        // Проверяем путь запроса. Более корректно ограничить запросы, на которые отрабатывает этот фильтр, в 
        //  конфигурации. Или же сформировать список путей, доступных без авторизации, и проверять по нему. Но для 
        //  примера пойдет. В данном случае получается, что без авторизации доступна только страница логина. Вопрос
        //  регистрации остается открытым:)
        if ("/login".equals(req.getServletPath())) {
            // Если запрос на логин - пускаем дальше по цепочке без дополнительных проверок
          super.doFilter(request, response, chain);
        }

        // Получаем объект сессии. Если сессии не существует - отправляем ошибку. 
        HttpSession session = request.getSession(false);
        
        if (session == null) {
          response.sendError(401);
          return;
        }

        // Если сессия есть - пользователь авторизован, можно пропускать его дальше. Такое возможно, если далее еще 
        // будет проверка с учетом роли или иных данных. Или если любому авторизованному пользователю доступны все 
        // ресурсы в системе, что бывает намного реже. Наиболее простой шаг к улучшению текущей реализации - определить 
        // возможные роли и список путей, доступных различным ролям. И проверять в данном фильтре роль пользователя на 
        // доступность для нее пути текущего запроса. По сути - развитие идеи, примененной выше для '/login'
        super.doFilter(request, response, chain);
    }
}
```

В текущем подходе важно, что сессия может быть создана лишь при логине. Если это не так и в приложении допустимо 
создание сессии для не авторизованного пользователя - скажем, для сбора метрик или работы с какой-то гостевой
функциональностью, то в фильтре выше надо в обязательном порядке проверять атрибуты, которые будут гарантировать, что
отправитель запроса авторизован. С другой стороны, это мало отличается от создания ролевой модели, описанной в 
комментариях к фильтру.

## Заключение

На этом мы заканчиваем блок знакомства с механизмами аутентификации в сервлетных приложениях. На мой взгляд, 
рассмотренный механизм достаточно прост, что позволяет его имплементировать даже новичкам. Со временем вы 
познакомитесь и поработаете с более продвинутыми подходами - возможно, даже с более продвинутыми реализациями 
сессионных механизмов, они тоже существуют.

В следующий раз к напрямую к теме аутентификации мы вернемся лишь в Spring Security, практически в самом конце курса.

Как направление для самостоятельного изучения рекомендую познакомиться с моделями аутентификации по токенам - они 
достаточно популярны в современной разработке. Также имеет смысл глубже изучить OAuth 2.0, описанный в предыдущей 
статье. Остальные подходы менее распространены и в силу этого менее актуальны на начальных этапах. 

#### С теорией на сегодня все!

![img.png](../../../commonmedia/defaultFooter.jpg)

Переходим к практике:

## Задача

Доработайте задачу, реализованную в рамках статей
[147](https://github.com/KFalcon2022/lessons/blob/master/lessons/web-and-java-ee/147/Servlet%20API.%20ServletConfig.%20ServletContext.%20Listeners.md)
и
[148](https://github.com/KFalcon2022/lessons/blob/master/lessons/web-and-java-ee/148/Servlet%20API.%20Filter.%20Filter%20chain.%20Request%20and%20response%20modification.md).

Необходимо добавить функциональность регистрации, аутентификации и авторизации пользователей:

1. Создайте сущность `User` со свободным набором атрибутов. В базовом сценарии можно ограничиться логином и паролем.
   Обеспечьте хранение информации о пользователях в БД;
2. Пароль в БД должен храниться в виде хеша. Реализовать хеширование можно как с использованием одного из алгоритмов,
   реализованных в `java.security`, так и с использованием внешней библиотеки на ваш вкус. В наиболее простой
   реализации в качестве хеша можно использовать результат `Object#hashcode()`;
3. Реализуйте сервлет, ответственный за регистрацию пользователей. Запрос должен содержать все необходимые для
   регистрации атрибуты пользователя, включая пароль. Если пользователь с указанным логином существует - необходимо
   возвращать ошибку;
4. Реализуйте сервлет, ответственный за логин пользователя. Должен принимать логин и пароль, в случае, если
   пользователь с указанным логином не существует или пароль неверный - возвращать сообщение о неудачном логине;
5. Реализуйте сервлет, ответственный за логаут пользователя;
6. Доступ к ранее созданный сервлетам должен быть ограничен и предоставляться только авторизованным пользователям.
   В противном случае должен возвращаться код ответа `401 (Unauthorized)`.
7. Любая полезная нагрузка в теле запроса или ответа должна передаваться в виде JSON-объектов.

Ветка для PR: [for-pr](https://github.com/KFalcon2022/car-servlet-practical-task/tree/for-pr/auth).

> Если что-то непонятно или не получается – welcome в комменты к посту или в лс:)
>
> Канал: https://t.me/ViamSupervadetVadens
>
> Мой тг: https://t.me/ironicMotherfucker
>
> **Дорогу осилит идущий!**