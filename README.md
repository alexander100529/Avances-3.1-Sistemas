# ![image](https://github.com/user-attachments/assets/c8e698d0-d2e4-4c6c-a0ea-885d90addfb6)

## Procedimiento Seguido para Configurar el DNS en CentOS Stream 9
				
## ##
**Paso 1: Actualizar**

Para instalar el servidor DNS Bind en CentOS 9 usaremos los paquetes de la distribución, así que como primer paso actualizaremos la información de los repositorios.

`sudo dnf update -y`

**Output:**


## ##
**Paso 2: Instalar paquetes**

El paquete que necesitamos es bind, aunque también añadiremos el paquete bind-utils para poder usar algunas herramientas interesantes.

`sudo dnf install bind bind-utils`

## ##
**Paso 3: Configuración del archivo named.conf**

Primero, navegue hasta el directorio /etc de la siguiente manera:

`cd /etc`

Copiamos el archivo named.conf para tener un respaldo en caso de cometer algún error."

`cp named.conf named.conf.original`

Editamos el archivo named.conf, que es el archivo principal de configuración de BIND. Este archivo define cómo se comporta el servidor DNS y especifica las zonas que el servidor gestionará, utilizando el siguiente comando:

`nano named.conf` 

1.  Se ha editado la línea listen-on port 53 para que el servidor escuche solicitudes DNS en la IP `192.168.1.9`, que es el servidor, a través del puerto 53.

2. Se agregaron las siguientes líneas para configurar el reenvío de consultas DNS:
  ```
  forward first;
  forwarders { 8.8.8.8; };
  ```
   Estas líneas indican que el servidor intentará resolver las solicitudes localmente primero (forward first;). Si no puede, reenviará las solicitudes al servidor DNS especificado en forwarders, que en este caso 
   es 8.8.8.8, el servidor DNS de Google. De esta forma también podríamos tener acceso a internet.

3. Se modificó la línea allow-query y la ponemos como `any` para que el servidor DNS acepte consultas desde cualquier dirección IP.

