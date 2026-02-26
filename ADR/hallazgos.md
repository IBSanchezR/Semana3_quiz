# Hallazgos de Auditoría y Resultados de Pruebas

## 🔍 Tabla de Hallazgos (FASE 2)

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

## 🧪 Resultados de Pruebas Funcionales (FASE 3)

### Prueba 1 — Login válido

```bash
curl -i -X POST "http://localhost:8080/login?u=admin&p=12345"

Resultado:HTTP/1.1 500 Internal Server Error

Conclusión: El sistema presenta fallos internos y manejo inadecuado de excepciones.

---

Prueba 2 — Intento de SQL Injection
curl -i -X POST "http://localhost:8080/login?u=admin'--&p=cualquiercosa"

Resultado: HTTP/1.1 500 Internal Server Error

Conclusión: Existe riesgo potencial de SQL Injection debido a consultas construidas dinámicamente.

---

Prueba 3 — Registro con contraseña débil

Contraseña corta: curl -i -X POST "http://localhost:8080/register?u=test&p=123&e=test@test.com"

Resultado: {"ok":false}

Contraseña 1234: HTTP/1.1 500 Internal Server Error

Conclusión: La validación es insuficiente y no cumple estándares de seguridad.

