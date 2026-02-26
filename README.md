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