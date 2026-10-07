# Mundial-Bot: chatbot conversacional con RAG sobre Mundiales de fútbol (1930–2026)

Mini-proyecto del curso de Procesamiento de Lenguaje Natural, Maestría en IA Aplicada, Universidad Icesi.

**Autores:** José Luis Realpe M., Alejandra Forero, Santiago Aristizabal, Sandra Orozco.

Es una versión a pequeña escala de un ChatGPT: un modelo de lenguaje, conocimiento recuperado (RAG), memoria de la conversación y una interfaz web. A diferencia de ChatGPT, usa un modelo de 3B parámetros y responde **solo** con el corpus de Mundiales; si el dato no está, lo dice.

- **Notebook:** [`mini_proyecto_RAG.ipynb`](mini_proyecto_RAG.ipynb)


## Descripción del proyecto

Se pregunta en español (por ejemplo, «¿Cómo terminó Colombia contra la Unión Soviética en 1962?») y el chatbot responde con la información recuperada de partidos, plantillas y artículos de Wikipedia, citando la fuente (partido y edición, o artículo). Los datos están en inglés, por eso se usan embeddings multilingües.

```
Pregunta (español) → Reformulación con historial → Recuperador (denso · BM25 · híbrido)
→ Índice FAISS (fragmentos + metadatos) → LLM llama3.2:3b (Ollama) → Respuesta con fuentes
```

## Tecnologías

LangChain, Ollama (`llama3.2:3b`), `intfloat/multilingual-e5-large` (con prefijos `query:` / `passage:`), FAISS (CPU), BM25 y Gradio. Versiones y semilla fijas en la primera celda del notebook.

## Corpus

| Fuente | Contenido |
|---|---|
| [openfootball/worldcup](https://github.com/openfootball/worldcup) | Partidos, alineaciones y plantillas 1930–2026 |
| Wikipedia en español | Artículo de la selección de Colombia y de los Mundiales de 1962, 1990, 1994, 1998, 2014, 2018 y 2026 |

## Cómo ejecutarlo (Google Colab)

1. Abre el notebook en Colab.
2. Elige **Entorno de ejecución → Cambiar tipo de entorno → GPU T4**.
3. Sube `data_wikipedia.zip` (está en este repositorio) al panel de archivos de Colab, junto al notebook. Es la copia congelada de Wikipedia con su fecha de descarga y revisión. Si no se sube, el notebook descarga los artículos actuales y los resultados pueden cambiar.
4. Ejecuta **Entorno de ejecución → Ejecutar todo**. El notebook instala las dependencias, clona openfootball, levanta Ollama y descarga el modelo.
5. La última celda lanza la interfaz de Gradio. La interfaz y el enlace público duran solo mientras la sesión de Colab siga activa.

Banderas al inicio del notebook: `SMOKE_TEST` (prueba rápida con un subconjunto), `REBUILD_INDEX` (reconstruir índices) y `USAR_DRIVE` (guardar índices en Google Drive).

## Resultados principales

Evaluación con 40 preguntas generadas automáticamente desde los registros.

- La recuperación densa con e5-large deja el fragmento correcto en primer lugar en el 95 % de los casos y entre los tres primeros en todos. BM25 y el híbrido quedaron por debajo.
- Dar contexto al modelo sube la exactitud en datos directos de 0,357 (sin RAG) a entre 0,786 y 0,821 (con RAG), y la abstención fuera de dominio de 0,2 a 0,9.
- Subir k de 3 a 10 no mejora la exactitud y duplica la latencia.

## Limitaciones

- Un modelo de 3B falla más con cifras y no sirve para contar o sumar entre partidos.
- La memoria de la conversación es irregular: a veces el seguimiento recupera el partido equivocado.
- Los datos de 2026 no se contrastaron con otra fuente y no tienen alineaciones.
- Wikipedia cambia: el notebook trabaja con una copia con fecha de descarga y revisión.
- Las 40 preguntas siguen plantillas, así que las métricas son optimistas respecto a preguntas reales.

El detalle está en la sección 10 del notebook.

## Fuentes y licencias

- openfootball/worldcup: CC0. Las carpetas `wikipedia/` y `rsssf/` son conversiones de páginas de terceros.
- Wikipedia en español: CC BY-SA.
- Lewis et al., *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*, 2020.
- Basado en el notebook guía `2-ollama-langchain` del curso.
