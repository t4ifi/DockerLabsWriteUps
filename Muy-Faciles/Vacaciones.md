# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a063e71f-8d4f-49aa-b928-a62d05a2af79)
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

1. **22/tcp (SSH):** OpenSSH 7.6p1 (Ubuntu)
2. Apache httpd 2.4.29 (Ubuntu)
### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Al acceder al puerto 80, se ve una pantalla completamente en blanco.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/f547dda7-2af3-48d2-be75-e57b18dce610)

# Fase 2- Exploración

#### CTRL + U
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3830b676-1d2f-42ac-ac8b-c56023f152f1)
Al hacer CTRL + U para ver el codigo nos encontramos con un mensaje <!-- De : Juan Para: Camilo , te he dejado un correo es importante... -->
Esos podrían ser 2 potenciales usuarios para el puerto 22/ssh.

#### Gobuster
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/8a49adb6-65d4-4201-bcc0-0478558b5126)
Con la ayuda de Gobuster encontramos un directorio de estado 301 llamado `javascript`

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/2b0c2887-288d-4306-bb36-8408a2ab678b)
Al ir al directorio/archivo nos encontramos con que no tenemos permiso para ver el recurso.

# Fase 3- Explotación
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/59211256-842a-472c-8d78-ed6d89fe3166)
Con hydra aplicamos fuerza bruta al usuario `camilo` y encontramos la contraseña `password1`.

#### SSH
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/860fa0a6-8339-441e-b50f-c2f6b966b1d8)
y accedemos dentro.

# Fase 4- Privilegios
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/da17f24d-3728-4538-8d9c-e70bb9a26ce9)
Se hizo el tratamiento TTY correspondiente.
Dentro del directorio  `/var/mail/camilo` vemos un archivo llamado `correo.txt` 

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/eb2d7123-ccb4-4dc1-a479-7efd80c6e117)
Dentro del correo.txt había el siguiente texto: 
```
Hola Camilo,

Me voy de vacaciones y no he terminado el trabajo que me dio el jefe. Por si acaso lo pide, aquí tienes la contraseña: 2k84dicb
```

Teniendo la contraseña intentamos acceder al usuario juan con la credencial obtenida.

#### Binario Ruby
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ffc3fb4d-8bf4-49b1-935a-45f79693588f)
Con el usuario juan tenemos permisos de ejecución el binario ruby.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/16092bbc-2820-4b5c-a0f5-abe46cace0e3)
Nos dirigimos a Gtfobins y vemos que para explotar el binario hace falta el comando 
`sudo ruby -e 'exec "/bin/sh"'`

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c4b1605f-0b73-461b-a3f7-44bbc5eeca58)
Lo ejecutamos y funciono! Somos root.
