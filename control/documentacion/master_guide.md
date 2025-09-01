# Guía Maestra del Proyecto: AI Software Factory

Este es el documento central y la fuente única de verdad sobre el funcionamiento, arquitectura y codificación de cada funcionalidad del proyecto. Mantenlo actualizado con cada nueva implementación.

---

## Sistema de Control de Desarrollo

- **ID:** CTRL-001
- **Descripción:** Es el conjunto de documentos y directrices, ubicados en la carpeta `control/`, que gobiernan el ciclo de vida del desarrollo de software asistido por IA.
- **Componentes Clave:** `gemini_prompt_maestro.md` (cerebro), `roadmap.md` (planificación), `master_guide.md` (documentación), `memoria/` (base de conocimiento), `snapshots/` (backups).

---

## Funcionalidad: Agente de Génesis y Auto-Gobernanza

- **ID:** BOOT-003
- **Descripción:** Representa una evolución del "Agente de Bootstrap". El Agente de Génesis es responsable de crear proyectos que no solo son funcionalmente básicos, sino también **filosóficamente completos**. Al arrancar un nuevo repositorio, este agente replica la totalidad del sistema de `control/` (incluyendo esta misma guía, los roadmaps, la memoria y las instrucciones maestras) dentro del nuevo proyecto. Además, todos los demás agentes de IA están instruidos para leer y obedecer las directrices del `gemini_prompt_maestro.md` **del repositorio en el que están trabajando**, no las de la aplicación central.
- **Componentes Involucrados:** `useProjectState.ts`, `geminiService.ts`.
- **Lógica de Implementación:**
  1.  **Bootstrap Mejorado:** El prompt de `geminiService.generateProjectBootstrapFiles` ahora incluye la instrucción explícita de generar todos los archivos del directorio `control/` con contenido boilerplate sensato.
  2.  **Contextualización de Agentes:** Antes de que `useProjectState` invoque a un agente de IA para una tarea (ej. `generateImplementationForComponent`), primero carga el contenido de `control/instrucciones_agente/gemini_prompt_maestro.md` desde el repositorio activo.
  3.  **Inyección de Directrices:** El contenido de estas instrucciones maestras se inyecta en el prompt del agente, asegurando que su comportamiento se alinee con las reglas específicas del proyecto.
- **Decisiones Clave de Diseño:** Este cambio arquitectónico transforma los proyectos generados de simples artefactos de código a ecosistemas de software autónomos y auto-gobernados. Cada proyecto es ahora portable y contiene su propia "constitución" para la IA, cumpliendo la visión de un desarrollo verdaderamente descentralizado y gestionado por agentes.

---

## Funcionalidad: Agente Perro Guardián (Watchdog)

- **ID:** WATCHDOG-001
- **Descripción:** Implementa una salvaguarda contra servicios que se congelan o entran en bucles infinitos durante el arranque en el WebContainer. Monitoriza el tiempo de inicio de cada servicio y, si excede un límite, termina el proceso y activa el ciclo de auto-corrección para errores de runtime.
- **Componentes Involucrados:** `features/development/hooks/useServiceOrchestrator.ts`.
- **Lógica de Implementación:**
  1.  **Temporizador de Arranque:** Al iniciar los servicios, el `useServiceOrchestrator` establece un temporizador individual para cada uno (ej. 60 segundos).
  2.  **Detección de Bloqueo:** Si un servicio se inicia correctamente y alcanza el estado `READY`, su temporizador se cancela.
  3.  **Intervención Automática:** Si el temporizador expira y el servicio no ha alcanzado el estado `READY`, el agente watchdog se activa.
  4.  El agente mata el proceso del servicio congelado (`process.kill()`) para liberar recursos.
  5.  **Ciclo de Auto-Corrección:** Se invoca la callback `onRuntimeError` con un mensaje específico de "Timeout de arranque del servicio". Esto transiciona la tarea correspondiente a `RUNTIME_FAILED`, permitiendo que el agente de IA intente generar una corrección para el código que causó el bloqueo.
- **Decisiones Clave de Diseño:** Este mecanismo previene que un único servicio defectuoso congele todo el entorno de desarrollo. Actúa como la última capa de resiliencia, asegurando que el sistema pueda intentar recuperarse incluso de los fallos de arranque más problemáticos sin requerir una recarga manual de la página.

---

## Funcionalidad: Manejador de Tests Intermitentes (Flaky Tests)

- **ID:** FLAKY-001
- **Descripción:** Implementa una estrategia de CI más resiliente para manejar tests que fallan de forma intermitente. El pipeline de CI reintenta automáticamente los trabajos fallidos una vez. Si un trabajo tiene éxito en un segundo intento, el sistema lo detecta y registra una advertencia, ayudando a identificar deuda técnica en la suite de tests.
- **Componentes Involucrados:** `useProjectState.ts`, `geminiService.ts`, `core/types/index.ts`, `AgentLogWidget.tsx`.
- **Lógica de Implementación:**
  1.  **Configuración de CI:** Los agentes de IA en `geminiService` que generan el archivo `ci.yml` (`generateMissingBoilerplate`, `generateProjectBootstrapFiles`) son instruidos para configurar el trabajo principal de build y test con una estrategia de reintentos (`max-attempts: 2`).
  2.  **Detección:** El `useProjectState`, en la fase `PR_CREATED`, ahora utiliza `getWorkflowRuns` en lugar de `getCommitChecks` para obtener información más detallada del pipeline, incluyendo la propiedad `run_attempt`.
  3.  **Notificación:** Si un pipeline de CI se completa con éxito (`conclusion: 'success'`) pero el `run_attempt` es mayor que 1, el sistema registra una advertencia en el `AgentLogWidget` (ej: "CI para la tarea #X pasó en el intento #2. La suite de tests puede contener tests intermitentes.").
  4.  La tarea procede a la siguiente fase (`RUNTIME_VALIDATION`) como si el CI hubiera sido exitoso desde el principio.
