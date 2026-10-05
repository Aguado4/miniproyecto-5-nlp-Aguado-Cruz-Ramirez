# Miniproyecto 5 - NLP

**Un asistente de viajes con RAG: hechos de Wikipedia, experiencias de viajeros reales**

Maestría · Universidad Icesi · Curso de Procesamiento de Lenguaje Natural

Autores: Juan José Aguado · Juan David Cruz · Juan Diego Ramírez

---

## El problema

Un chatbot con RAG sobre un corpus **híbrido** de dos fuentes opuestas: **40 fichas de destino**
de Wikipedia en español (factuales y largas) y **reseñas reales de viajeros** del corpus Rest-Mex
de los Miniproyectos [1](../miniproyecto%201/), [2](../miniproyecto%202/),
[3](../miniproyecto%203/) y [4](../miniproyecto%204/) (subjetivas, cortas y **etiquetadas**).

> **¿Puede un RAG responder con hechos cuando le preguntan por hechos y con experiencias cuando
> le preguntan por experiencias, y se puede *medir* si recupera lo correcto en lugar de juzgarlo
> a ojo?**

- **H1.** Los *embeddings* densos recuperan por **tema** y no por **polaridad**: ante «¿qué
  problemas reportan los hoteles de Tulum?» traerán reseñas de hoteles de Tulum sin distinguir
  las buenas de las malas.
- **H2.** El tamaño óptimo de trozo difiere entre las dos fuentes.
- **H3.** Filtrar por metadatos rinde más que subir la `k` del recuperador.

## Por qué este caso

El notebook guía monta un RAG sobre WikiHow y lo evalúa leyéndolo: «vemos que responde
coherentemente». Aquí hay dos cosas que ese montaje no permite y este sí.

**Se puede medir.** Las reseñas traen destino, tipo de establecimiento y estrellas, así que la
respuesta correcta de cada pregunta se puede expresar **en metadatos** y comprobar
automáticamente, sin que nadie lea nada. Casi ningún proyecto de RAG puede hacer esto, porque no
sabe cuál era el documento correcto.

**Hay dos fuentes compitiendo.** Con un solo tipo de documento el recuperador no decide nada. Con
fichas y reseñas en el mismo índice, cada pregunta tiene una fuente correcta y acertarla es
medible.

## La propuesta

| Sección | Qué hace |
|---|---|
| §2 | Corpus híbrido y EDA propio de las dos fuentes |
| §3 | Partición en trozos, distinta para cada fuente (H2) |
| §4 | *Embeddings*, vector store y comparación de dos codificadores con su costo |
| §5 a §6 | Recuperador denso y **evaluación medible con metadatos** |
| §7 | **El agujero de la polaridad** (H1) y el recuperador con filtro |
| §8 | Reordenamiento con *cross-encoder* y recuperación híbrida con BM25 (H3) |
| §9 a §10 | Generación con Ollama, chatbot con citas y fidelidad de la cita |
| §11 | La misma cadena con LangChain |
| §12 a §13 | Demo en Gradio y conclusiones |

## Qué corregimos o añadimos frente al notebook guía

El guía evalúa a ojo, usa un corpus homogéneo y no mide el costo de ninguna etapa. Aquí la
evaluación es automática y con métricas, el corpus es híbrido a propósito, y se mide el costo por
etapa para poder decidir con criterio. Detalle y alternativas descartadas en
[`docs/DECISIONS.md`](docs/DECISIONS.md).

**Nota sobre la continuidad de la serie:** a diferencia de MP2, MP3 y MP4, esta entrega **no
hereda el bloque de celdas de MP1**. No es un olvido: aquí no hay tarea de clasificación, así que
el protocolo de *splits* y los *baselines* no se usan en ninguna celda, y la retroalimentación del
profesor sobre el MP1 pidió notebooks menos extensos. El razonamiento completo está en D-502.

## Resultados

> Pendiente de la corrida de referencia ([`docs/EXPERIMENTS.md`](docs/EXPERIMENTS.md)).

## Cómo ejecutarlo

1. **Ollama** (no es un paquete de Python):
   - Windows y macOS: instalador de <https://ollama.com/download>
   - Linux y Colab: `curl -fsSL https://ollama.com/install.sh | sh`
2. El modelo: `ollama pull llama3.2:3b` (unos 2 GB).
3. Las dependencias de Python: `pip install -r requirements.txt`
4. Abrir `notebooks/miniproyecto5_restmex_rag.ipynb` y ejecutar de arriba abajo.

Las fichas de destino ya vienen en `data/fichas_destino.json`, así que el notebook no depende de
la API de Wikipedia. Las reseñas se descargan de HuggingFace en la primera ejecución.

## El video

La consigna exige un video de **máximo 2 minutos** con la celda que levanta Gradio y un par de
interacciones donde el bot cite los documentos. §12 deja la interfaz lista y un guion de
preguntas pensado para que la grabación sea corta y las citas se vean.

> Enlace al video: *pendiente*
