# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/105eccec-e5e9-46d8-9341-2a55f6d59ca3)
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

1. **22/tcp (SSH ):** OpenSSH 8.2p1
2. **80/tcp (HTTP ):** Apache httpd 2.4.52
3. **27017/tcp (MongoDB ):** MongoDB 7.0.9

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a acceder a la página por defecto de Apache Ubuntu.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/939f5571-c27b-473c-8835-166148ef63a0)

#### Fase 2- Exploración

#### feroxbuster
```python
301      GET        9l       28w      312c http://172.17.0.2/wordpress
```
Encontramos un `/wordpress`.

#### /Wordpress
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/eea5b4be-34a1-4062-988e-456010a2fb7d)
Es una web un poco destruida...

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/30fecbd0-9d7c-4f91-9ef8-70aaba680675)

Lo tenemos que agregar al `/etc/hosts`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/f5cd2b47-0e33-445f-9043-de2442dfcc6f)

Buscando en la pagina encuentro una parte de la pagina que fue creada por un usuario llamado `chocolate`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/19dc3ace-51c8-4276-b214-a6b31d621f81)

Es un autor... Estaría interesante probarlo en wordpress.

# Fase 3- Explotación

#### Wordpress R.I.P
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/269645ef-6a37-49af-8ba2-05ef2bf78134)

Probamos el usuario con una contraseña cualquiera y nos dice que la contraseña que hemos introducido para el usuario `chocolate` no es correcta. Esto significa que el usuario existe dentro de wordpress.

```python
wpscan --url http://collections.dl/wordpress/ --usernames chocolate --passwords /usr/share/wordlists/rockyou.txt
```
Con este comando vamos a realizarle fuerza bruta al usuario `chocolate` con el diccionario `rockyou`.
```python
| Username: chocolate, Password: chocolate
```
Y encontramos la contraseña `chocolate`.

#### File Manager
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/693b00c9-a8b0-4ba9-9beb-cb2f263c7de3)
Ahora para la explotación vamos a agregar el plugin `File Manager`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/048d4cce-f174-4708-8986-a380caf55039)

```python
http://collections.dl/wordpress/wp-content/themes/twentytwentyfour/
```
Si vamos a esta url deberíamos ver un directory listing.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/8088a566-5bc0-4930-86bc-2a5fc642bdf0)
En el `File Manager` vamos a la misma carpeta y creamos un archivo `txt`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c2b7ce61-acf0-4998-9da4-89d55fe0a930)
Lo guardamos como un `.php` y dentro ponemos el codigo de una reverse shell. Link: https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

Nos ponemos en modo escucha con el siguiente comando:
```python
❯ nc -nlvp <puerto>
```
Donde `<puerto>` fue el puerto que pusimos en el codigo.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/88c80434-9a8a-4e4c-8322-f8a470be00b6)

Y le ejecutamos el archivo, en este caso yo le puse `shell.php`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/6052c3d2-11fc-47b2-a5c0-2e957454521f)

Y ya estaríamos dentro!.

# Fase 4- Privilegios

#### sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/8b863437-6d8f-4c6f-a414-f4772d55fbeb)

Despues de realizar el tratamiento de la TTY, no nos deja ejecutar el comando `sudo -l` ni encontramos ningún binario con el comando `find / -perm -4000 -user root 2>/dev/null`
así que tocara buscar...

#### ssh
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/91ff833a-25e2-4db7-9263-ed930754c6e8)

Encontramos los usuarios `chocolate` y `dbadmin`. Haremos ataque de diccionario al ssh con los usuarios.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/f5f29cd4-0e50-4de6-9b11-8491018c27d8)

Y encontramos la contraseña `estrella` para el usuario `chocolate`.

#### mongodb
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/05ec9da9-f6be-4b2b-8c98-e37bf46f0849)
En el directorio `/tmp` encontramos un archivo llamado `mongodb-27017.sock` y al ejecutar el comando `ps -e -f` para listar que se esta ejecutando en el sistema vemos lo siguiente:
```python
root           1       0  0 22:46 ?        00:00:00 /bin/sh -c service ssh start &&     service mariadb start &&     service apache2 start ; sleep 5 &&     mongod --bind_ip_all --quiet &     sleep 10 &&  
```
Tiene el parametro `--bind_ip_all` activo, esto significa que escucha en todas las interfaces de red disponibles. Esto significa que MongoDB aceptará conexiones en cualquier dirección IP asignada a la máquina en la que se está ejecutando.

```python
❯ mongo --host 172.17.0.2 --port 27017
> show databases;
accesos  0.000GB
admin    0.000GB
config   0.000GB
local    0.000GB
> use accesos;
switched to db accesos
> show tables;
usuarios
> db.usuarios.find().pretty();
{
	"_id" : ObjectId("6645f4456682cdae1b46b799"),
	"nombre" : "dbadmin",
	"contraseña" : "chocolaterequetebueno123"
}
> 
```
Nos conectamos y encontramos una base llamada `accesos` y con un nombre que es el mismo que vimos anteriormente como el usuario `www-data` y una contraseña.

#### dbadmin
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/1bde0b69-1f87-47c4-8773-bb2f836c5642)

Y listo! somos el usuario dbadmin.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3fa7c154-df84-4314-8f67-81c587d49d49)

No encontramos nada raro en los binarios, y comprobamos contraseñas encontradas anteriormente y después de varios intentos somos root!. Contraseña del root: `chocolaterequetebueno123`
