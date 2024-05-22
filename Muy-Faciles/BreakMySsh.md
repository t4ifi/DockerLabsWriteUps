# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ed6b2475-1e54-4424-9583-d6497f963b1b)

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

1. **22/tcp (SSH ):** OpenSSH 7.7

# Fase 2- Exploración
Vemos que solamente esta el puerto 22 abierto. Procederemos a buscar si hay algún tipo de vulnerabilidad en esa versión.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/91bc3ac4-017c-4f3a-8ba7-385c96622e12)
Y si que la hay.

# Fase 3- Explotación
Procedemos a usar MsfConsole para usar ese Username Enumeration.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/048f239a-a3d1-4ac3-b576-c8191b049e03)
Nos encontró el usuario `lovely`.

### Hydra
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/af22e511-47dc-4780-80c1-42fdeb9f084a)
Utilizamos hydra para hacer fuerza bruta al usuario `lovely` y encontramos la contraseña `rockyou`.

# Fase 4- Privilegios
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/5321c0d4-7649-4135-8bf6-4ff3466139a2)
Tras no ver nada de primeras, vamos a /opt por si tenemos algo y vemos un archivo oculto.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/614e5299-73ab-4865-a735-2412c4d60c1d)
El archivo .hash contiene: `aa87ddc5b4c24406d26ddad771ef44b0` debe ser una especie de contraseña encriptada. Tras pasarla por crackstation vemos que la contraseña es: `estrella`,
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/5e95d4e8-14d1-409c-8df2-6a3f5dd0ed31)
La probamos con `su root` y es la contraseña del usuario root. Alcanzamos el nivel de privilegios máximos en la maquina.
