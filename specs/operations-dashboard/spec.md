# Feature Specification — Operations Dashboard

**Product:** Yoga Studio
**Feature:** Operations Dashboard
**Delivery target:** Local academic MVP
**Status:** Specification for review

## 1. Problem

Andrea, propietaria de un estudio de yoga independiente, necesita saber rápidamente qué clases requieren atención operativa. Hoy debe revisar por separado la agenda, las reservas, las listas de espera y el historial de créditos. Esto dificulta detectar clases completas, lugares disponibles, actividad reciente y resultados de clase sin recorrer varias secciones.

El dashboard reunirá indicadores y registros ya disponibles en la aplicación. Será una vista de consulta: no creará ni modificará reservas, créditos, clases, listas de espera ni políticas.

## 2. ICP and User Persona

### ICP — Ideal Customer Profile

Estudio de yoga independiente pequeño, con una o dos instructoras, que administra agenda, capacidad, reservas y paquetes de clases en un navegador compartido o de uso local. Necesita una vista operativa sencilla y no requiere en esta entrega integraciones, informes financieros ni coordinación entre dispositivos.

### User Persona — Andrea

- **Rol:** propietaria y administradora del estudio.
- **Contexto:** consulta la aplicación local para organizar clases y revisar el estado operativo del estudio.
- **Necesidades:** identificar clases próximas, ocupación, lugares disponibles, clases completas, listas de espera y actividad reciente del ledger.
- **Objetivo:** encontrar en menos de 30 segundos las clases completas o con personas en espera y entender qué cambió recientemente.
- **Límite:** Andrea es un perfil ficticio de demostración. La selección de rol no verifica identidad ni protege datos.

## 3. Objectives and Functional Scope

### Objetivos

1. Presentar los datos operativos existentes en un solo lugar y con fórmulas transparentes.
2. Permitir revisar períodos distintos sin alterar el estado de reservas.
3. Hacer visibles las clases completas, las listas activas de espera y los movimientos recientes.
4. Actualizar los resultados cuando cambie el estado de la aplicación.

### Alcance incluido

- Vista de dashboard de solo lectura para el perfil simulado de administración.
- Indicadores de clases programadas, reservas confirmadas, capacidad, ocupación, lugares disponibles y clases completas.
- Conteo de personas en listas de espera activas.
- Conteos de cancelaciones, asistencias y ausencias registradas.
- Lista de movimientos recientes del ledger de créditos.
- Filtro temporal y filtro de estado para la lista de clases.
- Estados de carga inicial, sin datos y sin resultados para los filtros seleccionados.
- Actualización del dashboard después de operaciones existentes y al volver a mostrarlo.

El dashboard usa los datos y las operaciones implementados en `index.html`; no mantiene una copia paralela ni una colección persistida de indicadores. Los filtros son estado de presentación y no cambian reservas, créditos ni otros datos del dominio.

### Fuera de alcance

- Crear, editar o cancelar clases y reservas desde el dashboard.
- Cambiar capacidad, registrar asistencia, cambiar políticas o administrar créditos desde esta vista.
- Ingresos, pagos, ventas, rentabilidad, paquetes comerciales, retención, predicciones o notificaciones.
- Gráficos o tendencias que requieran datos históricos no registrados.
- Exportación, impresión, búsquedas avanzadas, filtros por alumna o instructora.
- Autenticación, backend, sincronización entre dispositivos, dependencias externas nuevas o telemetría.

## 4. Business Rules and Calculation Formulas

Los cálculos deben reutilizar las fuentes y reglas actuales de la aplicación, en particular `classStatus(source, item)` para reservas confirmadas, espera y disponibilidad, y los movimientos del ledger para el historial. No se debe volver a implementar la lógica de reservas ni calcular saldos de crédito desde una fuente distinta al ledger.

### 4.1 Universo de clases

- Una clase pertenece al período si su `startsAt` cae dentro del intervalo seleccionado.
- Una clase sigue siendo parte del registro aunque su fecha haya pasado; el dashboard no elimina ni oculta registros del modelo.
- Para cada clase se muestran nombre, fecha/hora local, instructora, confirmadas, capacidad, lugares disponibles, estado de ocupación y cantidad de personas en espera activas.

