# Laboratorio Pingpong

## Descripción
Writeup del laboratorio **Pingpong** de DockerLabs - Dificultad: Medium

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

Originalmente se  usaría -T1 (Sneaky) pero por motivo de optimización se usó -T5

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

