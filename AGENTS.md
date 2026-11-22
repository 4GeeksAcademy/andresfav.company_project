# AGENTS.md — Instrucciones para agentes de programación

Instrucciones generales y protocolo de trabajo para cualquier agente de programación que opere en el monorepo de HealthCore.

Convención de este documento:

- **DEBE / NO DEBE**: obligación. No se omite sin autorización explícita del desarrollador.
- **Recomendado**: buena práctica; puede adaptarse justificándolo.

---

## A. Contexto obligatorio al iniciar cada sesión

Al comenzar cada sesión, el agente **DEBE** leer:

1. [`memory-bank/projectbrief.md`](./memory-bank/projectbrief.md): contexto empresarial y objetivos.
2. [`memory-bank/techContext.md`](./memory-bank/techContext.md): estructura, tecnologías, arquitectura y restricciones.
3. [`memory-bank/progress.md`](./memory-bank/progress.md): estado actual y próximos pasos.

Además, el agente **DEBE** consultar:

- [`BRIEFING.md`](./BRIEFING.md) cuando necesite detalle empresarial que el banco de memoria no recoja.
- El `README.md` de cada carpeta antes de trabajar en ella, y el [`README.md`](./README.md) principal para dudas sobre la ubicación de los elementos.

Reglas sobre el contexto:

- **DEBE** identificar las decisiones pendientes que afecten a la tarea (ver "Restricciones y decisiones pendientes" en `techContext.md` y "Próximos pasos y cuestiones abiertas" en `progress.md`).
- **NO DEBE** inventar información ausente (stack, frameworks, comandos, integraciones, datos o requisitos de cumplimiento). Si falta, lo indica y pregunta.
- **NO DEBE** presentar como implementado algo que solo está previsto en la documentación.

Reglas específicas (`.agents/rules/`):

- Antes de comenzar una tarea, el agente **DEBE** determinar si implica tratamiento de información sanitaria o datos personales de pacientes. Si es así, **DEBE** leer [`.agents/rules/health-data-safety.md`](./.agents/rules/health-data-safety.md) antes de modificar archivos y aplicar sus convenciones. Si duda de su aplicabilidad, consulta la sección "Alcance" de la regla.
- Las reglas específicas complementan este `AGENTS.md`; no sustituyen sus instrucciones generales.

## B. Procedimiento general de trabajo

1. **Comprender** el objetivo y el alcance de la tarea antes de modificar archivos. Si son ambiguos, preguntar.
2. **Inspeccionar** el código existente, el README de la carpeta y las convenciones aplicables. Leer solo lo necesario.
3. **Planificar** brevemente los cambios antes de ejecutarlos en tareas no triviales.
4. **Respetar** la estructura del monorepo (sección E).
5. **Solicitar confirmación** antes de:
   - tomar decisiones arquitectónicas o tecnológicas (frameworks, bases de datos, autenticación, despliegue, integraciones EHR);
   - modificar archivos protegidos (sección D);
   - añadir dependencias o crear aplicaciones o carpetas de primer nivel nuevas.
6. **Limitar los cambios** al alcance solicitado. No refactorizar ni corregir código ajeno a la tarea; anotar los problemas detectados.
7. **Verificar** los resultados con las comprobaciones disponibles (sección C, paso 3) y **comunicar** cualquier limitación, suposición o verificación no realizada.
8. **NO DEBE** crear commits ni hacer push sin aprobación explícita del desarrollador.

## C. Flujo obligatorio antes de cada commit

El agente **DEBE** completar estos pasos en orden. Si alguno falla, se detiene e informa.

Para los pasos 1 a 5, el agente **DEBE** leer y ejecutar la skill [`.agents/skills/validate-delivery/SKILL.md`](./.agents/skills/validate-delivery/SKILL.md) y presentar su informe en el paso 6. Si el estado no es APTO, no solicita autorización para el commit sin exponer antes las incidencias. Esta referencia hace obligatorio el procedimiento, pero no garantiza que una herramienta descubra la skill automáticamente sin leer este archivo.