![image](https://github.com/user-attachments/assets/cf9860a7-9080-49c1-869f-ac64409254b2)

Al final del archivo de configuración, se definen las zonas para resolver nombres de dominio y direcciones IP:

Zona directa (laboratorio.local): Resuelve nombres de dominio a direcciones IP.
```
zone "laboratorio.local" IN {
    type master;
    file "directa.laboratorio.local";
};
```
Zona inversa (1.168.192.in-addr.arpa): Resuelve direcciones IP a nombres de dominio.

```
zone "1.168.192.in-addr.arpa" IN {
    type master;
    file "inversa.laboratorio.local";
};
```
![image](https://github.com/user-attachments/assets/4d9fc6f8-0e1e-495d-80a3-0681b3d35417)

Por último guardamos los cambios y verificamos la configuración con el siguiente comando:

`named-checkconf /etc/named.conf`

![image](https://github.com/user-attachments/assets/80813b0d-e88f-4a27-ad11-fd2bd23b01f6)

Si este comando no produce ninguna salida, la sintaxis de la configuración es correcta.

## 
**Paso 4: Configuración de la zona directa**

Nos dirigimos al directorio `named` con el siguiente comando:

`cd /var/named`

Creamos el archivo para la zona directa con el comando:

`nano directa.laboratorio.local`

Luego, agregamos los siguientes datos al archivo:

![image](https://github.com/user-attachments/assets/fd9c6615-7c11-4be4-9746-153b2f5ada28)

Dicho archivo configura los registros DNS para el dominio laboratorio.local. Establece que el servidor DNS principal es srvcentos.laboratorio.local con la IP 192.168.1.9. Además, define alias para www, web, y ftp, todos apuntando a srvcentos.laboratorio.local.

Guardamos los cambios y comprobamos la sintaxis del archivo de zona con el comando named-checkzone, indicando la ip del servidor y el nombre del archivo:

`named-checkzone 192.168.1.9 directa.laboratorio.local`

![Captura de pantalla 2024-09-01 175715](https://github.com/user-attachments/assets/23d25c6c-7971-474b-98d6-1261e3848408)


## ##

**Paso 5: Configuración de la zona inversa**

Creamos el archivo para la zona inversa con el comando:

`nano inversa.laboratorio.local`

Luego, agregamos los siguientes datos al archivo:

![Captura de pantalla 2024-09-01 175246](https://github.com/user-attachments/assets/78aeeaff-4f0b-4b9d-9748-661232935a23)

Este archivo configura la zona inversa DNS para el dominio laboratorio.local. Establece que srvcentos.laboratorio.local es el servidor de nombres y define registros PTR para resolver direcciones IP a nombres de dominio, incluyendo www, web, ftp, y el dominio principal laboratorio.local, todos apuntando a srvcentos.laboratorio.local.

Guardamos los cambios y comprobamos la sintaxis del archivo de zona con el comando named-checkzone, indicando la ip del servidor y el nombre del archivo:

![image](https://github.com/user-attachments/assets/7ddc9943-09e8-46eb-8e27-f762d5dbe23e)

## ##

**Paso 6: Cambio de grupo de las zonas**

En el mismo directorio /var/named, cambiamos el grupo de los archivos que creamos para asegurar que el servidor DNS (BIND) pueda acceder a ellos correctamente. Utilizamos los siguientes comandos:
 
```
chgrp named directa.laboratorio.local 
chgrp named inversa.laboratorio.local
```
Para comprobar el cambio de grupo colomamos el comando: 

`ls -l`

![image](https://github.com/user-attachments/assets/e728c9e4-42b7-48ac-bbd5-fc3d4fc4bf04)


## ##

**Paso 7: Verificar resolv.conf**

Se abre el archivo con el siguiente comando:

`nano /etc/resolv.conf`

![image](https://github.com/user-attachments/assets/b16794da-0061-45b3-8994-aa07c6dfd397)

Se verifica que la línea nameserver tenga la IP del servidor DNS. Este archivo debe apuntar a la IP del servidor DNS para que las solicitudes de resolución de nombres se dirijan correctamente al servidor DNS configurado.

## ##

**Paso 8: Actualizar el archivo hosts**

Se abre el archivo con el siguiente comando:

`nano /etc/hosts`

Se añade una línea con la IP del servidor y el dominio.

![image](https://github.com/user-attachments/assets/90886761-5c4b-45a6-a5d0-4c309506e91c)

Esto permite que el sistema resuelva el dominio a la IP localmente, lo que es útil para pruebas y configuraciones internas.


## ##

**Paso 9: Configurar el firewall para el servicio DNS**

En CentOS 9 el firewall suele estar activado por defecto, así que es necesario añadir una regla que permita el acceso al servicio DNS desde la red local:

`sudo firewall-cmd --permanent --zone=public --add-service=dns`

Aplicamos los cambios recargando la configuración del firewall:

`sudo firewall-cmd --reload`

Para verificar que el servicio DNS se ha añadido correctamente al firewall, se coloca el siguiente comando:

`firewall-cmd --list-all`

![image](https://github.com/user-attachments/assets/5abc0712-c1c7-4025-898c-6e76fe7fdf32)

**Paso 10: Activación y Verificación del Servicio DNS**

Una vez completados todos los pasos anteriores, habilite el servicio named (para que se inicie automáticamente al arrancar el sistema), inicie el servicio y verifique el estado utilizando los siguientes comandos:

```
sudo systemctl enable named
sudo systemctl start named
sudo systemctl status named
```

![image](https://github.com/user-attachments/assets/ac17de67-464a-4170-b740-46bc8aa0ff69)

Con esto podemos comprobar que el servicio está funcionando correctamente.

**Paso 11: Verificaciones del servicio DNS**
Por último, se probó el servicio DNS con el comando nslookup. Primero, se realizó una consulta usando la IP del servidor para verificar si se resuelven correctamente los nombres de dominio asociados a esa IP. Luego, se realizó una consulta usando un nombre de dominio para comprobar si el servidor devuelve la IP correcta. 

![image](https://github.com/user-attachments/assets/c697a430-2a3f-4764-b920-41e5bcb623c8)

Esto confirma que el servicio DNS está funcionando adecuadamente tanto para la resolución inversa (IP a nombre) como para la resolución directa (nombre a IP).








