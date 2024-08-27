# ![image](https://github.com/user-attachments/assets/c8e698d0-d2e4-4c6c-a0ea-885d90addfb6)

<span style="color: red;">Este texto es rojo.</span> Procedimiento seguido para configurar el DNS.				
## ##
Paso 1: actualizar:
Para instalar el servidor DNS Bind en CentOS 9 usaremos los paquetes de la distribución, así que como primer paso actualizaremos la información de los repositorios.

sudo dnf update
## ##
Paso 2: instalar paquetes
El paquete que necesitamos es bind, aunque también añadiremos el paquete bind-utils para poder usar algunas herramientas interesantes.

sudo dnf install bind bind-utils
## ##
Paso 3: habilitar e iniciar el servicio:
Tras la descarga e instalación de estos paquetes, para que el servicio DNS named arranque con cada inicio de CentOS 9 tendremos que habilitarlo con systemctl.

sudo systemctl enable named
## ##
Paso 4: configurar el firewall
En CentOS 9 el firewall suele estar activado por defecto, así que es necesario añadir una regla que permita el acceso al servicio DNS desde la red local y posteriormente se recarga el firewall.

sudo firewall-cmd --permanent --add-service=dns

sudo firewall-cmd --reload
## ##
Paso 5: configurar el servidor dns:

Para configurar el servicio DNS Bind en CentOS 9 debemos saber que el archivo principal es /etc/named.conf.

nano /etc/named.conf

Para ello buscaremos las directivas listen-on y listen-on-v6:

...

listen-on port 53 { 127.0.0.1; };

listen-on-v6 port 53 { ::1; };

...

Y las desactivaremos:

...

#listen-on port 53 { 127.0.0.1; };

#listen-on-v6 port 53 { ::1; };

Aunque ya podría recibir peticiones desde red, el servicio sólo responde a peticiones locales debido a la configuración de la directiva allow-query:

...

allow-query { localhost; };

...

En un entorno de red local querremos que las consultas de los clientes sean tramitadas, así que tendremos que cambiar el valor de este bloque. Podríamos añadir direcciones IP individuales, rangos de red, etc. pero en este tipo de entornos es más fácil usar el valor any:

...

allow-query { any; };

Paso 6: Comprobación de errores:
Guardamos los cambios y antes de iniciar Bind con la nueva configuración comprobamos que no haya errores en la configuración con el comando named-checkconf:

sudo named-checkconf

Si este comando no produce ninguna salida, la sintaxis de la configuración es correcta, y podemos arrancar el servicio Bind
sudo systemctl start named

Paso 7: Verificar el estado del servicio:

sudo systemctl status named

Paso 8: probar el servicio:

se usa para consultar información sobre nombres de dominio y registros DNS. En CentOS 9, el comando dig te permite realizar consultas DNS desde la línea de comandos.

dig @localhost example.com