- **Decisiones Clave de Diseño:** Este enfoque evita que el desarrollo se bloquee por fallos esporádicos que no son errores de código reales. La advertencia registrada sirve como un recordatorio de deuda técnica para que los tests intermitentes puedan ser investigados y corregidos posteriormente.

---

## Funcionalidad: Agente de Regresión

- **ID:** REGRESS-001
- **Descripción:** Implementa una lógica de depuración avanzada para solucionar regresiones. Cuando un cambio de código para una tarea rompe una funcionalidad existente en otra parte del proyecto, este agente se activa para proporcionar a la IA el contexto necesario para entender y solucionar el problema de forma más eficiente.
- **Componentes Involucrados:** `useProjectState.ts`, `geminiService.ts`.
- **Lógica de Implementación:**
  1.  **Detección de Regresión:** En la fase `GENERATING_FIX`, el `useProjectState` analiza los logs del CI fallido para identificar el archivo de test específico que ha fallado.
  2.  **Recopilación de Contexto Adicional:** Si se encuentra un archivo de test fallido, el sistema deduce la ruta del archivo de implementación correspondiente (ej: `componente.test.tsx` -> `componente.tsx`).
  3.  El sistema utiliza el `githubService` para obtener el contenido de ambos archivos (el test roto y su implementación) directamente desde la rama de la feature.
  4.  **Prompt Mejorado:** Este contexto adicional se inyecta en el prompt del agente de corrección (`generateFixForFailedCI`). Al agente se le instruye explícitamente que ha causado una regresión y que debe utilizar el código del test fallido y de la implementación rota para generar un arreglo que no solo cumpla con la tarea original, sino que también repare la funcionalidad que se rompió.
- **Decisiones Clave de Diseño:** Este enfoque simula cómo un desarrollador humano depuraría una regresión. Al proporcionar a la IA el "qué se rompió" (el test) y el "dónde se rompió" (la implementación), se aumenta drásticamente la probabilidad de que genere una corrección precisa en el primer intento.

---

## Funcionalidad: Agente Supervisor con Interruptor (Circuit Breaker)

- **ID:** SUPERVISE-001
- **Descripción:** Implementa un mecanismo de seguridad para prevenir que el agente de IA se quede atascado en bucles de corrección infinitos. Si un test es particularly difícil de pasar, la IA podría intentar arreglarlo repetidamente sin éxito. Este agente limita el número de intentos de corrección para una tarea fallida.
- **Componentes Involucrados:** `useProjectState.ts`, `IssuesWidget.tsx`, `core/types/index.ts`.
- **Lógica de Implementación:**
  1.  **Contador de Reintentos:** Se añade una propiedad `fixRetries` a la interfaz `Issue`.
  2.  **Límite Máximo:** Se define una constante `MAX_FIX_RETRIES` en `useProjectState.ts` (ej. 3).
  3.  **Lógica de Incremento:** Cuando una tarea entra en el estado `CI_FAILED` y no es un error de dependencias, el sistema comprueba el contador `fixRetries`. Si es menor que el máximo, se incrementa y se procede a `GENERATING_FIX`.
  4.  **Bloqueo Automático:** Si el contador alcanza el límite máximo, la tarea se transiciona al estado `ERROR` con un mensaje indicando que se ha alcanzado el límite de reintentos y se requiere intervención manual.
  5.  **Notificación en la UI:** El `IssuesWidget` ahora muestra el número de intento actual (ej. "Generando Corrección CI... (Intento 2/3)") y un mensaje de error específico cuando se alcanza el límite.
- **Decisiones Clave de Diseño:** Este mecanismo de "interruptor de circuito" previene el consumo excesivo de recursos y tiempo en problemas que superan la capacidad de auto-corrección actual de la IA, notificando al usuario para que pueda intervenir.

---

## Funcionalidad: Agente Gestor de Dependencias

- **ID:** DEPS-001
- **Descripción:** Implementa un agente especializado que se activa cuando el CI falla debido a una dependencia de npm faltante. El agente analiza los logs de error, identifica el paquete que falta, lo añade al `package.json` correspondiente, y comitea el cambio para re-lanzar el pipeline de CI automáticamente.
- **Componentes Involucrados:** `useProjectState.ts`, `geminiService.ts`, `core/types/index.ts`.
- **Lógica de Implementación:**
  1.  **Detección Específica:** En la máquina de estados de `useProjectState`, cuando una tarea entra en `CI_FAILED`, los logs se analizan en busca de patrones como "Cannot find module" o "module not found".
  2.  **Estado Dedicado:** Si se detecta un error de este tipo, la tarea transiciona al nuevo estado `MANAGING_DEPENDENCIES`, en lugar del estado genérico `GENERATING_FIX`.
  3.  **Corrección Automática:** En este estado, se invoca a un nuevo agente de IA, `geminiService.addMissingDependency`, pasándole el contenido del `package.json` y los logs del error.
  4.  La IA devuelve el contenido del `package.json` actualizado con la nueva dependencia y una versión estable.
  5.  **Reintento de CI:** El `useProjectState` comitea este `package.json` corregido a la rama de la feature y vuelve a transicionar la tarea al estado `PR_CREATED`, lo que efectivamente reinicia el proceso de validación del CI.
