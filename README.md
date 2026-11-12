# Identificación y extracción de datos

Aplicación de Python que obtiene publicaciones públicas de una cuenta de Bluesky, limpia sus textos y los guarda en MongoDB. También puede generar un resumen y extraer temas comunes con Gemini.

## Características principales

- Consulta hasta 100 publicaciones del perfil de Bluesky configurado.
- Elimina enlaces, emojis y espacios sobrantes del texto.
- Guarda o actualiza las publicaciones en MongoDB, evitando duplicados mediante el URI de Bluesky.
- Conserva el actor, URI, CID, texto procesado y fecha de creación disponible.
- Genera un resumen general y hasta tres temas principales con Gemini.
- Devuelve el texto del análisis y lo guarda en un archivo UTF-8 y en un documento de MongoDB asociado a las URI de las publicaciones analizadas.
- Si Gemini no está configurado o no está disponible, las publicaciones se guardan igualmente.

## Tecnologías utilizadas

- **Python**
- **Bluesky AT Protocol:** `atproto`
- **MongoDB:** `pymongo`
- **Google Gemini:** `google-genai`
- **Configuración de entorno:** `python-dotenv`
- **HTTP:** `httpx`

Las versiones de las dependencias están fijadas en [`requirements.txt`](requirements.txt).

## Requisitos previos

- Python 3.10 o posterior.
- Una cuenta de Bluesky y una contraseña de aplicación.
- MongoDB en ejecución local o una URI de conexión a una instancia remota.
- Una clave de Gemini para habilitar el análisis (opcional).
- Git, si se va a clonar el repositorio.

## Instalación y configuración

Clona el repositorio (sustituye la URL por la dirección del repositorio):

```powershell
git clone <URL-del-repositorio>
cd IdentificacionExtraccionDatosUD
```

Crea y activa un entorno virtual en PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Instala las dependencias:

```powershell
python -m pip install -r requirements.txt
```

Crea un archivo `.env` en la raíz del proyecto:

```env
BSKY_IDENTIFIER=tu_usuario.bsky.social
BSKY_PASSWORD=tu_contrasena_de_aplicacion
MONGO_URI=mongodb://localhost:27017
GENAI_API_KEY=tu_clave_de_gemini
```

`MONGO_URI` es opcional cuando MongoDB está disponible localmente con la configuración predeterminada. La base de datos y la colección predeterminadas son `bluesky.posts`; puedes cambiarlas añadiendo:

```env
MONGO_DATABASE=bluesky
MONGO_COLLECTION=posts
MONGO_ANALYSIS_COLLECTION=feed_analyses
```

La clave `GENAI_API_KEY` es opcional. Sin ella, la aplicación omite el análisis de Gemini, pero continúa guardando las publicaciones. No compartas ni subas el archivo `.env` al repositorio.

## Guía de uso

Con MongoDB en ejecución y el entorno virtual activado, ejecuta:

```powershell
python main.py
```

El programa consulta por defecto las últimas 100 publicaciones de `hablemosofertas.bsky.social`, guarda los textos procesados y, si Gemini está configurado, muestra el análisis en la terminal.

Cuando Gemini devuelve un análisis, este se imprime y se devuelve desde `get_author_feed`. Además, se crea un archivo `analisis_<id-del-feed>.txt` en la raíz del proyecto que contiene únicamente la respuesta. Los datos para asociar el análisis al feed se guardan en un documento de la colección `feed_analyses` (o en la colección configurada mediante `MONGO_ANALYSIS_COLLECTION`), junto con la respuesta y la fecha del análisis. Si se analiza de nuevo exactamente el mismo feed, el documento correspondiente se actualiza.

Para consultar otra cuenta o cambiar el límite, modifica la última línea de `main.py`:

```python
get_author_feed("nombre-de-cuenta.bsky.social", 50)
```

El análisis usa actualmente el modelo `gemini-3.5-flash-lite`. Si el servicio de Gemini no está disponible, se muestra un aviso y la ejecución continúa sin análisis.

## Autor y agradecimientos

- **Autor:** Lorenzo Cruz Fernández.
- Gracias a Bluesky por el protocolo AT, a MongoDB por su base de datos y a Google por la API de Gemini y sus herramientas.