### 4.2 Indicadores principales

1. **Clases programadas:** cantidad de clases del período.
2. **Reservas confirmadas:** suma de reservas cuyo estado actual es `confirmed` en las clases del período.
3. **Capacidad total:** suma de `capacity` de las clases del período.
4. **Ocupación:** `reservas confirmadas / capacidad total × 100`, usando sumas agregadas (ocupación ponderada), redondeada a un decimal. Si no hay clases o la capacidad total es cero, mostrar `—`.
5. **Lugares disponibles:** suma de `capacity - reservas confirmadas` en las clases del período.
6. **Clases completas:** cantidad de clases del período cuya disponibilidad calculada por la aplicación es cero.

El contador de reservas confirmadas representa el estado actual de las reservas, no la cantidad histórica de solicitudes. Las clases completas se identifican con texto además de cualquier color o icono.

### 4.3 Listas de espera

- Una entrada cuenta como activa únicamente si su estado actual es `waiting`.
- El conteo y el listado se limitan a clases del período seleccionado.
- La posición se calcula con el orden FIFO que ya utiliza la aplicación; no se vuelve a ordenar ni se modifica la cola desde el dashboard.
- Las entradas `promoted` y `removed` no cuentan como personas esperando.
- La espera no ocupa capacidad ni consume créditos.

### 4.4 Cancelaciones, asistencias y ausencias

- Se cuentan reservas que actualmente tengan estado `cancelled`, `attended` o `absent`.
- Para estos tres conteos, la fecha del evento es `reservation.updatedAt`, que la aplicación actualiza al registrar ese resultado.
- Cada reserva se cuenta una sola vez en el grupo correspondiente a su estado actual. No se presenta como un historial completo de cambios de estado.
- El período se aplica a `updatedAt`, independientemente de la fecha de la clase. La lista de clases a la que corresponde cada resultado puede mostrar la fecha de clase como contexto.

### 4.5 Movimientos recientes

- La fuente es `state.movements`; no se derivan movimientos a partir de estados de reserva.
- Se ordenan por `createdAt` descendente. Si dos movimientos tienen la misma marca temporal, se muestra primero el que aparece después en el arreglo persistido.
- Se muestran como máximo los 10 movimientos más recientes que coincidan con el período.
- Cada fila presenta fecha/hora local, estudiante ficticia, tipo de movimiento, monto, operación y clase relacionada cuando exista. Los movimientos iniciales sin clase se identifican como crédito inicial/paquete, según el texto guardado.
- Los registros de tipo `grant`, `reserve`, `release` y `consume` se muestran tal como existen; no se infieren pagos ni compras.

## 5. Periods and Filters

### 5.1 Reloj de referencia

Todos los intervalos se calculan respecto de `currentTime(state)`: el reloj de demostración cuando está configurado; si no, la fecha y hora del dispositivo. El dashboard debe indicar cuando está activo el reloj de demostración. El reloj solo afecta la selección temporal; no cambia marcas temporales guardadas ni modifica el reloj del dispositivo.

Los límites se comparan como instantes locales coherentes con el modelo actual de fechas. Los intervalos son semiabiertos `[inicio, fin)`, excepto “Últimos 7 días” y “Últimos 30 días”, cuyo extremo superior es el instante de referencia e incluye registros con esa misma marca temporal.

### 5.2 Opciones del filtro temporal

- **Todas las fechas:** no aplica límites temporales; es la selección inicial para que la semilla ficticia sea visible incluso si sus fechas no coinciden con la fecha real del equipo.
- **Hoy:** desde el inicio del día local de referencia hasta el inicio del día local siguiente.
- **Últimos 7 días:** desde `currentTime - 168 horas` hasta `currentTime`, inclusive en el extremo superior.
- **Últimos 30 días:** desde `currentTime - 720 horas` hasta `currentTime`, inclusive en el extremo superior.
- **Próximos 7 días:** desde `currentTime` inclusive hasta `currentTime + 168 horas` exclusivo.

El filtro aplica a cada conjunto usando su fecha de referencia definida:

