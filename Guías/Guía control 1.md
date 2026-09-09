## Caso de práctica para control 1: Veterinaria Coco

> La Veterinaria Coco quiere digitalizar el agendamiento de horas médicas. Los pacientes deben poder ingresar su RUT desde el sitio web y ver los horarios disponibles de un especialista, y luego confirmar una hora. El sistema debe validar que el RUT sea válido antes de mostrar horarios.

> La recepcionista comenta: "hoy anotamos todo a mano y se nos duplican las horas, necesitamos que el sistema bloquee automáticamente un horario apenas alguien lo reserva, para que no se agende dos veces la misma hora con el mismo médico."

> El jefe de TI advierte: "en horario de alta demanda (lunes en la mañana) llegan hasta 3.000 solicitudes de agendamiento simultáneas, y el sistema no puede demorar más de 2 segundos en confirmar disponibilidad, o los pacientes abandonan la reserva."

> El equipo médico pide: "necesitamos que cada especialista pueda ver, desde una app móvil, su lista de pacientes agendados del día."

> El área de cumplimiento normativo exige: "toda ficha clínica y dato de salud debe quedar cifrado en la base de datos, es obligación legal según la Ley 19.628 de protección de datos."

> Los desarrolladores evaluaron dos alternativas de base de datos para guardar las reservas: PostgreSQL, por su soporte transaccional fuerte para evitar choques de horarios, y Firebase Realtime Database, por su facilidad de sincronización en tiempo real con la app móvil de los médicos. Finalmente el equipo decidió usar PostgreSQL, priorizando la integridad transaccional por sobre la sincronización en tiempo real, y resolver la app móvil con consultas periódicas (polling) en vez de sincronización nativa.

> El sistema debe enviar un recordatorio por correo 24 horas antes de la hora agendada.

### Preguntas

1. Diagrama usando 4+1: Elige al menos 2 vistas distintas (por ejemplo Lógica + Procesos, o Escenario + Desarrollo) y diagrama el caso de "un paciente reserva una hora médica".

2. Identifica los RF del enunciado y determina cuáles se convierten en ASR, justificando.

3. Identifica los stakeholders.

4. Identifica el ADR presente en el caso.

---

### Yapo intenten hacerlo sin pauta... yo sé q pueden...





















### ### Pauta sugerida... hay hartas respuestas q son válidas! 

### (2) RF y ASR

RF 1: El sistema debe validar que el RUT sea válido antes de mostrar horarios.

ASR: No.

Justificación: Es una validación local que afecta solo al módulo de ingreso de datos. No obliga a cambiar la arquitectura si se corrige después.

RF 2: El sistema debe bloquear automáticamente un horario apenas se reserve, evitando duplicados.

ASR: Sí.

Justificación: El estímulo corresponde a dos solicitudes simultáneas por el mismo horario. La condición es una situación de concurrencia alta y la respuesta esperada es que solo una reserva se confirme.

Este requerimiento obliga a decidir mecanismos de concurrencia y transacciones, por ejemplo locks o transacciones ACID. Por lo tanto, afecta a la capa de datos completa y no solamente a una función puntual.

RF 3: El sistema debe responder disponibilidad en menos de 2 segundos con hasta 3.000 solicitudes simultáneas durante el lunes en la mañana.

ASR: Sí.

Justificación: Tiene un estímulo de 3.000 solicitudes simultáneas, una condición de horario peak y una medida explícita de 2 segundos.

Este requerimiento obliga a decidir sobre mecanismos como balanceo de carga, caching o escalamiento horizontal, por lo que impacta a toda la infraestructura.

RF 4: Cada especialista debe ver su lista de pacientes del día desde una app móvil.

ASR: No.

Justificación: Es una funcionalidad concreta sin un umbral que fuerce una decisión estructural.

Sería ASR si se agregara una condición medible, por ejemplo: "debe sincronizar en menos de X segundos".

RF 5: Toda ficha clínica debe quedar cifrada en la base de datos.

ASR: Sí.

Justificación: Es un NFR de seguridad con una obligación legal explícita. Afecta a toda la capa de datos y a cómo se diseña el acceso a la base de datos.

Corregirlo después de tener datos en texto plano sería muy costoso debido a la migración y al riesgo legal.

RF 6: El sistema debe enviar un recordatorio por correo 24 horas antes de la hora agendada.

ASR: No.

Justificación: Es una función puntual que puede implementarse como una tarea programada y no compromete la estructura general del sistema.

### (3) Stakeholders

Desarrolladores (Developers): Son responsables de implementar el sistema y de tomar decisiones técnicas. En este caso, evaluaron PostgreSQL y Firebase Realtime Database y participaron en la decisión arquitectónica.

Testers: Son responsables de verificar que el sistema cumpla los requisitos. Deben comprobar, entre otras cosas, que no se produzcan reservas duplicadas, que el sistema soporte 3.000 solicitudes simultáneas y que los datos clínicos se encuentren cifrados.

Mantenedores (Maintainers): Son responsables de mantener el sistema una vez implementado, corrigiendo errores y realizando modificaciones y mejoras.

Administradores del sistema (System Administrators): Se encargan de administrar servidores, bases de datos, permisos, configuraciones y otros componentes necesarios para el funcionamiento del sistema.

Ingenieros de producción / DevOps (Production Engineers): Son responsables del despliegue y operación del sistema en producción. En este caso son especialmente relevantes por el requisito de soportar 3.000 solicitudes simultáneas y responder en menos de 2 segundos.

Proveedores (Suppliers): Son las organizaciones que proporcionan servicios o tecnologías externas utilizadas por el sistema. Por ejemplo, un proveedor de correo electrónico para enviar los recordatorios.

Adquirentes / Compradores (Acquirers): Corresponden a la organización que solicita o adquiere el sistema. En este caso, la Veterinaria Coco.

Evaluadores / Auditores (Assessors): Verifican que el sistema cumpla requisitos, normativas y estándares. En este caso, el área de cumplimiento normativo tiene especial importancia debido a la exigencia de cifrar los datos clínicos.

Comunicadores (Communicators): Son responsables de comunicar información relacionada con el sistema a los distintos stakeholders, como cambios, procedimientos o instrucciones de uso.

Personal de soporte (Support Staff): Ayuda a los usuarios cuando tienen problemas con el sistema, por ejemplo, dificultades para reservar una hora o consultar una agenda.

Usuarios (Users): Son las personas que utilizan directamente el sistema. En este caso corresponden principalmente a los pacientes, especialistas médicos y recepcionistas.

### (4) ADR

Título: ADR-001. Usar PostgreSQL para el módulo de reservas de horas médicas.

Estado: Aceptado.

Contexto: Se requiere evitar duplicidad de horarios bajo alta concurrencia, manteniendo la integridad transaccional. Al mismo tiempo, se necesita dar visibilidad casi inmediata de la agenda a la app móvil de los médicos.

Se evaluaron dos alternativas:

(a) Firebase Realtime Database, por su sincronización nativa en tiempo real.

(b) PostgreSQL, por su soporte robusto de transacciones ACID.

Decisión: Usaremos PostgreSQL como motor de base de datos para el módulo de reservas, y resolveremos la actualización de la app móvil mediante consultas periódicas (polling).

Consecuencias:

(+) Se garantiza la integridad transaccional y se evita la duplicación de horarios bajo concurrencia.

(-) La app móvil no tendrá actualización en tiempo real instantánea, sino que dependerá de la frecuencia con que se realice el polling.
