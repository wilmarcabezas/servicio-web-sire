# 🌐Servicio Web Sire: Despliegue de la aplicación

![Ejecucion comando Docker Compose](https://interactuemos.com/dockercompose.png)

### Descripción proceso
Basta con ir a una terminal y desde el directorio de la aplicacion correr el comando: **`docker-compose up`**. El comando **`docker-compose up`** es esencial en Docker Compose, utilizado para iniciar y ejecutar aplicaciones definidas en un archivo `docker-compose.yml`.

1. **Lectura del Archivo**: 
   - Docker Compose lee el `docker-compose.yml` que define servicios, redes y volúmenes.
   - 📁 Ubicación: Directorio actual o especificado con `-f`.

2. **Construcción de Imágenes**:
   - 🏗 Construye imágenes según el `Dockerfile` si es necesario.
   - 🔄 Solo se realiza si hay cambios o si no se ha construido previamente.

3. **Creación de Contenedores**:
   - 📦 Crea contenedores para cada servicio con las configuraciones establecidas.
   - ⚙ Configuraciones: Mapeo de puertos, variables de entorno, volúmenes, etc.

4. **Conexión de Redes**:
   - 🔗 Configura y conecta los contenedores a las redes definidas.

5. **Inicio de Contenedores**:
   - ▶️ Inicia los contenedores ejecutando comandos definidos en `Dockerfile` o `docker-compose.yml`.

6. **Logs en Consola**:
   - 📋 Muestra los logs de los contenedores en tiempo real en la consola.
   - 👁️‍🗨️ Útil para seguimiento y diagnóstico.

7. **Modo Detachado**:
   - 🌐 Opción `-d`: Ejecuta los contenedores en segundo plano y libera la terminal.
   - 🤫 No muestra logs en la consola.

####  <p style="color: red;"> El despliegue requiere que el host se encuentr con acceso a: 10.6.10.10 </p> 
<br>

---

<br>
<br>

# 🌐Servicio Web Sire: Descripcion tecnica de archivos base para el despliegue.

## 🐳 Dockerfile

### Descripción General
Este archivo contiene todas las instrucciones necesarias para construir una imagen de Docker. Estas imágenes son la base para crear contenedores que ejecutan aplicaciones o servicios específicos. 

El Dockerfile define un proceso paso a paso, comenzando desde una imagen base, e incluye instrucciones para instalar software necesario, copiar archivos, establecer variables de entorno, exponer puertos y definir comandos de inicio, entre otras acciones.

### 🏗️Etapa de Construcción

- `FROM maven:3.6.3-jdk-8 as builder`: Este comando inicia la construcción dela imagen de Docker. En este caso es Maven 3.6.3 con JDK 8, y la etiqueta que se usara parar  identificarla en esta etapa sera 'builder'.
<br/>
- `WORKDIR /opt/app`: Establece `/opt/app` como el directorio de trabajo en el contenedor. Esto quiere decir que la aplicacion y las futuras referencias del proyecto dentro del contenedor seran en /opt/app.
<br/>
- `COPY *.xml ./`: Copia todos los archivos `.xml` (como `pom.xml`) del directorio actual al directorio de trabajo en el contenedor.
<br/>
- `COPY ./src ./src`: Copia el directorio `src` del proyecto al directorio `src` en el contenedor. Este directorio contiene el código fuente. En la carpeta src encontramos:
  -  controller:
     -  el JwtAuthenticationController.java
     -  TransporteNacionalApiController.java
  -  dto:
     -  AuthRequest.java
     -  AuthResponse.java
     -  Datostransporte.java
     -  DetalleReporte.java
     -  ErrorResponse.java
     -  Pasajero.java
     -  ReporteResponse.java
     -  Ubicacion.java
  -  model:
     -  CargaReporteEmpresaModel.java
     -  CiudadModel.java
     -  CiudadPK.java
     -  DepartamentoModel.java
     -  DepartamentoPK.java
     -  EmpresaJuridicaModel.java
     -  EmpresaWwebApiModel.java
     -  PaisModel.java
     -  ReporteTransporteNacionalModel.java
     -  TipoDocumentoModel.java
  -  service:
     -  JwtUserDetailsRepository.java
     -  TransporteNacionalApiService.java
  -  util
     -  ServicioWebSireApplicationService.java  
<br/>
- `RUN mvn clean package -s settings.xml -Dmaven.test.skip=true`: Ejecuta Maven para limpiar, compilar y empaquetar la aplicación, utilizando `settings.xml` para la configuración y saltándose las pruebas.


### 🏃Etapa de Ejecución
- `FROM devexdev/8-jdk-alpine`: Inicia una nueva etapa con una imagen base de Java 8 sobre Alpine Linux.
  
- `WORKDIR /opt/app`: Establece `/opt/app` como el directorio de trabajo en esta etapa del contenedor.
  
- `COPY --from=builder /opt/app/target/*.jar /opt/app/app.jar`: Copia el archivo JAR compilado de la etapa 'builder' al directorio de trabajo.
- `EXPOSE 8080`: Indica que el contenedor escuchará en el puerto 8080.
  
- `ENTRYPOINT ["java","-jar","/opt/app/app.jar"]`: Define el comando para ejecutar la aplicación, arrancando el JAR con Java.  Este comando indica basicamente que el contenedor al iniciar debe ejecutar el comando, el cual a su vez es la aplicación en si misma.
<br/><br/>

## 🧩🐳Docker Compose

### Descripción General
Este archivo Docker Compose se utiliza para definir y ejecutar aplicaciones Docker con múltiples contenedores. Incluye configuraciones para construir y ejecutar un servicio web.

### Detalles de la Configuración

### Versión
- `version: '2'`
  - Especifica la versión 2 de la sintaxis del archivo Docker Compose.

### Servicios
- `services`
  - Define los servicios en la aplicación.

#### web-sire-service
- `web-sire-service`
  - El nombre del servicio.
  - `container_name: web-sire-service`
    - Asigna un nombre al contenedor para facilitar su referencia.
  - `build`
    - Define las instrucciones de construcción para la imagen del contenedor.
    - `dockerfile: Dockerfile`
      - Especifica el Dockerfile para construir la imagen.
  - `image: migracion-colombia/web-sire-service:latest`
    - El nombre y la etiqueta de la imagen que se utilizará.
  - `ports`
    - Mapea los puertos entre el contenedor y el host.
    - `8080:8080`
      - Mapea el puerto 8080 del contenedor al puerto 8080 del host.
  - `networks`
    - Redes a las que se conectará el servicio.
    - `- spring-cloud-network`
      - Conecta el servicio a la red `spring-cloud-network`.
  - `env_file: config.env`
    - Especifica un archivo de entorno con variables para el contenedor.

### Redes
- `networks`
  - Define las redes para los servicios.
  - `spring-cloud-network`
    - Nombre de la red definida por el usuario.
    - `driver: bridge`
      - Utiliza el controlador de red `bridge`.

<br/><br/>

## 📚Documentación del Archivo POM

### Descripción General
Este archivo `pom.xml` es utilizado por Maven para configurar un proyecto Java, especificando sus dependencias, plugins y otras configuraciones.

#### Información Básica
- `modelVersion`: `4.0.0`
- `groupId`: `co.gov.migracioncolombia.web`
- `artifactId`: `sire`
- `version`: `0.0.1-SNAPSHOT`
- `name`: Servicio Web SIRE
- `description`: Proyecto para la administración de registros de migración

### Configuración del Proyecto Padre
- `parent`
  - `groupId`: `org.springframework.boot`
  - `artifactId`: `spring-boot-starter-parent`
  - `version`: `2.7.14`

### Propiedades
- `java.version`: `1.8`

### Dependencias
Se incluyen dependencias para el web, seguridad, base de datos, pruebas, documentación y entorno de desarrollo.

### Construcción
- `sourceDirectory`: `src/main/java`
- Plugins usados incluyen:
  - `spring-boot-maven-plugin`
  - `maven-compiler-plugin`
  - `maven-surefire-plugin`

### Plugins Utilizados

#### 🌸Spring Boot Maven Plugin
- `groupId`: `org.springframework.boot`
- `artifactId`: `spring-boot-maven-plugin`
- Descripción: Este plugin es específico para proyectos Spring Boot. Facilita la empaquetación de la aplicación en un jar ejecutable y la ejecución de la aplicación durante el desarrollo.
- Configuraciones:
  - `execution`
    - `goals`
      - `repackage`: Para crear un jar ejecutable.
      - `build-info`: Genera información adicional de compilación.

#### 🛠️Maven Compiler Plugin
- `groupId`: `org.apache.maven.plugins`
- `artifactId`: `maven-compiler-plugin`
- Descripción: Este plugin se utiliza para compilar el código fuente del proyecto.
- Configuración:
  - `source`: `1.8`
  - `target`: `1.8`
  - Explicación: Establece la versión de Java (Java 8) para la compilación del código fuente y la compatibilidad del bytecode.

#### 🧪Maven Surefire Plugin
- `groupId`: `org.apache.maven.plugins`
- `artifactId`: `maven-surefire-plugin`
- Descripción: Utilizado para ejecutar pruebas unitarias. Este plugin asegura que todas las pruebas unitarias se ejecuten y valida sus resultados.
- Configuración:
  - `argLine`: `-Dspring.profiles.active=test`
  - Explicación: Configura el perfil de Spring activo para `test` durante la ejecución de las pruebas, lo cual puede ser útil para usar configuraciones específicas de prueba.


## 🛠️ Configuración de Maven - `settings.xml`

### 📄 Descripción General
El archivo `settings.xml` es utilizado por Maven para configurar el entorno de construcción y despliegue, incluyendo detalles sobre proxies, servidores, y repositorios. Este archivo en particular configura un proxy para Maven.

#### 🌍 Proxies
- `<proxies>`: Define la configuración de proxy para Maven.
  - `<proxy>`: Especifica un solo proxy.
    - `<id>`: `optional` - Identificador del proxy.
    - `<active>`: `true` - Indica que el proxy está activo.
    - `<protocol>`: `http` - Protocolo utilizado por el proxy.
    - `<username>`: `1032434237c` - Nombre de usuario para autenticación.
    - `<password>`: `Colomb1*` - Contraseña para autenticación.
    - `<host>`: `10.6.10.10` - Dirección del host del proxy.
    - `<port>`: `3128` - Puerto a través del cual el proxy se comunica.
    - `<nonProxyHosts>`: `localhost` - Hosts que no deben ser accedidos a través del proxy.

#### 🎯 Propósito
La configuración de proxy en `settings.xml` es crucial cuando se trabaja en entornos con restricciones de acceso a Internet. Permite que Maven se comunique con repositorios externos y herramientas a través de un proxy, asegurando el flujo normal de dependencias y paquetes necesarios para la construcción y despliegue de proyectos.
