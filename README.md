
# 🤖 Chatbot de Telegram para Consultas de PDFs

##  Resumen del Proyecto

Este es un proyecto de Programación Lógica cuyo objetivo es implementar una solución de **Retrieval Augmented Generation (RAG)** completamente local. El sistema permite a los usuarios de Telegram subir un documento **PDF** y, posteriormente, realizar preguntas contextuales sobre el contenido del archivo.

El proyecto se despliega de manera **contenida y reproducible** utilizando Docker Compose, integrando un LLM local (**Ollama**) con una Base de Datos Vectorial (**Qdrant**). Para que Telegram pueda enviar los mensajes al bot que corre en la máquina local, se utiliza **ngrok** para crear un túnel HTTP público y seguro.

---

##  Stack Tecnológico Clave

| Componente | Tecnología | Rol Principal en el Workflow |
| :--- | :--- | :--- |
| **Modelo de Lenguaje (LLM)** | **Ollama** | Motor para ejecutar el modelo **[Nombre del Modelo, ej: Llama 3 8B]**. Genera respuestas coherentes basadas en el contexto recuperado. |
| **Base de Datos Vectorial** | **Qdrant** | Almacena y gestiona los *embeddings* (vectores numéricos) del contenido del PDF para una recuperación de información precisa (**Vector Search**). |
| **Túnel HTTP Público** | **ngrok** | Crea un túnel seguro desde el host local para exponer la URL privada del bot a Telegram, permitiendo la comunicación vía **Webhooks**. |
| **Orquestación** | **Docker Compose** | Define, configura y ejecuta los tres servicios (Bot, Ollama y Qdrant) en un solo comando, garantizando un entorno idéntico. |
| **Interfaz** | **Telegram Bot API** | Permite la interacción del usuario: subir el PDF y enviar las consultas. |

---

##  Guía de Instalación y Despliegue (100% Local)

Sigue estos pasos para levantar y ejecutar el chatbot en tu máquina local.

###  Requisitos Previos

Necesitas tener instalados y en ejecución los siguientes programas:

1.  **Git:** Para clonar el repositorio.
2.  **Docker & Docker Compose:** El motor de contenedores es esencial para ejecutar los servicios.
3.  **ngrok:** Descargado e instalado para crear el túnel HTTP.

###  Pasos de Configuración Rápida

1.  **Clonar el Repositorio:**
    ```bash
    git clone [https://www.youtube.com/watch?v=L_lWQZNhN7w](https://www.youtube.com/watch?v=L_lWQZNhN7w)
    cd [Nombre de la Carpeta de tu Proyecto]
    ```

2.  **Crear y Configurar el Archivo `.env`:**
    Crea un archivo llamado `.env` en el directorio raíz. Debes obtener un token de BotFather en Telegram.

    ```bash
    # Copiar este contenido en el archivo .env
    TELEGRAM_BOT_TOKEN=[TU_TOKEN_OBTENIDO_DE_BOTFATHER]
    
    # Nombre del modelo de Ollama que se utilizará
    OLLAMA_MODEL=llama3:8b
    
    # Puerto interno del contenedor donde escucha el bot (normalmente 8080 o 5000)
    BOT_PORT=[PUERTO_INTERNO_DEL_BOT] 
    
    # Directorio local para guardar los PDFs subidos
    PDF_STORAGE_PATH=./data/pdfs
    
    # Configuración de los hosts internos de Docker
    QDRANT_HOST=qdrant
    OLLAMA_HOST=ollama
    ```

3.  **Ejecución de los Contenedores:**
    Este comando levantará los servicios de **Qdrant**, **Ollama** y el **Bot de Telegram**.

    ```bash
    docker compose up -d
    ```

4.  **Crear el Túnel Público con ngrok:**
    El bot está escuchando en el puerto interno `BOT_PORT` del contenedor. Necesitas exponer ese puerto a Internet usando ngrok.

    ```bash
    # Reemplaza [PUERTO_DEL_CONTENEDOR] con el valor definido en BOT_PORT (ej: 8080)
    ngrok http [PUERTO_DEL_CONTENEDOR]
    ```
    ngrok te mostrará una URL pública (ej: `https://abcd-efgh-ijkl.ngrok-free.app`). **Copia esta URL.**

5.  **Configurar el Webhook de Telegram:**
    Debes indicarle a Telegram la URL pública de ngrok para que envíe los mensajes a tu bot. Ejecuta este comando en tu terminal, reemplazando las variables:

    ```bash
    NGROK_URL="[URL_PUBLICA_COPIADA_DE_NGROK]"
    TOKEN="[TU_TELEGRAM_BOT_TOKEN]"
    
    curl -F "url=$NGROK_URL" "[https://api.telegram.org/bot$TOKEN/setWebhook](https://api.telegram.org/bot$TOKEN/setWebhook)"
    ```
    *Verificación:* Si la operación fue exitosa, Telegram responderá con `{"ok":true, ...}`.

---

##  Instrucciones de Uso del Chatbot

Una vez que los contenedores estén activos y el **Webhook** configurado con ngrok, puedes interactuar con el bot en Telegram.

1.  **Iniciar la Interacción:** Abre Telegram y busca tu bot.
2.  **Fase de Ingesta (Carga del PDF):**
    * Envía un **archivo PDF** al bot.
    * El bot automáticamente: descarga, procesa el texto, genera los **embeddings** y los almacena en **Qdrant**.
    * Espera el mensaje de confirmación del bot (ej: *"Documento cargado con éxito. ¡Ya puedes preguntar!"*).
3.  **Fase de Consulta (RAG):**
    * Haz tu pregunta sobre el contenido del PDF.
    * El sistema utiliza **Qdrant** para encontrar los pasajes más relevantes del PDF y los pasa a **Ollama** para formular la respuesta final.

---

##  NOTA IMPORTANTE: Mantenimiento del Túnel (ngrok)

Dado que estás utilizando la versión gratuita de ngrok, la URL pública cambia cada vez que reinicias el servicio (`ngrok http ...`). **Debes repetir el Paso 4 y el Paso 5** de la configuración cada vez que reinicies el túnel de ngrok, ya que la URL pública es esencial para que Telegram pueda comunicarse con tu bot local.

---

##  Estructura del Repositorio

Ayuda a tu profesor a navegar por tus archivos clave.
