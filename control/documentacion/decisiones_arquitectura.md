# Registro de Decisiones de Arquitectura (ADR)

Este documento registra las decisiones arquitectónicas importantes tomadas durante el proyecto.

---

## 004: Adopción de Proyectos Auto-Gobernados con Sistema de Control In-Repo

- **Fecha:** 2024-06-01
- **Estado:** Aceptado
- **Contexto:** El sistema de IA operaba como un "Controlador Externo", donde las reglas de los agentes residían en la aplicación del dashboard, y los proyectos en GitHub eran entidades pasivas. Esto entraba en conflicto con la filosofía de crear ecosistemas de software autónomos. Se necesitaba que cada proyecto contuviera su propio sistema de gobierno.
- **Decisión:** Se ha refactorizado el sistema para adoptar un modelo de "Ecosistemas Autónomos".
  1.  **Agente de Génesis:** El "Agente de Bootstrap" ahora tiene la responsabilidad de replicar la estructura completa del directorio `control/` (guías, roadmaps, memoria, instrucciones maestras) dentro de cada nuevo repositorio que crea.
  2.  **Agentes Conscientes del Contexto:** Antes de que un agente de IA realice cualquier tarea, el sistema ahora carga el `gemini_prompt_maestro.md` desde el repositorio activo del usuario y lo inyecta en el prompt. Esto fuerza al agente a seguir las reglas específicas del proyecto en el que está trabajando.
- **Consecuencias:**
  - **Positivas:** Cada proyecto se convierte en una unidad verdaderamente autónoma y portable. La gobernanza del agente es transparente y está versionada junto con el código. Se alinea perfectamente con la visión a largo plazo del proyecto.
  - **Negativas:** El proceso de bootstrap es más pesado, ya que debe generar una cantidad significativa de archivos de control. Aumenta la complejidad de la lógica de los prompts, que ahora deben ser dinámicos.

---

## 001: Refactorización a Arquitectura por Capas y Funcionalidades

- **Fecha:** 2024-05-21
- **Estado:** Aceptado
- **Contexto:** La estructura inicial de la aplicación era "plana" y no escalaba bien. Se necesitaba una separación de conceptos más clara para mejorar la mantenibilidad y la organización del código.
- **Decisión:** Se reorganizó el proyecto en una estructura por capas (`core`, `features`, `lib`, `ui`). La lógica de estado se extrajo de los componentes a hooks personalizados (`useProjectState`, `useGitHubAuth`), y los servicios externos se aislaron en `lib/services`.
- **Consecuencias:**
  - **Positivas:** Mayor mantenibilidad, escalabilidad, y una clara separación de conceptos. Es más fácil añadir nuevas funcionalidades o reemplazar servicios.
  - **Negativas:** La estructura puede parecer más compleja inicialmente. Requiere una disciplina para mantener la separación de capas.

---

## 002: Gestión de Estado Centralizada con Hooks de React

- **Fecha:** 2024-05-22
- **Estado:** Aceptado
- **Contexto:** El estado global de la aplicación (autenticación, estado del proyecto, sistema de archivos) necesitaba ser accesible por múltiples componentes. Se consideraron librerías como Redux o Zustand.
- **Decisión:** Se optó por usar hooks nativos de React (`useState`, `useEffect`, `useCallback`) encapsulados en hooks personalizados (`useProjectState`). Esto evita añadir dependencias externas, reduce la complejidad y aprovecha el motor de React para la gestión de re-renderizados.
- **Consecuencias:**
  - **Positivas:** Menos "boilerplate", curva de aprendizaje más suave, y se mantiene dentro del ecosistema de React.
  - **Negativas:** Para aplicaciones extremadamente complejas, podría llevar a un "prop drilling" si no se gestiona con cuidado (aunque React Context podría mitigar esto si fuera necesario).

---

## 003: Adopción de un Sistema de Control Manual Asistido por IA

- **Fecha:** 2024-05-23
- **Estado:** Aceptado
- **Contexto:** El requisito del proyecto es explorar un paradigma de desarrollo donde la gestión del ciclo de vida del software es gobernada por un agente de IA, con supervisión humana, pero sin las herramientas tradicionales de control de versiones como Git. Se necesita un sistema que sirva como "memoria" y "guía" para el agente.
- **Decisión:** Se ha diseñado e implementado un sistema de control basado en archivos Markdown dentro de una carpeta `control/`. Este sistema incluye un roadmap, una guía maestra de funcionalidades, un registro de decisiones, un árbol de directorios, un mapa de dependencias y una "memoria" de problemas/soluciones. El agente de IA es instruido (`gemini_prompt_maestro.md`) para leer y actualizar estos archivos como parte de su ciclo de trabajo.
- **Consecuencias:**
  - **Positivas:** Permite un control total sobre el flujo de trabajo de la IA. Todos los artefactos de gestión del proyecto son legibles por humanos y están versionados junto con el código fuente (mediante snapshots). Sirve como un experimento autocontenido sobre desarrollo dirigido por IA.
  - **Negativas:** Es un sistema manual que carece de las robustas capacidades de Git (historial de cambios a nivel de línea, branching/merging eficiente, resolución de conflictos). Escala pobremente para equipos humanos. Su éxito depende enteramente de la disciplina del agente de IA para mantener los archivos actualizados.