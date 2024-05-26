# Fase 1- Tanteo

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/81be8da7-2a69-4eb0-a45d-21fd59bf62b3)
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

1. **80/tcp (HTTP):** Apache httpd 2.4.57 (Debian)
### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a acceder a la página por defecto de Apache en la máquina vulnerable, confirmando así el correcto funcionamiento del servidor web Apache en Debian. La página por defecto de Apache en Debian muestra el mensaje "Apache2 Debian Default Page".
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ac82bb13-c1bc-446c-9960-15500f8e1867)

# Fase 2- Exploración


#### Gobuster

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/e13740d9-7297-4882-8a71-ebcae576dc70)

- Se ejecuto un escaneo con Gobuster sobre la pagina objetivo y se encontró un directorio wordpress.

#### Wordpress?

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/19ce64a2-fee3-432b-ad13-30768fcfce40)

Al acceder al directorio nos encontramos con una pagina web interesante...

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b21bf1ee-019f-4399-bd42-c55c6d9fc4f5)

Efectivamente esta corriendo un wordpress esta pagina web, deberíamos probar con wpscan.

# Fase 3- Explotación

#### Wordpress Explotatión

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/abc48851-f427-4f53-9faf-4c44b3598dd6)


Realizando un escaneo con wpscan, mas concretamente con el siguiente comando: `sudo wpscan --url https://172.17.0.2/wordpress/ -e u vp` notamos varias cosas.
- **Servidor HTTP**: Apache/2.4.57 (Debian)
- **XML-RPC habilitado**: [http://172.17.0.2/wordpress/xmlrpc.php](http://172.17.0.2/wordpress/xmlrpc.php)
- **WordPress readme encontrado**: [http://172.17.0.2/wordpress/readme.html](http://172.17.0.2/wordpress/readme.html)
- **Directorio de carga habilitado con listado**: [http://172.17.0.2/wordpress/wp-content/uploads/](http://172.17.0.2/wordpress/wp-content/uploads/)
- **WP-Cron externo parece estar habilitado**: [http://172.17.0.2/wordpress/wp-cron.php](http://172.17.0.2/wordpress/wp-cron.php)
- **Versión de WordPress**: 6.5.2 (Lanzada el 2024-04-09)
- **Tema de WordPress en uso**: twentytwentytwo (Versión 1.6, actualización disponible a la 1.7)
- **Usuarios identificados**:
    - mario

#### Usuario mario

- vamos a probar realizar un ataque por fuerza bruta al usuario mario, con el siguiente comando: `sudo wpscan --url http://172.17.0.2/wordpress/ -u mario -p /usr/share/wordlists/rockyou.txt`

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/4bc46038-bf28-4b71-aff4-679ccd29d67f)

- y encontró la contraseña del usuario "mario" y es "love".


![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/2c60e7d3-a90c-4563-b4cb-65819a65f395)

- probamos las credenciales obtenidas y estamos dentro.

#### Apariencia
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/63604617-e16f-4e37-97e8-d9bc2c26fc06)

- hay un editor de Apariencia code editor, podríamos intentar colar codigo malicioso en php.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ea591869-3553-41f4-a273-df65f8c627f7)

- Después de colar el archivo malicioso con la Revershell.php se guardaba en `http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/index.php?cmd=id`
- Despues de ejecutarlo nos da la revershell.

# Fase 4- Privilegios


![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/57dbe05f-de6e-474a-bf47-16d8f016c3f7)

- Despues de realizar el tratamiento de la tty, no deja realizar el comando sudo -l por lo tanto  Buscaremos por binarios con permisos **SUID** desde la raíz:

#### ENV

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/aa50e2d5-b8a4-44a4-9a87-0a8d55b3cc0a)
```
find / -perm -4000 2>/dev/null
________________________________________________
/usr/bin/chfn
/usr/bin/chsh
/usr/bin/gpasswd
/usr/bin/mount
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/su
/usr/bin/umount
/usr/bin/env
```
- Nos llama la atención el binario **/usr/bin/env**.

- podemos recurrir a [GTFOBins](https://gtfobins.github.io/) , y si es crítico esta página nos indicará como podemos explotarlo. En este caso es sencillo ya que con env podemos lanzarnos directamente una consola, y al ser ejecutado como el root gracias al permiso **SUID**, esta consola será lanzada con sus permisos, de forma que hemos logrado el nivel de privilegios máximo sobre la máquina !
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a17e1b12-f121-461e-8ec9-83a821d0058f)

