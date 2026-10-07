# Guía paso a paso - Práctica 2

## 1. Preparar el notebook

- [ ] Crear un notebook llamado `apelidos_nome_entregable2.ipynb`.
- [ ] Añadir una portada con nombre, asignatura, curso y fecha.
- [ ] Explicar brevemente que la práctica compara búsqueda léxica y búsqueda semántica.
- [ ] Organizar el notebook alternando celdas Markdown explicativas y celdas de código.
- [ ] Guardar los resultados en el notebook para que no sea necesario volver a ejecutar las celdas.

## 2. Descargar y preparar la colección CACM

- [ ] Descargar la colección CACM desde [Search Engines Book](http://www.search-engines-book.com/collections/).
- [ ] Localizar los documentos HTML del corpus.
- [ ] Descargar el fichero de consultas *processed queries*.
- [ ] Descargar el fichero de juicios de relevancia [`cacm.rel`](http://www.search-engines-book.com/data/cacm.rel).
- [ ] Guardar los archivos en carpetas separadas, por ejemplo:

  ```text
  datos/
  ├── cacm/
  ├── queries/
  └── cacm.rel
  ```

- [ ] Leer los documentos HTML con Python.
- [ ] Eliminar las etiquetas HTML usando BeautifulSoup.
- [ ] Conservar el cuerpo en texto plano.
- [ ] Mostrar algunos documentos de ejemplo y el número total de documentos cargados.

### Estructura recomendada de un documento

```python
{
    "doc_id": "1",
    "title": "Título del artículo",
    "body": "Título del artículo. Texto completo del documento..."
}
```

## 3. Crear el índice léxico con Whoosh

- [ ] Instalar e importar Whoosh.
- [ ] Definir un esquema con estos campos:
  - [ ] `doc_id`: identificador almacenado y único.
  - [ ] `title`: título del documento.
  - [ ] `body`: texto completo del documento, incluyendo el título.
- [ ] Crear una carpeta para el índice.
- [ ] Crear el índice Whoosh.
- [ ] Añadir todos los documentos del corpus al índice.
- [ ] Cerrar el escritor del índice correctamente.
- [ ] Comprobar cuántos documentos contiene el índice.

### Ejemplo de esquema

```python
from whoosh.fields import ID, TEXT, Schema

esquema = Schema(
    doc_id=ID(stored=True, unique=True),
    title=TEXT(stored=True),
    body=TEXT(stored=True),
)
```

## 4. Leer las consultas y los juicios de relevancia

- [ ] Leer todas las consultas del fichero *processed queries*.
- [ ] Asociar cada consulta con su número identificador.
- [ ] Leer `cacm.rel` una sola vez al comienzo del programa.
- [ ] Conservar únicamente los documentos marcados como relevantes con valor `1`.
- [ ] Crear una estructura eficiente, por ejemplo:

  ```python
  relevantes = {
      1: {"12", "35", "98"},
      2: {"7", "44"},
  }
  ```

- [ ] Comprobar que las consultas y los identificadores del corpus utilizan el mismo formato.
- [ ] Mostrar una consulta y sus documentos relevantes como ejemplo.

## 5. Implementar las métricas de evaluación

- [ ] Implementar Precision@10.
- [ ] Implementar Reciprocal Rank (RR).
- [ ] Para Precision@10, contar cuántos de los 10 primeros resultados son relevantes.
- [ ] Para RR, localizar la posición del primer documento relevante.
- [ ] Usar RR igual a `0` cuando no haya ningún documento relevante entre los resultados.
- [ ] Calcular la media de Precision@10 sobre todas las consultas.
- [ ] Calcular la media de RR sobre todas las consultas.

### Fórmulas

$$
Precision@10 = \frac{\text{documentos relevantes entre los 10 primeros}}{10}
$$

$$
RR = \frac{1}{\text{posición del primer documento relevante}}
$$

## 6. Buscar con Whoosh y BM25F

Realizar dos experimentos léxicos para todas las consultas:

- [ ] Buscar únicamente en el campo `title`.
- [ ] Buscar únicamente en el campo `body`.
- [ ] Usar el modelo BM25F por defecto de Whoosh.
- [ ] Recuperar únicamente los 10 primeros resultados.
- [ ] Guardar los identificadores recuperados para cada consulta.
- [ ] Calcular Precision@10 y RR para cada consulta.
- [ ] Mostrar las métricas individuales.
- [ ] Calcular y mostrar las métricas medias de cada variante.
- [ ] Comparar si buscar en `title` o en `body` produce mejores resultados.

### Resultado recomendado

Crear una tabla con estas columnas:

| Variante | Precision@10 media | RR medio |
|---|---:|---:|
| BM25F sobre `title` |  |  |
| BM25F sobre `body` |  |  |

## 7. Preparar los embeddings

- [ ] Tokenizar el texto de los documentos y de las consultas.
- [ ] Decidir cómo tratar palabras desconocidas.
- [ ] Entrenar un modelo Word2Vec con el corpus CACM.
- [ ] Comprobar la dimensión de los vectores generados.
- [ ] Representar cada documento como la media de los vectores de sus palabras.
- [ ] Representar cada consulta con el mismo procedimiento.
- [ ] Gestionar documentos o consultas sin ninguna palabra conocida.
- [ ] Normalizar los vectores si se utiliza similitud coseno.

### Ejemplo de media de embeddings

```python
import numpy as np


def embedding_medio(tokens, modelo):
    vectores = [modelo.wv[token] for token in tokens if token in modelo.wv]
    if not vectores:
        return np.zeros(modelo.vector_size, dtype=np.float32)
    return np.mean(vectores, axis=0).astype(np.float32)
```

## 8. Crear los índices FAISS

- [ ] Instalar e importar FAISS.
- [ ] Crear una matriz con los embeddings de todos los documentos.
- [ ] Crear un índice `IndexFlatL2` para la colección CACM.
- [ ] Añadir los embeddings de los documentos al índice.
- [ ] Mantener una lista que relacione cada posición del índice con su `doc_id`.
- [ ] Comprobar que el número de vectores del índice coincide con el número de documentos.

### Modelos que se deben probar

- [ ] Word2Vec entrenado con el corpus CACM.
- [ ] `glove-twitter-25`.
- [ ] `word2vec-google-news-300`.
- [ ] Un modelo SBERT preentrenado.

## 9. Buscar con FAISS

Para cada modelo de embeddings:

- [ ] Convertir cada consulta en un vector.
- [ ] Consultar el índice FAISS.
- [ ] Recuperar los 10 documentos más cercanos.
- [ ] Traducir las posiciones del índice a identificadores `doc_id`.
- [ ] Calcular Precision@10 y RR.
- [ ] Guardar los resultados individuales y medios.
- [ ] Comparar los resultados semánticos con los resultados de BM25F.

### Resultado recomendado

| Modelo | Precision@10 media | RR medio |
|---|---:|---:|
| Word2Vec propio |  |  |
| GloVe Twitter 25 |  |  |
| Word2Vec Google News 300 |  |  |
| SBERT |  |  |

## 10. Comparar los métodos

- [ ] Crear una tabla conjunta con todas las variantes.
- [ ] Comparar BM25F sobre `title` y `body`.
- [ ] Comparar Word2Vec, GloVe, Word2Vec Google News y SBERT.
- [ ] Indicar qué método obtiene mayor Precision@10.
- [ ] Indicar qué método obtiene mayor RR.
- [ ] Analizar si los embeddings encuentran documentos semánticamente relacionados aunque no compartan exactamente las mismas palabras.
- [ ] Revisar consultas en las que los métodos producen resultados muy diferentes.
- [ ] Mostrar ejemplos de consultas y documentos recuperados.
- [ ] Explicar las limitaciones de cada representación.

## 11. Guardar resultados

- [ ] Guardar las métricas individuales en `resultados_consultas.csv`.
- [ ] Guardar las métricas medias en `resultados_medios.csv`.
- [ ] Guardar las tablas y gráficos dentro del notebook.
- [ ] Comprobar que los resultados se pueden interpretar sin volver a ejecutar el código.

## 12. Revisar y documentar el notebook

- [ ] Comprobar que todas las celdas aparecen en orden lógico.
- [ ] Añadir una explicación antes de cada bloque importante de código.
- [ ] Incluir ejemplos de documentos, consultas y resultados recuperados.
- [ ] Incluir las tablas de Precision@10 y RR.
- [ ] Incluir, si es útil, gráficos comparativos de las métricas.
- [ ] Explicar cómo se han tratado los documentos sin palabras conocidas.
- [ ] Comprobar que no aparecen credenciales ni datos confidenciales.
- [ ] Revisar ortografía, nombres de variables y unidades.
- [ ] Ejecutar el notebook completo desde el principio.
- [ ] Confirmar que todas las salidas quedan guardadas.
- [ ] Añadir una sección final de conclusiones.

### Ejemplo de conclusiones

> BM25F sobre el campo `body` obtuvo mejores resultados que la búsqueda limitada al título porque utiliza más información textual. Los modelos basados en embeddings recuperaron documentos relacionados semánticamente aunque no compartieran exactamente las palabras de la consulta. SBERT obtuvo el mejor resultado global, mientras que Word2Vec propio estuvo limitado por el tamaño del corpus y por las palabras desconocidas.

## 13. Entregar la práctica

- [ ] Confirmar que el notebook se llama `apelidos_nome_entregable2.ipynb`.
- [ ] Comprobar que el notebook se abre correctamente.
- [ ] Verificar que las tablas, métricas y gráficos tienen sus resultados guardados.
- [ ] Entregar la práctica antes del **20 de octubre a las 23:59**.
- [ ] Preparar la defensa de la práctica ante el profesor.
