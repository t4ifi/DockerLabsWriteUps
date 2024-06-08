# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/cba7855f-1a1d-4197-8f57-8b82b17bbf69)
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

1. **22/tcp (SSH ):** OpenSSH 8.9p1
2. **80/tcp (HTTP ):** Apache httpd 2.4.59

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a una pagina web con una imagen de un huevo kínder.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3f73ae59-20a2-48b6-bf2c-9c59522ffb93)

# Fase 2- Exploración

#### FeroxBuster
```
200      GET        1l        2w       50c http://172.17.0.2/index.html
200      GET      157l      365w    30574c http://172.17.0.2/imagen.jpeg
200      GET        1l        2w       50c http://172.17.0.2/
```
Con feroxbuster encontramos un `index.html` pero nos redirige a la misma pagina, y la `imagen.jpeg` es la imagen misma que la descargamos.

#### .jpeg
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/82f4f2f9-67cf-4af0-80d7-efed1159632a)

A la imagen la escaneamos con la herramienta `exiftool`, y en la Descripción de la imagen nos da el usuario `borazuwarah`, y en el titulo nos dice la contraseña pero esta vacia.

# Fase 3- Explotación

#### Hydra
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/662e22d1-9112-4957-9e44-e148205141f8)
Realizamos fuerza bruta al puerto SSH, con hydra y nos encuentra la contraseña `123456`. 

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/e97151ba-f43a-482c-9144-d1a78ebe2d88)
Probamos y estamos dentro!

# Fase 4- Privilegios

#### sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/34fd42c4-2944-475c-9797-511d829d2852)
Despues de hacer el tratamiento de la TTY, y al hacer el comando `sudo -l`, tenemos permisos de ejecutar el binario `bash` como root.

#### Binario Bash
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/77a80572-2c7a-4398-81fa-37718a8f7dbb)

Gtfobins nos dice que para escalar privilegios con ese binario ejecutemos el comando `sudo bash` y listo, somos root!.