| Dato | Fecha usada para el filtro |
|---|---|
| Clases, reservas confirmadas y listas de espera | `class.startsAt` |
| Cancelaciones, asistencias y ausencias | `reservation.updatedAt` |
| Movimientos recientes | `movement.createdAt` |

Así, un filtro futuro puede incluir clases programadas y no incluir movimientos pasados, y un filtro pasado puede incluir actividad aunque no haya clases en ese intervalo.

### 5.3 Filtro de estado de clases

La lista de clases ofrece **Todas**, **Completas** y **Con lista de espera**. Este filtro solo cambia la lista de clases; no altera los indicadores agregados, conteos de resultados, listas de espera ni movimientos. Si no hay coincidencias, se muestra un estado vacío específico del filtro con una acción para volver a **Todas**.

### 5.4 Estado inicial y persistencia de filtros

- Al abrir el dashboard, el período inicial es **Todas las fechas** y el filtro de clases es **Todas**.
- La selección de filtros dura mientras la vista permanece abierta; no se guarda en `localStorage` ni se mezcla con el estado del dominio.
- Si cambia el reloj de demostración o los datos mientras la vista está abierta, se recalculan los intervalos y resultados con el filtro seleccionado.

## 6. Acceptance Criteria

Todos los criterios se prueban con datos ficticios. Antes de cada escenario, usar el control visible de reinicio de la aplicación. Las cifras de ejemplo corresponden a la semilla definida en `specs/class-booking/spec.md`: dos clases, capacidad total 15, tres reservas confirmadas en `class-small`, ninguna lista activa y nueve movimientos iniciales.

### AC01 — Indicadores derivados del estado de la aplicación

**Preparación:** reiniciar a la semilla y abrir el perfil simulado de administración. Seleccionar **Todas las fechas**.

**Verificación:** el dashboard muestra dos clases, tres reservas confirmadas, capacidad total 15, ocupación 20,0%, 12 lugares disponibles y una clase completa. El valor de lista de espera es cero. Los movimientos mostrados pertenecen al ledger existente; inicialmente hay nueve.

**Pasa si:** todos los valores coinciden con los registros actuales y con las fórmulas de esta especificación; cambiar el estado mediante las operaciones existentes cambia los valores derivados y no una cifra fija.

### AC02 — Filtro temporal actualiza los conjuntos correspondientes

**Preparación:** reiniciar, establecer el reloj de demostración en 1 de junio de 2030 a las 12:00 hora local y abrir el dashboard.

**Acciones:** observar **Todas las fechas**, seleccionar **Últimos 7 días** y luego **Próximos 7 días**.

**Verificación:** en **Todas las fechas** hay dos clases. En **Últimos 7 días** no hay clases de la semilla, porque comienzan el 4 de junio; los movimientos iniciales fechados el 1 de junio aparecen dentro del período si su marca temporal cae en el intervalo. En **Próximos 7 días** aparecen las dos clases. Cada lista de movimientos y cada conteo de resultados se filtra por sus propias fechas de referencia definidas en la sección 5, sin cambiar los datos persistidos.

**Pasa si:** cambiar el período actualiza todos los componentes sujetos a fecha, respeta la fecha de referencia de cada conjunto y no modifica reservas, movimientos ni balances.

### AC03 — Ocupación y capacidad reflejan los registros

**Preparación:** reiniciar y seleccionar **Todas las fechas**.

**Acciones:** como administración, cambiar la capacidad de `class-small` de 3 a 4 en la gestión de clases existente y volver al dashboard.

**Verificación:** `class-small` muestra tres confirmadas, cuatro de capacidad, un lugar disponible y 75,0% de ocupación. Los agregados muestran capacidad 16, tres confirmadas, 18,8% de ocupación (3/16 redondeado a un decimal) y 13 lugares disponibles. La clase ya no figura como completa.

**Pasa si:** los valores se actualizan desde la capacidad guardada y las reservas confirmadas actuales, sin superar la capacidad ni duplicar reglas de disponibilidad.

### AC04 — Espera y movimientos proceden de sus registros existentes

