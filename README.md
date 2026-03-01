# Software Quality

# OWASP Top 10 – Seguridad en Aplicaciones Web
## Clase: Calidad y Pruebas del Software

---

# 1. Broken Access Control (Fallas en Control de Acceso)

## ¿Qué significa?
Un usuario puede acceder a información o acciones que no le corresponden.

## Ejemplo sencillo
URL:
https://app.com/user/123/profile

Si el usuario cambia el 123 por 124 y puede ver otro perfil → Vulnerabilidad.

## Cómo probarlo (QA paso a paso)
1. Crear dos usuarios.
2. Iniciar sesión con el Usuario A.
3. Intentar acceder al ID del Usuario B.
4. Verificar si el sistema bloquea el acceso.

---

# 2. Cryptographic Failures (Fallas Criptográficas)

## ¿Qué significa?
Datos sensibles no están protegidos correctamente.

## Ejemplo
Base de datos guarda:
password: 123456

En vez de:
password: $2b$10$kd83jf...

## Cómo probarlo
1. Verificar si la API devuelve contraseñas.
2. Validar que el sitio use HTTPS.
3. Confirmar que las contraseñas estén cifradas.

---

# 3. Injection (Inyección SQL)

## ¿Qué significa?
El sistema permite ejecutar código malicioso en consultas.

## Ejemplo

Consulta vulnerable:
SELECT * FROM users WHERE username = 'admin' AND password = '1234'

Entrada maliciosa:
admin' OR '1'='1

Resultado:
SELECT * FROM users WHERE username = 'admin' OR '1'='1'

## Cómo probarlo
1. Enviar ' OR '1'='1 en el login.
2. Verificar si el sistema permite acceso sin contraseña válida.

---

# 4. Insecure Design (Diseño Inseguro)

## ¿Qué significa?
La seguridad no fue considerada desde el diseño.

## Ejemplo
No hay límite de intentos en login.

## Cómo probarlo
1. Intentar múltiples contraseñas incorrectas.
2. Verificar si existe bloqueo de cuenta.

---

# 5. Security Misconfiguration (Mala Configuración)

## ¿Qué significa?
Errores en configuración del servidor o aplicación.

## Ejemplo
Error 500 muestra:
System.NullReferenceException at controller...

## Cómo probarlo
1. Provocar un error intencional.
2. Verificar si se exponen detalles técnicos.

---

# 6. Vulnerable and Outdated Components (Componentes Vulnerables)

## ¿Qué significa?
Uso de librerías desactualizadas.

## Ejemplo
Proyecto usa versión antigua de una librería con vulnerabilidades conocidas.

## Cómo probarlo
1. Revisar dependencias del proyecto.
2. Validar versiones contra bases de datos de vulnerabilidades.

---

# 7. Identification and Authentication Failures

## ¿Qué significa?
Problemas en autenticación o gestión de sesiones.

## Ejemplo
El sistema permite contraseña: 1234

## Cómo probarlo
1. Intentar crear contraseñas débiles.
2. Validar si hay expiración de sesión.

---

# 8. Software and Data Integrity Failures

## ¿Qué significa?
El sistema no valida correctamente archivos o datos externos.

## Ejemplo
Subir archivo virus.exe como si fuera imagen.

## Cómo probarlo
1. Intentar subir archivos con extensiones peligrosas.
2. Validar si el sistema los bloquea.

---

# 9. Security Logging and Monitoring Failures

## ¿Qué significa?
El sistema no registra eventos de seguridad.

## Ejemplo
100 intentos fallidos de login sin registro.

## Cómo probarlo
1. Intentar múltiples accesos fallidos.
2. Verificar si quedan registrados en logs.

---

# 10. Server-Side Request Forgery (SSRF)

## ¿Qué significa?
El servidor realiza solicitudes internas que no debería.

## Ejemplo
Un atacante hace que el servidor consulte recursos internos privados.

## Cómo probarlo
1. Analizar funcionalidades que consuman URLs externas.
2. Validar si se restringen direcciones internas.

---

# Conclusión

Un software no es de calidad si es funcional pero inseguro.

La seguridad es parte fundamental de las pruebas no funcionales y del aseguramiento de calidad del software.
