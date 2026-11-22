---
name: validate-delivery
description: Verifica que los cambios de una tarea en el monorepo de HealthCore están preparados para entregarse y genera un informe de validación con estado APTO, NO APTO o BLOQUEADO. Usar antes de proponer un commit o cuando el desarrollador pida validar una entrega. No crea commits.
---

# Skill: validar entrega (`validate-delivery`)

## A. Objetivo

Comprobar, de forma reproducible, que los cambios de una tarea cumplen el alcance, `AGENTS.md` y las reglas aplicables, y emitir un informe de validación con un estado final.

Resuelve un problema concreto: evitar que se proponga un commit con cambios fuera de alcance, comprobaciones sin ejecutar, documentación desactualizada, secretos o datos de pacientes expuestos.

**Cuándo usarla:**

- Antes del paso 6 ("Solicitar aprobación") del flujo pre-commit de [`AGENTS.md`](../../../AGENTS.md), sección C. Esta skill ejecuta los pasos 1 a 5 de ese flujo y documenta su resultado.
- Cuando el desarrollador pida validar una entrega o un milestone.

**Qué no hace:** no corrige los problemas que detecta (salvo que el desarrollador lo pida aparte), no crea commits, no hace push y no modifica archivos.

## B. Inputs

| Input | Obligatorio | Cómo obtenerlo si no se proporciona |
| --- | --- | --- |
| Objetivo y alcance de la tarea (incluidas las autorizaciones explícitas recibidas, p. ej. para archivos protegidos) | Sí | De la petición del desarrollador. Si no se puede determinar, preguntar; sin él el estado es BLOQUEADO. |
| Rama de trabajo | Sí | `git branch --show-current` |
| Rama base de comparación (contra la que se entregará la tarea, p. ej. `main`) | Sí | Rama upstream de la rama de trabajo (`git rev-parse --abbrev-ref @{upstream}`) o indicación del desarrollador. Si no se puede determinar, preguntar; sin ella el estado es BLOQUEADO. |
| Archivos creados, modificados o eliminados | Sí | Paso 1 del procedimiento. |
| Reglas aplicables | Sí | `AGENTS.md` siempre; reglas de `.agents/rules/` según su sección "Alcance" (p. ej. [`health-data-safety.md`](../../rules/health-data-safety.md)). |
| Herramientas y comandos de verificación disponibles | Sí | Paso 4 del procedimiento. Pueden no existir; en ese caso se documenta. |

## C. Procedimiento

Ejecutar los pasos en orden. Registrar en el informe cada comando y su resultado.

### 1. Identificar los cambios

1. `git branch --show-current` y `git status --short --untracked-files=all`.
2. `git diff` y `git diff --staged` para los archivos con seguimiento.
3. Para **cada archivo sin seguimiento** (`??`), revisar su contenido completo: `git diff --no-index /dev/null <archivo>` o lectura directa. `git diff` no los muestra.
4. Para archivos eliminados (`D`), confirmar que la eliminación es intencionada.
5. Revisar los cambios ya confirmados en commits locales respecto a la rama base (siempre, aunque el árbol de trabajo esté limpio):
   1. `git log --oneline <rama-base>..HEAD` para listar los commits de la tarea. Si la lista está vacía, anotarlo y continuar.
   2. `git diff --name-status <rama-base>...HEAD` para obtener los archivos afectados por esos commits.
   3. `git diff <rama-base>...HEAD -- <archivo>` para revisar el contenido de **cada** uno de esos archivos, con el mismo criterio que los cambios sin confirmar.
   4. Si la rama base no existe localmente o no se puede determinar, preguntar al desarrollador. Si no se resuelve, el estado es BLOQUEADO.

Resultado: lista completa y sin duplicados de archivos afectados, indicando su tipo de cambio (A/M/D/??) y su origen (commit local, staged, unstaged o sin seguimiento).

### 2. Verificar el alcance

Para cada archivo de la lista:

- Asociarlo a un objetivo de la tarea. Si no se puede asociar, marcarlo como **fuera de alcance**.
- Comprobar si es un archivo protegido (`AGENTS.md`, sección D). Si lo es, confirmar que existe autorización explícita en el alcance de la tarea.
- Comprobar que está en la ubicación correcta según `AGENTS.md`, sección E (sin estructuras paralelas ni aplicaciones nuevas no autorizadas).

### 3. Comprobar reglas aplicables

1. Revisar el cumplimiento de `AGENTS.md`, secciones D, E y F, para los cambios identificados.
2. Para cada regla de `.agents/rules/`, evaluar su sección "Alcance" y decidir si aplica. Registrar la decisión y su motivo.
3. Para cada regla aplicable, recorrer su lista de comprobación final y marcar cada punto como cumplido, incumplido o no aplicable, con evidencia (archivo y línea, o justificación).

### 4. Ejecutar las validaciones disponibles

1. Descubrir las validaciones **existentes** relevantes para los archivos afectados, consultando solo:
   - el `README.md` de cada carpeta afectada;
   - manifiestos y configuración presentes en esas carpetas o en la raíz (p. ej. `package.json` → `scripts`, `pyproject.toml`, configuraciones de linters o de pruebas);
   - pruebas existentes junto al código afectado.
2. Ejecutar únicamente esas validaciones, limitadas a los archivos o componentes afectados. **No inventar comandos** ni instalar herramientas nuevas sin autorización.
3. Registrar comando, resultado (OK / FALLO) y un resumen de la salida.
4. Si no existe ninguna validación aplicable, registrarlo como "comprobación no ejecutada: no existe herramienta" con la ruta inspeccionada. En archivos solo de documentación, basta revisión manual.

