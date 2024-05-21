# Fase 1- Tanteo

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/bfc93dea-2e95-4ab7-88ce-61ee6a51335d)


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

1. **22/tcp (SSH):** OpenSSH 9.2p1 Debian 2+deb12u2.
2. **80/tcp (HTTP):** Apache httpd 2.4.57 en Debian.
### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a acceder a la página por defecto de Apache en la máquina vulnerable, confirmando así el correcto funcionamiento del servidor web Apache en Debian. La página por defecto de Apache en Debian muestra el mensaje "Apache2 Debian Default Page".
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b815f92b-a5f3-41d0-873e-0c5f136f7ea2)

# Fase 2- Exploración

#### Gobuster

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3c6e2fda-d352-41bc-9b46-3a88a16cebcb)

- Se ejecutó un escaneo con Gobuster para encontrar directorios en la página web. Se descubrieron los siguientes directorios:
    - /.php (Estado: 403, Tamaño: 275 bytes)
    - /secret.php (Estado: 200, Tamaño: 927 bytes)
    - /.php (Estado: 403, Tamaño: 275 bytes)
    - /server-status (Estado: 403, Tamaño: 275 bytes)


#### Mario?

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c5328732-6d9d-4aab-9f12-9809af10e50c)
- se encontró el directorio "secret.php", que muestra un mensaje indicando que la web no se puede hackear.  Probar el acceso mediante SSH utilizando el usuario "mario". Es interesante explorar este nombre, ya que puede ser una pista para intentar acceder al sistema a través de SSH.

# Fase 3- Explotación

#### Hydra

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/57772e56-d251-4e94-ba32-fb0083dabd3e)
**Prueba de Credenciales con Hydra**

- **Usuario:** mario
- **Contraseña Encontrada:** chocolate
- **Protocolo:** SSH
- **Dirección IP:** 172.17.0.2
- **Resultado:** Se encontró una contraseña válida para el usuario mario.

**Observaciones:**
- Se ha encontrado la contraseña "chocolate" para el usuario mario en el servicio SSH.

**Comando Utilizado**
  `sudo hydra -l mario -P /usr/share/wordlists/rockyou.txt 172.17.0.2 ssh`

- `sudo`: Este comando permite ejecutar Hydra con privilegios de superusuario, necesario para realizar algunas operaciones que requieren permisos especiales.
- `hydra`: Es el comando principal de Hydra, una herramienta de prueba de penetración que se utiliza para realizar ataques de fuerza bruta o ataques de diccionario contra servicios de autenticación, como SSH en este caso.
- `-l mario`: Especifica el nombre de usuario que se utilizará en el ataque de fuerza bruta. En este caso, estamos atacando al usuario "mario".
- `-P /usr/share/wordlists/rockyou.txt`: Especifica la ubicación del archivo de contraseñas que se utilizará en el ataque. En este caso, estamos utilizando el archivo "rockyou.txt", que es un diccionario popular de contraseñas comunes.
- `172.17.0.2`: Es la dirección IP del objetivo que estamos atacando.
- `ssh`: Especifica el protocolo que estamos atacando, en este caso, SSH.

### Dentro
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/dfacd200-f649-4097-a833-d1ce538df069)
- Se intentó acceder al SSH utilizando el usuario `mario` y la contraseña `chocolate`, con éxito. y se realizo el tratamiento de la TTY.

# Fase 4- Privilegios

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/9e70f74c-4a2b-49ed-ace1-ad4f1a45ac43)
- Al revisar los privilegios de sudo del usuario `mario` con `sudo -l`, se observó que puede ejecutar el binario `/usr/bin/vim` con privilegios de sudo.

### GTFobins
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b9a889b5-36da-462b-9afe-40f76f9df5c5)
**Búsqueda en GTFOBins:**

- Encontramos que el binario sudo vim puede ser utilizado para ejecutar comandos como root sin ingresar la contraseña.

**Comando Utilizado para Escalar Privilegios:**

`sudo vim -c ':!/bin/sh'`

**Acceso Obtenido:**
- Se logró acceso con privilegios de root.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a03355e9-5494-4c3e-9dd9-da1482f4f7b5)

















