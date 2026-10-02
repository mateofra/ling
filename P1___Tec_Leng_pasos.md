# Guía paso a paso - Práctica 1

## 1. Preparar el notebook

- [ ] Crear un notebook llamado `apelidos_nome_entregable1.ipynb`.
- [ ] Añadir una portada con:
  - [ ] Nombre y apellidos.
  - [ ] Asignatura y curso.
  - [ ] Fecha.
  - [ ] Descripción breve de los objetivos.
- [ ] Separar el notebook en celdas de texto y celdas de código.
- [ ] Documentar los resultados para que no sea necesario volver a ejecutar el notebook.

## 2. Elegir las cuentas de Reddit

- [ ] Elegir cuatro cuentas o subreddits con bastante actividad.
- [ ] Comprobar que sus publicaciones estén principalmente en inglés.
- [ ] Formar dos pares de cuentas:
  - [ ] Un par de cuentas similares, por ejemplo, dos deportistas del mismo deporte.
  - [ ] Un par de cuentas diferentes, por ejemplo, Bill Gates y Arnold Schwarzenegger.
- [ ] Explicar en el notebook por qué se ha elegido cada cuenta y por qué los pares son similares o diferentes.

### Ejemplo

- **Par similar:** `r/tennis` y `r/soccer` no serían ideales si se quiere comparar autores concretos; sería mejor escoger dos cuentas de deportistas del mismo deporte con publicaciones suficientes.
- **Par diferente:** una cuenta relacionada con tecnología y otra relacionada con cocina.
- **Justificación posible:** “El primer par trata temas parecidos, por lo que esperamos que sus publicaciones compartan vocabulario y sean más difíciles de clasificar. El segundo par trata temas distintos, por lo que esperamos una clasificación más sencilla”.

> Las cuentas y la justificación anteriores son solo un ejemplo. Comprueba que las cuentas elegidas existen y tienen suficientes publicaciones antes de usarlas.

## 3. Obtener los datos manualmente

**Cambio realizado:** se elimina del notebook el script de descarga automática y
no será necesario configurar OAuth, PRAW ni peticiones JSON. Los datos se
descargarán manualmente antes de comenzar el análisis.

- [ ] Descargar manualmente las publicaciones de las cuatro cuentas o subreddits elegidos.
- [ ] Guardar los datos en un archivo local, preferiblemente `publicaciones.csv` o `publicaciones.json`.
- [ ] Comprobar que cada registro contiene el texto de la publicación, la fecha y la cuenta o subreddit de origen.
- [ ] Mantener una columna `autor` o `subreddit` que permita identificar la clase de cada publicación.
- [ ] Eliminar registros sin texto útil o explicar cómo se han tratado.
- [ ] Anotar en el notebook la fuente de los datos, la fecha de descarga y el número de publicaciones obtenido por cuenta.
- [ ] Colocar el archivo de datos en la misma carpeta que el notebook o indicar correctamente su ruta.

### Ejemplo de estructura del archivo

Un archivo `publicaciones.csv` puede tener esta estructura:

```text
texto,fecha,subreddit,autor
"I really enjoyed this match",2026-09-20,nba,
"Great discussion about the story",2026-09-19,gameofthrones,
```

La descarga y preparación manual de los datos sustituye únicamente al antiguo
paso de extracción. Los pasos de división, vectorización, clasificación y
evaluación siguen siendo los mismos.

## 4. Separar los datos de entrenamiento y test

Para cada cuenta:

- [ ] Ordenar las publicaciones temporalmente.
- [ ] Reservar el último 30 % como conjunto de test.
- [ ] Utilizar el 70 % restante como conjunto de entrenamiento.
- [ ] No utilizar las publicaciones de test durante el entrenamiento.
- [ ] Mostrar en el notebook el número de ejemplos de entrenamiento y test de cada cuenta.

### Ejemplo

Si una cuenta tiene 100 publicaciones:

- Entrenamiento: las primeras 70 publicaciones.
- Test: las últimas 30 publicaciones.

```python
publicaciones = sorted(publicaciones, key=lambda item: item["fecha"])
punto_corte = int(len(publicaciones) * 0.70)

entrenamiento = publicaciones[:punto_corte]
test = publicaciones[punto_corte:]

print(len(entrenamiento), len(test))  # 70 30
```

