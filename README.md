## Chatbot Telegram con n8n + Ollama + Qdrant + Docker + Ngrok


# Autor:
**Franco William Cuastumal** 
**Jhon Felipe Loaiza Restrepo**
-----------------------


# Descripción General

Este documento técnico detalla la instalación, configuración y funcionamiento de un chatbot en Telegram que opera completamente en un entorno local. El proceso comienza con la recepción de archivos  PDF  a través de Telegram,  seguido de la extracción de texto del documento. El texto extraído se utiliza  para generar embeddings empleando Ollama, que se ejecuta dentro de un contenedor Docker. Estos embeddings se almacenan localmente en Qdrant, también en un contenedor Docker, que actúa como el almacén vectorial (vector store). Para manejar la lógica de consulta, se emplea un Agente de IA configurado en n8n, el cual consulta al PDF a través de Qdrant. Finalmente, la respuesta generada por el Agente de IA se envía de vuelta al usuario en Telegram.  La exposición del webhook de Telegram al entorno local se gestiona mediante Ngrok, asegurando la comunicación bidireccional necesaria para el funcionamiento del chatbot.

# 1. Instalación de herramientas necesarias

# 1.1. Instalar Docker Desktop

Se utiliza para ejecutar tanto Ollama como Qdrant.

Pasos:

1. Descargar Docker Desktop desde https://www.docker.com/

2. Instalarlo y verificar el funcionamiento con: ***docker --version***

![Verificación de Docker](https://raw.githubusercontent.com/FWilliamC/chatbot-telegram-pdf-ollama/main/Captura%20de%20pantalla%202025-11-21%20172633.png)

# 1.2. Instalar Ollama (Local LLM)

Ollama permite ejecutar modelos como llama3, mistral, nomic-embed-text, etc.

Instalación:

En Windows:

**cmd:**

***winget install Ollama.Ollama***

Verificar:

**cmd:**

***ollama --version***


![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-21%20175544.png)


Descargar modelos necesarios:

**bash:**

***ollama pull llama3***

***ollama pull nomic-embed-text***


![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-21%20175926.png)

# 1.3. Instalar Qdrant con Docker

**cmd:**

***docker run -p 6333:6333 -p 6334:6334 qdrant/qdrant***


![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-21%20181205.png)

# 1.4. Instalar ngrok

Para exponer n8n hacia Telegram:

**cmd:**

***ngrok http 5678***

![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-21%20181553.png)

# 1.5. Instalar n8n

Puedes usar Docker:

***docker run -it --rm \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n***

![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-21%20181938.png)

## 2. Configuración del Bot en Telegram

# 2.1 Crear el bot con BotFather

Comandos:

- /newbot

- Elegir nombre y usuario del bot.

- Telegram entrega un TOKEN así:

  ***7887885099:AAHN-ag8JsC6a9Ly-WKtY8ecHxtW_Jf3g0I***

  
![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-25%20191349.png)

# 2.2 Registrar el Webhook

Usando la URL de Ngrok:

***https://TU-NGROK-URL/webhook***

# 3. Construcción del Workflow en n8n

A continuación se describe todo el workflow tal como aparece en la imagen.

![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-19%20175437.png)

# 4. Explicación detallada del workflow

# 4.1 Nodo Telegram Trigger – Documento

Activa el flujo cada vez que un usuario envía un mensaje o PDF.

- Extrae mensaje o archivo.

- Pasa al Switch.

  ![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-25%20194206.png)

# 4.2 Nodo Switch

Detecta si el usuario envió:

- Texto

- Archivo PDF

Rama 1 → Texto

Rama 2 → Archivo

![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-25%20194254.png)

# 4.3 Nodo Get a file

Descarga el archivo PDF desde Telegram.

![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-25%20194317.png)

# 4.4 Nodo Extract From File – PDF

Extrae el texto del PDF.

![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-25%20194331.png)

# 4.5 Default Data Loader

El nodo Default Data Loader se encarga de recibir los chunks de texto procesados por el Token Splitter y devolverlos en un formato estándar compatible con los Vector Stores de n8n, incluido Qdrant.

![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-25%20194406.png)

# 4.6 Token Splitter

Divide el texto del PDF en chunks.

Configuración recomendada:

- Chunk size: 512 tokens

- Overlap: 50 tokens

  ![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-25%20194414.png)

# 4.7 Nodo Embeddings – Ollama

  Genera los embeddings con:

  ***nomic-embed-text***

  ![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-25%20194356.png)

# 4.8 Nodo Qdrant Vector Store

 Guarda los embeddings del PDF.

 Parámetros:

- URL: http://localhost:6333

- Collection: mi_bot_ollama

- Upsert: ON

 ![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-25%20194345.png)

# 5. Inteligencia Artificial – Agente

Este es el cerebro del chatbot.

Funciones:

- Busca información en Qdrant.

- Usa el modelo de chat de Ollama.

- Mantiene memoria conversacional.

- Responde como asistente.

![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-25%20201109.png)

# 5.2 Modelo de Chat – Ollama

Modelo:

***llama3.2***

![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-25%20201121.png)

# 5.3 Simple Memory

Permite conversaciones largas.

![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-25%20201130.png)

# 5.4 Qdrant Vector Store (Consulta)

Consulta los embeddings del PDF cargado.

![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-25%20201143.png)

# 6. Nodo final – Enviar mensaje a Telegram

El agente genera la respuesta → se envía a Telegram.

![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-25%20201206.png)

# 7. Funcionamiento final

1.Usuario envía un PDF.

2.El workflow:

 - Lo descarga

 - Lo lee

 - Lo divide

 - Genera embeddings

 - Los guarda en Qdrant

3.Usuario pregunta algo del PDF.

4.El AI Agent:

 - Busca en Qdrant

 - Recupera el contenido

 - Genera respuesta con Ollama

5.El bot envía la respuesta por Telegram.


![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-25%20204804.png)
![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-25%20204813.png)

# 8. Conclusiones

**1. Es totalmente posible montar un sistema RAG completo de manera local**

El proyecto demostró que se puede armar un chatbot funcional sin depender de servicios externos:
Ollama, Qdrant, Docker, n8n y Ngrok permiten crear un flujo completo para procesar PDFs y responder preguntas sobre ellos. Esto lo hace ideal para personas que quieren trabajar offline o sin pagar APIs.

**2. El flujo quedó claro, modular y fácil de mejorar**

Organizar el proceso en nodos (extraer PDF, dividir texto, generar embeddings, guardarlos, responder) hizo que todo quedara entendible y fácil de mantener. Si más adelante se quiere agregar otra función, solo es añadir un nodo más al workflow.

**3. Qdrant y los embeddings permitieron manejar varios documentos de forma inteligente**

Gracias al sistema de embeddings con Ollama y al almacenamiento vectorial con Qdrant, el bot puede buscar información relevante dentro de uno o varios PDFs. Esto hace que las respuestas tengan sentido, incluso en documentos largos.

**4. La velocidad depende del hardware y del uso de modelos locales**

El rendimiento del bot cambia según el computador que tenga cada persona.
Modelos locales como Ollama funcionan muy bien, pero sí pueden tardar un poco en generar respuestas, especialmente si el equipo no tiene buena CPU o RAM.
Aun así, tener un sistema 100% local compensa esa demora porque ofrece privacidad total y cero costos.

# 9. Repositorio Final

https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/docker-compose.yml

https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/ChatBot.json

