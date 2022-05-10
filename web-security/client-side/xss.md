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

## DOM-based XSS
