# DATASET - Ficha del corpus híbrido

El RAG de esta entrega indexa **dos fuentes con naturalezas opuestas**, y esa oposición es el
punto del ejercicio (D-501): una es factual, larga y de autoría enciclopédica; la otra es
subjetiva, corta y viene con etiqueta.

| | Fichas de destino | Reseñas de viajeros |
|---|---|---|
| Origen | Wikipedia en español | `vg055/Rest-Mex2025` (HuggingFace) |
| Documentos | 40 | 208.051 disponibles, ~8.000 indexadas |
| Mediana de longitud | 2.152 palabras | 48 palabras |
| Se parte en trozos | **sí** | **no** (D-505) |
| Metadatos | destino, región, título, URL | destino, región, tipo, **estrellas** |
| Qué responde | hechos | experiencias |
| Licencia | CC BY-SA 4.0 | la del dataset original |

---

## 1. Fichas de destino (fuente propia de esta entrega)

Archivo: `data/fichas_destino.json`, **versionado** en el repositorio (D-503), 823 KB.

Una ficha por cada uno de los 40 destinos que aparecen en el corpus Rest-Mex, descargada con la
API de MediaWiki (`action=query&prop=extracts&explaintext`) en texto plano.

| Métrica | Valor |
|---|---:|
| Destinos | 40 |
| Palabras totales | 131.920 |
| Mediana / mínimo / máximo | 2.152 / 443 / 13.965 |
| Más largas | Teotihuacan (13.965), Orizaba (13.566), Metepec (8.463) |
| Más cortas | Ajijic (443), Tapalpa (582), Cuatro_Cienegas (636) |

### Cómo se construyó, y qué salió mal

El mapeo destino → artículo es **curado a mano**, no automático, por dos razones comprobadas:

1. **La búsqueda automática falla con los nombres sin tildes.** El corpus escribe los destinos
   sin acentos y con guiones bajos (`TodosSantos`, `Patzcuaro`, `Teotihuacan`). Buscar
   «TodosSantos Mexico» en Wikipedia devuelve *Santa Fe (Ciudad de México)*, que no tiene
   ninguna relación.
2. **Hay homónimos.** `Valladolid` es sobre todo la ciudad española, `Loreto` tiene artículos en
   varios países y `Tequila` es a la vez una bebida y un municipio de Jalisco.

Dos destinos no tienen artículo de localidad con contenido utilizable y se resolvieron con el
artículo más pertinente, lo que queda registrado en el propio archivo:

- **Metepec**: el artículo `Metepec` es una página de desambiguación, así que se usa
  `Municipio de Metepec (Estado de México)`.
- **Xilitla**: el artículo de la localidad es un esbozo de 110 palabras, así que se usa
  **`Las Pozas`**, el jardín surrealista de Edward James, que además es la atracción de la que
  hablan las reseñas de ese destino.

La API **estrangula las ráfagas**: a partir de unas quince peticiones seguidas responde HTTP 429
con HTML en lugar de JSON, y el extracto de texto completo solo se puede pedir **de un artículo
por petición** (`exlimit` solo funciona si se pide la introducción). El recolector es por eso
reanudable, con reintentos y pausas de dos segundos. Es exactamente la razón por la que el
archivo se versiona en lugar de descargarse en cada ejecución (D-503).

### Atribución

El texto es de Wikipedia en español bajo **CC BY-SA 4.0**. Cada ficha conserva su
`titulo_wikipedia` y su `url`, que es la atribución que la licencia requiere, y el chatbot cita
esos campos cuando responde con una ficha.

---

## 2. Reseñas de viajeros (heredadas de MP1 a MP4)

`vg055/Rest-Mex2025`, el mismo corpus de las cuatro entregas anteriores. 208.051 reseñas de
viajeros sobre destinos turísticos mexicanos, con título, texto, destino, región, tipo de
establecimiento y polaridad de 1 a 5 estrellas.

| Campo | Valores |
|---|---|
| `Polarity` | 1 a 5 estrellas. Muy desbalanceado: ~66 % son 5★ y ~2,6 % son 1★ |
| `Type` | Restaurant (86.720), Attractive (69.921), Hotel (51.410) |
| `Region` | 12 regiones; QuintanaRoo concentra 85.993 |
| `Town` | 40 destinos; Tulum concentra 45.345 |

Para el índice se toma una submuestra estratificada por destino y estrellas, de tamaño
configurable (~8.000 por omisión, menos en CPU), porque indexar 208.051 reseñas no añade nada al
ejercicio y multiplica el costo de los *embeddings*.

### Qué aporta la etiqueta

Es lo que distingue a esta entrega de un RAG cualquiera: **permite evaluar la recuperación sin
anotación humana** (D-504). Si la pregunta es «¿qué problemas reportan los hoteles de Tulum?», la
respuesta correcta no es un texto sino un conjunto de metadatos (`destino=Tulum`, `tipo=Hotel`,
`estrellas<=2`), y comprobar si lo recuperado los cumple es automático.

### Lo que el desbalance implica aquí

El 66 % de 5★ no es un problema de clasificación en esta entrega, sino el **sesgo del índice**:
si el recuperador no distingue polaridad (H1), cualquier pregunta por lo negativo devolverá
mayoritariamente reseñas positivas simplemente porque son las que abundan. Esa es la predicción
que §7 pone a prueba.
