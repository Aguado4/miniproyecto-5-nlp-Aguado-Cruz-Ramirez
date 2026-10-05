# DECISIONS - Registro de decisiones

> Una entrada por decisión que un lector podría cuestionar. Formato fijo: contexto, decisión,
> alternativas descartadas y consecuencias. Numeración desde D-501.

---

## D-501 · Corpus híbrido: fichas de destino más reseñas

**Estado:** aceptada · 2026-10-04

**Contexto.** El guía monta el RAG sobre un corpus de un solo tipo de documento (artículos de
WikiHow). Con documentos homogéneos el recuperador no tiene que decidir nada: cualquier
resultado del mismo corpus es «del tipo correcto».

**Decisión.** Mezclar dos fuentes deliberadamente distintas: **40 fichas de Wikipedia en
español** (una por destino de Rest-Mex: largas, factuales) y **reseñas de viajeros** (cortas,
subjetivas, etiquetadas). Un asistente de viajes real recibe las dos clases de pregunta, de
hecho («¿qué es Bacalar?») y de experiencia («¿cómo tratan en los hoteles de Bacalar?»), y debe
responder cada una con la fuente que corresponde.

**Alternativas.** (a) Solo reseñas: no habría conocimiento factual que citar y la mitad de las
preguntas de un viajero quedarían sin respuesta. (b) Solo fichas: sería un RAG sobre Wikipedia,
indistinguible del guía salvo por el tema, y se perdería la etiqueta, que es lo que permite
medir. (c) Un tercer corpus nuevo: más trabajo de limpieza sin añadir una pregunta nueva.

**Consecuencias.** El recuperador enfrenta un problema real de competencia entre fuentes, y eso
se puede medir (precisión de fuente, §6). A cambio, hay que resolver el *chunking* por separado
para cada tipo (§D-505) y cuidar que las 40 fichas no queden ahogadas entre miles de reseñas.

---

## D-502 · Sin bloque heredado de MP1

**Estado:** aceptada · 2026-10-04

**Contexto.** MP2, MP3 y MP4 copian sin modificar las celdas 3 a 65 del notebook de MP1 (63
celdas de entorno, EDA y protocolo), para garantizar que las comparaciones entre entregas se
hacen sobre el mismo *split*. Lo natural sería repetirlo aquí.

**Decisión.** **No heredar el bloque.** Esta entrega escribe su propio EDA, corto y orientado a
lo que un RAG necesita: longitud de los documentos de cada fuente, número de trozos, cobertura
por destino y solapamiento de vocabulario entre fuentes.

**Por qué.** Tres razones, en orden de peso. Primera: **no hay tarea de clasificación**, así que
el protocolo de Split A y B, la función `evaluar` y los *baselines* heredados no se usan en
ninguna celda; copiarlos sería arrastrar 63 celdas de código muerto. Segunda: la
retroalimentación del profesor sobre el MP1 señaló como único punto de mejora que **el notebook
era demasiado extenso**, y heredar un bloque que no se usa va justo en contra. Tercera: lo que sí
importa de la continuidad (que el corpus y la submuestra sean los mismos) se consigue citando la
ficha de datos y usando el mismo identificador de HuggingFace.

**Alternativas.** (a) Heredar el bloque completo: 63 celdas sin uso y un notebook mucho más
largo. (b) Heredar solo la carga y la limpieza: rompería la identidad byte a byte, que es lo
único que justificaba heredar.

**Consecuencias.** Esta es la primera entrega de la serie sin bloque heredado, y hay que decirlo
en el README para que no parezca un olvido. Los números de esta entrega no son comparables con
las tablas de MP1 a MP4, pero tampoco lo pretenden: aquí no se clasifica nada.

---

## D-503 · Las fichas de Wikipedia se versionan en el repositorio

**Estado:** aceptada · 2026-10-04

**Contexto.** Lo natural sería que el notebook descargara las 40 fichas de la API de Wikipedia al
ejecutarse. Al construir el corpus se comprobó que **la API estrangula las ráfagas**: a partir de
unas 15 peticiones seguidas responde con HTTP 429 y HTML en lugar de JSON, y hubo que hacer el
recolector reanudable, con reintentos y pausas de dos segundos. Una descarga completa tarda
varios minutos y puede fallar a mitad.

**Decisión.** Descargar las fichas **una vez**, guardarlas en `data/fichas_destino.json` y
**versionar ese archivo** (es texto, unos cientos de kilobytes). El notebook lee el archivo si
existe y solo llama a la API si falta, con el recolector reanudable incluido en una celda.

**Alternativas.** (a) Descargar siempre: convierte la reproducibilidad en una lotería que depende
del límite de peticiones de un tercero, y el criterio de reproducibilidad vale 2 de 7 puntos.
(b) Usar un *dump* de Wikipedia: decenas de gigabytes para 40 artículos.

**Consecuencias.** El notebook corre sin red salvo para el corpus de HuggingFace y el modelo de
Ollama. Las fichas quedan congeladas en la fecha de descarga, que se registra en el propio
archivo y en `DATASET.md`.

---

## D-504 · El banco de preguntas define la respuesta en metadatos

