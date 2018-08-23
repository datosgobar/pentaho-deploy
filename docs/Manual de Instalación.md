
# Manual de Instalación
Documento que detalla los pasos a realizar para instalar Pentaho.

## Índice
- [Introducción](#introducción)
- [Despliegue servidor de base de datos](#despliegue-servidor-de-base-de-datos)
   - [Pre requisitos](#pre-requisitos)
   - [Paso a paso](#paso-a-paso)
   - [Post requisitos](#post-requisitos)
- [Despliegue servidor BA Server (HTTP)](#despliegue-servidor-ba-server)
   - [Pre requisitos](#pre-requisitos-1)
   - [Paso a paso](#paso-a-paso-1)
- [Despliegue servidor DI Server (APP)](#despliegue-servidor-di-server)
   - [Pre requisitos](#pre-requisitos-2)
   - [Paso a paso](#paso-a-paso-2)

## Introducción
A continuación se detallan los pasos a seguir para la instalación inicial de los 3 servidores necesarios para la implementación de los proyectos que lleve a cabo el Ministerio cómo Tablero IME, Apertura, RUD, etc.
El proyecto involucra el despliegue de los siguientes servidores:
1. **Base de datos** donde se alojan
   - Metadata de Pentaho BA server (HTTP).
   - Metadata de Pentaho DI server (APP).
   - Datawarehouse.
2. **HTTP** donde se aloja el servidor web de Pentaho desde donde se accede a los tableros.
3. **APP** donde se ubican y ejecutan los procesos ETL necesarios para alimentar el Datawarehouse.

## Despliegue servidor de base de datos
### Pre requisitos
1. PostgreSQL instalado

### Paso a paso
1. Descargar el archivo **/base_de_datos/deploy_base_datos_ba_server.tar.gz** del repositorio GIT. Subirlo al servidor destino y descomprimirlo (de ahora en más se utiliza /ruta/archivo como la ubicación del mismo):
   1. **tar -xf /ruta/archivo/deploy_base_datos_ba_server.tar.gz**
2. Crear bases y usuarios para la metadata de pentaho BA server utilizando los siguientes comandos:
   1. **psql -U postgres -f /ruta/archivo/create_quartz_postgresql.sql**
   2. **psql -U postgres -f /ruta/archivo/create_repository_postgresql.sql**
   3. **psql -U postgres -f /ruta/archivo/create_jcr_postgresql.sql**
   >Los archivos contienen la creación de los usuarios con sus contraseñas. Se recomienda cambiarlas antes de ejecutar los comandos.
3. Restaurar los backups del entorno de desarrollo:
   1. **psql -U postgres -d quartz -f /ruta/archivo/quartz_postgresql.sql**
   2. **psql -U postgres -d hibernate -f /ruta/archivo/hibernate_postgresql.sql**
   3. **psql -U postgres -d jackrabbit -f /ruta/archivo/jackrabbit_postgresql.sql**
4. Descargar el archivo **/base_de_datos/deploy_base_datos_di_server.tar.gz** del repositorio GIT. Subirlo al servidor destino y descomprimirlo (de ahora en más se utiliza /ruta/archivo como la ubicación del mismo):
   1. **tar -xf /ruta/archivo/deploy_base_datos_di_server.tar.gz**
5. Crear bases y usuarios para la metadata de pentaho DI server utilizando los siguientes comandos:
   1. **psql -U postgres -f /ruta/archivo/create_di_quartz_postgresql.sql**
   2. **psql -U postgres -f /ruta/archivo/create_di_repository_postgresql.sql**
   3. **psql -U postgres -f /ruta/archivo/create_di_jcr_postgresql.sql**
   >Los archivos contienen la creación de los usuarios con sus contraseñas. Se recomienda cambiarlas antes de ejecutar los comandos.
6. Restaurar los backups del entorno de desarrollo:
   1. **psql -U postgres -d di_quartz -f /ruta/archivo/di_quartz_postgresql.sql**
   2. **psql -U postgres -d di_hibernate -f /ruta/archivo/di_hibernate_postgresql.sql**
   3. **psql -U postgres -d di_jackrabbit -f /ruta/archivo/di_jackrabbit_postgresql.sql**
7. Descargar el archivo **/base_de_datos/deploy_tablero_ministerial.tar.gz** del servidor de bases de datos. Subirlo al servidor destino y descomprimirlo (de ahora en más se utiliza /ruta/archivo como la ubicación del mismo):
   1. **tar -xf /ruta/archivo/deploy_tablero_ministerial.tar.gz**
8. Crear la base de datos tablero_ministerial y el usuario dueño de la base. Dentro de psql:
   1. `CREATE USER usr_ministerial WITH NOSUPERUSER ENCRYPTED PASSWORD 'cambiarpassword';`
   2. `CREATE DATABASE tablero_ministerial WITH OWNER usr_ministerial;`
9. Realizar un restore de la base de desarrollo:
   1. **psql -U usr_ministerial -d tablero_ministerial -f /ruta/archivo/tablero_ministerial.sql**

### Post requisitos
1. Permitir visibilidad y acceso a las bases de datos correspondientes desde los otros 2 servidores (servidor HTTP y APP).

## Despliegue servidor BA Server
### Pre requisitos
1. El servidor destino debe tener libre el puerto 8080.
2. El puerto 8080 debe ser accesible externamente.

### Paso a paso
1. Crear el usuario "pentaho" a nivel SO. Usuario que se deberá utilizar para realizar los próximos pasos.
2. Descargar el archivo **/instaladores/server80.tar.gz** del repositorio GIT. Subirlo al servidor destino y descomprimirlo en /home/pentaho (de ahora en más se utiliza /ruta/archivo como la ubicación del mismo):
   1. **tar -xf /home/pentaho/server80.tar.gz**
3. Modificar IP, puerto, usuario y clave utilizado en el **punto 2** de la sección "[Despliegue servidor de base de datos](#despliegue-servidor-de-base-de-datos)" de los siguientes archivos de configuración:
   1. /home/pentaho/server80/pentaho-server/pentaho-solutions/system/hibernate/**postgresql.hibernate.cfg.xml**
   2. /home/pentaho/server80/pentaho-server/pentaho-solutions/system/jackrabbit/**repository.xml**.
   >Revisar todo el archivo ya que hay varias secciones a modificar, para mayor información seguir los pasos detallados en la sección "[Step 3: Modify Jackrabbit Repository Information for PostgreSQL](https://help.pentaho.com/Documentation/8.0/Setup/Installation/Archive/PostgreSQL_Repository#Step_3:_Modify_Jackrabbit_Repository_Information_for_PostgreSQL "Step 3: Modify Jackrabbit Repository Information for PostgreSQL")".
   3. /home/pentaho/server80/pentaho-server/tomcat/webapps/pentaho/META-INF/**context.xml**. Para mayor información seguir los pasos detallados en la sección "[Step 2: Modify JDBC Connection Information in the Tomcat Context XML File](https://help.pentaho.com/Documentation/8.0/Setup/Installation/Archive/PostgreSQL_Repository#Step_2:_Modify_JDBC_Connection_Information_in_the_Tomcat_Context_XML_File)".
4. Todos los archivos con extensión SH deben tener permisos de ejecución
   - `chmod 774 -R /home/pentaho/server80/*.sh`
5. Iniciar el servicio de Pentaho BA Server
   - Ejecutar `sh start-pentaho.sh` (ubicado en ../server80/pentaho-server)
6. Validar Pentaho BA Server ingresando a http://IP:8080 desde cualquier navegador. Por ejemplo en el entorno de desarrollo es http://192.168.53.111:8080.
7. Ingresar con usuario **admin** con la contraseña entregada
8. En caso  de ser necesario instalar las licencias seguir los pasos detallados en la sección "[Manage Licenses Using PUC](https://help.pentaho.com/Documentation/8.0/Setup/Administration/Licenses)" de la documentación de pentaho.
9. Modificar los datos de conexión JDBC que se utilizan para conectarse al datawarehouse, según lo realizado en el **punto 2** de la sección "[Despliegue servidor de base de datos](#despliegue-servidor-de-base-de-datos)". Para ello seguir los siguientes pasos:
   1. Conectarse via SSH al servidor.
   2. Obtener los datos de conexión via API ejecutando: `curl -X GET --header 'Authorization: Basic usuario:password' "http://localhost:8080/pentaho/plugin/data-access/api/datasource/jdbc/connection/data_mart_ime" > data_mart_ime.json`, donde usuario:password debe estar codificado en Base64 (para más información consultar [REST API Reference](https://help.pentaho.com/Documentation/8.0/Developer_Center/REST_API#Basic_Authentication))
   3. Modificar las propiedades correspondientes en el archivo **data_mart_ime.json** obtenido en el punto anterior. Por ejemplo, para modificar la ip del servidor, debe modificarse **"hostname":"192.168.53.61"** por **"hostname":"IPNUEVA"**.
   4. Ejecutar el siguiente comando para actualizar los datos de conexión: `curl -v -X PUT --header 'Authorization: Basic usuario:password' --header 'Content-type: application/json' "http://localhost:8080/pentaho/plugin/data-access/api/datasource/jdbc/connection/data_mart_ime" --data @data_mart_ime.json`
10. Se recomienda cambiar la contraseña del usuario **admin**. Para ello seguir los siguientes pasos para hacerlo via API:
    1. Conectarse via SSH al servidor.
    2. Crear un archivo admin.json como el siguiente, adaptando los valores de las propiedades según sea necesario:
    **{"userName":"admin","newPassword":"password","oldPassword":"1234"}**
    3. Ejecutar el siguiente comando: `curl -v -X PUT --header 'Authorization: admin:passwordactual' --header 'Content-type: application/json' "http://localhost:8080/pentaho/api/userroledao/user" --data @admin.json`, donde admin:passwordactual debe estar codificado en Base64


## Despliegue servidor DI Server
### Pre requisitos
1. El servidor destino debe tener libre el puerto 9080.
2. El puerto 9080 debe ser accesible externamente.

### Paso a paso
1. Crear el usuario "pentaho" a nivel SO. Usuario que se deberá utilizar para realizar los próximos pasos.
2. Descargar el archivo **/instaladores/dis_61.tar.gz** del repositorio GIT. Subirlo al servidor destino y descomprimirlo en /home/pentaho (de ahora en más se utiliza /ruta/archivo como la ubicación del mismo):
   1. **tar -xf /home/pentaho/dis_61.tar.gz**
3. Modificar IP, puerto, usuario y clave utilizado en el **punto 5** de la sección "[Despliegue servidor de base de datos](#despliegue-servidor-de-base-de-datos)" de los siguientes archivos de configuración:
   1. /home/pentaho/dis_61/server/data-integration-server/pentaho-solutions/system/hibernate/**postgresql.hibernate.cfg.xml**
   2. /home/pentaho/dis_61/server/data-integration-server/pentaho-solutions/system/jackrabbit/**repository.xml**.
   >Revisar todo el archivo ya que hay varias secciones a modificar, para mayor información seguir los pasos detallados en la sección "[Step 3: Modify Jackrabbit Repository Information for PostgreSQL](https://help.pentaho.com/Documentation/6.1/0F0/0O0/035/010#Step_3:_Modify_Jackrabbit_Repository_Information_for_PostgreSQL "Step 3: Modify Jackrabbit Repository Information for PostgreSQL")".
   3. /home/pentaho/dis_61/server/data-integration-server/tomcat/webapps/pentaho/META-INF/**context.xml**. Para mayor información seguir los pasos detallados en la sección "[Step 2: Modify JDBC Connection Information in the Tomcat Context XML File](https://help.pentaho.com/Documentation/6.1/0F0/0O0/035/010#Step_2:_Modify_JDBC_Connection_Information_in_the_Tomcat_context.xml_File "Step 2: Modify JDBC Connection Information in the Tomcat Context XML File")".
4. Todos los archivos con extensión SH deben tener permisos de ejecución
   - `chmod 774 -R /home/pentaho/dis_61/*.sh`
5. Iniciar el servicio de Pentaho DI Server.
   - Ejecutar `sh start-pentaho.sh`
6. Validar Pentaho DI Server ingresando a http://IP:9080 desde cualquier navegador. Por ejemplo en el entorno de desarrollo es http://192.168.53.211:9080.
7. Ingresar con usuario **admin**. Solicitar contraseña a Datalytics
8. En caso  de ser necesario instalar las licencias seguir los siguientes pasos:
   1. Subir al servidor via sFTP el archivo "Pentaho PDI Enterprise Edition.lic"
   2. Ir a la ruta "/home/pentaho/dis_61/license-installer"
   3. Ejecutar `sh install_license.sh PATH/Pentaho PDI Enterprise Edition.lic` (Donde dice PATH se debe reemplazar por el directorio del servidor donde se encuentra la licencia)
   4. Todos estos pasos se deben realizar con el usuario pentaho
   5. Para mayor información ir a [Install License Keys Using the Command Line Interface](https://help.pentaho.com/Documentation/6.1/0P0/0U0/060)
9. Se recomienda cambiar la contraseña del usuario admin. Para ello se debe utilizar la herramienta cliente de PDI, **Spoon**. Loguarse al repositorio con usuario **admin** al repositorio de Pentaho y luego seguir los pasos de la siguiente documentación: [Change_User_Passwords](https://help.pentaho.com/Documentation/6.1/0H0/060/010/040/000#Change_User_Passwords)
