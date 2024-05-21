# Fase 1- Tanteo

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/7d1fb338-5755-4fe4-885d-2833e77e6604)
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

1. **80/tcp (HTTP):** Apache httpd 2.4.58 (Debian)
2. **21/tcp(FTP)**: vsftpd 3.0.3
3. **22/tcp (SSH )**:OpenSSH 9.6p1 Debian 4
4. **3000/tcp(PPP)**:Ns
### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a acceder a la página por defecto de Apache en la máquina vulnerable, confirmando así el correcto funcionamiento del servidor web Apache en Debian. La página por defecto de Apache en Debian muestra el mensaje "Apache2 Debian Default Page".
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/d88d3c91-8b16-4cd0-85ca-59d49ee21e68)

# Fase 2- Exploración

#### 3000?

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/0d8fa58f-3d04-468f-95d7-0f5e819865b4)
Si vamos al resultado de nmap y nos fijamos el resultado del puerto 3000, vemos que tiene 
```
ingerprint-strings: 
  37   │ |   FourOhFourRequest: 
  38   │ |     HTTP/1.0 302 Found
  39   │ |     Cache-Control: no-cache
  40   │ |     Content-Type: text/html; charset=utf-8
  41   │ |     Expires: -1
  42   │ |     Location: /login
  43   │ |     Pragma: no-cache
  44   │ |     Set-Cookie: redirect_to=%2Fnice%2520ports%252C%2FTri%256Eity.txt%252ebak; Path=/; HttpOnly; SameSite=Lax
  45   │ |     X-Content-Type-Options: nosniff
  46   │ |     X-Frame-Options: deny
  47   │ |     X-Xss-Protection: 1; mode=block
  48   │ |     Date: Mon, 22 Apr 2024 20:53:33 GMT
  49   │ |     Content-Length: 29
  50   │ |     href="/login">Found</a>.
  ```
Esto podría ser un indicador de que por ese puerto esta corriendo alguna especie de pagina web?

#### FTP
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/14e5a890-98c1-432f-89e7-3d027c1ea89a)
En el escaneo se puede ser que el puerto 21 ftp esta abierto.
nos conectamos por ftp con el usuario `anonymous` sin dar ninguna contraseña.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ef48afb6-0ef5-4630-86ca-b1c33b103c33)
existe un archivo llamado `mantenimiento` pero no lo podemos descargar. no hay nada mas que hacer aquí.


#### Gobuster
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a14b3a4b-e5c2-4375-9eb7-ff1158dd3abf)
Encontramos un archivo llamado `maintenance.html` con Gobuster.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/65b57bd4-f209-4787-91d5-83677cf678f9)
`Website under maintenance, access is in /tmp/pass.txt` Básicamente nos dice que el sitio web esta en mantenimiento, que el acceso esta en /temp/páss.txt

#### Login?
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/2bd71ff5-ce6c-461b-be58-a2ca93cb1dbf)
La maquina victima por el puerto esta corriendo un panel de login.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/abdd7405-2aae-4d8d-a8f3-1caffdc0ab4a)
Probamos las credenciales `admin` `admin` y funcionaron.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b0fc9bb6-21bb-465e-b29e-2bf84dc0abaa)
Con wapalyzzer encontramos que es un Grafana 8.3.0 quizás haya algún exploit...

#### Fase 3- Explotación

#### ExploitDB
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3fe52174-30f9-49eb-91b6-1cde5230b1f3)
Encontramos en la pagina https://www.exploit-db.com/exploits/50581 que esa versión de Grafana es vulnerable a directory traversal.

#### Searchsploit
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/4786db56-9789-4012-b207-881f64e27e33)
con Searchsploit encontramos un exploit que explota la vulnerabilidad presente, lo descargamos.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/1c1c0708-8c12-48e3-a06c-207d8e28958e)
Lanzamos el exploit y nos pregunta que archivo queremos leer.

#### Directory Traversal
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/70293ba8-a6de-4110-8b4d-8b61a90f7be2)
Le indicamos /etc/passwd y se puede ver un /home/freddy, un usuario llamado freddy.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/62e7fe34-48de-4b24-896e-381ea1aead2c)
y al indicarle la dirección /tmp/pass.txt vemos una especie de hash o algo así.

#### ssh
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a69bc01c-51b1-46ff-bc5a-60443df46efc)
probamos el usuario y la contraseña y estamos dentro.

#### Fase 4- Privilegios
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/61c2cbf9-32cf-4876-bb15-d62835afece4)
Al ejecutar `sudo -l` vemos que tenemos permisos de /usr/bin/python3 /opt/maintenance.py

#### GTFOBins
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/681724b3-a2ae-4ccf-9b23-b9c186556bb7)
Si vamos a Gtfobins nos sale que para escalar privilegios solamente hace falta el comando `sudo python -c 'import os; os.system("/bin/sh")'`

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3145276d-eefa-4759-af3d-1bec6fd8fff2)
Y vemos que al ejecutarlo funciona correctamente. Ya somos root!!






