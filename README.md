# Yoga Studio — Prototype III

Proyecto III · Fundamentos de Product Engineering con IA · Código Facilito

Prototipo web para la gestión operativa de un estudio de yoga independiente. Esta tercera entrega incorpora un **Operations Dashboard** que reúne información de clases, reservas, ocupación, listas de espera y movimientos de créditos en una vista de consulta.

## 1. Problema

En estudios pequeños, la agenda, las reservas, los paquetes de clases y la asistencia pueden administrarse mediante tarjetas, mensajes y registros separados. Esa fragmentación dificulta conocer rápidamente la disponibilidad de cada clase, detectar sobrecupos y revisar la actividad reciente.

Yoga Studio explora cómo centralizar esas operaciones en una interfaz sencilla, sin introducir la complejidad de una plataforma de gestión integral.

## 2. Usuario objetivo

### ICP — Ideal Customer Profile

Estudios de yoga independientes pequeños, gestionados por su propietaria o una coordinadora y con una o dos instructoras. Administran agenda, capacidad, reservas y paquetes de clases con procesos manuales o herramientas dispersas. Necesitan una vista operativa simple y no requieren, para este prototipo, integraciones ni coordinación entre múltiples dispositivos.

### User Persona — Andrea

Andrea tiene 42 años y es propietaria y administradora de un estudio de yoga independiente. Dedica parte de su tiempo a organizar clases, controlar reservas y créditos, responder consultas y resolver cancelaciones. Necesita identificar rápidamente qué clases están completas, dónde hay lugares disponibles y qué actividad se registró recientemente. Su objetivo es reducir la carga administrativa y tener una visión clara de la operación cotidiana.

*Andrea es una persona ficticia utilizada como referencia de diseño.*

## 3. Funcionalidades

### Gestión de clases y reservas

- Consulta y gestión de clases.
- Reservas con validación de créditos disponibles.
- Control de capacidad por clase.
- Lista de espera con orden FIFO.
- Cancelaciones y promoción de alumnas en espera según las reglas existentes.
- Registro de asistencia y ausencia.
- Historial de movimientos de créditos.
- Políticas configurables de cancelación y ausencia.
- Perfiles de demostración para administración, instructoras y alumnas.

### Nueva funcionalidad — Operations Dashboard

La novedad de Prototype III es un dashboard de administración de solo lectura. Reúne datos que antes debían consultarse en distintas secciones:

- Clases programadas y reservas confirmadas.
- Capacidad total, ocupación y lugares disponibles.
- Clases completas y listas de espera activas.
- Cancelaciones, asistencias y ausencias registradas.
- Movimientos recientes del ledger de créditos.
- Filtros temporales: todas las fechas, hoy, últimos 7 días, últimos 30 días y próximos 7 días.
- Filtro de clases por estado.

Los indicadores se calculan a partir del estado existente de la aplicación. El dashboard no crea ni modifica reservas, clases o créditos.

La especificación funcional está en [specs/operations-dashboard/spec.md](specs/operations-dashboard/spec.md), con reglas de cálculo, criterios de aceptación, métricas de éxito y casos límite.

## 4. Tecnología e implementación

- **HTML5**
- **CSS3**
- **JavaScript vanilla**
- **localStorage** para persistencia local en el navegador.
- **Netlify** para publicar la demo.

No utiliza framework ni requiere instalar dependencias para probar la versión publicada.

## 5. Alcance y limitaciones

Este proyecto es un prototipo académico funcional, no un sistema listo para operar un estudio real.

- Los datos precargados son ficticios.
- La persistencia está limitada al navegador y al origen donde se ejecuta la aplicación.
- No hay backend ni base de datos remota.
- No existe sincronización entre dispositivos o usuarios.
- Los perfiles y roles son simulados: no hay autenticación ni autorización real.
- No incluye pagos, ingresos, reportes financieros ni integraciones externas.
- Los datos locales pueden borrarse o modificarse desde el navegador.

Por estas limitaciones, no debe utilizarse para almacenar información personal real ni para administrar operaciones reales.

## 6. Cómo probar la demo

1. Abrir la [aplicación publicada](https://yoga-studio-prototype.netlify.app).
2. Explorar los perfiles de demostración disponibles.
3. Probar reservas, cancelaciones, listas de espera y registro de asistencia.
4. Acceder al perfil de administración y abrir **Operations Dashboard**.
5. Cambiar los filtros y realizar operaciones para observar cómo se actualizan los indicadores.
6. Utilizar la opción de reinicio para recuperar los datos ficticios iniciales.

## 7. Estructura del repositorio

- `index.html`: interfaz, estilos y lógica de la aplicación.
- `specs/class-booking/spec.md`: especificación de la gestión de clases y reservas.
- `specs/operations-dashboard/spec.md`: especificación del nuevo dashboard operativo.
- `AGENTS.md`: contexto e instrucciones del proyecto.
- `.agents/skills/yoga-brand-design/SKILL.md`: lineamientos de identidad visual.

## 8. Enfoque de desarrollo

El proyecto utiliza un enfoque de **Specification-Driven Development (SDD)**: definir el problema y el alcance, documentar reglas y criterios de aceptación, implementar y verificar el comportamiento esperado.

Prototype III parte de la gestión de clases y reservas de Prototype II y agrega una nueva capa de consulta operativa. La evolución se documenta en la especificación del dashboard de usuario Administradora · Andrea.
