# Pruebas unitarias con Docker, JUnit 5, Amazon Corretto 21 y Gradle Wrapper

Esta guía explica cómo ejecutar las pruebas de un proyecto Java existente en un contenedor Linux, desde macOS —incluido un MacBook Air con Apple Silicon M4— o Windows 11. Utiliza `build.gradle.kts`, Gradle Wrapper y el flujo validado con Gradle 9.0.0.

Los comandos marcados como **ambos sistemas** se ejecutan sin cambios en Terminal de macOS o PowerShell de Windows 11. También puedes utilizar la terminal integrada de IntelliJ, comprobando qué intérprete está seleccionado. Ejecuta los comandos uno por uno desde la raíz del proyecto, salvo indicación contraria.

## 1. Comprender qué se ejecuta dentro de Docker

```text
macOS / Windows 11 (equipo anfitrión)
└── Docker Desktop
    └── Entorno Linux virtualizado
        └── Contenedor basado en amazoncorretto:21
            ├── Archivos y utilidades de Amazon Linux
            ├── Amazon Corretto 21 (distribución de OpenJDK)
            ├── Gradle, descargado por el Wrapper
            └── Proyecto Java y pruebas JUnit 5
```

El equipo sigue usando macOS o Windows. Docker Desktop proporciona el kernel Linux mediante su entorno virtualizado; el contenedor utiliza los archivos y herramientas Linux de la imagen. En Windows, esta guía utiliza el backend WSL 2.

`FROM amazoncorretto:21` selecciona una imagen que incluye el JDK y su base Linux. No instala Corretto en el sistema anfitrión. IntelliJ permanece fuera del contenedor.

## 2. Instalar y comprobar Docker Desktop

### macOS, incluido Apple Silicon M4

