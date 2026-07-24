# Ecosistema de Automatización IA Autónomo - HiperMegaRed

Este proyecto consiste en un **ecosistema de automatización inteligente** diseñado para la compañía ficticia **HiperMegaRed**. Su objetivo principal es optimizar la atención de solicitudes de usuarios mediante la clasificación inteligente de mensajes, derivando de forma autónoma entre la adquisición de nuevos servicios (Rama Comercial) o la resolución de fallas técnicas (Rama de Soporte).

El sistema integra bases de datos dinámicas, modelos de lenguaje avanzados (LLM) con lógica de agentes, y un mecanismo interactivo de aprobación humana (*Human-in-the-loop*).

---

## Tecnologías Utilizadas

*   **Orquestador Principal:** n8n (flujo lógico autohospedado).
*   **Base de Datos / Memoria:** Airtable (tablas relacionales de clientes, ventas y soporte).
*   **Motor Cognitivo / IA:** OpenAI (GPT) configurado mediante un nodo **AI Agent** con herramientas de consulta dinámica.
*   **Canal de Comunicación:** Gmail (para notificaciones automáticas y flujos de aprobación interactivos).

---
##  Estructura del Repositorio

A continuación se detalla la organización de los archivos y carpetas contenidos en este repositorio:

```text
├── Screenshots/                   # Carpeta con las 72 evidencias de ejecución en formato .jpeg
├── Diagrama HiperMegaRed.pdf      # Diagrama completo de la arquitectura del ecosistema
├── HiperMegaRed.json              # Flujo exportado de n8n con todos los nodos y configuraciones
└── README.md                      # Documentación principal del proyecto
```
---

##  Arquitectura del Flujo

El sistema se dispara de manera automatizada ante cada nuevo formulario recibido (`On form submission`).

El flujo se compone de 4 capas fundamentales:

1. **Triggers / Captura:** Recepción de solicitudes a través de un formulario único (Trámite Comercial o Soporte Técnico).

2. **Capa de Persistencia (Base de Datos):** Sincronización y registro en tiempo real con Airtable.

3. **Cerebro Cognitivo (IA Agent):** Procesamiento de lenguaje natural mediante un Agente de LangChain (OpenAI/OpenRouter) capaz de consultar herramientas en tiempo real (Function Calling / RAG).

4. **Capa de Notificación y Aprobación (HITL & Mailer):** Envíos multicanal vía Gmail con confirmación interactiva (Send & Wait).

### Desencadenador y Enrutamiento Inicial
*   **Trigger:** El flujo se inicia con el nodo **Formulario Inicio de Trámite**.
*   **Bifurcación Principal:** Un nodo condicional (**IF Soporte / Contratación**) evalúa la naturaleza de la solicitud:
    *   **Ruta de Contratación (Comercial):** Deriva a la creación de registros de preventa.
    *   **Ruta de Soporte (Clientes Activos):** Deriva al protocolo de verificación técnica.
---

### Rama Comercial: Procesamiento con Inteligencia Artificial
*   **Ingesta de Datos:** Se ejecutan de manera consecutiva los nodos **Creación de Cliente** y **Ticket Venta** en la base de datos.
*   **Motor Cognitivo (AI Agent):** El núcleo de procesamiento no utiliza plantillas estáticas sino que utiliza un **AI Agent** conectado a:
    *   **Model:** `OpenAI Chat Model`.
    *   **Tools Integradas:** `Consulta Planes Internet` y `Consulta Planes TV`.

El agente analiza la consulta inicial del cliente, determina qué herramienta necesita ejecutar para consultar la disponibilidad real de planes y genera una propuesta comercial personalizada sin invención de datos (alucinaciones).
*   **Bifurcación de Resiliencia (Manejo de Errores):**
    *   **Camino Feliz (Éxito de la IA):** Envía un **Correo al Cliente** con la propuesta e ingresa al **IF Derivación**.
        *   *Sub-ruta de Servicio Técnico:* **Actualizar Ticket Ventas** → **Consulta Servicio Técnico** → **Asignación Técnico** → **Mensaje a ST** → **Espera 5  Días(u otro período indicado)** → **Confirmación ST**.
        *   *Sub-ruta de Asesor Comercial:* **Consulta Asesor** → **Asignación Asesor** → **Mensaje Asesor C** → **Espera 5 Días** → **Confirmación Asesor**.
    *   **Camino Infeliz (Fallo de la IA):** El flujo desvía la carga hacia el nodo **Actualizacion Ticket: Error** → **Consulta ST Alerta** → **Asignacion Tecnico** → emitiendo un **Mensaje ST Alerta** para intervención humana inmediata.

---