## 5. Preprocesar y vectorizar el texto

Para cada experimento de clasificación:

- [ ] Limpiar o preprocesar el texto de las publicaciones.
- [ ] Crear un `TfidfVectorizer` usando únicamente los datos de entrenamiento.
- [ ] Ajustar el vectorizador con `fit_transform` sobre el conjunto de entrenamiento.
- [ ] Aplicar el mismo vectorizador al conjunto de test usando `transform`.
- [ ] No volver a ajustar el vectorizador con los datos de test.
- [ ] Construir la matriz de características de entrenamiento.
- [ ] Construir el vector de etiquetas `y`, asignando una etiqueta distinta a cada cuenta.
- [ ] Indicar las dimensiones de la matriz resultante: número de publicaciones y número de palabras.

### Ejemplo

```python
from sklearn.feature_extraction.text import TfidfVectorizer

textos_entrenamiento = [item["texto"] for item in entrenamiento]
textos_test = [item["texto"] for item in test]

vectorizador = TfidfVectorizer(
  lowercase=True,
  stop_words="english",
  min_df=2
)

matriz_entrenamiento = vectorizador.fit_transform(textos_entrenamiento)
matriz_test = vectorizador.transform(textos_test)

print(matriz_entrenamiento.shape)
print(matriz_test.shape)
```

Si hay 140 publicaciones de entrenamiento y 2.500 palabras después de la
vectorización, la matriz de entrenamiento tendrá una forma parecida a
`(140, 2500)`. La matriz de test debe tener las mismas 2.500 columnas.

## 6. Entrenar los clasificadores SVM

Para cada uno de los dos pares de cuentas:

- [ ] Crear varios clasificadores `SVM`.
- [ ] Probar distintos valores del parámetro `C`.
- [ ] Probar distintos tipos de kernel, por ejemplo, `linear` y `rbf`.
- [ ] Entrenar cada clasificador únicamente con el conjunto de entrenamiento.
- [ ] Predecir las etiquetas del conjunto de test.
- [ ] Guardar los resultados de cada combinación de parámetros.

### Ejemplo

```python
from sklearn.svm import SVC

etiquetas_entrenamiento = [0] * len(textos_entrenamiento_a)
etiquetas_entrenamiento += [1] * len(textos_entrenamiento_b)

clasificador = SVC(C=1.0, kernel="linear")
clasificador.fit(matriz_entrenamiento, etiquetas_entrenamiento)

predicciones = clasificador.predict(matriz_test)
```

Para comparar configuraciones:

```python
configuraciones = [
  {"C": 0.1, "kernel": "linear"},
  {"C": 1.0, "kernel": "linear"},
  {"C": 10.0, "kernel": "rbf"},
]
```

## 7. Evaluar la clasificación

Para cada clasificador y cada par:

- [ ] Calcular el valor de `accuracy`.
- [ ] Generar una matriz de confusión.
- [ ] Mostrar los resultados en el notebook.
- [ ] Comparar los valores obtenidos con los distintos valores de `C` y kernels.
- [ ] Identificar la mejor configuración para cada par.
- [ ] Comparar la dificultad de distinguir el par de cuentas similares con la del par de cuentas diferentes.
- [ ] Explicar si los resultados apoyan la hipótesis inicial.
- [ ] Comentar posibles causas de errores o resultados inesperados.

### Ejemplo

```python
from sklearn.metrics import accuracy_score, confusion_matrix

accuracy = accuracy_score(etiquetas_test, predicciones)
matriz = confusion_matrix(etiquetas_test, predicciones)

print(f"Accuracy: {accuracy:.2%}")
print(matriz)
```

Una matriz como esta:

```text
[[24  6]
 [ 4 26]]
```

indica que se clasificaron correctamente 24 ejemplos de la clase 0 y 26 de la
clase 1. El valor de `accuracy` de este ejemplo sería `83,33 %`.

En la conclusión se puede escribir: “El par diferente obtuvo un `accuracy` del
92 %, mientras que el par similar obtuvo un 71 %. Esto sugiere que el vocabulario
compartido entre las cuentas similares dificulta la clasificación”.

## 8. Realizar el análisis de sentimiento con VADER