1. **Revisar los cambios.** Ejecutar `git status --untracked-files=all` y `git diff` (y `git diff --staged` si hay cambios preparados). Como `git diff` no muestra los archivos sin seguimiento, **DEBE** revisar además el contenido completo de cada archivo nuevo (por ejemplo, con `git diff --no-index /dev/null <archivo>` o leyéndolo). Comprobar que todos los cambios y archivos nuevos pertenecen a la tarea; revertir o señalar cualquier cambio ajeno.
2. **Verificar estructura y reglas.** Confirmar que cada archivo está en la carpeta correcta según la sección E, que no se han modificado archivos protegidos sin autorización (sección D) y que se cumplen las restricciones de HealthCore (sección F).
3. **Ejecutar las comprobaciones disponibles.** Ejecutar solo las pruebas, linters o validaciones que **existan** en el repositorio y sean relevantes para los archivos modificados (por ejemplo, las definidas en el README de la carpeta o en sus archivos de configuración). Documentar el comando y su resultado. Si no existe ninguna comprobación aplicable, indicarlo explícitamente; **NO DEBE** inventar comandos ni dar por superadas comprobaciones no ejecutadas.
4. **Actualizar la documentación.** Actualizar los README afectados cuando los cambios modifiquen el comportamiento, la estructura o las instrucciones que documentan. Actualizar [`memory-bank/progress.md`](./memory-bank/progress.md) cuando haya avances relevantes (hitos, componentes, decisiones o próximos pasos); **NO DEBE** hacer actualizaciones artificiales por cambios menores. Si la tarea cambia estructura, tecnologías o decisiones, actualizar también `memory-bank/techContext.md` (con autorización si implica una decisión arquitectónica).
5. **Revisar datos sensibles.** Confirmar en el diff y en el contenido de los archivos nuevos que no se incluyen secretos, tokens, credenciales, archivos `.env`, datos de pacientes (PHI/datos personales) ni otros datos sensibles.
6. **Solicitar aprobación.** Presentar al desarrollador un resumen con: archivos modificados, propósito de cada cambio, comprobaciones ejecutadas y sus resultados, limitaciones y decisiones pendientes. Crear el commit **solo** tras su aprobación explícita.

## D. Archivos y carpetas protegidos

### Requieren autorización explícita del desarrollador antes de modificarse

| Categoría | Ejemplos actuales o futuros |
| --- | --- |
| Documentación empresarial fundamental | `BRIEFING.md`, `company-choice.md`, `CONTEXT.md` / `CONTEXT.es.md`, este `AGENTS.md` |
| Guías globales del monorepo | `README.md` / `README.es.md` de la raíz |
| Configuración de agentes | `.agents/` (reglas, skills y configuración de los agentes de programación); ver la aclaración bajo esta tabla |
| Seguridad, autenticación, autorización y privacidad | Configuración de auth, permisos, cifrado, auditoría, políticas de acceso a datos, archivos `.env*` y secretos |
| Infraestructura y despliegue | `infra/`, `docker-compose.yml`, `.devcontainer/`, pipelines de CI/CD (`.github/workflows/`) |
| Dependencias y configuración global | `package.json`, lockfiles, `pyproject.toml`, `requirements*.txt`, configuración de workspace, `.gitignore` |
| Contratos y esquemas compartidos | En `packages/` (p. ej. `@repo/shared-types`), `shared/` o cualquier otra ubicación: cambios en contratos públicos (tipos o APIs exportados, contratos de API), esquemas compartidos, esquemas de base de datos y migraciones, y cualquier modificación incompatible que pueda afectar a varios componentes |
| Datos | `data/raw/` y cualquier conjunto de datos de origen |

