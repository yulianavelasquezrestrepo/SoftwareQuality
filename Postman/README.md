# Introducción a Postman

Postman es una herramienta para trabajar con APIs: permite enviar solicitudes, inspeccionar respuestas y comprobar cómo se comporta un servicio. Es útil para aprender, desarrollar y probar la comunicación entre aplicaciones. Consulta la [documentación oficial](https://learning.postman.com/docs/getting-started/overview).

## Conceptos básicos

Una **API** es una interfaz que permite a distintos programas intercambiar información. Al utilizar una API HTTP, un cliente envía una solicitud y un servidor devuelve una respuesta.

- **Endpoint:** dirección URL del recurso al que quieres acceder.
- **Método HTTP:** indica la acción solicitada.
- **Parámetros:** valores que se envían, por ejemplo, en la URL para filtrar resultados.
- **Headers (cabeceras):** información adicional, como el formato de los datos o las credenciales.
- **Body (cuerpo):** datos enviados en una solicitud, cuando corresponde.
- **Respuesta:** incluye un código de estado, cabeceras y, normalmente, un cuerpo con datos.

### Métodos HTTP habituales

| Método | Uso habitual |
| --- | --- |
| `GET` | Consultar información. |
| `POST` | Enviar datos para crear un recurso o ejecutar una operación. |
| `PUT` | Reemplazar la representación de un recurso. |
| `PATCH` | Actualizar parcialmente un recurso. |
| `DELETE` | Eliminar un recurso. |

El comportamiento concreto de cada endpoint depende de la API y de su documentación.

## Tu primera solicitud

El siguiente ejercicio utiliza Postman Echo, un servicio de ejemplo que devuelve información de la solicitud recibida. Sigue el mismo enfoque de la [guía oficial de inicio rápido](https://learning.postman.com/docs/getting-started/first-steps/overview).

1. Descarga e instala la aplicación desde el [sitio de Postman](https://www.postman.com/downloads/).
2. Abre Postman y crea una nueva solicitud HTTP.
3. Selecciona el método `GET`.
4. Introduce esta URL:

   ```text
   https://postman-echo.com/get?nombre=Ana
   ```

5. Pulsa **Send** para enviar la solicitud.
6. Revisa el código de estado y el cuerpo de la respuesta.

Si la solicitud se completa correctamente, recibirás un estado `200 OK`. El JSON de respuesta incluirá un campo como este, además de otros datos:

```json
{
  "args": {
    "nombre": "Ana"
  }
}
```

Prueba a cambiar `Ana` por otro nombre y vuelve a enviar la solicitud para observar el resultado.

## Cómo interpretar una respuesta

Estos son algunos códigos HTTP frecuentes:

| Código | Significado |
| --- | --- |
| `200 OK` | La solicitud se procesó correctamente. |
| `201 Created` | Se creó un recurso. |
| `400 Bad Request` | El servidor no puede procesar la solicitud por un problema del cliente. |
| `401 Unauthorized` | Faltan credenciales válidas de autenticación. |
| `403 Forbidden` | El servidor rechaza el acceso al recurso. |
| `404 Not Found` | No se encontró el recurso solicitado. |
| `500 Internal Server Error` | Ocurrió un error en el servidor. |

Además del código, revisa el contenido de la respuesta: un estado exitoso no garantiza por sí solo que los datos sean los esperados.

## Próximos pasos

Guarda tu solicitud en una **colección** para reutilizarla. Después, explora cómo escribir pruebas que validen las respuestas siguiendo la [guía de inicio rápido de Postman](https://learning.postman.com/docs/getting-started/first-steps/overview).
