[Access control](https://portswigger.net/web-security/access-control)

## Вертикальная эскалация привелегий
### Незащищенная функциональность
[Unprotected admin functionality](https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality)
[Unprotected admin functionality with unpredictable URL](https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality-with-unpredictable-url)

### Методы управления доступом на основе параметров
Некоторые приложения определяют права доступа или роль пользователя при входе в систему и сохраняют эту информацию в контролируемом пользователем месте, например, в скрытом поле, файле cookie или заданном параметре строки запроса:

	`https://insecure-website.com/login/home.jsp?admin=true`
	`https://insecure-website.com/login/home.jsp?role=1`

[User role controlled by request parameter](https://portswigger.net/web-security/access-control/lab-user-role-controlled-by-request-parameter)
[User role can be modified in user profile](https://portswigger.net/web-security/access-control/lab-user-role-can-be-modified-in-user-profile)

### Нарушение контроля доступа в результате неправильной конфигурации платформы
Некоторые фреймфорки поддерживают различные нестандартные HTTP-заголовки, которые можно использовать для переопределения URL в исходном запросе, например:

	X-Original-URL
	X-Rewrite-URL
 
[URL-based access control can be circumvented](https://portswigger.net/web-security/access-control/lab-url-based-access-control-can-be-circumvented)

В лабе выше ограничение доступа основано на URL-адресе и методе HTTP, в случае же если можно использовать GET (или другой) для выполнения действий на ограниченном URL'е, то можно обойти контроль доступа, например, вместо:

	POST /admin-role
	...
	username=wiener&action=upgrade

Сделать:

	GET /admin-roles?username=wiener&action=upgrade

[Method-based access control can be circumvented](https://portswigger.net/web-security/access-control/lab-method-based-access-control-can-be-circumvented)

## Горизонтальная эскалация привелегий
[User ID controlled by request parameter](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter)
[User ID controlled by request parameter, with unpredictable user IDs](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-unpredictable-user-ids)
[User ID controlled by request parameter with data leakage in redirect](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-data-leakage-in-redirect)

### Переход от горизонтальной  эскалации привилегий к вертикальной
[User ID controlled by request parameter with password disclosure](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-password-disclosure)

### IDOR
[Insecure direct object references](https://portswigger.net/web-security/access-control/idor)

**IDOR** возникает, когда приложение использует пользовательский ввод для прямого доступа к объектам, а злоумышленник может изменить этот ввод для получения несанкционированного доступа.

**IDOR с прямым обращением к объектам базы данных**
Здесь номер клиента используется непосредственно в качестве индекса записи в запросах, выполняемых к внутренней базе данных. Если нет других средств контроля, можно изменить значение `customer_number`, обойдя средства контроля доступа:

	`https://insecure-website.com/customer_account?customer_number=132355`

**IDOR с прямым обращением к статическим файлам**
Возникают, когда конфиденциальные ресурсы находятся в статических файлах в файловой системе на стороне сервера. Например, если веб-сайт может сохранять расшифровки сообщений чата на диск, используя увеличивающееся имя файла, и позволять пользователям получать их, посетив URL-адрес, то можно изменить имя файла, чтобы получить расшифровку, созданную другим пользователем:

	`https://insecure-website.com/static/12144.txt`

### Уязвимости управления доступом в многоступенчатых процессах
[Multi-step process with no access control on one step](https://portswigger.net/web-security/access-control/lab-multi-step-process-with-no-access-control-on-one-step)

### Контроль доступа на основе заголовка Referer
Например, приложение обеспечивает надежный контроль доступа к главной административной странице `/admin`, но для вложенных страниц, таких как `/admin/deleteUser`, проверяет только заголовок `Referer`. Если заголовок `Referer` содержит основной URL `/admin`, то запрос разрешен.

[Referer-based access control](https://portswigger.net/web-security/access-control/lab-referer-based-access-control)

### Контроль доступа на основе местоположения
Некоторые веб-сайты применяют контроль доступа к ресурсам на основе географического местоположения пользователя. Это может применяться, например, в банковских приложениях или медиа-сервисах, где действует государственное законодательство или ограничения на ведение бизнеса. Такие средства контроля доступа часто можно обойти, используя веб-прокси, VPN или манипулируя механизмами геолокации на стороне клиента.


