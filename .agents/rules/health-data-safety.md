---
name: health-data-safety
description: Convenciones obligatorias para código, configuración, pruebas o documentación que procesen, almacenen o transmitan información sanitaria o personal de pacientes de HealthCore.
scope: Cualquier ubicación del monorepo que gestione información sanitaria (ver sección "Alcance").
activation: manual
---

# Regla: seguridad de datos sanitarios

## Objetivo

Establecer convenciones concretas y comprobables para desarrollar funcionalidades de HealthCore que procesen, almacenen o transmitan información sanitaria, de modo que esa información no se exponga durante el desarrollo, las pruebas ni la operación.

Esta regla complementa la sección F de [`AGENTS.md`](../../AGENTS.md); no la sustituye.

## Alcance

### Cuándo se aplica

La regla es **obligatoria**, con independencia de la carpeta del monorepo (`uis/`, `services/`, `data/`, `agents/`, `skills/`, `mcps/`, `workflows/`, `packages/`, `shared/`, `infra/`, `scripts/`, `internal/`, etc.), cuando la tarea:

1. Lee, crea, transforma, almacena, muestra o transmite datos de pacientes: identificadores, datos demográficos o de contacto, citas, historia clínica, notas clínicas, diagnósticos, tratamientos, reclamaciones de facturación o seguros, o cualquier dato que permita identificar a un paciente junto con información sanitaria.
2. Integra o simula los sistemas EHR, de citas o de facturación.
3. Define esquemas, modelos, contratos de API, pipelines, prompts o herramientas de agentes que manejen esos datos.
4. Configura almacenamiento, logs, autenticación, autorización, auditoría, cifrado o despliegue de componentes que los manejen.
5. Crea pruebas, fixtures, seeds, ejemplos o datasets de evaluación con ese tipo de datos.

### Cuándo no se aplica

En tareas que no tocan información sanitaria ni personal (por ejemplo, estilos de la web pública o documentación general), salvo la sección "Credenciales y secretos", que sigue las restricciones generales de `AGENTS.md`.

En caso de duda, se considera que la regla se aplica.

### Cómo se carga

- **Integración mediante instrucciones (verificado):** [`AGENTS.md`](../../AGENTS.md), sección A, obliga al agente a determinar al iniciar cada tarea si aplica esta regla y, en ese caso, a leerla antes de modificar archivos. Funciona en cualquier agente que lea `AGENTS.md`.
- **Activación automática (no verificada):** el repositorio no configura ninguna herramienta para cargar `.agents/rules/` por sí sola. El frontmatter de este archivo es descriptivo y **no garantiza** la activación.
- **Pendiente (requiere autorización):** configurar, si se decide, la herramienta de programación utilizada para que cargue `.agents/rules/` automáticamente.

## Origen de las restricciones

- **[Briefing]**: restricciones que proceden de [`BRIEFING.md`](../../BRIEFING.md).
- **[AGENTS.md]**: obligaciones ya establecidas en [`AGENTS.md`](../../AGENTS.md).
- **[company-choice.md]**: requisitos de la idea de agente descrita en [`company-choice.md`](../../company-choice.md).
- **[Técnica]**: recomendaciones técnicas adicionales de esta regla. No proceden del briefing y pueden ajustarse con aprobación del desarrollador.

Ninguna medida de esta regla, por sí sola, garantiza el cumplimiento de HIPAA o UK GDPR. Los controles concretos deben definirse con Cumplimiento y Gobierno del Dato (Claire Whitfield).

## Convenciones

### 1. Marco normativo y jurisdicción

- **[Briefing]** Los datos de pacientes de EE. UU. están sujetos a HIPAA y los del Reino Unido a UK GDPR. Son marcos distintos, con reglas diferentes de almacenamiento, acceso y compartición.
- **[Briefing]** Todo sistema que gestione datos de pacientes debe evaluarse bajo ambos marcos.
- **[Técnica]** Cada componente nuevo que maneje datos de pacientes documenta en su `README.md` qué categorías de datos trata y a qué jurisdicción o jurisdicciones afecta.
- **[Técnica]** No se implementa ninguna transferencia de datos de pacientes entre EE. UU. y el Reino Unido (por ejemplo, el historial entre sedes de distintos países) sin confirmación humana (sección 7).

### 2. Datos para desarrollo y pruebas

- **[AGENTS.md]** Por defecto, en desarrollo y pruebas solo se usan datos sintéticos. Cualquier excepción para usar datos reales, anonimizados o seudonimizados requiere autorización explícita previa y garantías técnicas, organizativas y legales adecuadas (ver sección 7). La autorización del desarrollador no sustituye la evaluación de cumplimiento y privacidad.
- **[AGENTS.md]** En ningún caso se incorpora información identificable de pacientes a archivos versionados (código, pruebas, fixtures, seeds, ejemplos, prompts, datasets de evaluación o documentación), logs ni herramientas externas no autorizadas.
- **[Técnica]** Los datos sintéticos se generan o escriben a mano y se marcan como tales:
  - nombres claramente ficticios;
  - identificadores con prefijo reconocible (p. ej. `SYN-`);
  - fechas y contactos no reales, como dominios de correo `example.com` o `example.org`;
  - un comentario o campo que indique "datos sintéticos".