- [ ] Instalar e importar VADER.
- [ ] Analizar las publicaciones de las cuatro cuentas.
- [ ] Obtener las puntuaciones de sentimiento de cada publicación.
- [ ] Calcular y mostrar estadísticas agregadas por cuenta.
- [ ] Comparar los resultados de sentimiento entre las cuentas.
- [ ] Explicar brevemente las conclusiones obtenidas.

### Ejemplo

```python
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer

analizador = SentimentIntensityAnalyzer()
texto = "I really enjoyed this match!"
puntuaciones = analizador.polarity_scores(texto)
print(puntuaciones)
```

La salida tendrá una estructura parecida a:

```python
{'neg': 0.0, 'neu': 0.455, 'pos': 0.545, 'compound': 0.6696}
```

Para resumir una cuenta, calcula la media de `compound` para todas sus
publicaciones y cuenta cuántas son positivas, neutras o negativas.

## 9. Apartado opcional: modelo BERT

- [ ] Cargar el modelo `nlptown/bert-base-multilingual-uncased-sentiment` desde Hugging Face.
- [ ] Aplicarlo a las mismas publicaciones utilizadas con VADER.
- [ ] Obtener la puntuación numérica de 1 a 5 para cada publicación.
- [ ] Normalizar la escala de BERT para poder compararla con VADER.
- [ ] Comparar los resultados de ambos métodos.
- [ ] Explicar las diferencias observadas.
- [ ] Probar el modelo con un texto escrito en castellano.
- [ ] Explicar si funciona y por qué.

### Ejemplo de comparación de escalas

Una puntuación de BERT de 1 a 5 se puede transformar a una escala aproximada de
`-1` a `1` con:

```python
sentimiento_normalizado = (puntuacion_bert - 3) / 2
```

Por ejemplo:

| Puntuación BERT | Escala normalizada | Interpretación aproximada |
|---:|---:|---|
| 1 | -1,0 | Muy negativo |
| 3 | 0,0 | Neutro |
| 5 | 1,0 | Muy positivo |

Ejemplos de textos de prueba:

```python
textos_prueba = {
  "castellano": "La película me ha gustado mucho.",
  "galego": "A película gustoume moito.",
}
```

Después de obtener las predicciones, explica si el modelo entiende ambos idiomas
y si las valoraciones coinciden con el significado de los textos.
- [ ] Probar el modelo con un texto escrito en gallego.
- [ ] Explicar si funciona y por qué.

## 10. Revisar y documentar el notebook

- [ ] Comprobar que todas las celdas tienen un orden lógico.
- [ ] Añadir texto explicativo antes de cada bloque importante de código.
- [ ] Incluir los resultados, tablas, matrices de confusión y gráficos necesarios.
- [ ] Verificar que no aparecen errores ni datos confidenciales.
- [ ] Ejecutar el notebook completo desde el principio.
- [ ] Confirmar que las salidas quedan guardadas en el archivo.
- [ ] Revisar ortografía, nombres de variables y unidades.
- [ ] Añadir una sección final de conclusiones.

### Ejemplo de conclusión final

> El clasificador distinguió mejor las cuentas del segundo par porque sus temas y
> vocabulario eran más diferentes. El kernel lineal con `C=1.0` obtuvo el mejor
> resultado en este conjunto de test. VADER mostró una valoración media más
> positiva para la primera cuenta, aunque algunas publicaciones irónicas fueron
> clasificadas de forma incorrecta.

## 11. Entregar la práctica

- [ ] Confirmar que el archivo se llama `apelidos_nome_entregable1.ipynb`.
- [ ] Comprobar que el notebook se abre correctamente.
- [ ] Verificar que el archivo contiene los resultados sin necesidad de ejecutarlo.
- [ ] Entregarlo antes del **6 de octubre a las 23:59**.
- [ ] Preparar la defensa de la práctica para la sesión posterior a la entrega.

### Ejemplo de revisión final

Antes de entregar, comprueba que el notebook contiene:

```text
1. Introducción y cuentas elegidas
2. Descarga y descripción de los datos
3. División entre entrenamiento y test
4. Vectorización TF-IDF
5. Clasificadores SVM y comparación de parámetros
6. Accuracy y matrices de confusión
7. Análisis VADER
8. Apartado BERT, si se realiza
9. Conclusiones
```
