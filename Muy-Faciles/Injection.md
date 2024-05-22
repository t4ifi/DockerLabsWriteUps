# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/0b1b42ae-b166-484e-a31a-45e3219c4116)

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

1. **22/tcp (SSH ):** OpenSSH 8.9p1
2. **80/tcp (HTTP ):** Apache httpd 2.4.52

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a acceder a una pagina tipo Login.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/74d0ca68-03d6-457c-92d6-eca78526a71c)

# Fase 2- Exploración

### Login
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/7d81e559-86d5-4fd5-8ec1-35436d3fa7f0)

Al probar las credenciales `admin` `admin`, no entro.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/58d78554-59cb-423d-9352-b09247e6e00d)
Al probar `admin'` nos tira un mensaje de SQL Syntax error. El mensaje dice lo siguiente:
```sql
SQLSTATE[42000]: Syntax error or access violation: 1064 You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'admin'' at line 1
```
Este mensaje indica que se ha producido un error de sintaxis o violación de acceso al ejecutar una consulta SQL. En particular, el error 1064 señala que hay un problema con la sintaxis en la consulta SQL que se está intentando ejecutar. En este caso, el mensaje específico indica que hay un error cerca de 'admin' en la línea 1 de la consulta

### Feroxbuster
No se encontró nada.

# Fase 3- Explotación

### SQL Inyección
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/140692f1-0ba9-487e-a237-ac746b682c72)

Probamos la clásica inyección `admin' or 1=1 -- - `.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ea7b6181-d875-483e-8e87-0cd8c7244311)

Y estamos dentro. Nos dice que existe un usuario Dylan y que su contraseña es `KJSDFG789FGSDF78`.

### SSH

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/cce5d18a-63f1-492a-82e6-9ac82e7f0e38)

Probamos el usuario y la contraseña proporcionada y estamos dentro.

# Fase 4- Privilegios
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/89868f54-da71-40ad-ab3e-e012f3527269)

Se realizo el tratamiento de la TTY y se ejecutó el comando `find / -perm -4000 -user root 2>/dev/null` este comando busca archivos en todo el sistema que tengan el bit de setuid activado y que pertenezcan al usuario `root`, omitiendo cualquier mensaje de error que pueda surgir durante la búsqueda. Vemos el binario ENV. Explorado anteriormente.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/2e343170-7db1-4d05-9b6a-8e0fcc054f00)

Ejecutamos el comando que nos proporciono GTFobins para escalar privilegios y nos dice que no funciona el comando sudo.
Ejecutamos el siguiente comando `/usr/bin/env /bin/sh -p` y funciono. Somos root.
