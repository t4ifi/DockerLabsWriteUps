# Fase 1- Tanteo

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/334f82bb-4707-4a58-843a-5612c3a172dc)
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

1. **20/tcp(SSH):** OpenSSH 9.6p1 (Ubuntu)
2. **80/tcp(HTTP)** Apache httpd 2.4.58

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a acceder a una pagina por defecto de Apache Ubuntu.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/f8cd2d4a-bf18-46b7-a671-e5be4aacca45)

# Fase 2- Exploración

#### index.php
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/6856b803-f752-43b2-8b2c-5104e10e4c2c)
Realizando fuzzing con Feroxbuster encontramos un directorio llamado `index.php` que contenía una cadena de caracteres.

# Fase 3- Explotación

#### ssh
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/0f102627-5ad1-44eb-9da0-488b87dde5dc)
Como no encontramos nada mas, probé hacer fuerza bruta al protocolo ssh dándole esa cadena como contraseña y un diccionario de usuarios, y encontró el usuario Carlos.

# Fase 4- Privilegios

#### Sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/64ac61a9-c9d2-46f8-a5e6-118550071085)
Despues de hacer el tratamiento de la TTY, vemos que podemos ejecutar como root python3, específicamente el archivo script.py.

#### Script.py
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/8218592e-7b43-4a4a-ad38-efa2f445839f)
Cambie de nombre el archivo `script.py` original a `ss.py` y cree otro `script.py` en el que puse el siguiente codigo:
```python
import os

os.system("bin/sh")
```
Este código `import os` y `os.system("bin/sh")` abre una terminal de Unix (`sh`) desde un script de Python. En este caso como ejecutamos el script con permisos de root. Se abrió una shell con privilegios de root. Esto se debe a que el comando `os.system("bin/sh")` ejecuta un shell (`sh`) con los mismos permisos que el usuario que corre el script.
