# SPEC - Especificación del entregable

> **Estado:** borrador · **Versión:** 0.1 · **Última actualización:** 2026-10-04
>
> Contrato del entregable. Si la implementación se desvía, se actualiza primero este archivo.

---

## 1. Problema

### 1.1 Enunciado

Un asistente de viajes con RAG sobre un **corpus híbrido** de dos fuentes muy distintas:

- **40 fichas de destino** de Wikipedia en español, una por cada pueblo o ciudad del corpus
  Rest-Mex: documentos largos, factuales, con historia, geografía y clima.
- **Reseñas reales de viajeros** del corpus Rest-Mex de los Miniproyectos 1 a 4: documentos
  cortos, subjetivos y, lo que es decisivo aquí, **etiquetados** con estrellas (1 a 5), tipo de
  establecimiento (hotel, restaurante, atracción) y destino.

> **¿Puede un RAG responder con hechos cuando le preguntan por hechos y con experiencias cuando
> le preguntan por experiencias, y se puede *medir* si recupera lo correcto en lugar de
> juzgarlo a ojo?**

### 1.2 Por qué esta pregunta

El notebook guía de la Sesión 5 monta un RAG sobre WikiHow y evalúa el resultado leyéndolo:
«vemos que responde coherentemente y ha citado el artículo». Eso es razonable para una demo,
pero deja dos preguntas sin respuesta, y las dos son las que decidimos atacar.

**Primera: la evaluación.** Casi ningún proyecto de RAG puede medir su recuperación, porque no
hay forma de saber cuál era el documento correcto. **Aquí sí**, porque las reseñas traen
metadatos. Si la pregunta es sobre Tulum, se puede calcular qué fracción de lo recuperado es de
Tulum. Si la pregunta es por hoteles malos, se puede comprobar si lo recuperado son de verdad
reseñas de 1★ y 2★. La etiqueta convierte un juicio de gusto en una métrica.

**Segunda: la mezcla de fuentes.** Un corpus de un solo tipo de documento no obliga al
recuperador a decidir nada. Con dos tipos, cada pregunta tiene una fuente correcta («¿qué es
Bacalar?» es un hecho; «¿cómo tratan en los hoteles de Bacalar?» es una experiencia), y se puede
medir si acierta la fuente.

Hipótesis:

> **H1 (el agujero de la polaridad).** Los *embeddings* densos recuperan por **tema** y no por
> **polaridad**: ante «¿qué problemas reportan los hoteles de Tulum?», la similitud semántica
> traerá reseñas de hoteles de Tulum sin distinguir las buenas de las malas, porque «el servicio
> fue excelente» y «el servicio fue pésimo» viven casi en el mismo punto del espacio. La
> distribución de estrellas de lo recuperado será parecida a la del corpus (66 % de 5★) en lugar
> de concentrarse en 1★ y 2★.
>
> **H2 (el tamaño del trozo no es único).** El tamaño óptimo de *chunk* difiere entre las dos
> fuentes: las fichas necesitan trozos grandes para no partir un hecho a la mitad, y las reseñas
> ya son unidades completas, así que partirlas solo las empeora.
>
> **H3 (filtrar vale más que ampliar).** Para la fidelidad de la respuesta, filtrar por metadatos
> (destino y estrellas) rinde más que subir la `k` del recuperador: subir `k` añade contexto
> irrelevante y diluye, filtrar cambia lo que se recupera.

### 1.3 Por qué la propuesta es adecuada

1. **Es el caso de uso natural del corpus.** Las cuatro entregas anteriores clasificaron la
   polaridad de estas reseñas; esta las usa como fuente de conocimiento, que es lo que haría un
   buscador de viajes real.
2. **La etiqueta permite evaluar sin anotación humana**, que es el cuello de botella de todo
   proyecto de RAG.
3. **El corpus híbrido es un problema de recuperación de verdad**, no una demo: dos tipos de
   documento, 40 destinos y tres tipos de establecimiento compitiendo por el mismo espacio de
   *embeddings*.

### 1.4 Relación con las entregas anteriores

Se reutilizan el corpus (`vg055/Rest-Mex2025`) y la ficha de datos, pero **no se hereda el
bloque de celdas de MP1** como hicieron MP2, MP3 y MP4. Las razones están en D-502: aquí no hay
tarea de clasificación, así que el protocolo de *splits* no aplica, y la retroalimentación del
MP1 pidió expresamente notebooks menos extensos. El EDA de esta entrega es propio y mira lo que
un RAG necesita (longitudes, trozos, cobertura por destino), no lo que necesita un clasificador.

## 2. Datos