- **Decisiones Clave de Diseño:** Al especializarse en una clase común de errores de CI, el sistema se vuelve más rápido y eficiente que un agente de corrección genérico. Esto aumenta la autonomía del sistema al permitirle gestionar sus propias dependencias de software.

---

## Funcionalidad: Agente Guardián de Sintaxis

- **ID:** SYNTAX-001
- **Descripción:** Implementa un control de calidad "shift-left" que valida la sintaxis del código generado por la IA *antes* de que se realice un commit. Esto previene que se introduzca código roto en el repositorio, acelera el ciclo de feedback al no depender de la fase de CI, y hace al agente de IA más autónomo al permitirle corregir sus propios errores de sintaxis.
- **Componentes Involucrados:** `useProjectState.ts`, `useServiceOrchestrator.ts`, `geminiService.ts`, `core/types/index.ts`.
- **Lógica de Implementación:**
  1.  **Validación Pre-Commit:** Después de que la IA genera el código de implementación, la tarea entra en un nuevo estado, `VALIDATING_SYNTAX`.
  2.  **Ejecución de TSC:** `useProjectState` utiliza una nueva función `executeCommand` expuesta por el orquestador para ejecutar el compilador de TypeScript (`npx tsc --noEmit`) sobre el archivo recién generado (que se escribe temporalmente en el WebContainer).
  3.  **Flujo de Éxito:** Si el comando `tsc` devuelve un código de salida 0, la sintaxis es válida. El código se commitea y la tarea continúa su flujo normal.
  4.  **Flujo de Fallo (Auto-Corrección):** Si el código de salida es distinto de 0, la tarea pasa al estado `GENERATING_SYNTAX_FIX`. El error del compilador se guarda.
  5.  Se invoca a un nuevo agente en `geminiService`, `generateFixForSyntaxError`, pasándole el código erróneo y el mensaje de error de `tsc`.
  6.  El código corregido por la IA se recibe y la tarea vuelve al estado `VALIDATING_SYNTAX` para re-validar el arreglo.
  7.  **Prevención de Bucles:** Un contador de reintentos (`syntaxFixRetries`) limita el número de intentos de corrección. Si se supera el límite, la tarea se marca como fallida para permitir una intervención manual.
- **Decisiones Clave de Diseño:** Este proceso se integra directamente en la máquina de estados de la tarea, tratándolo como un paso más del desarrollo. El uso del propio compilador de TypeScript del entorno WebContainer asegura que la validación sea idéntica a la que se realizaría en un entorno de desarrollo local o en CI.

---

## Funcionalidad: Agente de Linting de Configuración

- **ID:** LINT-001
- **Descripción:** Extiende la capacidad del "Orquestador Auto-Corrector" para actuar como un agente de validación proactiva. Antes de intentar instalar dependencias (`npm install`), el orquestador ahora valida la sintaxis de archivos de configuración JSON críticos (`package.json`, `tsconfig.json`, `.eslintrc.json`) además del `services.json`.
- **Componentes Involucrados:** `features/development/hooks/useServiceOrchestrator.ts`, `useProjectState.ts` (para la corrección).
- **Lógica de Implementación:**
  1.  Dentro de `useServiceOrchestrator`, después de determinar los servicios a levantar, se itera sobre cada uno.
  2.  Para cada servicio, el sistema intenta leer y parsear (`JSON.parse()`) los archivos de configuración JSON relevantes que encuentre en su directorio.
  3.  Si un archivo existe pero su contenido no es un JSON válido, el orquestador lanza un `InvalidConfigError`.
  4.  Este error es capturado por el hook `useProjectState`, que activa el mismo flujo de auto-corrección que ya existía: invoca a la IA para reparar el archivo, comitea el arreglo y reintenta la orquestación.
- **Decisiones Clave de Diseño:** Esta validación proactiva previene fallos comunes durante la fase de instalación de dependencias, haciendo el arranque del entorno mucho más robusto y reduciendo la necesidad de intervención manual para corregir errores de sintaxis generados por la IA.

---

## Funcionalidad: Agente Validador de Grafos

- **ID:** PLAN-002
- **Descripción:** Actúa como un guardián de la lógica de planificación del proyecto. Antes de que un plan generado por la IA pueda ser comiteado, este agente lo analiza para detectar errores lógicos que podrían bloquear el desarrollo, como dependencias rotas o circulares.
- **Componentes Involucrados:** `features/development/components/AIArchitectWidget.tsx`, `lib/utils.ts`.
- **Lógica de Implementación:**
  1.  Después de que la IA genera el plan y la lista de tareas, pero antes de `onCommitPlan`, el `AIArchitectWidget` invoca a la nueva función `validateProjectPlan` de `lib/utils.ts`.
  2.  La función `validateProjectPlan` primero construye un grafo de dependencias y verifica que cada ID de dependencia exista realmente en la lista de tareas (previniendo dependencias rotas).
  3.  A continuación, ejecuta un algoritmo de **Búsqueda en Profundidad (DFS)** para recorrer el grafo y detectar ciclos (ej: Tarea A -> Tarea B -> Tarea A).
  4.  Si se encuentra un error (dependencia rota o ciclo), la función devuelve un mensaje de error descriptivo.
  5.  El `AIArchitectWidget` muestra este error en el modal y deshabilita el botón de "Generar Plan", forzando al usuario a refinar la descripción del proyecto para que la IA genere un plan lógicamente válido.