### 5. Revisar documentación, secretos y datos sensibles

1. **Documentación:** si los cambios modifican comportamiento, estructura o instrucciones documentadas, comprobar que los README afectados se han actualizado. Comprobar que `memory-bank/progress.md` refleja los avances relevantes (sin exigir actualizaciones artificiales).
2. **Secretos:** buscar en el diff y en los archivos nuevos patrones como `password`, `secret`, `token`, `api_key`, `apikey`, `Authorization`, `BEGIN .*PRIVATE KEY`, cadenas de conexión (`://usuario:clave@`) y archivos `.env` no excluidos. Por ejemplo: `git diff | grep -n -i -E "password|secret|token|api_?key|private key"` y la misma búsqueda sobre los archivos sin seguimiento. Revisar manualmente cada coincidencia; una coincidencia no implica necesariamente un secreto.
3. **Datos de pacientes:** comprobar que fixtures, ejemplos, logs y documentación solo usan datos sintéticos identificados como tales, o que existe la excepción autorizada que exige `AGENTS.md`, sección F. Comprobar que no hay llamadas de log que vuelquen objetos de paciente completos.

### 6. Generar el informe

Redactar el informe con el formato de la sección D, determinar el estado según la sección E y presentarlo al desarrollador. **No crear el commit**: la decisión corresponde al desarrollador (paso 6 del flujo pre-commit de `AGENTS.md`).

## D. Output esperado

Informe en Markdown con esta estructura:

```markdown
# Informe de validación de entrega

- **Tarea:** <objetivo y alcance resumidos>
- **Rama:** <rama> (base: <rama-base>; commits locales revisados: <n>)
- **Fecha:** <fecha>
- **Estado final:** APTO | NO APTO | BLOQUEADO

## Archivos afectados
| Archivo | Cambio (A/M/D/??) | Origen (commit local / staged / unstaged / sin seguimiento) | Revisado | Dentro de alcance | Protegido / autorización |

## Reglas aplicables
| Regla | ¿Aplica? | Motivo | Resultado |

## Comprobaciones realizadas
| Comprobación | Resultado (OK / FALLO) | Evidencia |

## Comandos ejecutados
| Comando | Resultado | Resumen de salida |

## Comprobaciones no ejecutadas
| Comprobación | Justificación |

## Problemas detectados
| # | Severidad (bloqueante / no bloqueante) | Descripción | Archivo | Acción propuesta |

## Siguientes pasos
<qué debe decidir o corregir el desarrollador antes del commit>
```

## E. Criterios de aceptación

### Criterios obligatorios

| # | Criterio | Cómo se verifica |
| --- | --- | --- |
| C1 | Todos los archivos afectados han sido revisados: commits locales respecto a la rama base, cambios staged, unstaged y archivos nuevos sin seguimiento. | Cada archivo de `git diff --name-status <rama-base>...HEAD` y de `git status --untracked-files=all` figura en el informe marcado como revisado. |
| C2 | No hay cambios fuera del alcance autorizado. | Ningún archivo marcado como fuera de alcance; los protegidos tienen autorización explícita. |
| C3 | Las reglas aplicables se han comprobado. | Cada regla de `.agents/rules/` tiene su decisión de aplicabilidad y, si aplica, su lista de comprobación completada. |
| C4 | Las validaciones relevantes disponibles se han ejecutado correctamente. | Todos los comandos registrados terminan en OK. |
| C5 | No hay secretos ni datos sensibles expuestos. | Búsquedas del paso 5 ejecutadas y coincidencias revisadas sin hallazgos. |
| C6 | La documentación relevante está actualizada cuando corresponde. | README afectados y `memory-bank/progress.md` revisados según el paso 5.1. |
| C7 | El informe identifica toda comprobación que no se ha podido realizar. | Sección "Comprobaciones no ejecutadas" completa, con justificación. |

### Determinación del estado

- **APTO:** se cumplen C1–C7. Las validaciones no ejecutadas solo pueden deberse a que no existe herramienta aplicable, y está documentado (C7).
- **NO APTO:** falla al menos un criterio obligatorio (p. ej. cambio fuera de alcance, prueba fallida, secreto detectado, documentación desactualizada). Un fallo en un criterio obligatorio impide declarar APTO.
- **BLOQUEADO:** falta información o una herramienta imprescindible para evaluar un criterio obligatorio (p. ej. alcance desconocido, rama base desconocida o inexistente localmente, comando documentado que no puede ejecutarse en el entorno). Nunca se sustituye por APTO suponiendo que todo es correcto.

Si concurren NO APTO y BLOQUEADO, se informa NO APTO y se listan también los bloqueos.

## Descubrimiento e invocación

- **Integración mediante instrucciones (verificado):** [`AGENTS.md`](../../../AGENTS.md), sección C, obliga a leer y ejecutar esta skill para los pasos 1 a 5 del flujo pre-commit y a presentar su informe antes de solicitar autorización para el commit. Funciona en cualquier agente que lea `AGENTS.md`.
- **Invocación manual:** el desarrollador pide "valida la entrega con `.agents/skills/validate-delivery/SKILL.md`" indicando el objetivo, el alcance y la rama base; el agente lee este archivo y sigue el procedimiento.
- **Descubrimiento automático (no verificado):** el repositorio no configura ninguna herramienta para descubrir `.agents/skills/`. El frontmatter (`name`, `description`) sigue el formato habitual de `SKILL.md`, pero no está comprobado que ninguna herramienta lo cargue desde esta ruta. Configurarlo requiere autorización.
