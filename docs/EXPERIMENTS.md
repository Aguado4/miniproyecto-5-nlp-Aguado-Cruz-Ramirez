# EXPERIMENTS - Bitácora de resultados

> **Regla:** solo números efectivamente ejecutados. **Estado:** sin corrida de referencia.

| Campo | Valor |
|---|---|
| Fecha | — |
| Plataforma / GPU | — |
| Ollama / modelo | 0.35.1 / `llama3.2:3b` |
| Codificador principal | — |
| Semilla | 42 |
| Ejecución | — |

## 1. Corpus

| Fuente | Documentos | Trozos | Palabras | Mediana |
|---|---:|---:|---:|---:|
| Fichas de destino | 40 | | 131.920 | 2.152 |
| Reseñas | | | | |

## 2. Codificadores (§4)

| Modelo | Dimensión | Segundos en indexar | Precisión@5 de destino | MRR |
|---|---:|---:|---:|---:|

## 3. Recuperador: evaluación con metadatos (§6)

| Variante | Precisión@5 destino | Precisión@5 fuente | Recall@5 | MRR |
|---|---:|---:|---:|---:|
| denso, k=5 | | | | |

## 4. H1 · El agujero de la polaridad (§7)

| Pregunta por lo negativo | % recuperado con 1★-2★ | Distribución del corpus |
|---|---:|---:|
| denso sin filtro | | 2,6 % de 1★ |
| con filtro de metadatos | | |

## 5. H3 · Filtrar frente a ampliar k (§8)

| Variante | Precisión@k | Fidelidad | Latencia (ms) |
|---|---:|---:|---:|

## 6. Costo por etapa (§9)

| Etapa | ms |
|---|---:|
| Embedding de la consulta | |
| Búsqueda en el store | |
| Reordenamiento | |
| Generación | |

## 7. Veredicto de hipótesis

| Hipótesis | Veredicto |
|---|---|
| **H1** · los embeddings densos recuperan por tema y no por polaridad | pendiente |
| **H2** · el tamaño de trozo óptimo difiere entre fichas y reseñas | pendiente |
| **H3** · filtrar por metadatos rinde más que subir k | pendiente |