### Rama de Soporte Técnico: Validación y Asignación Dinámica
*   **Filtro de Seguridad:** El sistema realiza una **Busqueda Cliente** en la base de datos. 
*   **IF Existe Cliente?:**
    *   **Falso:** El flujo se detiene de forma segura notificando al remitente mediante el estado **Cliente No Encontrado**.
    *   **Verdadero:** Inicia el circuito operativo automatizado:
        1.  **Creación Ticket Soporte** en la base de datos relacional.
        2.  **Consulta ST** para validar la disponibilidad del personal de guardia.
        3.  **Asignación Técnico** mediante distribución balanceada de carga de trabajo.
        4.  Despacho de la orden mediante el nodo **Mensaje ST S**.

---

### Ciclos de Espera y Cierre de Tickets (HITL)
Todas las rutas convergentes finalizan en un esquema de **Human-in-the-loop** diseñado para resguardar la calidad del servicio técnico:
*   **Nodos de Espera:** El sistema mitiga el "Efecto Metralleta" deteniendo la ejecución mediante nodos de **Espera de 5 Días(u otro período indicado)**.
*   **Nodos de Confirmación:** Se dispara un **Correo Confirmación Cliente** o **Confirmación ST** interactivo.
*   **Cierre de Estados (IF Finalizacion / Incidencia):** Dependiendo del feedback del usuario o del asesor, la lógica actualiza el registro en Airtable hacia los estados conclusivos de **Finalización** (Éxito) o **Incidencia** (Reabierto/Auditoría).

## Resiliencia y Gestión de Errores

El flujo está diseñado bajo estrictos estándares de estabilidad y tolerancia a fallos:
*   **Contingencia en IA:** El nodo `AI Agent` cuenta con una **salida de error física (Separate Error Route)**. Si la API de OpenAI experimenta latencia o caídas, el flujo desvía la operación de forma segura para notificar al equipo de soporte interno y actualizar la base de datos de Airtable con el registro técnico del fallo (`{{ $error.message }}`).
*   **Tolerancia en Integraciones:** Los nodos clave de comunicación y consultas a bases de datos están configurados para evitar bloqueos del flujo mediante directivas de control de errores nativas.

---

## Enlaces y Recursos

*   **Base de Datos en Airtable (Modo Lectura):** https://airtable.com/appkFBDLqSpUrBHY0/shrsFras6ko2GeR8R
*   **Diagrama de Arquitectura:** [Descargar / Ver Diagrama PDF](./Diagrama%20HiperMegaRed.pdf)
*   **Video Demostración:** https://drive.google.com/file/d/1IwcSPUUIKc72ZYV6wQ2jnTppGTDi6rPk

---

## Evidencias de Funcionamiento
<details>
<summary> <b>Haga clic aquí para desplegar las 72 capturas de evidencia del workflow</b></summary>

### Arquitectura completa

![Arquitectura completa](./Screenshots/SS00001.jpeg)
![Arquitectura completa](./Screenshots/SS00002.jpeg)

### Formulario de inicio de trámite

![Formulario de inicio de trámite](./Screenshots/SS00003.jpeg)
![Formulario de inicio de trámite](./Screenshots/SS00004.jpeg)

### Rama Comercial

Formulario de prueba para contratación

![Formulario de prueba para contratación](./Screenshots/SS00005.jpeg)

Comportamiento del workflow

![Comportamiento del workflow](./Screenshots/SS00006.jpeg)

Nodo IF Comercial/Soporte

![Nodo IF Comercial/Soporte](./Screenshots/SS00007.jpeg)

Nodo creación de cliente (Airtable)

![Nodo creación de cliente](./Screenshots/SS00008.jpeg)

Nodo creación de Ticket de Venta (Airtable)

![Nodo creación de Ticket de Venta](./Screenshots/SS00009.jpeg)

#### AI Agent

Nodo AI Agent

![Nodo AI Agent](./Screenshots/SS00010.jpeg)

Workflow luego de iniciar en nodo send and wait de correo al cliente (Gmail)

![Workflow luego de iniciar en nodo send and wait de correo al cliente](./Screenshots/SS00011.jpeg)

Correo recibido por el cliente

![Correo recibido por el cliente](./Screenshots/SS00012.jpeg)
![Correo recibido por el cliente](./Screenshots/SS00013.jpeg)

Respuesta de cliente eligiendo uno de los productos propuestos por el Agente de IA

![Respuesta de cliente eligiendo uno de los productos propuestos por el Agente de IA](./Screenshots/SS00014.jpeg)

Comportamiento del workflow según la respuesta del cliente

![Comportamiento del workflow según la respuesta del cliente](./Screenshots/SS00015.jpeg)

Nodo de correo al cliente con su respuesta (Gmail)

![Correo recibido por el cliente](./Screenshots/SS00016.jpeg)

