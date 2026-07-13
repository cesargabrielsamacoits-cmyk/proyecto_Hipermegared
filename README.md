# Ecosistema de Automatización IA Autónomo - HiperMegaRed

Este proyecto consiste en un **ecosistema de automatización inteligente** diseñado para la compañía ficticia **HiperMegaRed**. Su objetivo principal es optimizar la atención de solicitudes de usuarios mediante la clasificación inteligente de mensajes, derivando de forma autónoma entre la adquisición de nuevos servicios (Rama Comercial) o la resolución de fallas técnicas (Rama de Soporte).

El sistema integra bases de datos dinámicas, modelos de lenguaje avanzados (LLM) con lógica de agentes, y un mecanismo interactivo de aprobación humana (*Human-in-the-loop*).

---

##  Tecnologías Utilizadas

*   **Orquestador Principal:** n8n (flujo lógico autohospedado).
*   **Base de Datos / Memoria:** Airtable (tablas relacionales de clientes, ventas y soporte).
*   **Motor Cognitivo / IA:** OpenAI (GPT) configurado mediante un nodo **AI Agent** con herramientas de consulta dinámica.
*   **Canal de Comunicación:** Gmail (para notificaciones automáticas y flujos de aprobación interactivos).

---

##  Arquitectura del Flujo

El sistema se dispara de manera automatizada ante cada nuevo formulario recibido (`On form submission`) y divide la lógica en dos grandes vertientes:

### Desencadenador y Enrutamiento Inicial
*   **Trigger:** El flujo se inicia con el nodo **Formulario Inicio de Trámite**.
*   **Bifurcación Principal:** Un nodo condicional (**IF Soporte / Contratación**) evalúa la naturaleza de la solicitud:
    *   **Ruta de Contratación (Comercial):** Deriva a la creación de registros de preventa.
    *   **Ruta de Soporte (Clientes Activos):** Deriva al protocolo de verificación técnica.
---

### Rama Comercial: Procesamiento con Inteligencia Artificial
*   **Ingesta de Datos:** Se ejecutan de manera consecutiva los nodos **Creación de Cliente** y **Ticket Venta** en la base de datos.
*   **Motor Cognitivo (AI Agent):** El núcleo de procesamiento utiliza un **AI Agent** conectado a:
    *   **Model:** `OpenAI Chat Model`.
    *   **Tools Integradas:** `Consulta Planes Internet` y `Consulta Planes TV`.
*   **Bifurcación de Resiliencia (Manejo de Errores):**
    *   **Camino Feliz (Éxito de la IA):** Envía un **Correo al Cliente** con la propuesta e ingresa al **IF Derivación**.
        *   *Sub-ruta de Servicio Técnico:* **Actualizar Ticket Ventas** $\rightarrow$ **Consulta Servicio Técnico** $\rightarrow$ **Asignación Técnico** $\rightarrow$ **Mensaje a ST** $\rightarrow$ **Espera 5 Días** $\rightarrow$ **Confirmación ST**.
        *   *Sub-ruta de Asesor Comercial:* **Consulta Asesor** $\rightarrow$ **Asignación Asesor** $\rightarrow$ **Mensaje Asesor C** $\rightarrow$ **Espera 5 Días** $\rightarrow$ **Confirmación Asesor**.
    *   **Camino Infeliz (Fallo de la IA):** El flujo desvía la carga hacia el nodo **Actualizacion Ticket: Error** $\rightarrow$ **Consulta ST Alerta** $\rightarrow$ **Asignacion Tecnico** $\rightarrow$ emitiendo un **Mensaje ST Alerta** para intervención humana inmediata.

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
*   **Nodos de Espera:** El sistema mitiga el "Efecto Metralleta" deteniendo la ejecución mediante nodos de **Espera de 5 Días**.
*   **Nodos de Confirmación:** Se dispara un **Correo Confirmación Cliente** o **Confirmación ST** interactivo.
*   **Cierre de Estados (IF Finalizacion / Incidencia):** Dependiendo del feedback del usuario o del asesor, la lógica actualiza el registro en Airtable hacia los estados conclusivos de **Finalización** (Éxito) o **Incidencia** (Reabierto/Auditoría).

## Resiliencia y Gestión de Errores

El flujo está diseñado bajo estrictos estándares de estabilidad y tolerancia a fallos:
*   **Contingencia en IA:** El nodo `AI Agent` cuenta con una **salida de error física (Separate Error Route)**. Si la API de OpenAI experimenta latencia o caídas, el flujo desvía la operación de forma segura para notificar al equipo de soporte interno y actualizar la base de datos de Airtable con el registro técnico del fallo (`{{ $error.message }}`).
*   **Tolerancia en Integraciones:** Los nodos clave de comunicación y consultas a bases de datos están configurados para evitar bloqueos del flujo mediante directivas de control de errores nativas.

---

## Estructura del Repositorio

*   `/archivos-tecnicos`: Contiene el archivo `.json` de n8n para importar el flujo de forma directa.
*   `/diagramas`: Diagrama de arquitectura del ecosistema en formato PDF.
*   `/evidencias`: Capturas de pantalla que demuestran el funcionamiento óptimo del flujo en vivo.

---

## Enlaces y Recursos

*   **Base de Datos en Airtable (Modo Lectura):** https://airtable.com/appkFBDLqSpUrBHY0/shrsFras6ko2GeR8R
*   **Video Demo del Funcionamiento (3 min):** [Pegá aquí tu enlace de YouTube / Loom]

---
