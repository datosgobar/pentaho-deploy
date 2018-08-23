# Manual de Configuración Inicial de Entornos
Documento que detalla los pasos a realizar para la configuración de los entornos Pentaho.

## Índice
- [Introducción](#introducción)
- [Servidor BA Server (HTTP)](#ba-server-o-servidor-http)
- [Servidor DI Server (APP)](#di-server-o-servidor-app)

## Introducción
Se asume que los 3 entornos ya se encuentran instalados.

### BA Server o Servidor HTTP
Estos pasos se deben realizar por única vez.

1. Configurar la parámetro _requestParameterAuthenticationEnabled_ con el valor _true_ en el archivo `../pentaho-server/pentaho-solutions/system/security.properties`
2. Agregar al archivo `../pentaho-server/tomcat/conf/server.xml`la línea `<Valve className="org.apache.catalina.valves.rewrite.RewriteValve" />` antes del último tag `</host>`
3. Agregar al archivo `../pentaho-server/tomcat/conf/Catalina/localhost/rewrite.config`la línea `RewriteRule ^/tableros/(api|management|swagger-resources|v2/api-docs|h2-console)($|/.*$) /tableros-back/$1$2`. Crear el archivo en caso que no exista.
4. Reiniciar el servicio de pentaho
5. Se deben crear un conjunto de roles y usuarios que son utilizados por los tableros.
	 - Crear el role **tablero** que tenga permisos de *leer contenido*
	 - Crear el usuario **tablero** que pertenezca al *role tablero*
	 - Crear el usuario **admin_cambios** que pertenezca al *role Administrator*
	 - Para crear usuarios y roles se debe contar con un usuario administrador. Para saber como realizar estos puntos ir al documento [Manage User and Roles in PUC](https://help.pentaho.com/Documentation/8.0/Products/User_Console/Manage_Users_Roles)

### DI Server o Servidor APP

Para poder realizar ejecuciones remotas o via API en el DI Server se deben realizar los siguientes pasos:

 1. Editar el archivo */home/pentaho/.kettle/repositories.xml* donde se debe agregar lo siguiente:

    > `<repository> <id>PentahoEnterpriseRepository</id>
    > <name>NOMBRE_REPOSITORIO</name> <description>DESCRIPCIÓN</description>
    > <repository_location_url>http&#x3a;&#x2f;&#x2f;IP_SERVER_APP&#x3a;PUERTO&#x2f;pentaho-di</repository_location_url>  <version_comment_mandatory>N</version_comment_mandatory>
    > </repository>`

donde:

 - **NOMBRE_REPOSITORIO:** es el nombre del repositorio; por ejemplo *Modernizacion_DEV*
 - **DESCIPCIÓN:** descripción (opcional); por ejemplo *Entorno de desarrollo*.
 - **IP_SERVER_APP:** se debe reemplazar por la IP del servidor; por ejemplo 192.168.70.201
 - **PUERTO:** se debe reemplazar por el puerto donde esta escuchando el DI Server; por ejemplo 9080

**NOTA:** Esta configuración también se debe realizar en el entorno local _(PC de los desarrolladores)_; y para evitar conflictos se recomienda poner exactamente el mismo nombre del repositorio. El archivo se encuentra en la carpeta home del usuario  del sistema operativo con el cual esta logueado y en caso de no existir lo debe crear.

 2. Editar el archivo */home/pentaho/.kettle/kettle.properties* y agregar la siguiente definición de variable:

		> `CARPETA_PROYECTOS=/home/pentaho/proyectos`

 3. Editar el archivo */home/pentaho/dis_61/server/data-integration-server/pentaho-solutions/system/kettle/slave-server-config.xml* y agregar la siguientes lineas debajo del tag *<slave_config>*:


>     `<repository>
>       <name>NOMBRE_REPOSITORIO</name>
>       <username>USUARIO</username>
>       <password>CLAVE</password>
>     </repository>`
donde:

 - **NOMBRE_REPOSITORIO:** Debe ser el mismo nombre definido en el punto 1.
 - **USUARIO:** Nombre de usuario para que se conecte al repositorio de Pentaho. El mismo debe ser administrador o al menos tener permisos de ejecución.
 - **CLAVE:** Clave del usuario definido en el item anterior.

 4. Reiniciar el servicio de Pentaho; para saber como iniciar/detener el servicio leer el documento [Start and Stop the Data Integration Server](https://help.pentaho.com/Documentation/6.1/0H0/070/010/000)
 5. En caso de necesitar dar de alta usuarios y roles para acceder al repositorio de Pentaho DI Server leer el documento [Managing Content in the Pentaho Repository Explorer](https://help.pentaho.com/Documentation/6.1/0L0/0Y0/040/010)

### Cargas Masivas

Para poder ejecutar procesos que realicen cargas masivas, es necesario que se instale **psql** en el servidor.
Además debe existir el archivo `.pgpass` (ver estructura del archivo [aquí](https://www.postgresql.org/docs/9.4/static/libpq-pgpass.html)) en la carpeta home del usuario que ejecuta Pentaho.

El archivo `.pgpass` debe tener sólo permisos de lectura y escritura (600).


### Configuración de variables
A continuación se explica los pasos para configurar: host(s), puerto(s), usuario(s) y contraseña(s), que utiliza el proceso de integración de datos para realizar las siguientes actividades:

 - Operar sobre las bases de datos del proyecto.
 - Transferencia de archivos entre servidores a través del protocolo FTP
 - Notificaciones vía email a usuarios (casos de éxito y error)
 - Para esto, se utiliza un archivo _.properties_ que contiene diferentes propiedades de configuración del sistema con la estructura:
	 - NOMBRE_PROPIEDAD=VALOR_ASIGNADO
	 - ORBEON_HOST=IP_SERVIDOR_BDD, por ejemplo aquí estamos definiendo la IP del servidor de base de datos en la propiedad "ORBEON_HOST".
	 - El símbolo # sirve para comentar una línea de texto, por ejemplo: *#Envío de E-mail*
 - El archivo se encuentra ubicado en la carpeta config de cada proyecto: */home/pentaho/proyectos/**NOMBRE_PROYECTO**/configuracion* del servidor de integración de datos con el nombre: _variables.properties_ donde el **NOMBRE_PROYECTO** es el nombre del proyecto. Por ejemplo */home/pentaho/proyectos/tablero_IME/configuracion*

#### Ofuscar password
Se recomienda, evitar escribir passwords de manera plana en archivos de configuración pues podrían quedar disponibles para el uso de personas no autorizadas, entonces un primer nivel para aumentar la seguridad del proceso de autenticación es ofuscar el password. A continuación explicaremos como ofuscar el password del usuario de base de datos.

 - Vamos a utilizar la herramienta **Encr** de Pentaho Data Integration, ubicada en la ruta: *../design-tools/data-integration* y una vez ubicados sobre el directorio ejecutar el siguiente comando
	 - `./encr.sh -kettle password_a_ofuscar`
 - Recuerde reemplazar **password_a_ofuscar** por la contraseña del usuario de base de datos. Cuando realice el comando anterior, el resultado será algo similar a esto: *Encrypted 2be98afc86aa7f2e4cb79ce71dc91abdf* (es un ejemplo).
 - Ahora debemos copiar el password "ofuscado" en el archivo de configuración (debe copiar todo el texto que retorna la herramienta **Encr**, es decir incluyendo la palabra *Encrypted*).
