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


