- **Decisiones Clave de Diseño:** La validación se realiza en el cliente antes de comitear, lo que proporciona un feedback inmediato al usuario y evita que se guarden planes de proyecto inválidos en el historial del repositorio.

---

## Funcionalidad: Agente de Resiliencia de Red

- **ID:** NET-001
- **Descripción:** Implementa un sistema de resiliencia de red global para que la aplicación maneje de forma elegante las pérdidas de conectividad. Detecta el estado de la red, notifica al usuario, deshabilita acciones de la UI y hace que las llamadas a API externas (GitHub, Gemini) sean resistentes a fallos temporales mediante reintentos automáticos.
- **Componentes Involucrados:** `App.tsx`, `ui/layout/Layout.tsx`, `ui/components/NetworkStatusBanner.tsx`, `core/hooks/useNetworkStatus.ts`, `lib/networkUtils.ts`, `lib/services/githubService.ts`, `lib/services/geminiService.ts`, y todos los widgets con acciones de red.
- **Lógica de Implementación:**
  1.  **Detección:** El hook `useNetworkStatus` escucha los eventos 'online' y 'offline' del navegador para proporcionar un estado booleano `isOnline`.
  2.  **Notificación:** `App.tsx` utiliza este hook y pasa el estado a `Layout.tsx`, que renderiza un `NetworkStatusBanner` global si `isOnline` es `false`.
  3.  **Resiliencia (Fetch):** Se ha creado una utilidad `resilientFetch` en `lib/networkUtils.ts` que envuelve `fetch` con una lógica de reintentos con "exponential backoff". El `githubService` se ha refactorizado para usar esta utilidad en todas sus llamadas.
  4.  **Resiliencia (SDK):** El `geminiService` utiliza una función de orden superior `withRetry` que envuelve las llamadas al SDK de Gemini, implementando la misma lógica de reintentos para errores de red.
  5.  **Control de UI:** El estado `isOnline` se pasa a todos los widgets relevantes para deshabilitar botones de acción que dependen de la red, evitando que el usuario inicie acciones que fallarían.
- **Decisiones Clave de Diseño:** La lógica de reintentos está centralizada para ser reutilizable (`resilientFetch` y `withRetry`), evitando duplicación de código. El feedback al usuario es inmediato y global, mejorando la experiencia de usuario durante las interrupciones de conectividad.

---

## Funcionalidad: Agente Vigilante de Recursos (Monitor de Salud del Entorno)

- **ID:** RES-001
- **Descripción:** Implementa un monitor de recursos en tiempo real para proporcionar visibilidad sobre la salud de la aplicación, específicamente el uso de memoria del heap de JavaScript. Ayuda a prevenir bloqueos del navegador al trabajar con repositorios grandes o procesos intensivos.
- **Componentes Involucrados:** `features/monitoring/components/EnvironmentHealthWidget.tsx`, `features/monitoring/hooks/useMemoryUsage.ts`, `ui/layout/Sidebar.tsx`.
- **Lógica de Implementación:**
  1.  El hook `useMemoryUsage` utiliza `setInterval` para sondear la API `performance.memory` del navegador, disponible en contextos seguros. Calcula el porcentaje de memoria usada y los valores en megabytes.
  2.  El `EnvironmentHealthWidget` consume este hook y renderiza una barra de progreso que cambia de color (verde, amarillo, rojo) según el consumo de memoria, proporcionando una alerta visual.
  3.  El `Sidebar` ha sido modificado para usar un layout flex, posicionando la navegación principal arriba y el `EnvironmentHealthWidget` en la parte inferior, asegurando que siempre esté visible.
- **Decisiones Clave de Diseño:** La monitorización es pasiva y solo de lectura para minimizar el impacto en el rendimiento. El widget maneja de forma segura los casos en que la API `performance.memory` no está disponible en el navegador del usuario.

---

## Funcionalidad: Agente de Persistencia de Sesión

- **ID:** PERSIST-001
- **Descripción:** Implementa la persistencia de la sesión del usuario y el estado del proyecto para evitar la pérdida de datos al recargar la página. La aplicación recuerda las credenciales, el repositorio activo, el estado de las tareas y la última vista activa, restaurándolos automáticamente al volver a abrir la aplicación.
- **Componentes Involucrados:** `App.tsx`, `core/hooks/useGitHubAuth.ts`, `core/hooks/useProjectState.ts`.
- **Lógica de Implementación:**
  1.  **Autenticación:** El hook `useGitHubAuth` guarda la sesión del usuario (usuario, token, repo) en `localStorage` tras una conexión exitosa. Al iniciar, el hook intenta leer y validar esta sesión para saltar el login. Al desconectar, la sesión se elimina.
  2.  **Estado del Proyecto:** `useProjectState` guarda el estado de las tareas y el plan del proyecto en `localStorage`, utilizando una clave única por repositorio (`project_state_${repoFullName}`). Al cargar un repositorio, intenta restaurar desde esta caché.
  3.  **Vista de la UI:** `App.tsx` guarda la última vista activa (ej. 'development') en `localStorage` y la restaura al cargar, devolviendo al usuario al panel donde se encontraba.
- **Decisiones Clave de Diseño:** Se utiliza `localStorage` por su simplicidad y amplio soporte. La separación de la caché de autenticación y la caché de estado del proyecto permite una gestión más granular y evita conflictos. Todos los accesos a `localStorage` se envuelven en bloques `try-catch` para manejar posibles errores o un almacenamiento deshabilitado en el navegador.

