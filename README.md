# Laboratorio - SSH

El laboratorio consiste en conectarse a un servidor de ssh y configurar
autenticación por llave y utilizar sftp para transferir un archivo

## Prerequisitos

* Mac OS/Linux/Windows con WSL2
* Terminal bash o zsh

## Práctica

El laboratorio dispone de dos servidores que comprenden la siguiente topología:

```
 .----------------.                            .----------------.
|  bastion server  |   ====[ firefall ]===>   |  private server  |
 `----------------´                            `----------------´
  bastin.protocolos.app                       ssh-privado.protocolos.app

```

El servidor bastion es públicamente accesible desde internet, sin embargo el 
servidor privado solo es accesible dentro de una red privada. Para poder acceder
al servidor privado es necesario hacerlo a través de bastion.

En el GES se encuentra la llave privada para conectarse al servidor bastion.
Al descargarla en la carpeta actual, uno puede conectarse al servidor bastion
utilizando el comando de ssh:

```
ssh -i galileo-bastion.pem bastion@bastion.protocolos.app
```

Si al intentar ejecutar el comando anterior recibes este mensaje: 

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for './galileo-bastion.pem' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "./galileo-bastion.pem": bad permissions
bastion@bastion.protocolos.app: Permission denied (publickey).
ssh_exchange_identification: Connection closed by remote host
```

Esto significa que la llave privada tiene demasiados permisos (otros usuarios
del equipo pueden ver el contenido de la llave). Para evitar este mensaje, los
permisos de la llave deben ser 600 (escritura y lectura para el usuario actual
únicamente).

Con los permisos en orden y ejecutando el comando anterior, la respuesta debe
mostrar: 

```
PTY allocation request failed on channel 0
Port forwarding only account.
Connection to bastion.protocolos.app closed.
```

Esto es un mensaje que nos indica que la llave solo está configurada para port
forwarding. Esto significa que uno no puede loguearse al servidor de bastion y 
obtener una terminal o ejecutar cualquier comando. Sin embargo, indica que 
lograste autenticar exitosamente con el servidor.

Una vez comprobando que se puede acceder al servidor bastion, se puede probar
el ingreso al servidor privado. Para esto, se ejecuta el siguiente comando 
(suponiendo que la llave de galileo-bastion.pem está en la carpeta actual).

```
# Recuerda reemplazar tu número de carné
# Usuario: galileo_<número de carné>
# Password: dirección de correo de galileo

ssh -o ProxyCommand="ssh -W %h:%p -i ./galileo-bastion.pem bastion@bastion.protocolos.app" galileo_<número de carné>@ssh-privado.protocolos.app
```

La contraseña asignada es el correo electrónico de galileo. Si ingresas 
correctamente tu contraseña, deberás ver el siguiente "prompt"

```
galileo_<número de carné>@galileo-private:~$
```

La opción `-o ProxyCommand` permite utilizar un servidor intermediario (bastion)
para alcanzar el servidor destino `ssh-privado.protocolos.app`. Al resolver el
dominio con el comando nslookup o dig, uno puede percatarse que dicho dominio
resuelve a una IP privada: 10.136.0.3. Esta IP es relativa al proxy utilizado. 
Lo que significa que aunque uno esté en otro segmento de red, o uno esté 
utilizando una IP pública para acceder al servidor bastión, este nos ayuda a 
alcanzar un servidor que no es accesible desde internet.

### Subiendo un archivo SFTP

SFTP es un subsistema de transferencia de archivos soportado por SSH. No se debe
confundir con FTPS (que es FTP sobre TLS). La mayoría de sistemas unix vienen 
con sftp incluido. 

Prueba crear un archivo de texto en la carpeta actual con el nombre ejemplo.txt
con cualquier contenido adentro. Usaremos este archivo para subirlo al servidor
privado.

Para abrir una sesión de sftp, se corre un comando muy 
similar al anterior:

```
sftp -o ProxyCommand="ssh -W %h:%p -i ./galileo-bastion.pem bastion@bastion.protocolos.app" galileo_<número  de carné>@ssh-privado.protocolos.app
```

Luego de ingresar la contraseña, esto abrirá una consola de sftp que lucirá así:

```
Connected to ssh-privado.protocolos.app.
sftp>
```

Para subir el archivo podemos ejecutar la instrucción:

```
# dentro de sftp
put ejemplo.txt
```

Esto subirá el archivo al servidor, mostrando un input similar a este:

```
Uploading ejemplo.txt to /home/galileo_<número de carné>/ejemplo.txt
ejemplo.txt                   100%   10     0.1KB/s   00:00
```

Una vez subido el archivo podemos corroborar que el archivo está arriba 
ejecutando `ls`. Una vez subido, podemos salir de la consola de sftp escribiendo
`exit`. 

### Usando SCP

Una alternativa a usar SFTP es utilizando el comando SCP (Secure Copy). Esta 
utilidad es más sencilla que su contraparte sftp en la que su única función es
copiar archivos desde o hacia el servidor que especifiquemos. Para poder probar
dicho comando podemos volver a subir el mismo archivo de ejemplo pero con un
nuevo nombre:

```
scp -o ProxyCommand="ssh -W %h:%p -i ./galileo-bastion.pem bastion@bastion.protocolos.app" ejemplo.txt galileo_<número de carné>@ssh-privado.protocolos.app:~/ejemplo2.txt
```

Notese como tanto `sftp` y `scp` permite la opción `-o ProxyCommand` para que se
pueda utilizar el servidor bastion como servidor intermediario.

## Ejercicio
1. Investiga el uso del comando `ssh-keygen` y genera un par de llaves 
público-privadas con una fortaleza de 4096 bits. No le coloques passphrase a la 
llave privada.
2. Ya sea configurandolo manualmente o usando el comando ssh-copy-id configura 
la llave que recien creaste para que puedas loguear al servidor privado 
(ssh-privado.protocolos.app) utilizando tu llave en vez de la contraseña.
3. Sube al home (carpeta `~`) de tu usuario una selfie utilizando sftp o scp.
4. Crea o sube un archivo llamado `respuestas.txt` en el home de tu usuario
respondiendo las siguientes preguntas:
    - Lista y explica los diferentes tipos de túneles de SSH que hay
    - Ejemplifica la sintaxis para levantar cada tipo de túnel
    - Qué tipo de tunel hizo posible que desde bastion pudiersa conectarte al 
    servidor privado

## Entrega
1. Sube al GES un archivo zip con el nombre `lab4-<número de carné>.zip` que
contenga la llave pública y la llave privada que generaste durante el ejercicio
2. Asegurate de haber cumplido con todos lo indicado en la práctica y en los 
ejercicios conectado al servidor provisto y que las llaves que envíaste al GES
funcionan ya que estas se utilizarán para evaluar lo realizado.
