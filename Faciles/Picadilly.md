# Fase 1- Tanteo

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c7e53a74-b4c7-4eec-a6ab-ffff94517ac7)
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

1. **80/tcp (HTTP):** Apache httpd 2.4.59 (Debian)
2. **443/tcp(SSL/HTTP)**: Apache httpd 2.4.59 (Debian)
### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a acceder a una pagina con un archivo llamado backup.txt.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/60d5a429-7011-4384-b116-97b1afa83d9b)

# Fase 2- Exploración

#### 443
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/9aebf466-6cf0-4401-9936-511754b51e92)
En el puerto 443, que es https/ssl hay una pagina web que nos permite subir posts.

#### Uploads
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/4a9d0392-f43c-4087-832d-63e9a28d133e)
Aplicando fuzzing encontramos que existe el directorio /uploads.

#### Mateo?
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/f1db7a3e-0a52-4485-8741-654041786fa9)
En la pagina web, dentro del archivo `backup.txt` dice esto:
```python
/// The users mateo password is ////


----------- hdvbfuadcb ------------

"To solve this riddle, think of an ancient Roman emperor and his simple method of shifting letters."

////////////////////////////////////
```
El archivo nos dice que el usuario es mateo y la contraseña es `hdvbfuadcb`, pero que contiene una especie de cifrado, específicamente el cifrado César. El cifrado César es una técnica de cifrado simple en la que cada letra del texto es reemplazada por otra letra que se encuentra un número fijo de posiciones adelante en el alfabeto.

#### Cifrado Cesar
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/aa9a6729-9965-4490-b861-be3ecfa00048)

Vamos a la pagina https://www.dcode.fr/cifrado-cesar, para poder descubrir la contraseña.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/bc4864fa-c747-49ca-a87f-3bcdc5f22b7b)

Según la pagina la contraseña es: `easycrxazy`.

# Fase 3- Explotación

#### Reverse shell
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/eb9e0a30-16fb-44df-813a-322ac6ce1a89)

Subimos una reverse shell hecha en php.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/d2e9962f-21b9-4ca5-a976-eedb1a3c1a78)

Nos llega y somos el usuario www-data lo que sigue es el tratamiento de la TTY y Escalada de privilegios.


# Fase 4- Privilegios

#### Mateo.-.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/7a25bb15-97bf-4125-8720-c68a44b3e3fe)

Vemos que existe el usuario `mateo`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a723d284-17b7-49bd-ae1e-ffa692d99629)

Intentamos cambiar a el con el comando `su mateo` y probamos la contraseña previamente encontrada pero no podemos. Mirando la contraseña:
```python
easycrxazy
```
Vemos que esta mal, seria `easycreazy`, así que la probamos.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/fad57fc4-2561-4257-8bf2-108b1dd92293)

y funciono, somos el usuario mateo.

#### php
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/f76adc5d-f89b-4bc8-a25b-1507f8735d34)

Vemos que podemos ejecutar el binario php como usuario root.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/9607af15-8ce0-4aed-b055-9a0c0ccebf92)

En gtfobins, nos dice que para escalar privilegios tenemos que ejecutar una shell de esa forma, yo utilizo esta personalmente:
```php
`sudo php -r 'system("/bin/bash");'`
```

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/16a56ed5-8cc4-4eb5-8ed1-0c724753b3e3)
Ejecutamos y funciono, somos root!.
