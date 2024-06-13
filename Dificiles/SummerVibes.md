# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/9524c8d4-c41a-45a0-9651-101f1c916e26)
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

1. **22/tcp(SSH):** OpenSSH 8.2p1
2.  **80/tcp(HTTP):** Apache httpd 2.4.52

### Puerto 8080
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Se encontró la pagina de apache2 por defecto de Ubuntu.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/270b1e40-85f7-4859-9eac-dbd6743f4687)

# Fase 2- Exploración

#### ViewSource
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/1301ea2d-98b7-4361-a00e-2dd2ad40ee15)

En la ultima linea del codigo se encontró un texto que dice lo siguiente:
```python
<!-- cms made simple is installed here - Access to it - cmsms -->
En Español:
<!-- cms made simple está instalado aquí - Acceso al mismo - cmsms -->
```

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/dee4d56d-437d-4079-9b89-5a5bc5d53123)
Encontramos cms.

#### Home
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/176808a3-12b5-45ed-bf44-8bd5f5604503)
En `home` encontramos que podemos acceder al `/admin` del cms.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/9a48a1a9-eafa-46bc-b441-8ceefc090513)

Y ya estamos aquí.

# Fase 3- Explotación

#### Hydra
Usamos el siguiente comando con Hydra para realizar fuerza bruta al usuario `admin`.
```python
hydra -l admin -P /usr/share/wordlists/rockyou.txt "http-post-form://172.17.0.2/cmsms/admin/login.php:username=^USER^&password=^PASS^&loginsubmit=Submit:User name or password incorrect"
```

Unos segundos después...
```python
[80][http-post-form] host: 172.17.0.2   login: admin   password: chocolate
```
Tenemos el usuario `admin` con la contraseña `chocolate`.

#### CMS
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/978a0538-c2ff-413e-aa96-fcecd02be514)

Nos dirigimos al apartado de `Extensiones` y `Tags Personalizados`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/93d51949-8ff4-48af-9301-c347eb9e7e4c)

Creamos una nueva Tag con cualquier nombre y probamos codigo malicioso en php.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/78500c45-c134-4efb-9251-4f40a8e314df)

Ejecutamos y funciono! Tenemos un RCE.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/33e6babf-e1f6-4aad-a994-a6ed45e8d16a)

Nos enviamos una reverse shell en bash.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/47ae22a5-3afa-48dd-a27b-e73c190b93d3)

Y estamos dentro!

# Fase 4- Privilegios

#### sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/e8447fe6-caec-4d47-b155-9fe1b1968167)

Despues de hacer un tratamiento de la TTY y no funciona el comando sudo, y no encontramos nada raro con find.

#### su root
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/9b818327-a843-46cb-be8d-52f62301aa45)

De pura suerte probe la misma contraseña para `cms` para `root` y funciono! Somos root! JAJAJAJ