**Preparación:** reiniciar, elegir perfil de administración y **Todas las fechas**.

**Acciones:** agregar a Brisa y Celia, en ese orden, a la lista de espera de `class-small` usando la herramienta de administración existente.

**Verificación:** el dashboard presenta dos entradas activas en orden FIFO y posición correcta; ninguna cuenta como reserva confirmada. La cantidad de movimientos y los balances permanecen iguales porque ingresar a la espera no genera movimientos ni consume créditos. La lista de movimientos coincide con los registros del ledger y muestra primero los de fecha más reciente.

**Pasa si:** la espera se deriva solo de entradas `waiting`, y cada fila de movimientos corresponde a un movimiento guardado con sus datos asociados.

### AC05 — Resultados operativos y sus fechas

**Preparación:** reiniciar; configurar la política de demostración con plazo de 24 horas, cancelación tardía `release` y ausencia `consume`.

**Acciones:** establecer el reloj de prueba en 3 de junio de 2030 a las 12:00 y, como administración, cancelar la reserva de `student-01` en `class-small`; establecerlo en 5 de junio de 2030 a las 12:00 y, como instructora Laura, registrar asistencia para `student-02` y marcar ausente a `student-03`. Seleccionar **Todas las fechas**.

**Verificación:** el dashboard cuenta una cancelación, una asistencia y una ausencia, cada una una sola vez. Los tres resultados aparecen asociados a las fechas `updatedAt` de sus reservas. El ledger muestra los movimientos creados por esas operaciones, según la política configurada.

**Pasa si:** los conteos reflejan estados/resultados y marcas temporales existentes, sin inferir resultados ni contar dos veces una reserva.

### AC06 — Actualización después de operaciones y presentación responsive

**Preparación:** reiniciar; abrir el dashboard en el perfil de administración con **Todas las fechas** y viewport de escritorio.

**Acciones:** reservar `class-open` como `student-alma` mediante la vista existente; regresar al perfil de administración y observar el dashboard. Repetir la verificación visual con anchos de 320, 375 y 414 CSS px.

**Verificación:** después de la operación aparecen cuatro reservas confirmadas, 11 lugares disponibles, 26,7% de ocupación total (4/15) y el movimiento de reserva correspondiente. En los tres anchos, los controles y datos permanecen legibles y accesibles, y `document.documentElement.scrollWidth` no supera `window.innerWidth`.

**Pasa si:** los indicadores se recalculan desde el estado actualizado y no hay desbordamiento horizontal en ninguno de los anchos móviles definidos.

## 7. Quantifiable Success Metrics

1. **Exactitud de datos:** 100% de coincidencia entre cada indicador/lista y los registros fuente en los escenarios de aceptación AC01–AC05.
2. **Cobertura de indicadores:** al menos cinco indicadores principales disponibles; la especificación define seis.
3. **Rapidez de identificación:** 5 de 5 ejecuciones de una prueba manual con datos de semilla identifican una clase completa o con lista de espera en menos de 30 segundos desde que se muestra el dashboard.
4. **Overflow móvil:** cero desbordamientos horizontales a 320, 375 y 414 CSS px.
5. **Actualización:** tras una operación válida, los indicadores reflejan el estado guardado en el siguiente render, sin requerir recargar la página.

La prueba de rapidez es una comprobación manual de usabilidad académica, no telemetría ni un estudio estadístico de clientes.

## 8. Edge Cases and Empty States

