# Jenkins-PCF-Example
Despliega en Pivotal Cloud Foundry una aplicación Spring Boot mediante un Pipeline de Jenkins

## Introducción
Antes de desplegar una aplicación a PCF debemos validar los buildpacks disponibles, así determinamos si hay que utilizar una versión fija de Java y si estamos limitados a las herramientas de PCF o podemos obtener más de internet.

Una vez delimitadas nuestras versiones nos aseguramos que nuestro código funciona en ellas, configuramos el plugin de Cloud Foundry en Jenkins y creamos nuestro Pipeline.

### ¿Qué es un _buildpack_?
Permite compilar y empaquetar aplicaciones para que puedan ejecutarse en *Cloud Foundry*

Cuando se hace _push_ de una aplicación, PCF detecta el buildpack a utilizar, el buildpack examina el código, determina si es necesario descargar herramientas y configura la aplicación para poder comunicarse con los servicios asociados.

### Tipos de buildpacks PCF
Pivotal Cloud Foundry provee de algunos buildpacks con un nombrado especial para clasificarlos:

- Offline
Estos buildpacks no requieren conexión a internet, contienen versiones fijas de Java y herramientas, no podremos usar herramientas que no se encuentran en el buildpack.

Para Java estos buildpacks terminan en *_offline*. Para cualquier otra tercnología terminan en *-cached*.

- Online
Estos _buildpacks_ pueden acceder a internet para descargar herramientas o nuevas versiones de Java.

Para Java estos buildpacks terminan en *_online* y para cualquier otra tecnología no tienen una terminación especial.

- Custom
Los _buildpacks_ personalizados pueden ser nombrados de cualquier forma pero es buena practica utilizar una de las terminaciones anteriores para clasificarlos.

### Conocer nuestros _buildpacks_
Antes de tirar código hay que conocer los _buildpacks_ que tenemos disponibles en nuestra instalación de Pivotal Cloud Foundry:

```bash
cf buildpacks
```

```bash
Getting buildpacks...

buildpack                position   enabled   locked   filename                                              stack
staticfile_buildpack     1          true      false    staticfile_buildpack-cached-cflinuxfs3-v1.4.42.zip    cflinuxfs3
java_buildpack_offline   2          true      false    java-buildpack-offline-cflinuxfs3-v4.18.zip           cflinuxfs3
ruby_buildpack           3          true      false    ruby_buildpack-cached-cflinuxfs3-v1.7.38.zip          cflinuxfs3
nginx_buildpack          4          true      false    nginx_buildpack-cached-cflinuxfs3-v1.0.11.zip         cflinuxfs3
nodejs_buildpack         5          true      false    nodejs_buildpack-cached-cflinuxfs3-v1.6.49.zip        cflinuxfs3
go_buildpack             6          true      false    go_buildpack-cached-cflinuxfs3-v1.8.39.zip            cflinuxfs3
r_buildpack              7          true      false    r_buildpack-cached-cflinuxfs3-v1.0.9.zip              cflinuxfs3
python_buildpack         8          true      false    python_buildpack-cached-cflinuxfs3-v1.6.32.zip        cflinuxfs3
php_buildpack            9          true      false    php_buildpack-cached-cflinuxfs3-v4.3.76.zip           cflinuxfs3
dotnet_core_buildpack    10         true      false    dotnet-core_buildpack-cached-cflinuxfs3-v2.2.11.zip   cflinuxfs3
binary_buildpack         11         true      false    binary_buildpack-cached-cflinuxfs3-v1.0.32.zip        cflinuxfs3
binary_buildpack         12         true      false    binary_buildpack-cached-windows2012R2-v1.0.32.zip     windows2012R2
binary_buildpack         13         true      false    binary_buildpack-cached-windows2016-v1.0.32.zip       windows2016
binary_buildpack         14         true      false    binary_buildpack-cached-windows-v1.0.32.zip           windows
```

En este caso solo tenemos disponible un *buildpack* de Java offline y la versión es 4.18.

Para conocer las versiones de Java sorportadas y otras herramientas incluidas se pude revisar las notas del *release* https://github.com/cloudfoundry/java-buildpack/releases/tag/v4.18.

De quí lo relevante es que tenemos disponible Java 8, Java 11 y [Spring Boot CLI](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-cli.html) (aunque no lo usaremos).


## Código

Para este ejercicio usaremos [el código de la guía de Spring Boot](https://spring.io/guides/gs/spring-boot/) y [Gradle](https://gradle.org/install/) para la construcción.

