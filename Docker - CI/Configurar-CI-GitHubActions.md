# Configuración de CI con JUnit 5, Gradle, Docker y GitHub Actions

## 1. Objetivo

Configurar un proceso de **Integración Continua (CI)** para un proyecto
Java que utiliza:

-   Java 21 con Amazon Corretto
-   JUnit 5
-   Gradle
-   Docker
-   GitHub
-   GitHub Actions

Cada vez que se realice un `push` a la rama `main` o se cree un
`pull request` hacia `main`, GitHub Actions ejecutará automáticamente
las pruebas unitarias dentro de Docker y almacenará el reporte HTML
generado por Gradle como artefacto.

El flujo será:

``` text
IntelliJ
   |
   | git push
   v
GitHub
   |
   v
GitHub Actions
   |
   +-- Checkout repository
   |
   +-- Build Docker image
   |
   +-- Run Docker container
   |       |
   |       +-- Amazon Corretto 21
   |       +-- Gradle
   |       +-- JUnit 5
   |
   +-- Copy test report
   |
   +-- Upload artifact
   |
   v
Resultado del pipeline
```

------------------------------------------------------------------------

## 2. Prerrequisitos

Antes de configurar CI, el proyecto debe tener las pruebas funcionando
localmente.

La estructura base puede ser:

``` text
UnitTest1/
├── gradle/
├── src/
│   ├── main/
│   │   └── java/
│   └── test/
│       └── java/
├── .dockerignore
├── .gitignore
├── build.gradle.kts
├── compose.yaml
├── Dockerfile
├── gradlew
├── gradlew.bat
└── settings.gradle.kts
```

También se debe haber comprobado que:

``` bash
docker build -t junit-test .
docker run --rm junit-test
```

ejecuta correctamente las pruebas.

------------------------------------------------------------------------

## 3. Dockerfile

En la raíz del proyecto debe existir el archivo:

``` text
Dockerfile
```

Contenido:

``` dockerfile
FROM amazoncorretto:21

RUN yum install -y findutils

WORKDIR /app

COPY . .

RUN chmod +x gradlew

CMD ["./gradlew", "test"]
```

### Explicación

`FROM amazoncorretto:21`

Define Amazon Corretto 21 como entorno Java del contenedor.

`RUN yum install -y findutils`

Instala utilidades necesarias por el Gradle Wrapper, incluyendo `xargs`.

`WORKDIR /app`

Define `/app` como directorio de trabajo dentro del contenedor.

`COPY . .`

Copia el proyecto al contenedor.

`RUN chmod +x gradlew`

Asigna permisos de ejecución al Gradle Wrapper.

`CMD ["./gradlew", "test"]`

Ejecuta las pruebas mediante Gradle cuando inicia el contenedor.

------------------------------------------------------------------------

## 4. Configurar `.dockerignore`

En la raíz del proyecto se recomienda tener:

``` text
.dockerignore
```

Con:

``` text
.gradle
build
.idea
*.iml
.git
.github
```

Esto evita copiar archivos innecesarios durante la construcción de la
imagen.

------------------------------------------------------------------------

## 5. Configurar `.gitignore`

Se recomienda:

``` text
.gradle
build
.idea
*.iml
```

La carpeta `.github` **no debe ignorarse**, ya que contiene la
configuración del pipeline.

------------------------------------------------------------------------

## 6. Crear las carpetas para GitHub Actions

En la raíz del proyecto crear:

``` text
.github
```

Dentro:

``` text
workflows
```

Y dentro de `workflows`:

``` text
tests.yml
```

La estructura queda:

``` text
UnitTest1/
├── .github/
│   └── workflows/
│       └── tests.yml
├── gradle/
├── src/
├── .dockerignore
├── .gitignore
├── build.gradle.kts
├── compose.yaml
├── Dockerfile
├── gradlew
├── gradlew.bat
└── settings.gradle.kts
```

Es importante escribir `.github` con el punto inicial. GitHub Actions
busca los workflows específicamente en:

``` text
.github/workflows/
```

------------------------------------------------------------------------

## 7. Configurar `tests.yml`

Contenido completo:

