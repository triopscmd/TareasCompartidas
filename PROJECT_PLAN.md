# PROJECT_PLAN.md

## 1. Autenticación de Usuarios (autenticacion-usuarios)
Esta sección aborda la implementación de los mecanismos de registro, inicio de sesión y gestión de sesiones de usuario.

### Archivos:
- `src/api/auth.ts`
- `src/components/AuthForm.tsx`
- `src/pages/LoginPage.tsx`
- `src/pages/RegisterPage.tsx`
- `src/context/AuthContext.tsx`
- `src/hooks/useAuth.ts`
- `src/services/authService.ts`
- `backend/src/controllers/authController.js`
- `backend/src/routes/authRoutes.js`
- `backend/src/models/User.js`
- `backend/src/utils/jwt.js`

## 2. Gestión de Perfil de Usuario (gestion-perfil-usuario)
Permite a los usuarios ver y actualizar su información personal, como nombre, email y avatar.

### Archivos:
- `src/api/user.ts`
- `src/pages/ProfilePage.tsx`
- `src/components/ProfileForm.tsx`
- `src/components/AvatarUpload.tsx`
- `src/services/userService.ts`
- `backend/src/controllers/userController.js`
- `backend/src/routes/userRoutes.js`
- `backend/src/models/User.js` (modificación)

## 3. Creación y Gestión de Grupos (gestion-grupos)
Habilita la creación de grupos, la adición/eliminación de miembros y la gestión de la información del grupo.

### Archivos:
- `src/api/groups.ts`
- `src/pages/GroupListPage.tsx`
- `src/pages/GroupDetailsPage.tsx`
- `src/components/GroupForm.tsx`
- `src/components/GroupCard.tsx`
- `src/services/groupService.ts`
- `backend/src/controllers/groupController.js`
- `backend/src/routes/groupRoutes.js`
- `backend/src/models/Group.js`
- `backend/src/models/UserGroup.js`

## 4. Creación de Tareas (creacion-tareas)
Permite a los usuarios definir nuevas tareas, incluyendo título y descripción.

### Archivos:
- `src/api/tasks.ts`
- `src/components/TaskForm.tsx`
- `src/pages/TaskCreationPage.tsx`
- `src/services/taskService.ts`
- `backend/src/controllers/taskController.js`
- `backend/src/routes/taskRoutes.js`
- `backend/src/models/Task.js`

## 5. Asignación de Tareas (asignacion-tareas)
Funcionalidad para asignar tareas a uno o varios miembros de un grupo.

### Archivos:
- `src/components/TaskAssignmentForm.tsx`
- `src/components/TaskDetailsModal.tsx`
- `src/api/tasks.ts` (modificación)
- `src/services/taskService.ts` (modificación)
- `backend/src/controllers/taskController.js` (modificación)
- `backend/src/models/Task.js` (modificación)
- `backend/src/models/UserTask.js`

## 6. Establecimiento de Fechas Límite (fechas-limite-tareas)
Capacidad para fijar fechas y horas límite para cada tarea.

### Archivos:
- `src/components/DateTimePicker.tsx`
- `src/components/TaskForm.tsx` (modificación)
- `src/api/tasks.ts` (modificación)
- `src/services/taskService.ts` (modificación)
- `backend/src/controllers/taskController.js` (modificación)
- `backend/src/models/Task.js` (modificación)

## 7. Gestión del Estado de Tareas (gestion-estado-tareas)
Permite a los usuarios cambiar el estado de una tarea (ej. 'pendiente', 'en progreso', 'completada').

### Archivos:
- `src/components/TaskStatusToggle.tsx`
- `src/components/TaskItem.tsx` (modificación)
- `src/api/tasks.ts` (modificación)
- `src/services/taskService.ts` (modificación)
- `backend/src/controllers/taskController.js` (modificación)
- `backend/src/models/Task.js` (modificación)

## 8. Sistema de Notificaciones (notificaciones-sistema)
Implementación de un sistema para enviar y mostrar notificaciones dentro de la aplicación.

