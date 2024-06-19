# Fase 1- Tanteo

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
![[Pasted image 20240619194959.png]]

#### Fase 2- Exploración

#### feroxbuster
```python
301      GET        9l       28w      312c http://172.17.0.2/wordpress
```
Encontramos un `/wordpress`.

#### /Wordpress
![[Pasted image 20240619195238.png]]
Es una web un poco destruida...

![[Pasted image 20240619195254.png]]
Lo tenemos que agregar al `/etc/hosts`.

![[Pasted image 20240619200009.png]]
Buscando en la pagina encuentro una parte de la pagina que fue creada por un usuario llamado `chocolate`.

![[Pasted image 20240619200049.png]]
Es un autor... Estaría interesante probarlo en wordpress.

# Fase 3- Explotación

#### Wordpress R.I.P
![[Pasted image 20240619200139.png]]
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
![[Pasted image 20240619200416.png]]
Ahora para la explotación vamos a agregar el plugin `File Manager`.

![[Pasted image 20240619200520.png]]
```python
http://collections.dl/wordpress/wp-content/themes/twentytwentyfour/
```
Si vamos a esta url deberíamos ver un directory listing.

![[Pasted image 20240619200632.png]]
En el `File Manager` vamos a la misma carpeta y creamos un archivo `txt`.

![[Pasted image 20240619200756.png]]
Lo guardamos como un `.php` y dentro ponemos el codigo de una reverse shell. Link: https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

Nos ponemos en modo escucha con el siguiente comando:
```python
❯ nc -nlvp <puerto>
```
Donde `<puerto>` fue el puerto que pusimos en el codigo.

![[Pasted image 20240619201020.png]]
Y le ejecutamos el archivo, en este caso yo le puse `shell.php`.

![[Pasted image 20240619201058.png]]
Y ya estaríamos dentro!.

# Fase 4- Privilegios

#### sudo -l
![[Pasted image 20240619201233.png]]
Despues de realizar el tratamiento de la TTY, no nos deja ejecutar el comando `sudo -l` ni encontramos ningún binario con el comando `find / -perm -4000 -user root 2>/dev/null`
así que tocara buscar...

#### ssh
![[Pasted image 20240619202002.png]]
Encontramos los usuarios `chocolate` y `dbadmin`. Haremos ataque de diccionario al ssh con los usuarios.

![[Pasted image 20240619202158.png]]
Y encontramos la contraseña `estrella` para el usuario `chocolate`.

#### mongodb
![[Pasted image 20240619202540.png]]
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
![[Pasted image 20240619203553.png]]
Y listo! somos el usuario dbadmin.

![[Pasted image 20240619203724.png]]
No encontramos nada raro en los binarios, y comprobamos contraseñas encontradas anteriormente y después de varios intentos somos root!. Contraseña del root: `chocolaterequetebueno123`