- **No hay clases en el período:** mostrar clases programadas, confirmadas, capacidad, lugares y completas como `0`; ocupación como `—`; explicar que no hay clases para ese período.
- **No hay coincidencias con el filtro de clases:** conservar los indicadores agregados y mostrar un mensaje en la lista con opción de volver a **Todas**.
- **No hay listas activas:** mostrar contador `0` y un mensaje sin lista de filas vacía ambigua.
- **No hay movimientos en el período:** mostrar un estado vacío que explique que no hay movimientos registrados para ese intervalo; no crear movimientos sintéticos.
- **Capacidad agregada cero:** no dividir por cero; mostrar ocupación `—`. La aplicación actual valida capacidad de clase como entero positivo.
- **Clases con la misma fecha o movimientos con la misma marca temporal:** conservar un orden determinista; para clases, fecha ascendente y orden de origen para empates; para movimientos, regla de desempate definida en 4.5.
- **Clase completa y con espera:** ambas condiciones se muestran simultáneamente; el estado no depende solo del color.
- **Lista activa en clase no completa:** mostrar los registros activos sin eliminarlos ni promocionarlos desde el dashboard. No inventar una regla de promoción por cambio de capacidad.
- **Reserva en estado terminal:** solo se cuenta en el resultado que representa su estado actual. El dashboard no reconstruye estados previos que no estén en el modelo.
- **Reloj de demostración activo:** indicar visiblemente que los rangos usan hora de prueba. Si se limpia, recalcular con la hora del dispositivo.
- **Estado local no disponible o inválido:** reutilizar el error visible de la aplicación; no presentar indicadores como si fueran datos guardados correctamente ni sustituir silenciosamente el estado por valores vacíos.

## 9. Responsive and Accessibility Requirements

- Aplicar la identidad visual `yoga-brand-design`: fondo cálido, superficies claras, paleta natural, tipografía legible, espacio generoso y decoración contenida.
- Evitar dashboards densos, gradientes llamativos, animaciones innecesarias y acentos no funcionales.
- Mantener tarjetas de indicadores y listas en una columna en pantallas estrechas; no requerir desplazamiento horizontal de página entre 320 y 414 CSS px.
- Si una tabla no puede adaptarse sin perder legibilidad, convertir sus filas en tarjetas compactas en móvil, siguiendo el patrón existente.
- Usar HTML semántico, encabezados jerárquicos, etiquetas asociadas a filtros y controles operables con teclado.
- Mostrar foco visible y contraste de texto/controles conforme al objetivo WCAG 2.1 AA.
- Comunicar estados (completa, con lugares, con espera, vacío) con texto y no únicamente con color o icono.
- Anunciar cambios de resultados producidos por filtros mediante una región accesible apropiada, sin mover el foco de forma inesperada.
- Respetar controles y accesibilidad ya existentes en la selección de rol, reloj de demostración y reinicio.

## 10. Technical Limitations and Data Boundaries

- Implementación prevista: HTML, CSS y JavaScript vanilla en el `index.html` existente; no agregar framework, backend, base externa, dependencia nueva ni servicio de autenticación.
- Los datos son ficticios y se leen del estado local persistido por la aplicación en `localStorage`. El dashboard no añade una segunda persistencia ni almacena agregados.
- `localStorage` se limita al mismo navegador y origen; puede borrarse, bloquearse, editarse o perderse. No sincroniza dispositivos, no autentica roles y no ofrece respaldo, auditoría protegida ni coordinación transaccional entre usuarios/pestañas.
- Los roles de administración, instrucción y alumnado son perfiles simulados del cliente. El dashboard restringido visualmente a administración no constituye autorización de seguridad.
- El reloj de demostración es un control de prueba persistido por la aplicación y puede diferir del reloj del dispositivo. Los filtros deben usar la función de tiempo vigente de la aplicación, y los timestamps guardados conservan su propio valor.
- Los resultados disponibles son estados actuales y movimientos efectivamente registrados. No se pueden calcular ingresos, pagos, compras, pérdidas, rendimiento comercial, cambios de estado no guardados ni tendencias históricas completas.
- El dashboard no es una fuente de verdad independiente: sus valores dependen de la validez y disponibilidad del estado local y no proporciona garantías de concurrencia o exactitud multiusuario.

## 11. Definition of Done

- Los seis criterios de aceptación pasan con la semilla ficticia y pasos reproducibles.
- Los indicadores y registros se derivan del estado y reglas existentes, sin duplicar lógica del dominio.
- Los filtros y el reloj de demostración respetan las fechas de referencia indicadas.
- No se agregan operaciones de negocio ni datos comerciales fuera del alcance.
- La vista se adapta a móvil sin overflow horizontal y cumple los requisitos de accesibilidad y diseño establecidos.
- La interfaz comunica sus límites de demo local y no presenta perfiles simulados como autenticación.
