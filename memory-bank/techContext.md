# Contexto técnico

## Organización del monorepo

La distribución por responsabilidades procede de la guía de la plantilla; no implica que existan productos funcionales en cada carpeta:

| Ubicación | Responsabilidad |
| --- | --- |
| `uis/website` | Ubicación prevista para la web pública. |
| `uis/backoffice` | Ubicación prevista para la aplicación interna. |
| `services/` | APIs, lógica backend y procesos de fondo. |
| `data/raw`, `data/pipelines`, `data/process`, `data/eval` | Datos de origen, transformación, resultados y evaluación, respectivamente. |
| `agents/`, `skills/`, `mcps/` | Componentes del producto empresarial: agentes de IA, capacidades reutilizables y servidores MCP de HealthCore. |
| `AGENTS.md` | Instrucciones generales de los agentes de programación. |
| `.agents/rules/` | Reglas específicas de desarrollo para los agentes de programación. |
| `.agents/skills/` | Procedimientos reutilizables para los agentes de programación. No forman parte del producto (a diferencia de `agents/` y `skills/`). |
| `workflows/` | Automatizaciones y orquestación entre sistemas. |
| `packages/`, `shared/` | Paquetes compartidos y recursos reutilizables no empaquetados. |
| `docs/` | Arquitectura, decisiones y documentación transversal. |
| `infra/`, `scripts/`, `internal/` | Infraestructura, scripts auxiliares y herramientas internas. |
| `memory-bank/` | Contexto empresarial, técnico y de progreso para futuros agentes. |

## Presente y convenciones

- El repositorio contiene principalmente estructura y documentación de plantilla; no se han identificado aplicaciones de HealthCore ejecutables ni un gestor de workspace en la raíz.
- Hay una plantilla de agente en `agents/_template/`, un ejemplo de limpieza en Python que usa pandas y metadatos del paquete privado `@repo/shared-types` con tipos TypeScript genéricos. Son recursos de partida, no funcionalidades de HealthCore.
- El devcontainer configura Ubuntu 24.04, Python 3.12, Node.js 22, Docker, GitHub CLI y herramientas de desarrollo; contempla pnpm y uv. Esta configuración de desarrollo no determina el stack de producción.
- La guía separa interfaces, servicios, datos y agentes, y pide documentar cada aplicación, servicio, agente o pipeline en su propia carpeta. Los recursos compartidos deben situarse según sean paquetes o activos sueltos.
- El README principal propone una API centralizada con FastAPI y evitar microservicios prematuros. Es una orientación de la plantilla, no una arquitectura de HealthCore confirmada ni implementada.

## Restricciones y decisiones pendientes

- Evaluar los sistemas que traten datos de pacientes bajo HIPAA y UK GDPR; definir con Cumplimiento los controles, el acceso autorizado y la auditoría. No incluir datos clínicos reales en recursos de ejemplo sin las garantías correspondientes.
- Decidir frameworks y lenguajes de las aplicaciones y servicios, organización del backend, bases de datos, autenticación, despliegue y observabilidad.
- Definir integraciones con los dos EHR, intercambio de datos entre jurisdicciones y mecanismos concretos de seguridad y cumplimiento.
- Definir alcance, fuentes autorizadas, herramientas y evaluación del futuro agente clínico, manteniendo la revisión profesional prevista.

Fuentes: [README.md](../README.md), [uis/README.md](../uis/README.md), [services/README.md](../services/README.md), [agents/README.md](../agents/README.md), [docs/README.md](../docs/README.md), [devcontainer.json](../.devcontainer/devcontainer.json) y recursos de plantilla citados. Los README de `data/` no pudieron consultarse por la política de contenido de la organización; sus responsabilidades se basan en el README principal.
