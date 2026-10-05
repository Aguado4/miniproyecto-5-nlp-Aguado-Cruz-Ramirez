# PLAN - Ejecución por fases

> Estado global: **corpus construido y notebook escrito (51 celdas, 25 de código), sin ejecutar.**
> Las fichas de destino ya están descargadas y versionadas; falta la corrida de referencia, que
> necesita la GPU libre para descargar el modelo de Ollama.

Leyenda: `[ ]` pendiente · `[~]` en curso · `[x]` hecho

---

## Fase 0 - Scaffolding

- [x] Revisar consigna, rúbrica y los dos notebooks de `icesi-nlp/Sesion5`
- [x] Repositorio público y colaboradores invitados
- [x] Docs (`SPEC`, `DATASET`, `DECISIONS`), `CLAUDE.md`, `requirements.txt`, `.gitignore`
- [x] Instalar Ollama (0.35.1)
- [ ] Descargar `llama3.2:3b` (pendiente: requiere GPU libre)

## Fase 1 - Corpus (SPEC §2)

- [x] Mapeo curado de los 40 destinos a artículos de Wikipedia, con los homónimos resueltos
- [x] Recolector reanudable con reintentos; 40 de 40 fichas, 131.920 palabras
- [x] Corpus versionado en `data/fichas_destino.json` con procedencia y licencia
- [~] EDA propio de las dos fuentes (longitudes, cobertura por destino, sesgo del índice)

## Fase 2 - Recuperación (SPEC §3 a §8)

- [~] 3 Partición en trozos por tipo de documento
- [~] 4 Embeddings y vector store
- [~] 5 Recuperador denso con filtros
- [~] 6 Banco de 23 preguntas con respuesta en metadatos y evaluación medible
- [~] 7 El agujero de la polaridad (H1) y el filtro
- [~] 8 Estudios: tamaño de trozo (H2), codificador, reordenamiento, BM25 (H3)

> **Prueba en CPU (2026-10-04).** §1 a §8 se ejecutaron de punta a punta sin errores en **315 s**
> con 1.622 reseñas indexadas (configuración degradada de CPU). Resultados preliminares, no
> definitivos: la corrida de referencia usará 8.000 reseñas. Hallazgos que ya cambiaron el
> diseño: la columna de precisión no puede llevar la `k` en el nombre (hacía incomparables las
> filas de §8), y el tamaño de trozo pequeño gana (H2 apunta a quedar refutada).

## Fase 3 - Generación y chatbot (SPEC §9 a §11)

- [~] 9 Ollama con `llama3.2:3b`; costo por etapa
- [~] 10 ChatBot con citas, memoria y fidelidad de la cita
- [~] 11 Versión con LangChain

## Fase 4 - Cierre (SPEC §12 a §13)

- [~] 12 Demo en Gradio, con guion de preguntas para el video
- [ ] 13 Conclusiones; `EXPERIMENTS.md`; Restart & Run All
- [ ] **Grabar el video de 2 minutos** (lo hace el equipo, no el agente)

## Presupuesto de tiempo (estimado, por verificar)

La lección del MP4 es que el presupuesto se fija **midiendo**, no a ojo, así que estas cifras son
una hipótesis que la corrida de referencia debe confirmar o corregir.

| Bloque | min |
|---|---:|
| Corpus y EDA | 2 |
| Embeddings de ~8.000 reseñas más trozos de fichas | 4 a 6 |
| Comparación de codificadores (segundo store) | 3 a 4 |
| Evaluación del recuperador (banco de preguntas) | 1 |
| Reordenamiento y BM25 | 2 |
| Generación: respuestas del banco con `llama3.2:3b` | 4 a 6 |
| ChatBot, LangChain y demo | 2 |
| **Total** | **~20** |

Aparte, y una sola vez: descargar `llama3.2:3b` (unos 2 GB).