**Estado:** aceptada · 2026-10-04

**Contexto.** Evaluar un RAG suele exigir anotación humana: alguien lee la respuesta y dice si
está bien. Eso no escala en un trabajo de curso y es justo lo que el guía evita haciendo la
evaluación a ojo.

**Decisión.** Construir un banco de preguntas propio donde **la respuesta correcta se expresa en
metadatos, no en texto**. Cada pregunta declara qué debería recuperar: la fuente (ficha o
reseña), el destino y, cuando aplica, el tipo de establecimiento y el rango de estrellas. Así
«¿Qué problemas reportan los hoteles de Tulum?» se evalúa comprobando que lo recuperado tenga
`destino=Tulum`, `tipo=Hotel` y `estrellas<=2`, sin que nadie lea nada.

**Alternativas.** (a) Juzgar a ojo como el guía: no es una métrica y no permite comparar
variantes del recuperador. (b) Usar un LLM como juez: añade otra fuente de error y, sobre todo,
el juez sería el mismo modelo que genera, con la circularidad que eso implica (es la lección que
dejó §12 del MP4).

**Consecuencias.** La evaluación es automática, repetible y permite comparar recuperadores. La
limitación es que mide **recuperación**, no calidad de la redacción final; esa se declara como
cualitativa en §10 y §13.

---

## D-505 · Las reseñas no se parten en trozos

**Estado:** aceptada · 2026-10-04

**Contexto.** El guía parte todos los documentos en trozos de tamaño fijo con solapamiento, lo
cual tiene sentido para artículos largos de WikiHow.

**Decisión.** Partir **solo las fichas** de Wikipedia, y dejar cada reseña como un documento
completo. Una reseña tiene una mediana de 48 palabras y es una unidad de sentido cerrada: tiene
un autor, una valoración y una estrella. Partirla produce fragmentos sin polaridad reconocible y
rompe la correspondencia entre documento y etiqueta, que es la base de toda la evaluación de
§6 y §7.

**Alternativas.** (a) Partir todo por igual: destruye la unidad reseña-etiqueta. (b) Agrupar
varias reseñas del mismo destino en un documento: mezclaría estrellas distintas en un mismo
trozo y haría imposible citar «esta reseña de 1★».

**Consecuencias.** El corpus queda con dos poblaciones de documentos de longitud muy distinta, y
eso hay que tenerlo en cuenta al interpretar la similitud: los trozos largos tienden a
puntuar distinto que los cortos. Se mide en §4 y se discute en §6.

---

## D-506 · El tamaño de trozo sale de medirlo, y contradice la intuición

**Estado:** aceptada · 2026-10-04

**Contexto.** Había que elegir el tamaño de trozo de las fichas. El razonamiento *a priori*, que
es el que escribimos primero en el SPEC como H2, era que las fichas necesitarían trozos
**grandes**: un hecho partido a la mitad pierde el contexto que lo hace interpretable, y una ficha
de Wikipedia está escrita en párrafos largos.

**Hallazgo.** La prueba en CPU de §8.1 lo contradice, y de forma monótona. Con las preguntas de
hecho (familia A), la precisión@5 fue:

| Palabras por trozo | Solape | Trozos | Precisión@5 |
|---:|---:|---:|---:|
| **110** | 20 | 1.467 | **0,975** |
| 220 | 40 | 744 | 0,875 |
| 450 | 80 | 369 | 0,825 |

El mecanismo se ve en un ejemplo concreto: con trozos de 220 palabras, la pregunta «¿qué es la
laguna de Bacalar?» devolvía como primer resultado el párrafo del **censo de población** del
artículo de Bacalar. El nombre del destino aparece en todos los trozos del artículo, así que
domina la similitud, y lo que decide el desempate es cuánto *otro* tema arrastra el trozo. Cuanto
más corto, más puro el tema y menos ruido.

**Decisión.** El valor por omisión es **110 palabras con 20 de solape**, elegido por la medición y
no por el razonamiento previo. La narrativa de §3 se reescribió para presentar el compromiso y
remitir a §8.1, en lugar de afirmar una conclusión que los datos no sostienen.

**Alternativas.** Dejar 220 porque «suena razonable»: es exactamente el error que la rúbrica de
reproducibilidad y la retroalimentación del MP1 («que el equipo pueda justificar la profundidad
técnica») penalizan.

**Consecuencias.** El índice tiene el doble de documentos (1.467 trozos de ficha en lugar de 744),
lo que encarece algo el indexado y la memoria del store, a cambio de 0,10 de precisión en las
preguntas de hecho. H2, tal como la escribimos en el SPEC, **queda refutada** y así se reportará:
no es que el tamaño óptimo difiera por fuente en la dirección que supusimos, sino que para las
fichas gana el trozo corto. Falta confirmarlo en la corrida de referencia con 8.000 reseñas.

---

## Plantilla para nuevas entradas

## D-5NN · Título breve

**Estado:** aceptada | revisada | descartada · fecha

**Contexto.** Qué problema había.

**Decisión.** Qué se hizo.

**Alternativas.** Qué se descartó y por qué.

**Consecuencias.** Qué implica, incluido lo que empeora.
