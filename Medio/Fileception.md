# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/313e3067-7dc4-4307-ba50-6e1c3f29ff21)


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

1. **21/tcp (FTP ):** vsftpd 3.0.5
2. **22/tcp (SSH ):** OpenSSH 9.6p1
3.  **80/tcp (HTTP ):** Apache httpd 2.4.58

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Y se encontró la pagina por defecto de Apache2 Debian.
![[Pasted image 20240605152628.png]]

# Fase 2- Exploración

#### Feroxbuster
No encontró nada.

#### Codigo
![[Pasted image 20240605152830.png]]
Revisando el codigo fuente encontramos este mensaje...
```
<!-- 
¡Hola, Peter!

¿Te acuerdas los libros que te presté de esteganografía? ¿A que estaban buenísimos?

Aquí te dejo una clave que usaras sabiamente en el momento justo. Por favor, no seas tan obvio, la vida no se trata de fuerza bruta.

@UX=h?T9oMA7]7hA7]:YE+*g/GAhM4

Solo te comento, recuerdo que usé este método porque casi nadie lo usa... o si. Lamentablemente, a mi también se me olvido. Solo recuerdo que era base
-->
```

![[Pasted image 20240605153845.png]]
Investigando bastante encontré que estaba encriptado en base85, la contraseña sin encriptar es: base_85_decoded_password.

# Fase 3- Explotación

#### Anonymous
![[Pasted image 20240605154026.png]]
En el escaneo de nmap, pudimos ver que existe el usuario Anonymous en el puerto 21 ftp, nos conectamos y descargamos una imagen llamada `hello_peter.jpg`.

#### Steghide
![[Pasted image 20240605154324.png]]
Con la herramienta Steghide, logramos extraer un archivo llamado  `you_find_me.txt` de la imagen.

#### What
![[Pasted image 20240605154423.png]]
Revisando el contenido encontramos esto.
```
Hola, Peter!

Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook! Ook?  Ook. Ook?  Ook. Ook.  Ook. Ook?  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook?  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook?  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook? Ook.  Ook? Ook.  Ook? Ook.  Ook? Ook.  Ook! Ook!  Ook? Ook!  Ook. Ook?  Ook. Ook?  Ook. Ook?  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook.  Ook. Ook?  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook! Ook.  Ook? Ook.  Ook! Ook!  Ook! Ook.  Ook! Ook.  Ook. Ook.  Ook! Ook.  Ook. Ook?  Ook! Ook.  Ook? Ook.  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook.  Ook. Ook.  Ook! Ook.  Ook. Ook?  Ook! Ook.  Ook! Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook! Ook.  Ook! Ook.  Ook? Ook.  Ook! Ook!  Ook! Ook.  Ook. Ook.  Ook! Ook.
```

![[Pasted image 20240605154516.png]]
Revisando un poco encontramos que lo que estamos viendo es un lenguaje de programación esotérico.

![[Pasted image 20240605154740.png]]
Encontré una pagina que al decodificarlo nos entrega una contraseña: `9h889h23hhss2`.

#### SSH
![[Pasted image 20240605154944.png]]
Nos intentamos conectar por ssh y tuvimos éxito!.

# Fase 4- Privilegios

#### sudo -l
![[Pasted image 20240605155354.png]]
Despues de realizar el tratamiento de la TTY, y realizar el comando sudo -l, no podemos.

#### ls
![[Pasted image 20240605155617.png]]
Vemos que en una nota importante nos dice que hay un archivo importante en tmp.

![[Pasted image 20240605155701.png]]
Hay 2 archivos, un recuerdo que nos dice que cuando era niño creía que para pasar los archivos de flv a mp4, solamente había que cambiar la extensión. y el otro que es una nota importante para octopus.

![[Pasted image 20240605155812.png]]
Con el comando: 
```python
sudo scp peter@172.17.0.2:/tmp/importante_octopus.odt /home/sansett/t4ifi/Maquinas/DockerLabs/Medias/fileception/content
```
Nos pasamos el archivo .odt a nuestra maquina.


#### .odt
![[Pasted image 20240605155956.png]]
Dentro del archivo hay varias cosas, pero dentro del leerme.xml nos dice que existe un usuario octopus y su contraseña es: `ODBoMjM4MGgzNHVvdW8zaDQ=`. O tiene algún tipo de encriptación.

![[Pasted image 20240605160214.png]]
Probando cosas encontré que su encriptación es base64. La contraseña es: `80h2380h34uouo3h4`

#### Octupus
![[Pasted image 20240605160326.png]]
Pudimos cambiar al usuario octupus, y para escalar podemos ejecutar todo, asi que hacemos un sudo su.

![[Pasted image 20240605160422.png]]
Y somos root!
