[Information disclosure](https://portswigger.net/web-security/information-disclosure)

[Exploiting](https://portswigger.net/web-security/information-disclosure/exploiting)
## Общие источники раскрытия информации
Ниже приведены некоторые общие примеры мест, где можно проверить.
**Файлы для веб-краулеров**
Например, `/robots.txt` или `/sitemap.xml`

- **Листинги каталогов**
	- DIRB
	- dirsearch
- **Комментарии разработчиков**
- **Сообщения об ошибках**
- **Данные для отладки**
- **Страницы учетных записей пользователей**
- **Файлы резервного копирования**
- **Небезопасная конфигурация**
	- http trace
- **История контроля версий**
	- GitTools
- **Burp's engagement tools**
	- Search
	- Find comments
	- Discover content