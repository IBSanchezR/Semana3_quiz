📦 Semana 3 — Auditoría de Clean Code y Seguridad + ADR
🎯 Objetivo

Identificar violaciones a principios de Clean Code y buenas prácticas de seguridad en un sistema heredado, y documentar decisiones de refactorización mediante un Architecture Decision Record (ADR).

🧪 FASE 1 — Levantamiento del Entorno
🖥️ Entorno de Ejecución
| Componente        | Versión             |
| ----------------- | ------------------- |
| Sistema Operativo | Ubuntu 24.04 (WSL2) |
| Docker            | 28.5.1              |
| Docker Compose    | v2.40.3             |
| Backend           | Spring Boot 3.2.0   |
| Base de datos     | PostgreSQL 15       |

El entorno se ejecuta sobre Windows utilizando WSL2 para garantizar compatibilidad con contenedores Linux y reproducibilidad del entorno de desarrollo.

🚀 Proceso de Levantamiento

Desde la raíz del proyecto se ejecutó: docker compose up --build

Posteriormente se dejó ejecutando en segundo plano: docker compose up -d

Verificación del estado de contenedores: docker compose ps

Servicios levantados:
app → Backend Spring Boot (puerto 8080)
db → PostgreSQL (puerto 5432)

🔎 Validación del Servicio

Se validó el endpoint de salud expuesto por la aplicación: curl -s http://localhost:8080/health

✅ Resultado obtenido: {"ok": true} 


---

🔍 FASE 2 — Auditoría del Código

📋 Tabla de Hallazgos

| # | Descripción técnica del hallazgo | Archivo | Línea aprox. | Principio violado | Nivel |
|---|----------------------------------|---------|-------------|-------------------|-------|
| 1 | Construcción dinámica de consulta SQL mediante concatenación de input (`username = '" + u + "'"`). Permite SQL Injection. | repository/UserRepository.java | 18–22 | Seguridad (SQL Injection) | Alto |
| 2 | Inserción SQL vulnerable por concatenación directa de atributos del objeto `User`. | repository/UserRepository.java | 31–37 | Seguridad (SQL Injection) | Alto |
| 3 | Credenciales de base de datos hardcodeadas (`admin/admin123`). | repository/UserRepository.java | 10–14 | Seguridad (exposición de secretos) | Alto |
| 4 | Uso de MD5 para hash de contraseñas. Algoritmo inseguro. | service/AuthService.java | 60–73 | Seguridad (hashing débil) | Alto |
| 5 | Exposición del hash en la respuesta (`res.put("hash", hp)`). | service/AuthService.java | 25 y 33 | Seguridad (exposición de datos) | Alto |
| 6 | Campos públicos en modelo `User`. Falta encapsulación. | model/User.java | 3–7 | Clean Code / Encapsulación | Medio |
| 7 | Naming poco descriptivo (`u`, `p`, `e`, `s`, `r`). | controller/AuthController.java | varios | Clean Code (Naming) | Medio |
| 8 | No se cierran recursos JDBC (`Connection`, `Statement`, `ResultSet`). | repository/UserRepository.java | 16–29 | Clean Code (manejo de recursos) | Medio/Alto |
| 9 | Validación débil de contraseña (`p.length() > 3`). | service/AuthService.java | 46–52 | Seguridad (validación insuficiente) | Medio |

---

 📌 Conclusión de Auditoría

El sistema presenta vulnerabilidades críticas de seguridad como SQL Injection, uso de hashing inseguro (MD5) y exposición de datos sensibles.  
También se identifican malas prácticas relacionadas con encapsulación, manejo de recursos y principios de Clean Code.  
Se recomienda refactorización prioritaria enfocada en seguridad y aplicación de principios SOLID.

---

🧪 FASE 3 — Pruebas Funcionales

Se realizaron pruebas manuales enviando solicitudes HTTP mediante curl contra la API en http://localhost:8080.

🔎 Prueba 1 — Login válido

Comando ejecutado: curl -i -X POST "http://localhost:8080/login?u=admin&p=12345"
Resultado obtenido:
HTTP/1.1 500 Internal Server Error
Content-Type: application/json
...
{"timestamp":"2026-02-26T06:34:35.474+00:00","status":500,"error":"Internal Server Error","path":"/login"}

Análisis:

El endpoint existe, pero se produce un error interno (500).

No se retornan datos del usuario.

El sistema no maneja adecuadamente el error.

En producción, los errores deberían manejarse con mensajes controlados y sin exponer información interna.

Conclusión:

El login no funciona correctamente debido a un error interno del servidor, lo que indica posibles problemas de conexión a base de datos o manejo de excepciones.

🔎 Prueba 2 — Intento de SQL Injection

Comando ejecutado: curl -i -X POST "http://localhost:8080/login?u=admin'--&p=cualquiercosa"

Resultado obtenido:
HTTP/1.1 500 Internal Server Error
...
Análisis:

Se intentó manipular la consulta SQL utilizando admin'--.
Esto busca comentar el resto de la sentencia SQL y omitir la validación de contraseña.
El sistema no valida ni sanitiza correctamente los parámetros.
Si la consulta estuviera construida dinámicamente sin prepared statements, podría permitir acceso no autorizado.

Riesgo en producción:

Acceso indebido a cuentas.
Exposición o manipulación de datos.
Escalada de privilegios.
Compromiso total del sistema.

🔎 Prueba 3 — Registro con contraseña débil
Caso 1 — Contraseña muy corta: curl -i -X POST "http://localhost:8080/register?u=test&p=123&e=test@test.com"

Caso 2 — Contraseña ligeramente mayor: curl -i -X POST "http://localhost:8080/register?u=test2&p=1234&e=test2@test.com"

Análisis:
Se evaluó si el sistema rechaza contraseñas débiles.
La validación aplicada parece basarse únicamente en longitud mínima.
No se verifican criterios de seguridad como:
Uso de mayúsculas
Uso de números
Uso de caracteres especiales
Complejidad mínima
Hash seguro de la contraseña

Conclusión:
La validación actual no es suficiente para un entorno productivo.
Se recomienda implementar:

Políticas de complejidad de contraseña.
Hash seguro (BCrypt o similar).
Validaciones de email.
Mensajes de error controlados.


🛡 Conclusión General FASE 3

Las pruebas funcionales evidencian:
Manejo inadecuado de errores (HTTP 500).
Posible vulnerabilidad a SQL Injection.
Validación insuficiente de contraseñas.
Falta de controles de seguridad robustos.

El sistema presenta debilidades que lo hacen inseguro para un entorno productivo sin mejoras adicionales.


