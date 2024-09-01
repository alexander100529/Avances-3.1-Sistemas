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

1.  Se ha editado la línea `listen-on port 53` para que el servidor escuche solicitudes DNS en la IP `192.168.1.9`, que es el servidor, a través del puerto 53.

2. Se agregaron las siguientes líneas para configurar el reenvío de consultas DNS:
  ```
  forward first;
  forwarders { 8.8.8.8; };
  ```
   Estas líneas indican que el servidor intentará resolver las solicitudes localmente primero (forward first;). Si no puede, reenviará las solicitudes al servidor DNS especificado en forwarders, que en este caso 
   es 8.8.8.8, el servidor DNS de Google. De esta forma también podríamos tener acceso a internet.

3. Se modificó la línea allow-query y la ponemos como `any` para que el servidor DNS acepte consultas desde cualquier dirección IP.

![image](https://github.com/user-attachments/assets/cf9860a7-9080-49c1-869f-ac64409254b2)

Al fianl del archivo colocaremos las zonas que son explica esto todo corto 


![image](https://github.com/user-attachments/assets/4d9fc6f8-0e1e-495d-80a3-0681b3d35417)

























## ##

**Paso 3: habilitar e iniciar el servicio**  talves 

Tras la descarga e instalación de estos paquetes, para que el servicio DNS named arranque con cada inicio de CentOS 9 tendremos que habilitarlo con systemctl.

sudo systemctl enable named

## ##

**Paso 4: configurar el firewall**

En CentOS 9 el firewall suele estar activado por defecto, así que es necesario añadir una regla que permita el acceso al servicio DNS desde la red local y posteriormente se recarga el firewall.

sudo firewall-cmd --permanent --add-service=dns

sudo firewall-cmd --reload

## ##


...

allow-query { localhost; };

...

En un entorno de red local querremos que las consultas de los clientes sean tramitadas, así que tendremos que cambiar el valor de este bloque. Podríamos añadir direcciones IP individuales, rangos de red, etc. pero en este tipo de entornos es más fácil usar el valor any:

...

allow-query { any; };

## ##

**Paso 6: Comprobación de errores**

Guardamos los cambios y antes de iniciar Bind con la nueva configuración comprobamos que no haya errores en la configuración con el comando named-checkconf:

sudo named-checkconf

Si este comando no produce ninguna salida, la sintaxis de la configuración es correcta, y podemos arrancar el servicio Bind
sudo systemctl start named

## ##

**Paso 7: Verificar el estado del servicio**

sudo systemctl status named

## ##

**Paso 8: probar el servicio**

se usa para consultar información sobre nombres de dominio y registros DNS. En CentOS 9, el comando dig te permite realizar consultas DNS desde la línea de comandos.

dig @localhost example.com
## ##
**Paso 9:**

sudo nano /etc/bind/named.conf.local
zone "example.local" {
    type master;
    file "/etc/bind/zones/db.example.local";
};

## ##

**Paso 10:**

sudo mkdir -p /etc/bind/zones
sudo nano /etc/bind/zones/db.example.local
$TTL    604800
@       IN      SOA     ns1.example.local. admin.example.local. (
                           1         ; Serial
                      604800         ; Refresh
                       86400         ; Retry
                     2419200         ; Expire
                      604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.example.local.
@       IN      A       192.168.100.101
ns1     IN      A       192.168.100.101
asterisk IN      A       192.168.100.101

## ##

**Paso 11:**

sudo named-checkconf
sudo named-checkzone example.local /etc/bind/zones/db.example.local

## ##

**Paso 12:**

sudo systemctl restart named

## ##

**Paso 13:** 

dig asterisk.example.local

## ##