---

## Funcionalidad: Sistema de Doble Roadmap para Gobernanza del Agente

- **ID:** CTRL-004
- **Descripción:** Implementa un sistema de doble roadmap para asegurar que **todos** los cambios en el código, ya sean planificados o emergentes, sigan un proceso de trazabilidad estricto.
- **Componentes Involucrados:** `control/seguimiento/roadmap.md`, `control/seguimiento/roadmap_second.md`, `control/instrucciones_agente/gemini_prompt_maestro.md`.
- **Lógica de Implementación:**
  1.  **Roadmap Principal (`roadmap.md`):** Se utiliza exclusivamente para las fases y características planificadas del proyecto.
  2.  **Roadmap Secundario (`roadmap_second.md`):** Se utiliza para registrar todos los cambios no planificados, como hotfixes, refactorizaciones emergentes o pequeñas mejoras solicitadas que no estaban en el plan original.
  3.  **Protocolo del Agente:** Las instrucciones maestras del agente ahora le obligan a verificar si una tarea está en el roadmap principal. Si no lo está, tiene prohibido modificar el código hasta que haya creado una nueva entrada en el `roadmap_second.md` y documentado el ciclo completo (rama, commit, merge) allí.
- **Decisiones Clave de Diseño:** Esta separación previene cambios "ocultos" o no documentados, mejorando drásticamente la gobernanza y la auditoría del trabajo del agente. Fuerza una disciplina absoluta, asegurando que cada línea de código modificada esté justificada en uno de los dos roadmaps.

---

## Funcionalidad: UI/UX Overhaul con Layout de Sidebar

- **ID:** UI-002
- **Descripción:** Una refactorización completa de la interfaz de usuario para reemplazar el diseño de grid monolítico por un layout moderno y estructurado que utiliza una barra lateral de navegación (sidebar). Esto organiza los widgets en "vistas" contextuales, mejorando la usabilidad y reduciendo la carga cognitiva.
- **Componentes Involucrados:** `App.tsx`, `ui/layout/Layout.tsx`, `ui/layout/Sidebar.tsx`, `features/shared/components/Header.tsx`.
- **Lógica de Implementación:**
  1. El componente `App.tsx` ahora gestiona un estado de `view` que determina qué panel está activo (`dashboard`, `development`, etc.).
  2. Un nuevo componente `Layout.tsx` envuelve la aplicación en la etapa de desarrollo, organizando el `Header`, el `Sidebar` y el área de contenido principal.
  3. El `Sidebar.tsx` contiene los enlaces de navegación. Al hacer clic en un enlace, se actualiza el estado de la `view` en `App.tsx`.
  4. `App.tsx` utiliza un `switch` para renderizar condicionalmente el conjunto de widgets apropiado para la vista seleccionada en el área de contenido.
- **Decisiones Clave de Diseño:** Este cambio arquitectónico separa la navegación principal de la presentación del contenido. Permite que cada vista tenga su propio layout de grid optimizado para la tarea en cuestión, en lugar de un único grid para todo.

---

## Sistema Avanzado de Control de Calidad

- **ID:** CTRL-003
- **Descripción:** Un conjunto de herramientas y procesos para asegurar la consistencia, calidad и mantenibilidad del código.
- **Componentes Involucrados:**
  - `control/pull_requests/PULL_REQUEST_TEMPLATE.md`: Una checklist de calidad que el agente debe completar antes de finalizar una tarea. Actúa como un "Quality Gate".
  - `control/plantillas/`: Contiene plantillas de código base para nuevos archivos (componentes, hooks) para asegurar una estructura y estilo consistentes.
  - `control/seguimiento/deuda_tecnica.md`: Un registro para que el agente anote proactivamente áreas de mejora que no son errores, pero que deben ser abordadas en el futuro.
  - `control/documentacion/configuracion.md`: Un lugar centralizado para documentar variables de entorno, feature flags y otras configuraciones del proyecto.
- **Lógica de Implementación:** El agente de IA está instruido a través del `gemini_prompt_maestro.md` para utilizar estas herramientas como parte integral de su ciclo de desarrollo.

---

## Funcionalidad: Sistema de Snapshots

- **ID:** CTRL-002
- **Descripción:** Proporciona un mecanismo de copias de seguridad manual para preservar el estado completo del proyecto en puntos clave del desarrollo.
- **Componentes Involucrados:** `control/snapshots/`, `roadmap.md`.

---

## Funcionalidad: Sistema de Logging del Agente

- **ID:** LOG-001
- **Descripción:** Proporciona una vista en tiempo real de las acciones y pensamientos del agente de IA, mejorando la transparencia y la depuración.
- **Componentes Involucrados:** `AgentLogWidget.tsx`, `logger.ts`, `useProjectState.ts`.
- **Lógica de Implementación:** El `logger.ts` se ha refactorizado para ser un emisor de eventos. El hook `useProjectState` se suscribe a estos eventos, almacena los logs en un estado y los pasa al `AgentLogWidget`, que los renderiza en la UI.

---

## Funcionalidad: Agente Depurador y Validación en Runtime