#### Derivación a Técnico e Instalación

Nodo IF de Derivación a actualización de ticket de ventas

![Nodo IF de Derivación a actualización de ticket de ventas](./Screenshots/SS00017.jpeg)

Nodo de actualización de Ticket de Ventas (Airtable)

![Nodo de actualización de Ticket de Ventas](./Screenshots/SS00018.jpeg)

Nodo de Consulta al Servicio Técnico (Airtable)

![Nodo de Consulta al Servicio Técnico](./Screenshots/SS00019.jpeg)

Nodo de Código para Asignación de Técnico

![Nodo de Código para Asignación de Técnico](./Screenshots/SS00020.jpeg)

Nodo de Mensaje vía correo al técnico asignado (Gmail)

![Nodo de Mensaje vía correo al técnico asignado](./Screenshots/SS00021.jpeg)

Correo recibido por el técnico asignado

![Correo recibido por el técnico asignado](./Screenshots/SS00022.jpeg)

Nodo Wait de espera para que transcurrido un tiempo se vuelva a enviar un mensaje al técnico para saber si se realizó la instalación 

![Nodo Wait de espera para que transcurrido un tiempo se vuelva a enviar un mensaje al técnico para saber si se realizó la instalación](./Screenshots/SS00023.jpeg)

Nodo Send and Wait para el técnico asignado (Gmail)

![Nodo Send and Wait para el técnico asignado](./Screenshots/SS00024.jpeg)

Correo enviado al técnico asignado

![Correo enviado al técnico asignado](./Screenshots/SS00025.jpeg)

Nodo IF para el caso de que la respuesta de la instalación haya sido Aprobada

![Nodo IF para el caso de que la respuesta de la instalación haya sido Aprobada](./Screenshots/SS00026.jpeg)

Nodo de finalización del trámite (Airtable)

![Nodo de finalización del trámite](./Screenshots/SS00027.jpeg)

Nodo IF para el caso de que la respuesta de la instalación no haya sido Aprobada

![Nodo IF para el caso de que la respuesta de la instalación haya sido Aprobada](./Screenshots/SS00028.jpeg)

Nodo de Incidencia del trámite (Airtable)

![Nodo de finalización del trámite](./Screenshots/SS00029.jpeg)

#### Derivación a Asesor Comercial

Nodo IF de Derivación para el caso que el cliente prefiera ser atendido por un ASESOR

![Nodo IF de Derivación para el caso que el cliente prefiera ser atendido por un asesor](./Screenshots/SS00030.jpeg)

Nodo de Consulta Asesor (Airtable)

![Nodo de finalización del trámite](./Screenshots/SS00031.jpeg)

Nodo de Código de Asignación de Asesor

![Nodo de Código de Asignación de Asesor](./Screenshots/SS00032.jpeg)

Nodo de Mensaje a Asesor Comercial (Gmail)

![Nodo de Mensaje a Asesor Comercial](./Screenshots/SS00033.jpeg)

Correo recibido por el Asesor Comercial

![Nodo de Mensaje a Asesor Comercial](./Screenshots/SS00034.jpeg)

Nodo Wait de espera para que transcurrido un tiempo se vuelva a enviar un mensaje al asesor para saber si se realizó la instalación 

![Nodo Wait de espera para que transcurrido un tiempo se vuelva a enviar un mensaje al asesor para saber si se realizó la instalación](./Screenshots/SS00035.jpeg)

Nodo Send and Wait para el Asesor Comercial asignado (Gmail)

![Nodo Send and Wait para el Asesor Comercial asignado](./Screenshots/SS00036.jpeg) 

Correo recibido por el Asesor Comercial

![Correo recibido por el Asesor Comercial](./Screenshots/SS00037.jpeg) 

Nodo IF para el caso de que la respuesta de la instalación haya sido Aprobada

![Nodo IF para el caso de que la respuesta de la instalación haya sido Aprobada](./Screenshots/SS00038.jpeg)

Nodo de finalización del trámite (Airtable)

![Nodo de finalización del trámite](./Screenshots/SS00039.jpeg)

Nodo IF para el caso de que la respuesta de la instalación no haya sido Aprobada

![Nodo IF para el caso de que la respuesta de la instalación haya sido Aprobada](./Screenshots/SS00041.jpeg)

Nodo de Incidencia del trámite (Airtable)

![Nodo de finalización del trámite](./Screenshots/SS00042.jpeg)

#### Manejo de fallos en IA Agent 

Comportamiento del Workflow en caso de error del Agente de IA

![Comportamiento del Workflow en caso de error del Agente de IA](./Screenshots/SS00043.jpeg)

Nodo de Actualización del estado del Ticket a Error (Airtable)

![Actualización del estado del Ticket a Error](./Screenshots/SS00044.jpeg)

