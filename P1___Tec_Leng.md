# Práctica 1 - Práctica de Clasificación

**Marcos Fernández Pichel**  
Curso 2026-2027 · Grao en Intelixencia Artificial - USC

## 1. Author profiling

As redes sociais representan unha importante fonte de acceso a contidos. Nesta práctica, centrarémonos en recuperar contido da rede social Reddit, que destaca por unha política aberta e por unha maior facilidade de acceso aos datos.

Nesta práctica, experimentaremos coa tarefa de *author profiling*. O perfilado de autores é unha tarefa de clasificación moi común en PLN que persegue determinar automaticamente quen é o/a autor/a dun determinado post. Estes clasificadores poden utilizarse posteriormente para realizar análises demográficas, como a idade ou o xénero, de determinados sectores da poboación nas redes sociais.

### 1.1. Pasos a seguir

- Os datos das catro contas ou subreddits descargaranse manualmente antes de comezar o análise. Gardade as publicacións nun ficheiro local, preferiblemente `publicaciones.csv` ou `publicaciones.json`, conservando o texto, a data e a conta ou subreddit de orixe.

- O primeiro obxectivo da práctica consiste en, para dous pares de personalidades/contas da vosa elección, reunir manualmente o máximo número posible de posts da súa liña temporal. Escollede 4 contas (ou subreddits) con ampla actividade en Reddit e cuxos posts estean en inglés, xa que posteriormente utilizaremos un clasificador de sentimento baseado en regras só para o inglés.

  Ademais, como efectuaremos experimentos de clasificación automática, tratade de escoller:

  - Un par de usuarios/as (ou subreddits) “similares” (por exemplo, dous deportistas profesionais do mesmo deporte, dous irmáns, etc.).
  - Outro par que non teñan nada que ver entre si (por exemplo, Bill Gates, `u/thisisbillgates`, e Arnold Schwarzenegger, `u/GovSchwarzenegger`).

- **Experimento de clasificación de posts.** Para familiarizarvos coas estratexias de clasificación de texto, imos construír un clasificador automático de posts que poida predicir quen escribiu un determinado post.

  Realizaranse dous experimentos de clasificación binaria asociados aos dous pares de contas escollidos e tratarase de verificar a hipótese de que as personalidades máis similares (p. ex., dous deportistas do mesmo deporte) son máis difíciles de distinguir entre si ca o outro par de usuarios/as escollidos/as, que non teñen nada que ver entre si.

  Por exemplo, se escollestes dous usuarios/as que falan sobre ciberseguridade, trataríase de adestrar un clasificador cun subconxunto da colección e logo, para un conxunto de datos de test separado, ver como funciona a clasificación. Por suposto, é de esperar que, en función do parecidas que sexan as contas que escollades, a clasificación sexa máis fácil ou máis difícil.

  **Pasos a seguir:**

  1. Separade o último 30 % dos posts de cada persoa como conxunto de test, que só se utilizará para medir o rendemento do clasificador.
  2. Co 70 % restante hai que adestrar o clasificador. Para iso, preprocesade os posts para extraer unha representación *bag of words* da colección.

     Como se comentou na clase, podedes usar scikit-learn e, en particular, as súas funcionalidades para extraer características a partir de texto:

     - [Feature extraction](https://scikit-learn.org/stable/modules/feature_extraction.html#feature-extraction)
     - [TfidfVectorizer](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfVectorizer.html)

     É importante que a estratexia de vectorización sexa homoxénea nos subconxuntos de adestramento e test. Para iso, será necesario conservar o obxecto *vectorizer* empregado sobre o conxunto de adestramento para logo aplicarlle a mesma vectorización aos posts de test.

     Unha vez vectorizados os posts de adestramento, a representación da colección é unha matriz $D \times T$, onde $D$ é o número de posts de adestramento e $T$ é o número de palabras tras a vectorización.

     Ademais, como para cada post de adestramento sabemos quen é o seu autor, podemos construír un vector binario $y = [0, 0, 1, \ldots]$ (por exemplo, un 0 indicaría a orixe nun usuario concreto e un 1 indicaría a orixe noutro) e utilizar calquera estratexia de clasificación binaria.

  3. Con esta matriz e o vector de etiquetas hai que construír un clasificador utilizando unha máquina de vectores de soporte (*Support Vector Machine*, SVM): [SVM - Classification](https://scikit-learn.org/stable/modules/svm.html#classification).

     Probade distintos axustes (distintos valores do parámetro `C` e distintos *kernels*), construíndo varios clasificadores e comparando a súa efectividade na predición dos posts de test.

     Para predicir a etiqueta dos posts de test basta con aplicarlles a vectorización utilizada no adestramento e invocar o método `predict` do clasificador creado durante o adestramento.

     Dadas as predicións emitidas polo clasificador e as etiquetas reais do test (de quen é realmente cada post de test), amosade:

     - Unha matriz de confusión. [Exemplo de matriz de confusión](http://www2.cs.uregina.ca/~dbd/cs831/notes/confusion_matrix/confusion_matrix.html)
     - O valor de *accuracy* (porcentaxe de decisións correctas do sistema).

     Isto debe facerse para as distintas variantes creadas coa colección de adestramento (distintos valores de `C`, distintos *kernels*, etc.). A descrición dos experimentos que se inclúa no notebook a entregar debe detallar os experimentos asociados aos dous clasificadores construídos e analizar as conclusións extraídas.

- **Realizar unha análise básica de sentimento.** Utilizando VADER, realizade unha análise básica do sentimento dos posts de cada unha das contas consideradas.

  Referencia útil: [VADER (Valence Aware Dictionary and sEntiment Reasoner)](https://github.com/cjhutto/vaderSentiment).

- **OPTATIVO:** Como veremos no Tema 3, existen librarías e repositorios que nos permiten utilizar de maneira moi sinxela modelos do estado da arte para diferentes tarefas. En PLN, os modelos do estado da arte son os *Transformers*.

  A través de Hugging Face, pódense utilizar facilmente Transformers xa adestrados para diferentes tarefas. Proponse que repitades a análise de sentimento anterior, pero utilizando o seguinte modelo de BERT adestrado para clasificación de sentimento:

  [BERT multilingual uncased sentiment](https://huggingface.co/nlptown/bert-base-multilingual-uncased-sentiment)

  Observades algunha diferenza coas saídas de VADER? Tede en conta que a saída é numérica do 1 ao 5; para facer unha comparación xusta das diferenzas teredes que normalizar esta escala.

  Probade agora a pasarlle un texto voso escrito en castelán. Funciona? Por que? E en galego?

## 2. Entregables

Debedes entregar un Python Notebook coa vosa práctica e que se chame `apelidos_nome_entregable1.ipynb`.

É fundamental que o Notebook sexa autoexplicativo en todos os pasos: debe conter celas de texto acompañando as celas con código e explicitar os resultados, sen necesidade de executar as celas de novo.

Recordade: o único entregable obrigatorio é o último, pero é preciso acadar unha nota igual ou superior a 5 nas prácticas para superar a asignatura.

Á parte da entrega do informe, esta práctica avaliarase cunha defensa ante o profesor, avaliada con rúbrica, na sesión posterior á entrega.

## 3. Valoración e data de entrega

Esta práctica ten unha valoración de 2 puntos sobre o total de 10 puntos da parte práctica da materia:

- 1,5 puntos correspóndense coa correcta realización dos apartados obrigatorios.
- 0,5 puntos correspóndense coa correcta realización do apartado optativo.

**Data límite de entrega:** 6 de outubro ás 23:59.
