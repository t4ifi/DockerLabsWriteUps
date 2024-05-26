# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/cc971242-980f-4a8f-8721-5bce9355d3a6)
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

1. **22/tcp(SSH):** OpenSSH 9.6p1 (Ubuntu)
2.  **80/tcp(HTTP):** Apache httpd 2.4.58
### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Y se encontró una pagina sobre información de seguridad.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c6afe27f-3aea-4f73-baa9-b09b356f29be)

# Fase 2- Exploración

### GoBuster
No se encontró nada interesante con GoBuster. Pero se encontró en la pagina el siguiente mensaje:
```shell
¡Importante! Despido de empleado
Juan fue despedido de la empresa por enviar un correo con la contraseña a un compañero.

Firmado: Carlota, Departamento de ciberseguridad
```
Ya tenemos dos posibles usuarios para hacer un ataque de fuerza bruta con hydra al puerto ssh `Carlota` y `Juan`.

# Fase 3- Explotación

### Carlota
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/4aca0421-97bb-4dd8-bfe8-1ddc64f12961)
Se encontró la contraseña del usuario `carlota` que era `babygirl` esto se hizo mediante fuerza bruta con hydra.

# Fase 4- Privilegios

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a251e597-23ea-47e1-930d-d77f116f141a)

Despues de completar el tratamiento de la TTY y entrar al directorio `/Desktop/fotos/vacaciones` tenemos un archivo llamado `imagen.jpg`.

### Imagen.jpg
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/84631dfa-34a4-4e97-bf00-77ab33a49a82)

```sql
scp carlota@172.17.0.2:/directorio/al/archivo/imagen.jpg /directorio/destino/
```
Con scp nos pasamos el archivo de la maquina victima a nuestra maquina.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3d3e1b4e-eb22-4486-bea4-5291df6d8000)

Es una foto normal sin nada a simple vista.

### datos
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/da7aa6c9-f0ed-4f53-9189-5f8fce909918)

```sql
steghide --extract -sf imagen.jpg
```
Este comando se utiliza para extraer datos ocultos de un archivo de imagen utilizando la herramienta Steghide. nos extrajo un secret.txt que tiene pinta de estar encriptado con base64.

### Base 64
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/8fea2810-fa89-4c0c-ad4c-5a2bec653092)

```python
`echo "ZXNsYWNhc2FkZXBpbnlwb24=" | base64 -d; echo`
```
Este comando decodifica el texto en base64 "ZXNsYWNhc2FkZXBpbnlwb24=" y lo muestra como texto plano en la terminal.

### Ruby
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/4aff376d-2f4e-46fe-b6f8-ae0f3c2dcc85)

Al migrar a otro usuario vemos que tenemos permisos de ejecutar el binario ruby.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3a88b3d2-ac86-4be9-bcf5-e66af192e2ce)

Si nos dirigimos a Gtfobins nos dice que para escalar privilegios con este binario tenemos que ejecutar el comando:
```sql
sudo /usr/bin/ruby -e 'exec "/bin/bash"'
```
y ya somos root!.