| Fuente | Documentos | Naturaleza | Metadatos |
|---|---:|---|---|
| Fichas de Wikipedia ES | 40 | factual, larga | destino, región, título, URL |
| Reseñas de Rest-Mex | configurable (~8.000) | subjetiva, corta | destino, región, tipo, estrellas |

Ficha completa en [`DATASET.md`](DATASET.md). Las fichas se versionan en
`data/fichas_destino.json` para que el notebook corra sin depender de la API de Wikipedia
(D-503); las reseñas se descargan de HuggingFace.

## 3. Protocolo y métricas

Todo se mide sobre un **banco de preguntas propio** con la respuesta correcta expresada en
metadatos, no en texto (D-504). Por ejemplo: «¿Qué es la laguna de Bacalar?» tiene como fuente
correcta `tipo=ficha, destino=Bacalar`; «¿Cómo es el servicio en los hoteles de Tulum?» tiene
`tipo=reseña, destino=Tulum, establecimiento=Hotel`.

| Métrica | Qué mide | Dónde |
|---|---|---|
| **Precisión@k de destino** | ¿lo recuperado habla del destino preguntado? | §6, §8 |
| **Precisión@k de fuente** | ¿acierta entre ficha y reseña? | §6 |
| **Acierto de polaridad** | ante una pregunta por lo malo, ¿recupera 1★-2★? | §7 (H1) |
| **MRR y recall@k** | calidad del orden del recuperador | §6 |
| **Fidelidad de la cita** | ¿lo que afirma la respuesta está en lo recuperado? | §10 |
| **Latencia por etapa** | costo de *embedding*, búsqueda, reordenamiento y generación | §9 |

## 4. Estructura del notebook

Archivo único: `notebooks/miniproyecto5_restmex_rag.ipynb`

| Sección | Contenido |
|---|---|
| 0 | Portada, problema, resultados principales enlazados por sección |
| 1 | Entorno, semilla, detección de GPU y de Ollama |
| 2 | **Corpus híbrido**: fichas de Wikipedia y reseñas; EDA propio de los dos |
| 3 | Partición en trozos: estrategia por tipo de documento (H2) |
| 4 | *Embeddings* y vector store; comparación de dos codificadores y su costo |
| 5 | Recuperador denso y búsqueda por similitud |
| 6 | **Banco de preguntas y evaluación medible** del recuperador |
| 7 | **El agujero de la polaridad** (H1) y el recuperador con filtro de metadatos |
| 8 | Reordenamiento con *cross-encoder* y recuperación híbrida con BM25 (H3) |
| 9 | El generador: Ollama con `llama3.2:3b`, y costo por etapa |
| 10 | El chatbot con citas, memoria de conversación y fidelidad de la cita |
| 11 | Versión con LangChain, para contrastar con el segundo notebook guía |
| 12 | **Demo en Gradio** (la del video) |
| 13 | Conclusiones y limitaciones |

## 5. Criterios de aceptación

- [ ] «Restart & Run All» sin errores, con presupuesto de ~20 min sin contar la descarga del
      modelo de Ollama.
- [ ] El notebook corre en CPU, degradando el número de reseñas, y nunca falla por falta de GPU.
- [ ] Toda gráfica y tabla con su lectura; conclusiones que responden H1, H2 y H3.
- [ ] El chatbot responde **citando** destino, fuente y, en las reseñas, las estrellas.
- [ ] Celda de Gradio que levanta la interfaz, lista para grabar el video.
- [ ] Sin `TODO` ni `<!-- LEER -->`.

## 6. Mapa rúbrica → notebook

| Criterio | Pts | Dónde |
|---|---:|---|
| Notebook completo | 1 | Narrativa por celda, lectura por sección, §13 |
| Reproducibilidad | 2 | `SEED`, fichas versionadas, *embeddings* en caché, degradación en CPU, modelo de Ollama fijado |
| Implementación del ChatBot con RAG | 2 | §5 a §10: recuperador, generador, citas, memoria; §12 demo |
| Innovación | 2 | Corpus híbrido (§2), evaluación medible con metadatos (§6), el agujero de la polaridad y el filtro (§7), reordenamiento y BM25 (§8), fidelidad de la cita (§10), LangChain (§11) |

## 7. Entregable aparte: el video

La consigna exige un **video de máximo 2 minutos** que muestre la celda que levanta Gradio y un
par de interacciones donde el bot responda citando documentos. **Lo graba el equipo**, no se
puede generar desde el notebook. §12 deja la interfaz y un guion de preguntas sugeridas para que
la grabación sea corta y muestre las citas.

## 8. Fuera de alcance

- Modelos por API de pago; el generador es local.
- Ajuste fino de los codificadores o del generador.
- Evaluación humana formal de la calidad de las respuestas.
