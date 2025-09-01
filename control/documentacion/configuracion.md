# Configuración del Proyecto

Este documento detalla los archivos y configuraciones clave que rigen el comportamiento y las dependencias de este proyecto.

## 1. Archivos de Configuración Globales

### `package.json`
Este archivo, ubicado en la raíz del proyecto, define los metadatos del proyecto, sus dependencias (en `dependencies` y `devDependencies`), y los scripts que automatizan tareas como el inicio del servidor de desarrollo, la compilación, la ejecución de pruebas y la linting. Es el corazón de la gestión de paquetes tanto para el frontend como para el backend.

### `.env` (o `.env.local`, `.env.production`, etc.)
Los archivos `.env` se utilizan para gestionar variables de entorno, cruciales para la configuración sensible como claves de API, cadenas de conexión a bases de datos o puertos de servicio. Se recomienda no comitear estos archivos al repositorio y utilizar ejemplos como `.env.example`.

### `Dockerfile` y `docker-compose.yml`
Estos archivos se utilizan para la containerización de la aplicación. `Dockerfile` define cómo construir la imagen de Docker para la aplicación (incluyendo dependencias, código y configuración de entorno), mientras que `docker-compose.yml` orquesta múltiples servicios Docker (como la aplicación frontend, backend y la base de datos) para un entorno de desarrollo o producción completo.

## 2. Configuración del Frontend

### `vite.config.ts`
Este archivo configura el proceso de construcción y desarrollo del frontend, utilizando Vite. Define cómo se compila el código, cómo se resuelven los módulos, las optimizaciones de producción y los plugins utilizados (por ejemplo, para React).

### `jest.config.js`
Configuración para Jest, el marco de pruebas unitarias y de integración para el código React. Aquí se especifican los patrones de archivos de prueba, la configuración de Babel (o `ts-jest`), y la recolección de cobertura de código.

### `cypress.config.js`
Configuración para Cypress, utilizado para pruebas end-to-end (E2E) del frontend. Define los directorios de pruebas, la URL base de la aplicación y cualquier plugin o configuración específica de Cypress.

## 3. Configuración del Backend

### `backend/src/config/database.js`
Contiene la configuración de conexión a la base de datos (por ejemplo, MongoDB o PostgreSQL). Aquí se definen la URL de conexión, credenciales y opciones de configuración específicas del cliente de la base de datos.

### `backend/src/config/roles.js`
Define los roles de usuario y los permisos asociados dentro del sistema. Este archivo es fundamental para la implementación del control de acceso basado en roles (RBAC).

## 4. Configuración de CI/CD

### `.github/workflows/main.yml`
Este archivo de GitHub Actions define el pipeline de Integración Continua y Despliegue Continuo (CI/CD). Especifica los pasos para construir la aplicación, ejecutar pruebas (unitarias, de integración, E2E), y potencialmente desplegar el código a un entorno de staging o producción tras cada `push` o `Pull Request` a la rama principal.