**Aclaración sobre `.agents/`:** una solicitud explícita del desarrollador para crear o modificar reglas o skills de `.agents/` constituye autorización para los cambios incluidos en el alcance de esa tarea. Cualquier modificación adicional o no solicitada en `.agents/` sigue requiriendo autorización.

### Se actualizan normalmente durante el trabajo (sin autorización previa)

- [`memory-bank/progress.md`](./memory-bank/progress.md): **DEBE** reflejar los avances relevantes (ver sección C, paso 4), sin actualizaciones artificiales.
- `memory-bank/techContext.md` y `memory-bank/projectbrief.md`: para reflejar hechos o decisiones ya aprobadas; no para introducir decisiones nuevas.
- El código, las pruebas y los `README.md` propios de la carpeta o componente en el que se trabaja dentro del alcance de la tarea.
- Cambios compatibles en `packages/` y `shared/` dentro del alcance de la tarea (p. ej. añadir recursos o tipos nuevos sin alterar contratos existentes).
- Documentación en `docs/` que describa hechos o funcionalidades existentes. Introducir **nuevas decisiones arquitectónicas** en `docs/` requiere confirmación explícita.

En caso de duda sobre si un archivo está protegido, se trata como protegido y se pregunta.

## E. Organización del monorepo

- Las futuras aplicaciones **DEBEN** ubicarse en las rutas definidas por la plantilla:
  - `uis/website/`: web pública.
  - `uis/backoffice/`: aplicación interna.
  - `services/`: backend y APIs.
- El resto de responsabilidades por carpeta (`data/`, `workflows/`, `packages/`, `shared/`, `docs/`, `infra/`, `scripts/`, `internal/`) está descrito en `memory-bank/techContext.md` y en la guía "Where should I put this?" del [`README.md`](./README.md).
- `.agents/` está destinada a **configurar los agentes de programación**: contiene sus reglas (`.agents/rules/`) y skills (`.agents/skills/`). No contiene producto.
- `agents/` y `skills/` (junto con `mcps/`) corresponden a **componentes del proyecto empresarial** de HealthCore (agentes de IA y capacidades del producto). No se usan para configurar al agente de programación.
- **NO DEBE** crear estructuras paralelas (p. ej. `frontend/`, `backend/`, `apps/`) ni aplicaciones nuevas sin justificación y autorización.
- Cada nueva aplicación, servicio, agente o pipeline **DEBE** tener su propia subcarpeta con `README.md`.

## F. Restricciones de HealthCore

- **Privacidad por defecto.** Los datos sanitarios y personales son sensibles. Minimizar los datos tratados y no exponerlos en código, logs, mensajes de error, prompts ni documentación.
- **Marco normativo.** Considerar **HIPAA** (Estados Unidos) y **UK GDPR** (Reino Unido) según la jurisdicción de los datos o de la funcionalidad. Si afecta a ambas, considerar ambos marcos.
- **Datos sintéticos por defecto.** En desarrollo y pruebas solo se usan datos sintéticos claramente identificados como tales. Cualquier excepción para usar datos reales, anonimizados o seudonimizados **DEBE** contar con autorización explícita previa del desarrollador y con garantías técnicas, organizativas y legales adecuadas; esa autorización no sustituye la evaluación de cumplimiento y privacidad. En ningún caso se incorpora información identificable de pacientes a archivos versionados, logs ni herramientas externas no autorizadas.
- **Validación previa.** Las decisiones relevantes para seguridad o cumplimiento (almacenamiento, acceso, auditoría, cifrado, transferencia entre jurisdicciones, uso de IA sobre datos clínicos) **DEBEN** validarse con el desarrollador antes de implementarse; los controles concretos están pendientes de definir con Cumplimiento y Gobierno del Dato.
- **Supervisión humana.** Las funcionalidades de IA clínica **DEBEN** contemplar revisión y aprobación profesional; no se presentan resultados del modelo como decisiones clínicas.