1. Descarga [Docker Desktop para Mac](https://docs.docker.com/desktop/setup/install/mac-install/).
2. En un Mac M4, elige **Apple silicon**; en un Mac Intel, elige **Intel chip**.
3. Abre el instalador, arrastra Docker a Aplicaciones y abre Docker Desktop.
4. Completa la configuración y espera a que el motor esté listo.

No necesitas forzar `--platform linux/amd64` para este proyecto Java: Docker selecciona la variante compatible de la imagen. En Apple Silicon se utiliza normalmente Linux ARM64; en equipos Intel/AMD, Linux AMD64. Si tu proyecto añade bibliotecas nativas, estas también deben ser compatibles con la arquitectura elegida.

### Windows 11

1. Comprueba que la virtualización del equipo esté habilitada.
2. Si WSL no está instalado, abre PowerShell **como administrador** y ejecuta:

   ```powershell
   wsl --install
   ```

3. Reinicia si Windows lo solicita. Actualiza y comprueba WSL:

   ```powershell
   wsl --update
   wsl --status
   ```

4. Instala [Docker Desktop para Windows](https://docs.docker.com/desktop/setup/install/windows-install/) con el instalador correspondiente a tu equipo.
5. Abre Docker Desktop y activa **Use the WSL 2 based engine** cuando esa opción esté disponible. Utiliza **contenedores Linux**. Si el menú ofrece **Switch to Linux containers**, selecciónalo.

Consulta los requisitos y la configuración en la [documentación del backend WSL 2](https://docs.docker.com/desktop/features/wsl/). Los comandos posteriores se ejecutan desde PowerShell; no es necesario abrir una terminal Ubuntu.

### Comprobación en ambos sistemas

```sh
docker --version
docker compose version
docker info
docker run --rm hello-world
```

`docker info` debe mostrar información del servidor y `OSType: linux`. `hello-world` comprueba que Docker puede descargar y ejecutar una imagen. Docker Desktop incluye [Docker Compose](https://docs.docker.com/compose/install/); se usa `docker compose`, con espacio.

Necesitas acceso a Internet para descargar la imagen, instalar `findutils` y obtener Gradle y las dependencias. Para ejecutar exclusivamente dentro de Docker no necesitas Java ni Gradle instalados en el anfitrión, siempre que el proyecto ya incluya el Wrapper completo.

## 3. Preparar la estructura del proyecto

Utilizaremos `UnitTest1` como nombre de ejemplo:

```text
UnitTest1/
├── .dockerignore
├── .gitattributes
├── Dockerfile
├── compose.yaml
├── build.gradle.kts
├── settings.gradle.kts
├── gradlew
├── gradlew.bat
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
└── src/
    ├── main/
    │   └── java/
    └── test/
        └── java/
            └── ExampleTest.java
```

Conserva los archivos y las pruebas existentes. Crea los archivos de configuración indicados en esta guía con IntelliJ o tu editor, respetando sus nombres exactos; `Dockerfile` no tiene extensión.

Entra a la carpeta del proyecto. Ajusta estas rutas de ejemplo a tu ubicación real.

**macOS:**

```sh
cd "$HOME/IdeaProjects/UnitTest1"
```

**Windows 11, PowerShell:**

```powershell
Set-Location "$HOME\IdeaProjects\UnitTest1"
```

La raíz es la carpeta que contiene `build.gradle.kts` y `gradlew`.

## 4. Comprobar JUnit 5 y Gradle Wrapper

Si las pruebas ya funcionan, conserva tu configuración. Este es un `build.gradle.kts` mínimo completo para un proyecto Java sencillo; integra lo necesario sin eliminar dependencias propias:

```kotlin
plugins {
    java
}

repositories {
    mavenCentral()
}

java {
    toolchain {
        languageVersion.set(JavaLanguageVersion.of(21))
    }
}

dependencies {
    testImplementation(platform("org.junit:junit-bom:5.10.2"))
    testImplementation("org.junit.jupiter:junit-jupiter")
    testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}

tasks.test {
    useJUnitPlatform()
}
```

La versión `5.10.2` es una versión fija de JUnit 5 para este ejemplo, no una afirmación de que sea la más reciente. Si el proyecto ya tiene otra versión de JUnit 5 validada, mantenla. El BOM alinea sus dependencias, Jupiter aporta las pruebas y `useJUnitPlatform()` permite ejecutarlas con Gradle. Véase [pruebas Java en Gradle](https://docs.gradle.org/current/userguide/java_testing.html).

Un `settings.gradle.kts` mínimo puede contener:

```kotlin
rootProject.name = "UnitTest1"
```

Si necesitas una prueba de comprobación, crea `src/test/java/ExampleTest.java`:

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertEquals;

class ExampleTest {
    @Test
    void sumaDosNumeros() {
        assertEquals(4, 2 + 2);
    }
}
```

El Wrapper debe incluir **los cuatro archivos**: `gradlew`, `gradlew.bat`, `gradle-wrapper.jar` y `gradle-wrapper.properties`. Deben estar disponibles en el repositorio, incluido el JAR.

En el flujo validado, `gradle/wrapper/gradle-wrapper.properties` apunta a:

```properties
distributionUrl=https\://services.gradle.org/distributions/gradle-9.0.0-bin.zip
```

Esta es una línea del archivo: conserva sus demás propiedades. El Wrapper descarga la versión indicada; no utiliza una instalación global de Gradle del anfitrión.

Si el Wrapper ya está completo, no tienes que regenerarlo. Solo si falta y ya dispones de una instalación local de Gradle compatible y Java, puedes generarlo desde la raíz, en ambos sistemas:

```sh
gradle wrapper --gradle-version 9.0.0 --distribution-type bin
```

## 5. Asegurar los saltos de línea para Linux

Dentro del contenedor se ejecuta **`gradlew`**, incluso cuando el anfitrión es Windows. `gradlew.bat` solo corresponde a una ejecución local desde Windows.

Guarda `gradlew` con saltos de línea **LF**. En IntelliJ puedes abrir el archivo y cambiar su indicador de finales de línea de `CRLF` a `LF`, y guardar. `chmod` concede permisos, pero no corrige saltos de línea.

Para mantener este comportamiento al compartir el proyecto por Git, crea o amplía `.gitattributes`:

```gitattributes
gradlew text eol=lf
gradlew.bat text eol=crlf
Dockerfile text eol=lf
*.yaml text eol=lf
*.kts text eol=lf
```

Agregar estas reglas no convierte automáticamente una copia de `gradlew` que ya esté abierta con CRLF: conviértela y guárdala en el editor antes de construir la imagen.

## 6. Crear el Dockerfile

Guarda este contenido completo en `Dockerfile`, en la raíz:

```dockerfile
FROM amazoncorretto:21

RUN yum install -y findutils

WORKDIR /app

COPY . .

RUN chmod +x gradlew

CMD ./gradlew test
```

| Instrucción | Propósito |
| --- | --- |
| `FROM amazoncorretto:21` | Usar la imagen que incluye Amazon Corretto 21 y su base Linux. |
| `RUN yum install -y findutils` | Instalar utilidades, entre ellas `xargs`, utilizada por el script del Wrapper. |
| `WORKDIR /app` | Establecer `/app` como directorio de trabajo dentro de la imagen. |
| `COPY . .` | Copiar el proyecto desde el contexto de construcción a `/app`, aplicando `.dockerignore`. |
| `RUN chmod +x gradlew` | Dar permiso de ejecución al Wrapper dentro de Linux. |
| `CMD ./gradlew test` | Ejecutar las pruebas al iniciar el contenedor. |

`RUN` actúa al construir la imagen; `CMD` define qué ejecutar al iniciar un contenedor. Por tanto, construir esta imagen **no ejecuta las pruebas**.

La imagen descargada en el flujo original utilizaba Amazon Linux 2023. En esa base, `yum` se mantiene como enlace a `dnf`, por lo que la instrucción validada sigue siendo válida. La etiqueta `21` puede actualizarse: verifica tu imagen con el paso 10. Fuente: [imágenes oficiales de Corretto](https://github.com/corretto/corretto-docker).

## 7. Crear .dockerignore

Guarda en `.dockerignore`:

```dockerignore
.git/
.gradle/
build/
**/build/
.idea/
*.iml
out/
.DS_Store
Thumbs.db
```

Así evitas copiar cachés locales, resultados anteriores, metadatos del editor e historial Git. **No excluyas `gradle/`, `gradlew`, `gradlew.bat` ni todos los archivos `*.jar`**, porque el Wrapper necesita `gradle-wrapper.jar`.

`.gradle/` es una caché; `gradle/` contiene archivos del proyecto. Son carpetas diferentes. `.dockerignore` controla la copia a la imagen, no los archivos que Git versiona ni el montaje del reporte.

## 8. Construir la imagen y ejecutar las pruebas

**Ambos sistemas, desde la raíz:**

```sh
docker build -t junit-test .
docker run --rm junit-test
```

`-t junit-test` asigna el nombre a la imagen. El punto final indica que la carpeta actual es el contexto de construcción. `docker run` crea un contenedor y ejecuta `./gradlew test`; `--rm` elimina ese contenedor al terminar, incluso si las pruebas fallan. La imagen permanece disponible.

Es normal que la primera ejecución muestre la descarga de `gradle-9.0.0-bin.zip` y de las dependencias. Esta guía no persiste la caché de Gradle, por lo que nuevos contenedores pueden repetir esas descargas.

### Interpretar el resultado

Con pruebas correctas, una salida típica es:

```text
> Task :compileJava
> Task :compileTestJava
> Task :test

BUILD SUCCESSFUL
```

Esto indica que la tarea terminó correctamente. Comprueba también que se hayan encontrado pruebas: `test NO-SOURCE` no demuestra que tus pruebas se ejecutaron. El reporte permite confirmar cuántas se ejecutaron y cuántas se omitieron.

Si una aserción falla, puede aparecer:

```text
ExampleTest > sumaDosNumeros() FAILED
    org.opentest4j.AssertionFailedError

1 test completed, 1 failed
> Task :test FAILED

BUILD FAILED
```

En ese caso, Docker sí ejecutó las pruebas y Gradle detectó el fallo. `BUILD FAILED` también puede deberse a compilación, dependencias o configuración: lee la causa concreta. No cambies una aserción solo para obtener éxito; revisa el comportamiento esperado.

Para consultar el código de salida, inmediatamente después de `docker run` o `docker compose run`:

**macOS:**

```sh
echo $?
```

**Windows 11, PowerShell:**

```powershell
$LASTEXITCODE
```

`0` significa éxito; otro valor indica un fallo del proceso o de Docker.

En esta ejecución básica, el reporte queda dentro del contenedor en `/app/build/reports/tests/test/index.html`. Al eliminarse el contenedor se pierde ese archivo. El siguiente paso permite conservarlo.

## 9. Persistir el reporte con Docker Compose

Crea `compose.yaml` en la raíz:

```yaml
services:
  tests:
    build: .
    image: junit-test
    volumes:
      - ./build/reports:/app/build/reports
```

`tests` es el nombre del servicio. `build: .` utiliza el Dockerfile del proyecto e `image: junit-test` asigna el mismo nombre que en la construcción manual. El servicio hereda `CMD ./gradlew test` del Dockerfile.

El montaje conecta una carpeta del anfitrión con una del contenedor:

```text
Anfitrión:  ./build/reports
                   ↕
Contenedor: /app/build/reports
```

La ruta relativa se resuelve desde la carpeta del archivo Compose en este proyecto. Funciona en macOS y Windows sin escribir rutas absolutas específicas. Compose puede crear la carpeta de origen si no existe. Este montaje conserva los reportes, pero no la caché de Gradle ni todo el directorio `build`. Consulta la [referencia de servicios y volúmenes de Compose](https://docs.docker.com/reference/compose-file/services/).

**Ambos sistemas:**

```sh
docker compose config
docker compose build tests
docker compose run --rm tests
```

El primer comando valida y muestra la configuración; el segundo construye la imagen; el tercero ejecuta las pruebas en un contenedor temporal con el montaje. `--rm` elimina el contenedor, pero los archivos escritos en la carpeta compartida permanecen. Véase [`docker compose run`](https://docs.docker.com/reference/cli/docker/compose/run/).

Cuando se genere el reporte tendrás:

```text
UnitTest1/
└── build/
    └── reports/
        └── tests/
            └── test/
                ├── index.html
                ├── classes/
                ├── packages/
                ├── css/
                └── js/
```

Abre **`build/reports/tests/test/index.html`** en el navegador. Conserva la carpeta completa para mantener los enlaces y estilos.

**macOS:**

```sh
open ./build/reports/tests/test/index.html
```

**Windows 11, PowerShell:**

```powershell
Start-Process .\build\reports\tests\test\index.html
```

También puedes abrir el archivo desde Finder o el Explorador. La URL `file:///app/build/reports/tests/test/index.html` del mensaje de Gradle describe una ruta **del contenedor**, no una ruta que debas pegar directamente en el navegador del anfitrión.

El reporte se conserva cuando fallan aserciones, siempre que la tarea de pruebas haya podido generarlo. Si falla antes por compilación o descarga, puede no existir un reporte nuevo; uno anterior puede seguir en la carpeta. Revisa su fecha y la salida de la ejecución.

### Después de modificar código o pruebas

El código se copia durante la construcción: el volumen solo comparte reportes. Reconstruye para incluir cambios recientes, en ambos sistemas:

```sh
docker compose build tests
docker compose run --rm tests
```

También puedes hacerlo en un solo comando:

```sh
docker compose run --build --rm tests
```

Si utilizas el flujo sin Compose, repite:

```sh
docker build -t junit-test .
docker run --rm junit-test
```

Ese último `docker run` no utiliza el volumen definido en Compose y no conserva reportes en el anfitrión.

## 10. Inspeccionar Amazon Linux y Java

**Ambos sistemas:**

```sh
docker run --rm amazoncorretto:21 cat /etc/os-release
```

Si no existe localmente, Docker descarga primero la imagen. En la ejecución validada se obtuvo, entre otros campos:

```text
NAME="Amazon Linux"
VERSION="2023"
ID="amzn"
VERSION_ID="2023"
PRETTY_NAME="Amazon Linux 2023.12.20260817"
```

La revisión exacta puede cambiar. Amazon Linux proviene de la construcción de `amazoncorretto:21`, aunque tu Dockerfile no tenga una línea `FROM amazonlinux`.

Otros comandos útiles, iguales en ambos sistemas:

```sh
docker image inspect amazoncorretto:21
docker run --rm amazoncorretto:21 java -version
docker run --rm amazoncorretto:21 uname -m
docker run --rm junit-test cat /etc/os-release
docker run --rm junit-test java -version
docker run --rm junit-test ./gradlew --version
```

`docker image inspect` solo inspecciona imágenes locales: si aparece `No such image`, ejecuta primero el comando anterior con `docker run` o descarga explícitamente:

```sh
docker pull amazoncorretto:21
```

Los comandos sobre `junit-test` comprueban el entorno de la imagen que construiste. Un `pull` posterior de la imagen base no modifica automáticamente esa imagen ya construida. `uname -m` muestra la arquitectura; para identificar la distribución utiliza `/etc/os-release`, no `uname`.

## 11. Resolver errores frecuentes

| Síntoma | Qué comprobar o corregir |
| --- | --- |
| Docker no puede conectarse al daemon | Abre Docker Desktop, espera a que arranque y repite `docker info`. |
| Windows intenta usar contenedores Windows | Cambia Docker Desktop a contenedores Linux y comprueba `OSType: linux`. |
| `xargs is not available` | Confirma `RUN yum install -y findutils` y reconstruye la imagen. |
| `Permission denied` al ejecutar `gradlew` | Confirma `RUN chmod +x gradlew` después de `COPY . .` y reconstruye. |
| `bad interpreter`, `^M` o `gradlew: not found` aunque exista | Convierte `gradlew` a LF, guarda y reconstruye. |
| No encuentra `GradleWrapperMain` | Comprueba `gradle/wrapper/gradle-wrapper.jar` y que `.dockerignore` no lo excluya. |
| Gradle no usa Java 21 | Comprueba la imagen y el bloque de toolchain; ejecuta `docker run --rm junit-test java -version`. |
| No aparecen los últimos cambios | Reconstruye: el código fuente está copiado en la imagen. |
| No aparece el HTML en el equipo | Usa Compose, verifica el montaje y que la tarea `test` haya generado un reporte. |
| Docker deniega el acceso a la carpeta compartida | Revisa los permisos del proyecto y el acceso a archivos de Docker Desktop. |
| Error al descargar Gradle o dependencias | Revisa Internet, proxy y certificados de tu entorno; lee el error concreto. |

Para obtener más detalle de un fallo de Gradle y conservar el reporte, ejecuta en ambos sistemas:

```sh
docker compose run --rm tests ./gradlew test --stacktrace --info
```

## 12. Secuencia habitual

Una vez creados los archivos, desde la raíz del proyecto, en **macOS y Windows 11**:

```sh
docker compose build tests
docker compose run --rm tests
```

Después abre `build/reports/tests/test/index.html` con el comando correspondiente a tu sistema del paso 9. Repite la construcción después de editar el proyecto.

Esta guía documenta el flujo confirmado en la conversación original y añade las instrucciones para Windows 11. No representa una nueva ejecución de las pruebas en ambos equipos. El ejemplo supone un proyecto Java de un solo módulo y la ubicación de reportes predeterminada de Gradle; en proyectos multimódulo deben adaptarse las rutas de cada módulo.