``` yaml
name: JUnit Tests

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  test:
    name: Run JUnit 5 Tests
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t junit-test .

      - name: Run JUnit tests
        run: docker run --name junit-test-container junit-test

      - name: Copy test report
        if: always()
        run: |
          mkdir -p test-report
          docker cp junit-test-container:/app/build/reports/tests/test/. test-report/

      - name: Upload test report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: junit-test-report
          path: test-report
```

------------------------------------------------------------------------

## 8. ¿Cuándo se ejecuta el pipeline?

Esta sección:

``` yaml
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
```

indica que el workflow se ejecuta cuando:

1.  Se realiza un `push` a `main`.
2.  Se abre o actualiza un `pull request` dirigido a `main`.

------------------------------------------------------------------------

## 9. Job de pruebas

``` yaml
jobs:
  test:
    name: Run JUnit 5 Tests
    runs-on: ubuntu-latest
```

GitHub crea temporalmente un runner basado en Ubuntu donde ejecutará el
pipeline.

No necesitamos instalar manualmente Amazon Corretto 21 ni Gradle en el
runner para ejecutar nuestras pruebas, porque esos elementos se
gestionan mediante Docker y el Gradle Wrapper.

------------------------------------------------------------------------

## 10. Checkout del repositorio

``` yaml
- name: Checkout repository
  uses: actions/checkout@v4
```

Descarga el código del repositorio dentro del runner de GitHub Actions.

------------------------------------------------------------------------

## 11. Construir la imagen Docker

``` yaml
- name: Build Docker image
  run: docker build -t junit-test .
```

Equivale al comando que ejecutamos localmente:

``` bash
docker build -t junit-test .
```

Docker lee el `Dockerfile` y construye la imagen `junit-test`.

------------------------------------------------------------------------

## 12. Ejecutar las pruebas

``` yaml
- name: Run JUnit tests
  run: docker run --name junit-test-container junit-test
```

Docker crea un contenedor y ejecuta:

``` text
./gradlew test
```

dentro de él.

No utilizamos:

``` bash
docker run --rm junit-test
```

porque necesitamos conservar temporalmente el contenedor para extraer
posteriormente el reporte.

Si las pruebas pasan:

``` text
JUnit 5
   |
   v
Gradle
   |
   v
BUILD SUCCESSFUL
   |
   v
GitHub Actions continúa
```

Si una prueba falla:

``` text
JUnit 5
   |
   v
Test FAILED
   |
   v
Gradle
   |
   v
BUILD FAILED
   |
   v
Job marcado como fallido
```

------------------------------------------------------------------------

## 13. Copiar el reporte

Gradle genera el reporte HTML dentro del contenedor en:

``` text
/app/build/reports/tests/test/
```

El workflow utiliza:

``` yaml
- name: Copy test report
  if: always()
  run: |
    mkdir -p test-report
    docker cp junit-test-container:/app/build/reports/tests/test/. test-report/
```

Primero crea:

``` text
test-report/
```

en el runner.

Luego:

``` bash
docker cp
```

copia el reporte desde el contenedor.

`if: always()` es importante porque permite intentar recuperar el
reporte incluso cuando las pruebas han fallado.

------------------------------------------------------------------------

## 14. Guardar el reporte como artefacto

``` yaml
- name: Upload test report
  if: always()
  uses: actions/upload-artifact@v4
  with:
    name: junit-test-report
    path: test-report
```

GitHub Actions guarda la carpeta `test-report` como un artefacto
denominado:

``` text
junit-test-report
```

Así el reporte permanece disponible después de que finalice el runner.

------------------------------------------------------------------------

## 15. Subir la configuración a GitHub

Antes del commit:

``` bash
git status
```

Se debe comprobar que Git detecta:

``` text
.github/workflows/tests.yml
```

Agregar los cambios:

``` bash
git add .
```

Crear el commit:

``` bash
git commit -m "Add CI pipeline for JUnit tests"
```

Subirlo:

``` bash
git push
```

Si la rama todavía no estuviera asociada con el repositorio remoto:

``` bash
git push -u origin main
```

------------------------------------------------------------------------

