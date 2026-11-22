# Progreso del proyecto

## Estado actual

El proyecto está en fase inicial: se ha seleccionado HealthCore y existe su briefing empresarial. La documentación y el código accesibles corresponden principalmente a la plantilla del monorepo; no se han identificado aplicaciones, APIs o agentes específicos de HealthCore implementados. El milestone "Agent Memory Bank" tiene sus componentes creados y está en validación final; los cambios aún no se han confirmado en ningún commit.

## Hitos

- **Completado:** selección de HealthCore y documentación de intereses e idea inicial del agente clínico en [company-choice.md](../company-choice.md).
- **Milestone actual — Agent Memory Bank (en validación final):**
  - Hecho: banco de memoria (`projectbrief.md`, `techContext.md`, `progress.md`).
  - Hecho: [`AGENTS.md`](../AGENTS.md) con el protocolo de trabajo, el flujo pre-commit y los archivos protegidos.
  - Hecho: regla [`health-data-safety.md`](../.agents/rules/health-data-safety.md) para el tratamiento de información sanitaria.
  - Hecho: skill [`validate-delivery`](../.agents/skills/validate-delivery/SKILL.md) para validar entregas.
  - Hecho: integración de la regla y la skill en `AGENTS.md` mediante instrucciones explícitas (no se ha verificado descubrimiento automático por ninguna herramienta).
  - Hecho: actualización de la documentación raíz (`README.md`, `README.es.md`, `CONTEXT.md`, `CONTEXT.es.md`).
  - En curso: validación final con `validate-delivery`.
  - Pendiente: commit, push y Pull Request.
- **Futuros:** aplicaciones y servicios que respondan a las necesidades descritas en el briefing; su alcance y orden aún deben definirse.

## Componentes

| Existente | Pendiente o previsto |
| --- | --- |
| Briefing, guías del monorepo y plantilla de agente. | Web pública en `uis/website` y aplicación interna en `uis/backoffice`. |
| Ejemplo de limpieza con pandas y tipos TypeScript genéricos. | APIs y servicios de HealthCore en `services/`, incluidas las integraciones EHR. |
| Configuración de devcontainer y este banco de memoria. | Agente clínico propuesto, dashboards, automatizaciones y sistemas específicos de la empresa. |
| `AGENTS.md`, regla `health-data-safety` y skill `validate-delivery` en `.agents/` (pendientes de commit). | Configuración, si se decide, del descubrimiento automático de `.agents/` por la herramienta de programación utilizada. |

Los componentes pendientes reflejan ubicaciones o necesidades previstas, no un compromiso de alcance o calendario.

## Próximos pasos y cuestiones abiertas

1. Revisar y mantener actualizado el banco de memoria conforme se tomen decisiones y se complete trabajo.
2. Definir prioridades, alcance y secuencia de los próximos requisitos del proyecto y del milestone.
3. Decidir stack, arquitectura, almacenamiento, despliegue y mecanismos de integración con los EHR.
4. Concretar controles de privacidad y cumplimiento para ambas jurisdicciones y el acceso autorizado del agente.
5. Determinar cómo se evaluará el agente clínico y cómo se incorporará la revisión y aprobación por profesionales.
6. Completar la validación del milestone y, tras aprobación, crear el commit, hacer push y abrir la Pull Request.

Fuentes: [BRIEFING.md](../BRIEFING.md), [company-choice.md](../company-choice.md) y documentación y archivos accesibles del repositorio. La política de contenido impidió revisar los README de datos, por lo que su contenido no se ha verificado.
