# Sistema de Control del Agente de Desarrollo IA

Bienvenido al "cerebro" del proyecto. Esta carpeta, `control/`, contiene todos los documentos y directrices que el agente de IA (Gemini) utiliza para gestionar el ciclo de vida de desarrollo de esta aplicación. Este sistema es un sustituto deliberado de herramientas como Git, Jira y Confluence, diseñado para explorar un paradigma de desarrollo gobernado por IA.

## Filosofía

La premisa es que, en lugar de que un humano use herramientas de control de versiones, el agente de IA utiliza un conjunto de documentos legibles (Markdown) como su fuente de verdad, su memoria y su plan de acción.

## ¿Cómo interactúan los documentos?

1.  **El Comienzo (`instrucciones_agente/gemini_prompt_maestro.md`):** Este es el documento más importante. Contiene las directrices fundamentales que dictan el comportamiento, la filosofía y el ciclo de trabajo del agente. Es su "sistema operativo".

2.  **El Plan (`seguimiento/roadmap.md`):** El agente consulta este archivo para saber cuál es la siguiente tarea a realizar. Es su "backlog" y su "historial de git".

3.  **El Conocimiento (`documentacion/` y `memoria/`):**
    *   `master_guide.md`: Antes de codificar, el agente lee este archivo para entender los requisitos exactos de la funcionalidad. Es la "documentación técnica".
    *   `decisiones_arquitectura.md`: Registra las decisiones de alto nivel para mantener la coherencia a largo plazo.
    *   `memoria/*.md`: La base de conocimiento. El agente consulta aquí para no cometer los mismos errores dos veces.

4.  **Las Herramientas de Calidad (`plantillas/` y `pull_requests/`):**
    *   `plantillas/`: Para asegurar consistencia, el agente usa estas plantillas como punto de partida para nuevo código.
    *   `PULL_REQUEST_TEMPLATE.md`: Antes de dar una tarea por finalizada, el agente completa esta checklist para verificar que ha cumplido con todos los estándares de calidad.

5.  **El Registro (`seguimiento/arbol.md` y `dependencias.md`):** Después de escribir código, el agente actualiza estos archivos para mantener un registro preciso del estado actual del proyecto.

6.  **La Red de Seguridad (`snapshots/`):** Después de un hito importante (`(merge)`), se crea una copia de seguridad completa del proyecto para poder restaurar en caso de un error catastrófico.

Este sistema, aunque manual en su concepción, crea un ciclo de desarrollo estructurado, documentado y auditable, impulsado por la inteligencia artificial.