### Archivos:
- `src/components/NotificationCenter.tsx`
- `src/context/NotificationContext.tsx`
- `src/hooks/useNotifications.ts`
- `src/api/notifications.ts`
- `src/services/notificationService.ts`
- `backend/src/controllers/notificationController.js`
- `backend/src/routes/notificationRoutes.js`
- `backend/src/models/Notification.js`
- `backend/src/utils/realtime.js`

## 9. Recordatorios Automáticos (recordatorios-automaticos)
Configuración y envío de recordatorios automáticos a los usuarios sobre tareas pendientes o próximas fechas límite.

### Archivos:
- `backend/src/services/schedulerService.js`
- `backend/src/utils/emailService.js`
- `backend/src/utils/notificationSender.js`
- `backend/src/jobs/reminderJob.js`
- `backend/src/models/Task.js` (consulta)
- `backend/src/models/User.js` (consulta)

## 10. Vista de Lista de Tareas (vista-lista-tareas)
Interfaz de usuario para visualizar todas las tareas, posiblemente con opciones de filtrado y ordenamiento.

### Archivos:
- `src/pages/TaskListPage.tsx`
- `src/components/TaskList.tsx`
- `src/components/TaskItem.tsx`
- `src/components/FilterSortControls.tsx`
- `src/api/tasks.ts` (modificación)
- `src/services/taskService.ts` (modificación)
- `backend/src/controllers/taskController.js` (modificación)

## 11. Edición Colaborativa de Tareas (edicion-colaborativa-tareas)
Permite a varios usuarios editar la misma tarea simultáneamente, con actualizaciones en tiempo real.

### Archivos:
- `src/components/TaskDetailsModal.tsx` (modificación)
- `src/hooks/useRealtimeEditing.ts`
- `backend/src/utils/websocket.js`
- `backend/src/controllers/taskController.js` (modificación para emitir eventos)
- `backend/src/models/Task.js` (modificación para gestión de concurrencia)

## 12. Autorización y Control de Acceso (autorizacion-acceso)
Define roles y permisos para controlar quién puede ver, crear, editar o eliminar tareas y grupos.

### Archivos:
- `src/hocs/withAuthorization.tsx`
- `src/utils/permissions.ts`
- `backend/src/middleware/authMiddleware.js`
- `backend/src/middleware/permissionMiddleware.js`
- `backend/src/models/User.js` (adición de roles/permisos)
- `backend/src/models/Group.js` (adición de roles/permisos)
- `backend/src/config/roles.js`

## 13. Diseño de API y Base de Datos (diseno-api-base-datos)
Documentación y esquemas iniciales para la API REST y la estructura de la base de datos.

### Archivos:
- `docs/api_design.md`
- `docs/database_schema.md`
- `backend/src/models/User.js`
- `backend/src/models/Group.js`
- `backend/src/models/Task.js`
- `backend/src/models/Notification.js`
- `backend/src/models/UserGroup.js`
- `backend/src/models/UserTask.js`
- `backend/src/config/database.js`

## 14. Pipeline de CI/CD (pipeline-ci-cd)
Configuración de un pipeline de Integración Continua/Despliegue Continuo para automatizar pruebas y despliegues.

### Archivos:
- `.github/workflows/main.yml`
- `Dockerfile`
- `docker-compose.yml`
- `scripts/deploy.sh`
- `package.json` (scripts)

## 15. Pruebas Automatizadas (pruebas-automatizadas)
Creación de pruebas unitarias, de integración y end-to-end para asegurar la calidad del software.

### Archivos:
- `src/tests/components/AuthForm.test.tsx`
- `src/tests/pages/LoginPage.test.tsx`
- `backend/src/tests/unit/authService.test.js`
- `backend/src/tests/integration/authRoutes.test.js`
- `jest.config.js`
- `cypress.config.js`
- `cypress/e2e/auth.cy.js`