- **ID:** DEBUG-001
- **Descripción:** Dota al sistema de la capacidad de detectar y corregir errores que solo se manifiestan en tiempo de ejecución (runtime), cerrando el bucle de feedback entre el código y la previsualización en vivo.
- **Componentes Involucrados:** `useProjectState.ts`, `useServiceOrchestrator.ts`, `LivePreviewWidget.tsx`, `IssuesWidget.tsx`, `geminiService.ts`.
- **Lógica de Implementación:**
  1.  Después de que una tarea pasa el CI, entra en un estado de `RUNTIME_VALIDATION`.
  2.  El hook `useServiceOrchestrator` monitoriza la salida de la consola de cada servicio.
  3.  Si se detecta un patrón de error, el hook invoca una callback `onRuntimeError` con el mensaje de error y el nombre del servicio que ha fallado.
  4.  `App.tsx` pasa esta llamada a `useProjectState`, que actualiza el estado de la tarea a `RUNTIME_FAILED` y almacena el error.
  5.  El agente de IA se activa, llama a `geminiService.generateFixForRuntimeError` y genera una corrección.
  6.  La corrección se commitea y el ciclo de CI/CD se reinicia.
- **Decisiones Clave de Diseño:** La detección de errores se basa en patrones de texto en la consola. La callback `onRuntimeError` incluye el nombre del servicio para permitir una depuración precisa en arquitecturas de microservicios.

---

## Funcionalidad: Agente de Bootstrap y Guardián

- **ID:** BOOT-001
- **Descripción:** El Agente de Bootstrap se activa para crear la estructura de archivos inicial y la configuración base de un proyecto. Lee el `PROJECT_PLAN.md` y genera los archivos esenciales (`package.json`, etc.). El "Guardián" es una sub-funcionalidad que asegura que los archivos JSON generados por la IA sean cadenas de texto válidas antes de ser comiteados, previniendo errores de tipo `[object Object]`.
- **Componentes Involucrados:** `useProjectState.ts`, `AIArchitectWidget.tsx`, `RepoStatusWidget.tsx`, `geminiService.ts`.
- **Lógica de Implementación:**
  1.  **Bootstrap Inicial:** El agente se activa si existe un `PROJECT_PLAN.md` pero faltan archivos clave como `package.json`.
  2.  **Auto-Reparación:** La acción "Reparar Estructura" invoca a este mismo agente.
  3.  **Guardián de Bootstrap:** Durante el proceso de bootstrap, `useProjectState` verifica el tipo de contenido de los archivos generados. Si un archivo JSON es un objeto, lo convierte a una cadena JSON (`JSON.stringify`) antes de realizar el commit.
- **Decisiones Clave de Diseño:** La validación se realiza en el momento de la creación para prevenir que se introduzcan configuraciones mal formadas en el repositorio desde el principio.

---

## Funcionalidad: Agente de Saneamiento de Configuración

- **ID:** BOOT-002
- **Descripción:** Complementa al "Guardián de Bootstrap". Este agente se activa al cargar un **repositorio existente**. Audita los archivos de configuración críticos (`services.json`, `package.json`) en busca de formatos incorrectos (ej. `[object Object]`). Si encuentra un error, lo corrige, comitea el arreglo automáticamente y luego procede con la orquestación.
- **Componentes Involucrados:** `useProjectState.ts`.
- **Lógica de Implementación:**
  1.  Durante la función `loadRepoAndPlan`, después de cargar el contenido de los archivos de configuración.
  2.  El agente itera sobre los archivos, verifica su `typeof`.
  3.  Si un archivo es un objeto en lugar de una cadena, se convierte a JSON, se comitea el cambio a GitHub y se actualiza el estado local antes de continuar.
- **Decisiones Clave de Diseño:** Este agente asegura la resiliencia contra repositorios que ya estaban en un estado corrupto, garantizando que el entorno de desarrollo siempre pueda intentar arrancar, sin importar si el proyecto es nuevo o existente.

---

## Funcionalidad: Agente de Contexto y Sistema RAG

- **ID:** CTX-001
- **Descripción:** Dota a la IA de una "memoria a largo plazo" del código base del proyecto. Indexa todos los archivos de código fuente y utiliza un sistema de Recuperación Aumentada por Generación (RAG) para encontrar fragmentos de código relevantes que se inyectan en los prompts de generación.
- **Componentes Involucrados:** `ContextAgentWidget.tsx`, `contextService.ts`, `useProjectState.ts`, `geminiService.ts`.
- **Lógica de Implementación:**
  1.  Al cargar un repositorio y después de cada `merge`, `useProjectState` invoca a `contextService.indexProject`.
  2.  `contextService` divide los archivos `.ts(x)` en "chunks" y los almacena en un índice en memoria.
  3.  Antes de generar código, `useProjectState` llama a `contextService.searchRelevantCode`.
  4.  Los chunks de código relevantes se inyectan en el prompt de `geminiService.generateImplementationForComponent`.
- **Decisiones Clave de Diseño:** Se implementó una búsqueda semántica mejorada utilizando el algoritmo **TF-IDF (Term Frequency-Inverse Document Frequency)**. Este método es superior a la simple búsqueda por palabras clave, ya que prioriza los términos que son significativos para un fragmento de código específico en el contexto de todo el proyecto. Es una simulación efectiva de un sistema RAG basado en embeddings, adaptada a las capacidades del entorno.

---

## Funcionalidad: Planificador de Tareas con Dependencias

