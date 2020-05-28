# Jenkins-PCF-Example

Despliega en Pivotal Cloud Foundry una aplicación Spring Boot mediante un Pipeline de Jenkins

## Introducción

Antes de desplegar una aplicación a PCF debemos validar los buildpacks disponibles, así determinamos si hay que utilizar una versión fija de Java y si estamos limitados a las herramientas de PCF o podemos obtener más de internet.

Una vez delimitadas nuestras versiones nos aseguramos que nuestro código funciona en ellas, configuramos el plugin de Cloud Foundry en Jenkins y creamos nuestro Pipeline.

### ¿Qué es un _buildpack_?

Permite empaquetar aplicaciones para que puedan ejecutarse en *Cloud Foundry*

Cuando se hace _push_ de una aplicación, PCF detecta el buildpack a utilizar, el buildpack examina el código, determina si es necesario descargar herramientas y configura la aplicación para poder comunicarse con los servicios asociados.

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

Para conocer las versiones de Java sorportadas y otras herramientas incluidas se pude revisar las notas del [*release*](https://github.com/cloudfoundry/java-buildpack/releases/tag/v4.18).

De quí lo relevante es que tenemos disponible Java 8, Java 11 y [Spring Boot CLI](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-cli.html) (aunque este no lo usaremos).


## Código

### Microservicio

Para este ejercicio usaremos [el código de la guía de Spring Boot](https://spring.io/guides/gs/spring-boot/) y [Gradle](https://gradle.org/install/) para la construcción.

El micro servicio de ejemplo solo envía un saludo: _Greetings from Spring Boot!_ cada que recibe una petición GET.

### Gradle wrapper

Es recomendable usar el wrapper de Gradle (gradlew) para la construcción, así podemos distribuir el código con la versión de exacta de Gradle que usamos para dearrollar y si otra persona necesita construir nuestro proyecto no tiene que instalarlo, el wrapper se encarga de descargarlo y configurar lo necesario para hacer el build. 

Para más detalles puedes revisar la [documentación oficial](https://docs.gradle.org/4.10.3/userguide/gradle_wrapper.html) y si aún no tienes muy claro que ventajas tiene usarlo puedes revisar [este post](https://medium.com/@bherbst/understanding-the-gradle-wrapper-a62f35662ab7).

### Plugin de Gradle

Para construir aplicaciones Spring Boot existe un [plugin oficial para Gradle](https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/gradle-plugin/reference/html/), incluso se agrega al usar el [Spring initializr](https://start.spring.io/) y seleccionar la opción de _Gradle Project_.

- EL plugin desactiva la tarea `jar` y en su lugar crea la tarea `bootJar` para crear un _fat jar_ (un jar con todo lo necesario para ejecutar la aplicación)
- Vincula la tarea `bootRun` a la tarea `assemble`
- Crea la tarea `bootRun` para ejecutar la aplicación.

Para más detalles sobre el uso del plugin puedes leer [este post](https://tomgregory.com/unleashing-the-spring-boot-gradle-plugin/).

## Cloud Foundry

Una vez construido el código podemos desplegar nuestro jar en cloud foundry con la herramienta  `cf`.

Para realizar el despliegue requerimos crear un archivo *manifest.yml*, en este especificamos la ubicación del jar, el nombre de nuestra aplicación y el número de instancias.

Existen más valores de configuración en el archivo manifest: memoria, buildpack, serviciso, etc...

Para más detalles del archivo manifest puedes revisar [la documentación oficial](https://docs.cloudfoundry.org/devguide/deploy-apps/manifest-attributes.html). 

## Pipeline

Para desplegar nuestra aplicación podemos ejecutar gradlew e instalar `cf` en el agente de Jenkins que ejecutara nuestras tareas.

Pero para tener una mejor integración con Jenkins usaremos los plugins de [Gradle](https://plugins.jenkins.io/gradle/) y [Cloud Foundry](https://plugins.jenkins.io/cloudfoundry/).

### Pruebas

Primero queremos ejecutar las pruebas unitarias de nuestro código, el reporte se genera por defecto en la ruta `build/reports/tests/test`

```groovy
withGradle {
	sh './gradlew check'
}
```

### Construcción

Si todo sale bien pasamos a creer nuestro _fat jar_, por defecto se genera en `build/libs`

```groovy
withGradle {
	sh './gradlew bootJar'
}
```

### Despliegue

Probablemente no queremos desplegar de todas las ramas, en mi caso quiero solo quiero desplegar cambios en la rama *master* así que agregué una condición en el `stage` de despliegue.

```groovy
when { branch 'master' }
```

Si el pipeline se ejecuta desde la rama master se usa el plugin de Cloud Fondry para desplegar el jar. 

```groovy
pushToCloudFoundry(
  target: 'api.run.pivotal.io',
  organization: 'organization.name-org',
  cloudSpace: 'development',
  credentialsId: 'pcfdev_user',
  manifestChoice: [manifestFile: 'manifest.yml']
)
```

Es importante configurar el archivo manifest.yml, debemos colocar la ruta en la que se guarda el jar y en mi caso uso Java 11 así que tengo que colocar una variable de entorno para indicar la versión del JRE. [Más detalles aquí]().

```yaml
---
applications:
  - name: springboot-testeando-2020
    path: build/libs/springboot-0.0.1-SNAPSHOT.jar
    instances: 1
    env:
      JBP_CONFIG_OPEN_JDK_JRE: '{ jre: { version: 11.+}}'
```

### Reportes

Para conservar los reportes en el pipeline hay que hacer uso de la sección `post`, los pasos que se definan aquí siempre se ejecutan incluso si el pipeline falla.

Podemos guardar el jar generado para inspeccionarlo después (en la sección artifacts del pipeline).

```groovy
archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true
```

Guardar los resultados en formato JUnit (sección Tests del pipeline)

```groovy
junit 'build/test-results/test/*.xml'
```


También podemos guardar el reporte de Gradle (sección artifacts)

```groovy
publishHTML([
	reportDir: 'build/reports/tests/test',
	reportFiles: 'index.html',
	reportName: 'Tests Report',
	keepAll: true,
	allowMissing: false, 
	alwaysLinkToLastBuild: false
])
```

## Ejecución

Si hacemos un _push_ a Github podremos ver un indicador verde o rojo a un lado del _commit id_ con el estado del build, al dar clic nos enviara a la Jenkins.

Si hacemos un pull request también podremos ver el estadodel branch de origen y del pull request.

![checks](https://resources.github.com/assets/img/whitepapers/gh-required-status.png)