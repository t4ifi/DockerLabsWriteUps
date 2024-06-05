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
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ec937c5b-1fa5-4d4b-9bc5-e9ee8bd157f8)

# Fase 2- Exploración

#### Feroxbuster
No encontró nada.

#### Codigo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ff341e9e-7466-4a61-b64b-34804d262194)
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

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ea90429b-aad5-4b70-9668-50e5d603488a)
Investigando bastante encontré que estaba encriptado en base85, la contraseña sin encriptar es: base_85_decoded_password.

# Fase 3- Explotación

#### Anonymous
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/d9e97cbb-b5a0-4525-b842-56659f342fd7)
En el escaneo de nmap, pudimos ver que existe el usuario Anonymous en el puerto 21 ftp, nos conectamos y descargamos una imagen llamada `hello_peter.jpg`.

#### Steghide
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/aea12683-9cac-4737-a8fa-98f05abbcda5)
Con la herramienta Steghide, logramos extraer un archivo llamado  `you_find_me.txt` de la imagen.

#### What
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b0864c74-2a78-4e76-802b-6bc8185ae625)
Revisando el contenido encontramos esto.
```
Hola, Peter!

Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook! Ook?  Ook. Ook?  Ook. Ook.  Ook. Ook?  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook?  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook?  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook? Ook.  Ook? Ook.  Ook? Ook.  Ook? Ook.  Ook! Ook!  Ook? Ook!  Ook. Ook?  Ook. Ook?  Ook. Ook?  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook.  Ook. Ook?  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook! Ook.  Ook? Ook.  Ook! Ook!  Ook! Ook.  Ook! Ook.  Ook. Ook.  Ook! Ook.  Ook. Ook?  Ook! Ook.  Ook? Ook.  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook.  Ook. Ook.  Ook! Ook.  Ook. Ook?  Ook! Ook.  Ook! Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook! Ook.  Ook! Ook.  Ook? Ook.  Ook! Ook!  Ook! Ook.  Ook. Ook.  Ook! Ook.
```

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/90d3c7d9-3f85-4781-a4fe-bd0ab169a1b1)
Revisando un poco encontramos que lo que estamos viendo es un lenguaje de programación esotérico.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a46aaa26-aa04-4e78-a4bc-02afe4e682b4)
Encontré una pagina que al decodificarlo nos entrega una contraseña: `9h889h23hhss2`.

#### SSH
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/7c6ad654-b170-4893-9df0-3900ac29e559)

Nos intentamos conectar por ssh y tuvimos éxito!.

# Fase 4- Privilegios

#### sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/84d6be44-b56d-41c9-9eea-5bab743277ee)

Despues de realizar el tratamiento de la TTY, y realizar el comando sudo -l, no podemos.

#### ls
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/259fd293-0f05-408d-9182-7c1b36989af1)

Vemos que en una nota importante nos dice que hay un archivo importante en tmp.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/0cda3270-a5a1-4cd4-8947-44a7a5460fd1)
Hay 2 archivos, un recuerdo que nos dice que cuando era niño creía que para pasar los archivos de flv a mp4, solamente había que cambiar la extensión. y el otro que es una nota importante para octopus.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c70ed996-4dc6-4c3e-9716-31aab4e0c456)
Con el comando: 
```python
sudo scp peter@172.17.0.2:/tmp/importante_octopus.odt /home/sansett/t4ifi/Maquinas/DockerLabs/Medias/fileception/content
```
Nos pasamos el archivo .odt a nuestra maquina.


#### .odt
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/706f5857-f528-4586-9264-6d9796aaba9c)

Dentro del archivo hay varias cosas, pero dentro del leerme.xml nos dice que existe un usuario octopus y su contraseña es: `ODBoMjM4MGgzNHVvdW8zaDQ=`. O tiene algún tipo de encriptación.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/0d6bb38b-b3c4-4029-81eb-b8f038df2e6c)

Probando cosas encontré que su encriptación es base64. La contraseña es: `80h2380h34uouo3h4`

#### Octupus
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c5e935e3-c7a5-425e-a79b-b74743f98c7e)
Pudimos cambiar al usuario octupus, y para escalar podemos ejecutar todo, asi que hacemos un sudo su.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/6429fae8-5baa-4e2d-99be-e36fbd5c3943)

Y somos root!