- **ID:** PLAN-001
- **Descripción:** Permite al sistema entender y respetar las dependencias lógicas entre las tareas, asegurando que el desarrollo se realice en un orden coherente (ej: implementar autenticación antes que el perfil de usuario).
- **Componentes Involucrados:** `useProjectState.ts`, `geminiService.ts`, `IssuesWidget.tsx`, `FeatureSuggestionModal.tsx`, `core/types/index.ts`.
- **Lógica de Implementación:**
  1.  **Definición:** La IA, al sugerir características en `generateFeatureSuggestions`, ahora puede añadir un campo `dependsOn: ['id-de-otra-tarea']`.
  2.  **Visualización:** El modal de sugerencias y el widget de tareas muestran estas dependencias.
  3.  **Bloqueo:** Una tarea no puede ser iniciada (ni manual ni automáticamente) si sus dependencias no están en estado "Closed". La UI refleja esto con un icono de candado y deshabilitando el botón de inicio.
  4.  **Ejecución:** La lógica del "Modo Dios" en `useProjectState` ahora solo considera elegibles a las tareas que no tienen dependencias o cuyas dependencias ya han sido completadas.
- **Decisiones Clave de Diseño:** El sistema de dependencias se basa en los IDs de las tareas, creando un grafo de dependencias acíclico dirigido (DAG) que el planificador respeta. Esto permite una planificación de proyectos más compleja y realista.

---

## Funcionalidad: Orquestador Auto-Corrector (Self-Healing)

- **ID:** ORCH-002
- **Descripción:** Implementa un sistema de resiliencia de 3 fases para asegurar que el entorno de desarrollo de WebContainer se inicie de forma robusta, previniendo y corrigiendo automáticamente los errores de configuración generados por la IA.
- **Componentes Involucrados:** `useServiceOrchestrator.ts`, `useProjectState.ts`, `geminiService.ts`.
- **Lógica de Implementación:**
  1.  **Fase 1 (Prevención - Prompt Shielding):** Los prompts en `geminiService.ts` contienen reglas estrictas para la generación de archivos como `services.json` (debe ser un array), `package.json` (versiones fijas de React 18) y `vite.config.ts` (configuración HMR).
  2.  **Fase 2 (Detección - Validación Segura):** El hook `useServiceOrchestrator` valida la estructura del `services.json` antes de llamar a `WebContainer.boot()`. Si es inválido, lanza un `InvalidConfigError` en lugar de fallar.
  3.  **Fase 3 (Solución - Auto-Corrección):** El hook `useProjectState` captura este `InvalidConfigError`, invoca a un nuevo agente en `geminiService.correctInvalidJsonFile` para corregir el archivo, comitea el arreglo y reintenta la orquestación.
- **Decisiones Clave de Diseño:** Este sistema convierte un punto de fallo crítico en un ciclo de auto-mejora, haciendo al agente más autónomo. El reintento está limitado para evitar bucles infinitos.

---

## Funcionalidad: Agente Orquestador y Soporte de Terminales Múltiples

- **ID:** ORCH-001
- **Descripción:** Permite al sistema desarrollar, ejecutar y depurar aplicaciones compuestas por múltiples servicios (microservicios) de forma concurrente, similar a Docker Compose, pero directamente en el navegador. Cada servicio tiene su propia instancia de terminal.
- **Componentes Involucrados:** `useServiceOrchestrator.ts`, `LivePreviewWidget.tsx`, `ServiceStatusWidget.tsx`.
- **Lógica de Implementación:**
  1.  El "Agente Arquitecto" genera un archivo `services.json` que define cada servicio.
  2.  El hook `useServiceOrchestrator` gestiona el ciclo de vida de cada proceso del WebContainer.
  3.  La `LivePreviewWidget` ahora gestiona las instancias de `xterm.js`, creando una pestaña y una terminal para cada servicio, a la que se le redirige el `stdout`/`stderr` del proceso correspondiente.
- **Decisiones Clave de Diseño:** La gestión de la UI de la terminal se ha movido al `LivePreviewWidget`, separándola de la lógica de orquestación de procesos. Esto mejora la separación de conceptos y permite una UI más rica y flexible.

---

## Funcionalidad: Cacheo de Dependencias en WebContainer

- **ID:** PERF-001
- **Descripción:** Una optimización de rendimiento crítica que acelera significativamente los tiempos de arranque del entorno de desarrollo en ejecuciones posteriores a la primera.
- **Componentes Involucrados:** `features/development/hooks/useServiceOrchestrator.ts`.
- **Lógica de Implementación:**
  1.  **Detección de Cambios:** Antes de instalar dependencias, el orquestador usa el contenido del `package.json` de un servicio como clave de caché.
  2.  **Primera Ejecución (Cacheo):** Tras un `npm install` exitoso, se toma un `snapshot()` del directorio `node_modules` y se guarda en memoria.
  3.  **Ejecuciones Posteriores (Restauración):** Si la clave de caché no ha cambiado, se omite `npm install` y se restaura el `node_modules` desde el snapshot usando `mount()`.
- **Decisiones Clave de Diseño:** Este mecanismo asegura que `npm install` solo se ejecute cuando las dependencias realmente han cambiado, imitando el comportamiento de un lockfile y mejorando drásticamente la experiencia del desarrollador.

---

## Funcionalidad: Sistema de Archivos Virtual y Carga Diferida Híbrida

