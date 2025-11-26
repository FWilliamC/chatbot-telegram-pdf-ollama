## Chatbot Telegram con n8n + Ollama + Qdrant + Docker + Ngrok


# Autor:
Franco William Cuastumal 
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

bash:

***winget install Ollama.Ollama***

Verificar:

bash:

***ollama --version***


![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-21%20175544.png)


Descargar modelos necesarios:

bash:

***ollama pull llama3***

***ollama pull nomic-embed-text***


![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-21%20175926.png)

# 1.3. Instalar Qdrant con Docker

bash:

***docker run -p 6333:6333 -p 6334:6334 qdrant/qdrant***


![Verificación de Docker](https://github.com/FWilliamC/chatbot-telegram-pdf-ollama/blob/main/Captura%20de%20pantalla%202025-11-21%20181205.png)

# 1.4. Instalar ngrok

Para exponer n8n hacia Telegram:

bash:

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

#4.1 Nodo Telegram Trigger – Documento

Activa el flujo cada vez que un usuario envía un mensaje o PDF.

- Extrae mensaje o archivo.

- Pasa al Switch.

  ![Verificación de Docker]

# 4.2 Nodo Switch

Detecta si el usuario envió:

- Texto

- Archivo PDF

Rama 1 → Texto
Rama 2 → Archivo

![Verificación de Docker]

# 4.3 Nodo Get a file

Descarga el archivo PDF desde Telegram.

![Verificación de Docker]

# 4.4 Nodo Extract From File – PDF

![Verificación de Docker]

# 4.5 Token Splitter

Divide el texto del PDF en chunks.

Configuración recomendada:

- Chunk size: 300 tokens

- Overlap: 50 tokens

  ![Verificación de Docker]

# 4.6 Nodo Embeddings – Ollama

  Genera los embeddings con:

  ***nomic-embed-text***

  ![Verificación de Docker]

# 4.7 Nodo Qdrant Vector Store

 Guarda los embeddings del PDF.

 Parámetros:

- URL: http://localhost:6333

- Collection: pdf_documents

- Upsert: ON

 ![Verificación de Docker]

# 5. Inteligencia Artificial – Agente

Este es el cerebro del chatbot.

Funciones:

- Busca información en Qdrant.

- Usa el modelo de chat de Ollama.

- Mantiene memoria conversacional.

- Responde como asistente.

![Verificación de Docker]

# 5.2 Modelo de Chat – Ollama

Modelo:

***llama3.2***

![Verificación de Docker]

# 5.3 Simple Memory

Permite conversaciones largas.

![Verificación de Docker]

# 5.4 Qdrant Vector Store (Consulta)

Consulta los embeddings del PDF cargado.

![Verificación de Docker]

# 6. Nodo final – Enviar mensaje a Telegram

El agente genera la respuesta → se envía a Telegram.

![Verificación de Docker]

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


![Verificación de Docker]

# 8. Conclusiones

# 9. Repositorio Final

