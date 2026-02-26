# ADR-001: Refactorización del módulo de autenticación por vulnerabilidades críticas de seguridad

## Contexto

El sistema actual implementa un módulo básico de autenticación que permite el registro y login de usuarios mediante consultas directas a una base de datos PostgreSQL. Durante la auditoría técnica y las pruebas funcionales realizadas, se identificaron problemas críticos relacionados con seguridad, manejo de errores y diseño del código.

Se detectó el uso de consultas SQL construidas dinámicamente, lo que puede permitir ataques de SQL Injection si no se sanitizan adecuadamente los parámetros de entrada. Además, las contraseñas no cuentan con un mecanismo robusto de protección, lo que representa un alto riesgo en caso de filtración de datos. También se evidenció un manejo inadecuado de excepciones, generando errores HTTP 500 sin control ni mensajes estandarizados.

Estas debilidades afectan directamente la seguridad de los usuarios, la estabilidad del sistema y la confiabilidad del producto. En un entorno productivo, podrían permitir accesos no autorizados, exposición de información sensible y pérdida de integridad de datos. Por lo tanto, es necesario tomar una decisión arquitectónica para mejorar el diseño y fortalecer la seguridad del módulo de autenticación.

## Decisión

Se decide refactorizar el módulo de autenticación aplicando principios de seguridad y buenas prácticas de diseño, implementando las siguientes mejoras:

1. **Uso de consultas seguras mediante PreparedStatement o herramientas del framework (Spring Data / JdbcTemplate).**  
   Esto elimina el riesgo de SQL Injection al separar la estructura de la consulta de los parámetros enviados por el usuario.

2. **Implementación de hash seguro de contraseñas utilizando BCrypt.**  
   Las contraseñas no deben almacenarse en texto plano. BCrypt permite almacenar credenciales de forma segura y resistente a ataques de fuerza bruta.

3. **Separación de responsabilidades siguiendo el principio de Single Responsibility (SRP).**  
   El módulo se organizará en:
   - Controlador (manejo de solicitudes HTTP)
   - Servicio (lógica de negocio)
   - Repositorio (acceso a datos)  
   Esto mejora la mantenibilidad, testabilidad y claridad del código.

4. **Implementación de un manejador global de excepciones.**  
   Se estandarizarán las respuestas de error para evitar exponer detalles internos del sistema y mejorar la experiencia del cliente.

5. **Fortalecimiento de validaciones de entrada.**  
   Se aplicarán reglas de validación para contraseñas (longitud mínima y complejidad) y validación de formato de correo electrónico.

## Consecuencias

### Consecuencias positivas

- Eliminación de vulnerabilidades críticas de seguridad.
- Protección adecuada de contraseñas mediante hash seguro.
- Código más organizado y mantenible.
- Mejor control y estandarización de errores.
- Mayor confiabilidad y preparación para producción.

### Consecuencias negativas o riesgos

- Incremento del tiempo de desarrollo debido a la refactorización.
- Posibles errores temporales durante la transición.
- Mayor complejidad inicial en la estructura del proyecto.

No obstante, los beneficios en seguridad y calidad del software superan ampliamente los riesgos asociados.

## Alternativas consideradas

1. **Reescribir completamente el módulo desde cero.**  
   Se descartó debido al alto costo en tiempo y al riesgo de introducir nuevos errores durante una reimplementación total.

2. **Corregir únicamente la vulnerabilidad de SQL Injection sin refactorizar el diseño.**  
   Se descartó porque no solucionaría los problemas estructurales ni mejoraría la mantenibilidad del código.

3. **Mantener la implementación actual agregando validaciones superficiales.**  
   Se descartó por no abordar el problema de raíz relacionado con seguridad y arquitectura.