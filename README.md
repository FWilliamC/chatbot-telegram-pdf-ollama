## Chatbot Telegram con n8n + Ollama + Qdrant + Docker + Ngrok


# Autor:
Franco William Cuastumal 
-----------------------


# Descripción General

Este documento técnico detalla la instalación, configuración y funcionamiento de un chatbot en Telegram que opera completamente en un entorno local. El proceso comienza con la recepción de archivos PDF a través de Telegram, seguido de la extracción de texto del documento. El texto extraído se utiliza para generar embeddings empleando Ollama, que se ejecuta dentro de un contenedor Docker. Estos embeddings se almacenan localmente en Qdrant, también en un contenedor Docker, que actúa como el almacén vectorial (vector store). Para manejar la lógica de consulta, se emplea un Agente de IA configurado en n8n, el cual consulta al PDF a través de Qdrant. Finalmente, la respuesta generada por el Agente de IA se envía de vuelta al usuario en Telegram. La exposición del webhook de Telegram al entorno local se gestiona mediante Ngrok, asegurando la comunicación bidireccional necesaria para el funcionamiento del chatbot.

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

  
![Verificación de Docker]


