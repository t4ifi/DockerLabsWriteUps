# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/26160cff-30de-4857-acc7-41728db64925)
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

1. **21/tcp(FTP):** vsftpd 2.0.8
2.  **22/tcp(SSH):** OpenSSH 9.6p1
3. **80/tcp(HTTP)**: Apache httpd 2.4.58
### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a acceder a la página web con un mensaje.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/956a96d9-da04-4c43-8e2f-d8d8c571039d)
`Welcome mwheeler!!` Esto podría ser un posible usuario en ssh o ftp.

# Fase 2- Exploración

#### Feroxbuster
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ba045203-f8d3-4fef-93b0-ae741edee7a3)
Se realizo un fuzzing con Feroxbuster y se encontró un directorio llamado `strange`.
Y dentro de strange se encontraron los siguientes directorios: `private.txt`, `secret.html`

#### Bayers
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3410978d-bede-4dba-bb32-de1107ea1251)
Al ingresar al directorio vemos una pagina web con información muy interesante...
```python
The Upside Down - My Experience
Hey everyone, it's Will. I want to share with you all my experience...

Read more here.

Archive
July 1985
Strange Dreams
The password for the encrypte file is iloveu
My Encounter with the Demogorgon
About Will
Hey, I'm Will Byers. I love playing Dungeons & Dragons, riding my bike...
```
Encontramos otro usuario posible para ftp o ssh `Will`. Y además nos dice que la contraseña del archivo cifrado `iloveu`.

#### Secret.html
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/eca1d80d-53c0-4246-ae34-b021bc66f770)

Dentro del directorio `/strange/secret.html` encontramos un texto muy interesante... El texto dice lo siguiente:
```ruby
The ftp user is admin, but the password is ...
You must discover it.
A hint: The rockyou diccionary is correct for use!
```
Básicamente nos dice que el usuario del servicio ftp es admin, y que la contraseña la tenemos que descubrir pero nos dice que el diccionario rockyou es correcto para ese uso.

#### Private.txt
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/0654c9d0-3582-495d-bf9c-3d4bde473980)

Descargamos el archivo private.txt pero parecen ser unos caracteres no legibles.

# Fase 3- Explotación

#### Hydra
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/e8b7983f-c116-43ff-a38e-97a907dee6e5)
Atacamos el puerto ftp con hydra con el usuario admin y el diccionario rockyou.txt y encontramos la contraseña `banana`

#### FTP
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/bfd8b6b1-90a1-409e-89e6-8c453289cd48)

Entramos en ftp con el usuario admin y encontramos un archivo llamado `private_key.pem`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b2706579-2825-4676-aac1-49efc81850a5)

Al descargarlo y leerlo parece ser una especie de contraseña o algo así encriptado.

#### Desencriptar
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/8c478417-77ea-42aa-8d1b-6b93bcdd795a)

Entonces, con un pem y un txt, que se nos ocurre? quizás desencriptar? Bueno con el comando: 
```ruby
sudo openssl rsautl -decrypt -in private.txt -out privateOUT.txt -inkey private_key.pem
```
Logramos desencriptar y encontramos `demogorgon`.

#### Ssh
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/21093ece-0de0-4de8-a5a2-deb86d086bbb)

Nos conectamos por ssh al usuario `mwheeler` con la contraseña `demogorgon`.

# Fase 4- Privilegios
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/337a19c5-3515-49b3-8f55-90facd18cd63)

Despues de hacer el tratamiento de la TTY, vemos que al ejecutar sudo -l no sucede nada. Tocara buscar.

Y estamos dentro, después de buscar y buscar, nos acordamos de la contraseña de admin el cual esta en el passwd.
```python
mwheeler@d7a455afaddd:~$ cat /etc/passwd|grep sh$

root:x:0:0:root:/root:/bin/bash

ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash

mwheeler:x:1001:1001::/home/mwheeler:/bin/bash

admin:x:1002:1002::/home/admin:/bin/sh

mwheeler@d7a455afaddd:~$ su admin

Password:

admin@d7a455afaddd:/home/mwheeler$
```

Probamos por si tenemos algún privilegio en cuanto a binarios o scripts como algún usuario y vemos que finalmente podemos escalar a root!
```python
admin@d7a455afaddd:/home/mwheeler$ sudo -l

[sudo] password for admin:

Matching Defaults entries for admin on d7a455afaddd:

    env_reset, mail_badpass,

    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User admin may run the following commands on d7a455afaddd:

    (ALL) ALL

admin@d7a455afaddd:/home/mwheeler$ sudo su

root@d7a455afaddd:/home/mwheeler# woami
root
```
