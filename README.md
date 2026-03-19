# Documentación Técnica: Ecosistema E2E de Automatización e IA

## Resumen del Proyecto
Solución funcional diseñada para procesar material académico extenso, transformándolo automáticamente en herramientas de evaluación interactivas (exámenes) y recursos multimedia (podcasts). La arquitectura se basa en un modelo orientado a eventos con 5 microservicios orquestados mediante n8n, cumpliendo con los estándares de ingesta, lógica de control, integraciones externas, IA avanzada y seguridad.

---

## 1. Servicio Activador
Este servicio actúa como el controlador de entrada, manejando la interacción inicial con el usuario y derivando la carga de trabajo según el rol y la intención.

<img src="Imagenes\Flujo1.jpeg" alt="Flujo 1">

**Paso a paso de ejecución:**
1. **Ingesta:** El flujo inicia mediante un Webhook que escucha eventos de WhatsApp o Telegram tras la recepción de un archivo adjunto.
2. **Estructura:** Se utilizan nodos de transformación para limpiar el JSON y aislar el ID del usuario (que se identifiquen por nro celular).
3. **Interacción Inicial:** El sistema pausa la ejecución para preguntarle al usuario de forma interactiva si desea generar un "Examen" o un "Podcast", esperando su respuesta en texto.
4. **Validación de Roles:** Se realiza una consulta a la base de datos externa para verificar si el usuario es "Profesor" o "Alumno".
5. **Enrutamiento:** Dependiendo de la intención capturada y el rol verificado, un nodo condicional deriva el payload al sub-workflow correspondiente.

---

## 2. Servicio Creador de Exámenes 
Microservicio encargado de la extracción de datos y la generación del contenido estructurado oficial.

<img src="Imagenes\Flujo2.jpeg" alt="Flujo 2">

**Paso a paso de ejecución:**
1. **Recepción:** Recibe el PDF y el ID del usuario desde el Orquestador.
2. **Procesamiento:** Descarga el binario del PDF y extrae el texto plano.
3. **Iteración:** Divide el texto en lotes mediante un bucle para procesar documentos largos sin saturar la ventana de contexto de la IA.
4. **Generación:** Un Agente de IA redacta las preguntas basándose en un System Prompt estructurado.
5. **Pausa de Seguridad:** El borrador se guarda temporalmente en la base de datos (Estado: PENDIENTE) y se envía un correo al profesor solicitando su revisión.

---

## 3. Servicio Editor y Publicador
Maneja la asincronía de la validación humana, procesando la respuesta del profesor al correo electrónico.

<img src="Imagenes\Flujo3.jpeg" alt="Flujo 3">

**Paso a paso de ejecución:**
1. **Trigger Asíncrono:** Un Webhook o nodo de Email escucha la respuesta del profesor.
2. **Bifurcación:** Si el profesor aprueba haciendo clic en un enlace directo, el examen cambia a estado "PUBLICADO" en la base de datos.
3. **Edición por IA:** Si el profesor responde el correo solicitando cambios en texto libre, un Agente de IA interpreta la corrección y modifica el JSON original de la base de datos.
4. **Notificación:** Se envía una alerta confirmando la publicación oficial a los canales correspondientes.


---

## 4. Servicio Tutor Interactivo y Evaluador (Alumno)
El motor interactivo del ecosistema que evalúa las respuestas en tiempo real y gestiona las calificaciones de las sesiones de práctica.

<img src="Imagenes\Flujo4.jpeg" alt="Flujo 4">

**Paso a paso de ejecución:**
1. **Contexto:** Utiliza el ID de sesión (sessionId) del usuario para mantener el hilo de la conversación y consultar en la base de datos qué pregunta se está respondiendo actualmente.
2. **Evaluación:** El Agente de IA compara semánticamente la respuesta del alumno con la respuesta modelo de la base de datos y asigna un puntaje.
3. **Iteración de Estado:** Avanza el índice de la pregunta y actualiza el estado de la sesión en la base de datos.
4. **Cierre y Reporte:** Al finalizar el quiz, recopila el rendimiento total, genera un archivo PDF dinámico con el boletín de calificaciones y lo despacha al correo del alumno.

---

## 5. Servicio Motor de Podcasts 
Microservicio independiente activado a demanda para transformar texto denso en material auditivo.

<img src="Imagenes\Flujo5.jpeg" alt="Flujo 5">

**Paso a paso de ejecución:**
1. **Recepción:** Recibe la orden explícita y el texto extraído desde el Orquestador.
2. **Adaptación:** La IA transforma el texto académico formal en un guion conversacional y ameno.
3. **Síntesis de Voz:** Realiza una petición HTTP a una API externa para generar el audio.
4. **Entrega:** Devuelve el archivo binario de audio (.mp3 o .ogg) directamente al canal de WhatsApp o Telegram del usuario.