## 16. Si el proyecto todavía no está en Git

Para un proyecto nuevo, el flujo inicial puede ser:

``` bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin URL_DEL_REPOSITORIO
git push -u origin main
```

Después, para cambios posteriores:

``` bash
git add .
git commit -m "Add CI pipeline for JUnit tests"
git push
```

------------------------------------------------------------------------

## 17. Verificar GitHub Actions

Después del `git push`:

1.  Abrir el repositorio en GitHub.
2.  Entrar en **Actions**.
3.  Seleccionar **JUnit Tests**.
4.  Abrir la ejecución correspondiente.
5.  Entrar al job **Run JUnit 5 Tests**.

Una ejecución correcta debe mostrar aproximadamente:

``` text
✓ Set up job
✓ Checkout repository
✓ Build Docker image
✓ Run JUnit tests
✓ Copy test report
✓ Upload test report
✓ Complete job
```

------------------------------------------------------------------------

## 18. Consultar el reporte

En el resumen de la ejecución de GitHub Actions se puede localizar el
artefacto:

``` text
junit-test-report
```

Al descargarlo se obtiene el reporte generado por Gradle.

El archivo principal es:

``` text
index.html
```

Al abrirlo en un navegador se puede observar información como:

``` text
Test Summary

Tests:        2
Failures:     0
Ignored:      0
Success rate: 100%
```

También permite consultar los paquetes, clases y pruebas individuales.

------------------------------------------------------------------------

## 19. ¿Qué ocurre cuando una prueba falla?

Si una prueba contiene una aserción incorrecta, por ejemplo:

``` java
@Test
void suma() {
    assertEquals(10, 2 + 2);
}
```

JUnit reportará el fallo y Gradle terminará con:

``` text
BUILD FAILED
```

Docker devolverá un código de salida diferente de cero y GitHub Actions
marcará el paso de pruebas y el job como fallidos.

Gracias a:

``` yaml
if: always()
```

los pasos encargados del reporte se intentan ejecutar incluso después
del fallo.

Esto permite analizar el reporte para determinar qué prueba falló.

------------------------------------------------------------------------

## 20. Flujo completo de CI

``` text
Desarrollador
     |
     | git push
     v
GitHub
     |
     v
GitHub Actions
     |
     +-- Checkout repository
     |
     +-- docker build
     |
     +-- docker run
     |       |
     |       +-- Amazon Corretto 21
     |       |
     |       +-- Gradle
     |       |
     |       +-- JUnit 5
     |              |
     |              +-- PASS
     |              |
     |              +-- FAIL
     |
     +-- Copy test report
     |
     +-- Upload artifact
     |
     v
Resultado del CI
```

------------------------------------------------------------------------

## 21. ¿Esto es CI o CD?

Lo implementado hasta este punto corresponde principalmente a
**Continuous Integration (CI)**.

Tenemos:

``` text
Código
   |
   v
Push / Pull Request
   |
   v
Build
   |
   v
Pruebas automáticas
   |
   v
Reporte
```

Todavía no estamos realizando un despliegue.

Un proceso de CD podría continuar posteriormente:

``` text
CI
 |
 +-- Build
 +-- Tests
 +-- Quality checks
 |
 v
PASS
 |
 v
CD
 |
 +-- Empaquetar aplicación
 +-- Publicar imagen
 +-- Desplegar
```

Para un proyecto cuyo objetivo es practicar pruebas unitarias, el
pipeline de CI es suficiente para demostrar la ejecución automática de
pruebas y el control de calidad ante cambios en el código.

------------------------------------------------------------------------

## 22. Resultado final

Con esta configuración obtenemos:

``` text
JUnit 5
   +
Gradle
   +
Amazon Corretto 21
   +
Docker
   +
GitHub Actions
   =
Integración Continua
```

Las pruebas que inicialmente se ejecutaban manualmente desde IntelliJ
ahora se ejecutan automáticamente cada vez que el código llega a `main`
o se crea/actualiza un `pull request` hacia `main`.

Esto permite detectar regresiones automáticamente y conservar evidencia
de cada ejecución mediante el reporte generado por Gradle.
