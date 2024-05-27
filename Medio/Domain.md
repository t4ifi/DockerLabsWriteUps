# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/d3c1e6f4-c91d-4178-be89-e487fb80a573)
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

1. **80/tcp(HTTP):** Apache httpd 2.4.52 (Ubuntu)
2.  **139/tcp(SMB):** Samba smbd 4.6.2
3. **445/tcp(SMB):** Samba smbd 4.6.2
### Puerto 8080
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Sobre que es y para que sirve samba.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ea892e02-691f-41b9-bf47-98e4a17e0e4b)

# Fase 2- Exploración

#### SMBMAP
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/2af67ce9-f971-4019-840b-55b1e9e89092)
Usando smbmap, vemos que no tenemos acceso a los recursos compartidos desde el usuario `anonymous`. El comando:
```python
smbmap -H 172.17.0.2 -u 'anonymous' -p ''
```

#### RPCCLIENT
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/9c3e7f31-553b-4d39-a45d-ebd04dbf2045)

El comando:`rpcclient -U "" -N 172.17.0.2` se usa para interactuar con un servidor SMB sin autenticación:
- `srvinfo`: Muestra información del servidor.
- `querydispinfo`: Lista las cuentas de usuario.
En este caso podemos ver que tenemos dos usuarios: `bob` y `james`.

# Fase 3- Explotación

#### bob
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/137c5c6e-a9aa-4363-bdfe-69be7ee9300b)
Con crackmapexec usamos el comando:
```python
crackmapexec smb 172.17.0.2 -u bob -p /usr/share/wordlists/rockyou.txt
```
Para realizar un ataque de fuerza bruta al protocolo smb a la ip `172.17.0.2` al usuario `bob` y nos encontró la contraseña `star`

#### SMB Explotation
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/54929cee-181b-40b0-9577-15b2cc36f05a)
Usamos el comando "crackmapexec smb 172.17.0.2 -u bob -p star --shares" enumera las carpetas compartidas en un servidor SMB usando las credenciales de usuario y contraseña proporcionadas.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b9a99f54-6264-4bc7-bba7-94bba7280bfa)
El comando: `smbclient -U 'bob' //172.17.0.2/html` Se usa para acceder a un recurso compartido SMB llamado "html" en la dirección IP 172.17.0.2 con las credenciales de usuario "bob" 
Y con el comando `put php.php` subimos una reverse shell ubicada en mi mismo directorio para la maquina victima.

#### Web
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/e0114d36-a9b1-40bb-8fdd-200fb3f3593e)

Llamamos al archivo malicioso `php.php` desde la web.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/1c6255d0-77c6-4849-8e1c-afdc601c9e24)
Y nos llega la reverse shell.

# Fase 4- Privilegios

####  find
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3788f4d6-208c-4ad1-918e-36fcbcd94fb3)

Nos cambiamos del usuario www.data al usuario bob, y vemos que tenemos permisos de ejecución del binario nano.

#### root
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/37ed1641-385b-46c1-825f-39bb42078c38)

Vamos al archivo `passwd` y le sacamos la X al usuario root.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/fc0c6114-df58-49f4-ad09-2a129e869100)

y al hacer `su root` ya somos root!