- **ID:** PERF-002
- **Descripción:** Optimiza la carga de repositorios al obtener inicialmente solo la estructura de archivos. El contenido de cada archivo se carga bajo demanda, ya sea por una acción del usuario (clic) o automáticamente por el agente de IA cuando lo necesita para una tarea ("hidratación just-in-time").
- **Componentes Involucrados:** `useProjectState.ts`, `githubService.ts`, `LivePreviewWidget.tsx`.
- **Lógica de Implementación:**
  1.  **Carga Inicial:** Se usa `githubService.getRepoStructure` para obtener solo el árbol de archivos. El estado `fileSystem` se inicializa con `code: null`.
  2.  **Carga por Usuario:** `handleSelectFile` se vuelve asíncrono y obtiene el contenido del archivo si es `null`.
  3.  **Hidratación para la IA:** Antes de que el agente ejecute una tarea (`GENERATING_TEST`, `IMPLEMENTING_CODE`), el hook `useProjectState` verifica si los archivos requeridos por la tarea están cargados. Si no, los obtiene automáticamente de GitHub antes de proceder.
- **Decisiones Clave de Diseño:** Este enfoque híbrido ofrece lo mejor de ambos mundos: una UI ultra-rápida para el usuario y un agente de IA completamente autónomo que nunca carece de contexto.

---

## Funcionalidad: Monitor de Orquestación en Tiempo Real

- **ID:** UI-001
- **Descripción:** Proporciona feedback visual detallado y en tiempo real durante la fase de preparación del entorno de desarrollo (arranque de WebContainer, instalación de dependencias, etc.).
- **Componentes Involucrados:** `IssuesWidget.tsx`, `useProjectState.ts`, `useServiceOrchestrator.ts`.
- **Lógica de Implementación:** `useServiceOrchestrator` ahora expone un objeto de progreso (`OrchestrationProgress`). `useProjectState` gestiona un temporizador y pasa el estado al `IssuesWidget`, que lo muestra en un overlay rediseñado. Esto elimina la ambigüedad sobre si el sistema está trabajando y qué está haciendo.

---

## Funcionalidad: Autenticación con GitHub

- **ID:** AUTH-001
- **Descripción:** Permite al usuario conectar su cuenta de GitHub a la aplicación utilizando un Nombre de Usuario y un Personal Access Token (PAT).
- **Componentes Involucrados:** `GitHubAuthWidget.tsx`, `GitHubRepoWidget.tsx`, `useGitHubAuth.ts`.

---

## Funcionalidad: Agente de Deuda Técnica

- **ID:** TD-001
- **Descripción:** Proporciona una herramienta para que el agente de IA, bajo supervisión humana, analice de forma proactiva toda la base de código en busca de deuda técnica.
- **Componentes Involucrados:** `ControlSystemWidget.tsx`, `useProjectState.ts`, `geminiService.ts`, `control/seguimiento/deuda_tecnica.md`.
- **Lógica de Implementación:** El usuario activa el análisis desde el `ControlSystemWidget`. El `useProjectState` orquesta la llamada a `geminiService.analyzeAndSuggestTechDebt`, que devuelve un análisis en Markdown. Este se añade a `deuda_tecnica.md` y se comitea al repositorio.

---

## Funcionalidad: Arquitectura por Capas y Gestión de Estado

- **ID:** `ARCH-001`
- **Descripción:** Una refactorización fundamental de la estructura del proyecto para mejorar la mantenibilidad, escalabilidad y una clara separación de conceptos. El código se organiza en capas lógicas y la gestión del estado se extrae de los componentes a hooks personalizados.
- **Componentes Clave:** `/core`, `/features`, `/lib`, `/ui`.

---

## Funcionalidad: Seguimiento de Issues (Issue Tracker)

- **ID:** TRACK-001
- **Descripción:** Un widget para la gestión y visualización de tareas o problemas generales del proyecto. Permite hacer un seguimiento de issues con título, descripción, prioridad y estado, de forma independiente al flujo de TDD automatizado.
- **Componentes Involucrados:** `IssueTrackerWidget.tsx`, `useProjectState.ts`, `core/types/index.ts`.

---

## Funcionalidad: Revisión de Código Asistida por IA

- **ID:** REVIEW-001
- **Descripción:** Transforma el flujo de aprobación de una tarea en un proceso de revisión de código interactivo. El usuario puede ver una comparación "diff" de los cambios realizados por la IA antes de fusionarlos.
- **Componentes Involucrados:** `CodeReviewModal.tsx`, `IssuesWidget.tsx`, `useProjectState.ts`, `githubService.ts`.
- **Lógica de Implementación:** Cuando una tarea está en `AWAITING_APPROVAL`, el `IssuesWidget` muestra un botón "Revisar Cambios". Esto abre un modal (`CodeReviewModal`) que utiliza el `DiffEditor` de Monaco para mostrar una vista lado a lado de los cambios.

---

## Funcionalidad: Diálogo de Clarificación con el Agente

- **ID:** INTERACTION-001
- **Descripción:** Permite que el agente de IA pause una tarea y solicite información adicional al usuario cuando los requisitos son ambiguos.
- **Componentes Involucrados:** `IssuesWidget.tsx`, `useProjectState.ts`, `geminiService.ts`.
- **Lógica de Implementación:** La IA puede devolver un `ClarificationNeededError`. El `useProjectState` captura este error, pone la tarea en estado `AWAITING_CLARIFICATION`, y la UI en `IssuesWidget` muestra un campo de texto para que el usuario responda.

---

## Funcionalidad: Testing del Dashboard de Desarrollo

- **ID:** TEST-001
- **Descripción:** Establece la estrategia y la infraestructura para escribir pruebas unitarias para la propia aplicación del dashboard, asegurando su robustez.
- **Componentes Involucrados:** `vitest.config.ts`, `vitest.setup.ts`, todos los archivos `*.test.tsx`.
- **Lógica de Implementación:** Utiliza `Vitest` y `@testing-library/react` para probar componentes y hooks críticos, como el flujo de autenticación.