Nodo de Consulta al Servicio Técnico para Alerta (Airtable)

![Nodo de Consulta al Servicio Técnico para Alerta](./Screenshots/SS00045.jpeg)

Nodo de Código para asignación de un técnico para ver el Alerta

![Nodo de Código para asignación de un técnico para ver el Alerta](./Screenshots/SS00046.jpeg)

Nodo de Mensaje de Alerta al técnico asignado (Gmail)

![Nodo de Mensaje de Alerta al técnico asignado](./Screenshots/SS00047.jpeg)

Correo enviado al técnico asignado

![Correo enviado al técnico asignado](./Screenshots/SS00048.jpeg)

### Rama Soporte Técnico

#### Clientes Activos

Formulario de prueba para servicio técnico

![Formulario de prueba para servicio técnico](./Screenshots/SS00049.jpeg)

Comportamiento del Workflow luego de la solicitud de soporte

![Comportamiento del Workflow luego de la solicitud de soporte](./Screenshots/SS00050.jpeg)

Nodo IF Comercial/Soporte

![Nodo IF Comercial/Soporte](./Screenshots/SS00051.jpeg)

Nodo de Búsqueda de Cliente (Airtable)

![Nodo de Búsqueda de Cliente](./Screenshots/SS00052.jpeg)

Nodo IF de confirmación de la existencia de cliente

![Nodo IF de confirmación de la existencia de cliente](./Screenshots/SS00053.jpeg)

Nodo de Creación de Ticket de Soporte (Airtable)

![Nodo de Creación de Ticket de Soporte](./Screenshots/SS00054.jpeg)

Nodo de Consulta Servicio Técnico (Airtable)

![Nodo de Consulta Servicio Técnico](./Screenshots/SS00055.jpeg)

Nodo de Código para asignación de un técnico

![Nodo de Código para asignación de un técnico](./Screenshots/SS00056.jpeg)

Nodo de Mensaje al Técnico Asignado (Gmail)

![Nodo de Mensaje al Técnico Asignado](./Screenshots/SS00057.jpeg)

Correo enviado al técnico asignado

![Correo enviado al técnico asignado](./Screenshots/SS00058.jpeg)

Nodo Wait de espera para que transcurrido un tiempo se vuelva a enviar un mensaje al cliente para saber si se realizó el servicio 

![Nodo Wait de espera para que transcurrido un tiempo se vuelva a enviar un mensaje al técnico para saber si se realizó el servicio](./Screenshots/SS00059.jpeg)

Nodo Send and Wait para el Cliente que solicitó el servicio técnico (Gmail)

![Nodo Send and Wait para el Cliente que solicitó el servicio técnico](./Screenshots/SS00060.jpeg) 

Correo recibido por el cliente

![Correo recibido por el cliente](./Screenshots/SS00061.jpeg) 

Nodo IF para confirmación de la resolución del su problema con ayuda del soporte técnico

![Nodo IF para confirmación de la resolución del su problema con ayuda del soporte técnico](./Screenshots/SS00062.jpeg)

Nodo de finalización del trámite (Airtable)

![Nodo de finalización del trámite](./Screenshots/SS00063.jpeg)

Nodo IF para confirmación de la NO resolución del su problema con ayuda del soporte técnico

![Nodo IF para confirmación de la NO resolución del su problema con ayuda del soporte técnico](./Screenshots/SS00064.jpeg)

Nodo de Incidencia del trámite (Airtable)

![Nodo de finalización del trámite](./Screenshots/SS00065.jpeg)

#### No Clientes

Caso de solicitud de servicio técnico por una persona que no es cliente

![Caso de solicitud de servicio técnico por una persona que no es cliente](./Screenshots/SS00066.jpeg)

Comportamiento del Workflow en caso de solicitud de servicio técnico por una persona que no es cliente

![Comportamiento del Workflow en caso de solicitud de servicio técnico por una persona que no es cliente](./Screenshots/SS00067.jpeg)

Nodo IF Comercial/Soporte

![Nodo IF Comercial/Soporte](./Screenshots/SS00068.jpeg)

Nodo de Búsqueda de Cliente (Airtable)

![Nodo de Búsqueda de Cliente](./Screenshots/SS00069.jpeg)

Nodo IF de confirmación de la inexistencia de cliente

![Nodo IF de confirmación de la existencia de cliente](./Screenshots/SS00070.jpeg)

Nodo de correo para informar al solicitante del servicio técnico que no es cliente

![Nodo de correo para informar al solicitante del servicio técnico que no es cliente](./Screenshots/SS00071.jpeg)

Correo enviado al solicitante informándole que no es cliente

![Correo enviado al solicitante informándole que no es cliente](./Screenshots/SS00072.jpeg)

</details>
---
