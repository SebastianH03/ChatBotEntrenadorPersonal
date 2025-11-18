# Asistente de Rutinas de Ejercicio (Audio y Texto)
# Descripción general
Este proyecto implementa un asistente inteligente de Telegram especializado en rutinas de ejercicio, capaz de interactuar con los usuarios mediante audio o texto, y responder generando textos, imágenes o mensajes de voz naturales, según la intención del usuario.
El sistema combina la potencia de N8N como motor de orquestación junto con modelos de Gemini y OpenAI para análisis, generación de contenido y síntesis de voz.

# 1. Requisitos previos:
## Docker
Docker nos permitió ejecutar n8n dentro de un contenedor aislado y estable, asegurando que funcione igual en cualquier equipo. Además, simplifica la instalación y el levantamiento del servicio con un solo comando.

**Enlace de descarga:** ***https://www.docker.com/products/docker-desktop/***

## Ngrok
Ngrok nos ayudó a exponer el servicio local a internet mediante un túnel seguro, necesario para que n8n pueda recibir llamadas externas vía API.

**Enlace de descarga:**
***https://ngrok.com/download***

Configurar el token una vez instalado:

`ngrok config add-authtoken <TU_AUTHTOKEN>`

# 2. Instalación y ejecución:

## Clonar el repositorio
`git clone https://github.com/SebastianH03/ChatBotEntrenadorPersonal.git`

## Ejecutar Docker Compose

`docker compose up -d`

Se reutilizó el `docker-compose.yml` compartido por el profesor, lo que permite:

* Descargar las imágenes necesarias (n8n, postgres)
* Construir los contenedores
* Iniciar el servicio en segundo plano

## Acceso a n8n en local

Una vez levantado el servicio, acceder a n8n desde tu navegador usando la siguiente URL:

`http://localhost:5678`

## Habilitar acceso público con ngrok

De forma predeterminada, n8n ejecutándose en Docker no puede recibir webhooks ni llamadas externas, ya que está limitado a la red local.

Para resolverlo, se ejecuta:

`ngrok http 5678`

Con ello, ngrok genera una URL pública HTTPS que apunta a la instancia local de n8n. En nuestro caso, el subdominio asignado es:

**https://balletic-marget-overdrily.ngrok-free.app -> http://localhost:5678**

Esta URL permite integrar servicios como Telegram y otras APIs externas.

# 3. Configuración de credenciales:

Para que el flujo funcione correctamente, se deben configurar credenciales para:

* Telegram
* Gemini
* OpenAI

## Telegram:
1. Abrir Telegram y buscar: BotFather

2. Crea un nuevo bot con /newbot

3. Seguir las instrucciones de BotFather para asignar: un nombre público para el bot y un username único (debe terminar en bot).

4. Una vez completado el proceso, BotFather proporcionará: un enlace para chatear con tu nuevo bot y una API Key (token de acceso).

5. Copia esa API Key y en un nodo Telegram Trigger en n8n, crea una nueva credencial de Telegram e introduce:
* la API Key generada por BotFather,
* cualquier configuración adicional requerida por el nodo.

<img width="1280" height="943" alt="image" src="https://github.com/user-attachments/assets/c5f29726-ab9e-494e-85a5-4c3c500ad237" />


## Credenciales de Gemini (Google AI)

1. Ve a Google Cloud Console  
   `https://console.cloud.google.com/`

2. Crea un proyecto nuevo

3. Entra a Google AI Studio  
   `https://aistudio.google.com/`

4. Ve a **API Keys → Create API key**

5. Copia el **API Key**

6. En **n8n**:
   - Abre cualquier nodo de **Gemini**
   - Crea una nueva credencial
   - Pega la clave
   - Guarda

<img width="1280" height="435" alt="image" src="https://github.com/user-attachments/assets/a5eae152-b25e-4120-a955-a4fc754b8e16" />

---

## Credenciales de OpenAI

1. Ingresa a OpenAI  
    `https://platform.openai.com/usage`

2. Ve a **API Keys → “Create new secret key”**

3. Copia la clave privada

4. En **n8n**:
   - Abre un nodo de **OpenAI**
   - Crea una credencial nueva
   - Pega la clave
   - Guarda.

<img width="1280" height="322" alt="image" src="https://github.com/user-attachments/assets/c8c19fae-5d03-4ea3-88ba-a89d49ea173e" />

# 4. Funcionamiento del Asistente:

Este bot es capaz de:

- Recibir texto o audio  
- Detectar si el usuario pide una **imagen**  
- Responder con:  
  - Texto  
  - Audio  
  - Imagen generada  

## Lógica general del flujo

<img width="1280" height="514" alt="image" src="https://github.com/user-attachments/assets/b63e7768-62f0-4236-87bc-b588be959ff8" />


1. El usuario envía **texto o audio**.  
2. Si es audio → se **descarga y transcribe**.  
3. Se verifica si el texto comienza con **"imagen"**.  
4. Si inicia con “imagen” → **se genera una imagen**.  
5. En otros casos:  
   - Si el usuario envió texto → se responde en **texto**.  
   - Si el usuario envió audio → se responde con **audio generado**.

> El contexto del sistema restringe todas las respuestas al **dominio del entrenamiento físico**, actuando siempre como un entrenador personal profesional.

---

## Prompts Internos

### Prompt interno para respuestas sobre ejercicios (texto/audio)

```
Eres un entrenador personal profesional. Cuando te dé el nombre de un ejercicio o una pregunta sobre un movimiento, responde de forma muy clara, concisa y directa. No escribas párrafos largos ni explicaciones extensas.
Tu respuesta debe incluir únicamente:
- Descripción breve del ejercicio (1–2 líneas).
- Paso a paso para ejecutarlo correctamente (en viñetas).
- Músculos principales trabajados.
- Errores comunes a evitar (máximo 3).
- Recomendaciones de seguridad o técnica (máximo 3).

No agregues información innecesaria, teorías, fisiología ni advertencias genéricas.

Entrada: {{ $json.text }}
Salida: Respuesta precisa siguiendo el formato indicado, máximo 100 palabras.
```

### Prompt interno para generación de imágenes

```
Eres un generador de imágenes especializado en ejercicios físicos y entrenamiento.
Cuando reciba el nombre de un ejercicio, debes generar una imagen clara, precisa y funcional, enfocada en mostrar la técnica correcta.

Sigue estas reglas:
- Muestra una única persona realizando el ejercicio.
- Postura correcta, ángulo claro y detalles visibles.
- Fondo simple, estilo realista o ilustración técnica (sin distracciones).
- Sin efectos artísticos exagerados.
- Evita rostros famosos o rasgos reconocibles específicos.
- Enfócate en que la ejecución sea educativa y fiel a la técnica real.

Entrada: {{ $json.text }}
Salida: imagen que muestre el ejercicio con la técnica adecuada, desde un ángulo óptimo para comprenderlo.
```

# 5. Ejemplos de Casos de Uso

## Caso 1 — Pregunta por texto → Respuesta en texto

**Entrada:**  
¿Cómo hago una sentadilla con barra?

**Salida esperada:**  
Texto breve, técnico y estructurado según el prompt.

---

## Caso 2 — Pregunta por audio → Respuesta en audio

**Entrada (audio transcrito):**  
Explícame cómo hacer un peso muerto rumano.

**Salida:**  
Audio con instrucciones claras según formato.

---

## Caso 3 — Solicitud de imagen

**Entrada:**  
Imagen, de una plancha abdominal

**Salida:**  
Imagen técnica mostrando postura correcta.
