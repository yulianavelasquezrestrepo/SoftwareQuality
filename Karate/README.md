# Introducción a Karate

Karate es un framework de automatización de pruebas que permite enviar solicitudes a APIs y validar sus respuestas. Sus pruebas se escriben en archivos `.feature` con una sintaxis basada en Gherkin y palabras clave integradas, lo que permite describir solicitudes y verificaciones sin implementar cada paso en Java. Consulta la [documentación oficial de Karate](https://docs.karatelabs.io/).

## Conceptos básicos

- **API:** interfaz que permite la comunicación entre aplicaciones.
- **Endpoint:** URL que identifica un recurso u operación de una API.
- **Solicitud HTTP:** mensaje enviado al servidor con un método, una URL y, opcionalmente, cabeceras y datos.
- **Respuesta HTTP:** resultado del servidor, que incluye código de estado, cabeceras y cuerpo.
- **Aserción:** comprobación automática que compara un resultado con lo esperado.

### Estructura de una prueba

| Elemento | Propósito |
| --- | --- |
| `Feature` | Describe la funcionalidad que se prueba. |
| `Background` | Define pasos comunes que se ejecutan antes de cada escenario. |
| `Scenario` | Define un caso de prueba. |
| `Given` | Introduce la preparación de la solicitud. |
| `When` | Introduce la acción que se ejecuta. |
| `Then` | Introduce la validación del resultado. |
| `And` | Añade un paso. |
| `*` | Prefijo alternativo para escribir pasos. |

Los prefijos ayudan a organizar la lectura; las palabras clave de Karate determinan la operación. Puedes ampliar estos conceptos en la [guía de archivos feature](https://docs.karatelabs.io/core-syntax/feature-files/).

### Palabras clave frecuentes

| Palabra clave | Uso |
| --- | --- |
| `url` | Establece la URL base. |
| `path` | Añade segmentos a la ruta. |
| `param` | Añade un parámetro de consulta. |
| `header` | Define una cabecera HTTP. |
| `request` | Define el cuerpo de la solicitud. |
| `method` | Envía la solicitud con el método indicado. |
| `status` | Verifica el código de estado HTTP. |
| `match` | Comprueba valores o estructuras de datos. |
| `def` | Define una variable. |

Después de enviar una solicitud, `response` contiene el cuerpo de la respuesta. Consulta la [referencia de sintaxis](https://docs.karatelabs.io/api-reference/syntax-reference/) y la [guía de respuestas](https://docs.karatelabs.io/http-responses/response-handling/).

## Métodos HTTP

| Método | Uso habitual | Paso en Karate |
| --- | --- | --- |
| `GET` | Consultar información. | `When method get` |
| `POST` | Crear un recurso o ejecutar una operación. | `When method post` |
| `PUT` | Reemplazar la representación de un recurso. | `When method put` |
| `PATCH` | Actualizar parcialmente un recurso. | `When method patch` |
| `DELETE` | Eliminar un recurso. | `When method delete` |
| `HEAD` | Obtener cabeceras sin el cuerpo de respuesta. | `When method head` |
| `OPTIONS` | Consultar las opciones de comunicación disponibles. | `When method options` |

El comportamiento y los códigos de respuesta concretos dependen del contrato de cada API. La [guía de solicitudes HTTP de Karate](https://docs.karatelabs.io/http-requests/making-requests/) explica cómo construir estas llamadas.

## Ejemplo práctico

Este ejemplo consulta Postman Echo, un servicio público que devuelve información de la solicitud recibida. Permite comprobar tanto el estado HTTP como el parámetro enviado.

### 1. Preparar el entorno

Configura Karate siguiendo las opciones de instalación y ejecución de la [documentación oficial](https://docs.karatelabs.io/). Los requisitos dependen de la versión y del ejecutor elegido; utiliza los indicados para tu instalación.

### 2. Crear el archivo de prueba

Dentro de tu proyecto de pruebas Karate, crea un archivo llamado `consulta.feature` con este contenido:

```gherkin
Feature: Users API

  Scenario: Obtener un usuario

    Given url 'https://jsonplaceholder.typicode.com'
    And path 'users/1'
    When method GET
    Then status 200
```

### 3. Ejecutar e interpretar

Ejecuta `consulta.feature` con el ejecutor configurado en tu proyecto Karate, como la integración del editor, la línea de comandos o JUnit. La prueba requiere conexión a Internet y que el servicio esté disponible.

El escenario envía `GET https://jsonplaceholder.typicode.com` y verifica la condición:

1. El servidor responde con el código `200`.

Si la comprobación falla, el escenario se marca como fallido. Para observarlo, cambia únicamente el valor esperado de la última línea por `404` y vuelve a ejecutar la prueba.

## Buenas prácticas

- Escribe escenarios con nombres que expliquen el comportamiento esperado.
- Valida el contenido de la respuesta además del código HTTP.
- Incluye casos exitosos y casos de error previstos por la API.
- Usa `Background` para la configuración compartida entre escenarios.
- Mantén credenciales y tokens fuera de los archivos versionados.
- Para pruebas estables, utiliza datos y entornos de prueba controlados.

## Documentación oficial

- [Documentación de Karate](https://docs.karatelabs.io/).
- [Estructura de archivos feature](https://docs.karatelabs.io/core-syntax/feature-files/).
- [Construcción de solicitudes HTTP](https://docs.karatelabs.io/http-requests/making-requests/).
- [Manejo de respuestas](https://docs.karatelabs.io/http-responses/response-handling/).
- [Referencia de palabras clave](https://docs.karatelabs.io/api-reference/keywords/).
- [Repositorio oficial en GitHub](https://github.com/karatelabs/karate).
