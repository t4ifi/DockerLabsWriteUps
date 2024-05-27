# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/1f5cf63f-3ec4-4fe7-a3d0-1c261186b044)
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

1. **22/tcp(SSH):** OpenSSH 9.2p1 (Ubuntu)
2.  **80/tcp(HTTP):** Apache httpd 2.4.59
### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a acceder a la página por defecto de Apache en la máquina vulnerable, confirmando así el correcto funcionamiento del servidor web Apache en Debian. La página por defecto de Apache en Debian muestra el mensaje "Apache2 Debian Default Page".
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/93e7e03b-f07b-49b9-9daf-58225b169fa8)

# Fase 2- Exploración

### Feroxbuster
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/e641097f-93d8-4a53-bb29-5ea7909250e6)
Se uso Feroxbuster para la enumeración de directorios ocultos dentro de la pagina y se encontró /images/.

### /Images/
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/5d611576-3290-4e5e-a1b0-c21aba7cc6ec)

Al entrar se encontró una imagen llamada `agua_ssh.jpg`. Esto puede ser un posible usuario en ssh.

### Apache
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/41fcb3e1-1c71-4548-a341-0d4757577720)
Al revisar en el codigo fuente de la pagina de apache encontramos esto: 
```sql
<!--
++++++++++[>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++++>++++++++++>+++++++++++>++++++++++++>++++++++++>++++++++++++>++++++++++>+++++++++++>+++++++++++>+>+<<<<<<<<<<<<<<<<<-]>--.>+.>--.>+.>---.>+++.>---.>---.>+++.>---.>+..>-----..>---.>.>+.>+++.>.
-->
```
Al buscarlo en internet nos sale que es brianfuck, una especie de lenguaje de encriptación. Lo desencriptamos y nos da que el resultado es: `bebeaguaqueessano`.

# Fase 3- Explotación

### SSH
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/35064162-c9d1-48c8-9fe7-dd86b4d8ea95)

Intentamos acceder por ssh con el usuario `agua` y la contraseña `bebeaguaqueessano` y lo logramos.

# Fase 4- Privilegios

### Sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/6fe1270c-4791-4877-979d-e309ab489686)

Despues de realizar el tratamiento de la TTY, y al hacer el comando `sudo -l` vemos que tenemos permisos de ejecutar el binario `bettercap`

### Bettercap
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3f58ece9-cd75-4f45-a867-0d08e7eac495)

Ejecutamos el binario como root.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/357afd70-b364-45cc-9fd1-e46498c00d0b)

Usamos el comando help y vemos que nos permite ejecutar un comando.
podríamos intentar ejecutar esto: `chmod u+s /bin/bash` para escalar.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/8ebedc86-d821-49bc-ae0a-cd52848d9cee)

Ejecutamos y funciono, somos usuario root!
