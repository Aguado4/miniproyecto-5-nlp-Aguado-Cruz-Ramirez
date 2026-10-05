# CLAUDE.md — Instrucciones de trabajo para este repositorio

Fuente de verdad operativa para cualquier sesión de agente. Léelo completo antes de tocar nada.

---

## 1. Qué es este repositorio

Entregable del **Miniproyecto 5** del curso de NLP (Maestría, Universidad Icesi): **un único
Jupyter Notebook** que monta un **chatbot con RAG** sobre un corpus turístico **híbrido** (40
fichas de destino de Wikipedia en español más reseñas reales de Rest-Mex), mide la recuperación
con los metadatos de las reseñas en lugar de juzgarla a ojo, y expone el bot en una interfaz de
Gradio.

Guía: `icesi-nlp/Sesion5/1-ollama-rag.ipynb` (RAG sobre WikiHow con Ollama) y
`2-ollama-langchain.ipynb` (lo mismo con LangChain). **No se usa su dataset.**

`consigna.txt` y `rubrica.txt` están en la raíz y **no se modifican** (7 puntos, 4 criterios).

### Relación con las entregas anteriores

- Se reutiliza el corpus `vg055/Rest-Mex2025` de los Miniproyectos 1 a 4.
- **A diferencia de MP2, MP3 y MP4, aquí NO se hereda el bloque de celdas de MP1** (§D-502): no
  hay tarea de clasificación, el protocolo de *splits* no aplica, y la retroalimentación del MP1
  pidió notebooks menos extensos. El EDA es propio y orientado a lo que un RAG necesita.

## 2. Documentos (leer en este orden)

`docs/SPEC.md` (contrato) · `docs/PLAN.md` · `docs/DATASET.md` · `docs/DECISIONS.md` (desde
D-501) · `docs/EXPERIMENTS.md` (solo números ejecutados).

## 3. Reglas duras

### 3.1 Reproducibilidad — 2 de 7 puntos

- «Restart & Run All» limpio. `SEED = 42`, fijada antes de cada bloque de generación.
- Las fichas de destino **se versionan** en `data/fichas_destino.json`: el notebook no debe
  depender de la API de Wikipedia, que estrangula las ráfagas (§D-503).
- Modelo de Ollama **fijado por nombre y etiqueta** (`llama3.2:3b`), no «el último».
- GPU/CPU detectados; en CPU se degrada el número de reseñas, nunca falla.
- Los *embeddings* se guardan en caché en `data/`, que no se versiona, y se recalculan si falta.

### 3.2 Narrativa — 1 punto

Todo en español; markdown antes de cada celda de código explicando el porqué; toda gráfica y
tabla con su lectura; **sin guiones largos en la narrativa** (convención de MP1). Las lecturas
van dentro del markdown de la sección, no en celdas aparte (lección del MP4).

### 3.3 ChatBot con RAG — 2 puntos

Recuperador, generador y **citas verificables** (destino, fuente y estrellas). Mostrar ejemplos
de interacción y conclusiones con los conceptos bien usados (recuperación densa, *chunking*,
reordenamiento, fidelidad).

### 3.4 Innovación — 2 puntos

Corpus híbrido, evaluación medible con metadatos, el agujero de la polaridad y el filtro que lo
corrige, reordenamiento y BM25, fidelidad de la cita, versión con LangChain (SPEC §6).

## 4. Convenciones técnicas

- Un solo codificador para documentos y consultas; si se compara otro, se recalcula el store.
- Las reseñas **no se parten** en trozos: ya son unidades completas (§D-505).
- Toda respuesta del bot cita sus fuentes con metadatos, no solo con el texto.
- El banco de preguntas define la respuesta correcta **en metadatos**, nunca en texto libre
  (§D-504), para que la evaluación sea automática.

## 5. Qué NO hacer

- No usar WikiHow ni los datos del guía.
- No copiar los defectos del guía: evaluación solo a ojo, un único tipo de documento, sin
  medir el costo de cada etapa.
- No inventar resultados en `EXPERIMENTS.md`.
- No hacer `git push` ni crear *releases* sin pedirlo al usuario.
- No dejar `TODO` ni `<!-- LEER -->` en el entregable final.
- No ejecutar nada en la GPU sin confirmar con el usuario: la usa para otras cosas.

## 6. Pendiente que no depende del agente

El **video de 2 minutos** con la demo de Gradio lo graba el equipo. Su ausencia afecta
considerablemente la calificación (ver `consigna.txt`).