- **[Técnica]** No convertir datos reales en "anonimizados" por iniciativa propia. Al solicitar la excepción para datos anonimizados o seudonimizados, el desarrollador debe confirmar su origen y el procedimiento aplicado. La seudonimización no equivale a anonimización.
- **[Técnica]** No añadir a `data/` datasets con datos de pacientes sin autorización explícita.

### 3. Logs, errores y archivos temporales

- **[Técnica]** No registrar en logs, trazas, métricas ni eventos de telemetría:
  - nombres, contactos, identificadores clínicos o de seguro, ni contenido clínico;
  - cuerpos completos de peticiones o respuestas;
  - prompts o salidas de modelos que contengan esos datos.
- **[Técnica]** En los logs, identificar registros mediante IDs internos opacos, no mediante datos del paciente.
- **[Técnica]** Los mensajes de error devueltos al usuario o a clientes de API son genéricos. No incluyen datos del paciente, consultas SQL, trazas de pila ni detalles internos.
- **[Técnica]** No escribir datos de pacientes en archivos temporales, cachés, salidas de depuración ni notebooks versionados. Si un archivo temporal es imprescindible, se crea fuera del repositorio, se elimina al terminar y no se añade a Git.
- **[Técnica]** Antes de proponer un commit, buscar en el diff y en los archivos nuevos llamadas de log o `print` que puedan volcar objetos de paciente completos.

### 4. Credenciales y secretos

- **[Técnica]** No escribir credenciales, tokens, claves de API, cadenas de conexión ni certificados en el código, la configuración versionada, los ejemplos ni los logs.
- **[Técnica]** Leer los secretos de variables de entorno o de un gestor de secretos. Si hace falta documentar las variables, crear un archivo de ejemplo con valores ficticios (p. ej. `.env.example`).
- **[Técnica]** Antes de crear el primer archivo `.env` o equivalente, comprobar que `.gitignore` lo excluye. Si no lo excluye, solicitar autorización para modificar `.gitignore`, que es un archivo protegido.
- **[Técnica]** Si se detecta un secreto ya versionado, no se intenta reescribir el historial por iniciativa propia: se informa al desarrollador.

### 5. Autorización, control de acceso y trazabilidad

- **[Briefing]** Hoy los registros de acceso a datos se mantienen por separado en cada sistema EHR y las pistas de auditoría están incompletas; Cumplimiento necesita consolidarlas.
- **[Técnica]** Todo componente nuevo que acceda a datos de pacientes implementa controles de acceso, autorización y trazabilidad apropiados, según las convenciones siguientes.
- **[Técnica]** Ningún endpoint, herramienta de agente o servidor MCP expone datos de pacientes sin autenticación y comprobación de autorización.
- **[Técnica]** Aplicar el mínimo privilegio: devolver solo los campos necesarios para el caso de uso y no exponer modelos de datos completos.
- **[Técnica]** Diseñar el acceso a datos de pacientes para registrar quién accede, a qué recurso, cuándo y con qué propósito, sin copiar el contenido clínico en el registro de auditoría.
- **[Técnica]** Los agentes de IA del producto acceden a los datos solo mediante herramientas con permisos acotados. No se les da acceso directo a bases de datos con credenciales amplias.
- **[company-choice.md]** El agente clínico propuesto requiere revisión y aprobación profesional. **[Técnica]** Sus salidas no se presentan como decisiones clínicas.

### 6. Almacenamiento y transmisión

- **[Técnica]** Transmitir datos de pacientes solo por canales cifrados (p. ej. TLS).
- **[Técnica]** No enviar datos de pacientes a servicios externos (APIs de IA, analítica, mensajería SMS o email, monitorización) sin confirmación humana sobre el proveedor y las garantías contractuales aplicables.
- **[Técnica]** El cifrado en reposo, la retención y el borrado son decisiones de seguridad (sección 7). No se implementan con valores supuestos.

## 7. Confirmación humana obligatoria

El agente **DEBE** detenerse y pedir confirmación al desarrollador antes de implementar cualquiera de estas decisiones:

- Elección del almacenamiento de datos de pacientes, del cifrado, de los plazos de retención o de la política de borrado.
- Diseño de la autenticación, de los roles o de los permisos de acceso a datos de pacientes.
- Formato, contenido o almacenamiento de los registros de auditoría.
- Envío de datos de pacientes a terceros o entre jurisdicciones.
- Uso de datos reales, anonimizados o seudonimizados en cualquier entorno distinto de producción.
- Cualquier funcionalidad de IA que procese datos clínicos.
- Cualquier excepción a esta regla.

Al pedir confirmación, el agente expone la decisión, las alternativas, el marco normativo afectado (HIPAA, UK GDPR o ambos) y lo que queda pendiente de validar con Cumplimiento.

## Comprobación antes de cerrar la tarea

Estos puntos complementan el flujo pre-commit de `AGENTS.md`:

- [ ] Solo se usan datos sintéticos identificados como tales, o la excepción está autorizada explícitamente (sección 7).
- [ ] Ningún log, error ni archivo temporal contiene datos de pacientes.
- [ ] No hay secretos en el diff ni en los archivos nuevos.
- [ ] Todo acceso nuevo a datos de pacientes exige autenticación y autorización y contempla trazabilidad.
- [ ] Las decisiones de la sección 7 se han confirmado o se han dejado explícitamente pendientes.
- [ ] El `README.md` del componente indica las categorías de datos y la jurisdicción afectadas.
