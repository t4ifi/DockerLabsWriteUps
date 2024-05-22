# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/6348f6ca-627c-4c18-85ed-b0651440783b)
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

1. **80/tcp (HTTP):** Apache httpd 2.4.52 (Ubuntu)
### Puerto 80

Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Al acceder al puerto 80, se muestra una página que permite la subida de archivos. Esto podría ser potencialmente peligroso si se encontrara el directorio donde se guardan los archivos ya que podríamos colar un archivo malicioso.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/89eb5c62-7c3e-48fa-95b8-8369df5aecbe)

# Fase 2- Exploracion

#### Archivo php

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/d9725446-7aa2-40fe-bf50-1efcd0408b6f)
El archivo que vamos a intentar colar va a ser un archivo que es una revershell en php. Por dentro el script le debemos poner nuestra ip y el puerto por el que vamos a recibir la revershell.


![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ead52f94-7c3c-4721-8b72-0a608c1dfa7f)
Se subio correctamente, el siguiente paso es encontrar el directorio donde se almacenan los archivos con la herramienta Gobuster.


#### Gobuster

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/cb3e42e0-2e8a-4006-abba-55d43eef4871)
Con la ayuda de Gobuster haciendo un poco de fuzzing encontramos un directorio llamado "uploads"

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/6d3ae363-d4d2-48ce-a612-dc94f1399bc5)
Entramos dentro y ahí esta el archivo .php que subimos anteriormente.

# Fase 3- Explotación
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/1a48b6aa-807f-448f-a450-211e31bd348d)
Al ejecutar el archivo malicioso que colamos estamos dentro. Ahora a escalar privilegios :)

# Fase 4- Privilegios

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/746f3231-6a4b-4a78-b900-fc220b65822c)
Después de hacer el tratamiento de la TTY y ejecutar el comando `sudo -l` vemos que tenemos permisos de ejecutar el binario `env`.

#### GTFOBINS
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ee4a5abb-9164-444e-ae23-6cb37c7b0c4c)
Nos dirigimos a la pagina gtfobins y encontramos el binario `env` y nos dice que para escalar solamente hay que poner el comando `sudo env /bin/sh`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/774f1357-6820-4fb3-9378-7912f49cb86d)
Lo probamos y funciono, estamos dentro como el usuario root!
