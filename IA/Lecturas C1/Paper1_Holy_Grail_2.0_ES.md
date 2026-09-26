# Paper 1 — "Holy Grail 2.0: From Natural Language to Constraint Models"
### Resumen detallado en español (no es traducción literal del texto original; es una explicación exhaustiva sección por sección para fines de estudio)

**Autores:** Dimos Tsouros, Hélène Verhaeghe (KU Leuven), Serdar Kadıoğlu (Fidelity Investments / Brown University), Tias Guns (KU Leuven).
**Tipo de trabajo:** position paper (paper de postura/propuesta, no presenta un sistema totalmente evaluado, sino un framework conceptual con un ejemplo de prueba de concepto).

---

## 1. Introducción y motivación

El punto de partida es una cita clásica de Eugene Freuder (1996), quien describió a la programación por restricciones (CP) como una de las aproximaciones más cercanas al "Santo Grial de la programación": el usuario simplemente **enuncia el problema** y la computadora lo **resuelve**, sin que el usuario deba preocuparse de *cómo* resolverlo.

Los autores argumentan que, aunque la CP ya logra en parte esa promesa (el usuario no necesita programar un algoritmo de búsqueda, solo modelar restricciones), todavía exige que el usuario domine un **lenguaje de modelado formal** (MiniZinc, CPMpy, Essence, etc.) y sepa traducir un problema del mundo real a variables, dominios y restricciones. Eso sigue siendo una barrera de entrada importante para personas sin formación en optimización.

La propuesta del paper es entonces una **"Holy Grail 2.0"**: aprovechar los grandes modelos de lenguaje (LLMs) para que el usuario pueda describir su problema en **lenguaje natural (LN)**, sin conocer terminología de modelado, y que el sistema genere automáticamente el modelo de restricciones formal y lo resuelva. Esto eliminaría la última barrera: ya no haría falta saber *modelar*, solo *describir* el problema como lo haría cualquier persona.

---

## 2. Trabajo relacionado

- **NER4OPT** (Dakle et al.): trabajo que formaliza la subtarea de "reconocimiento de entidades nombradas" (Named Entity Recognition) aplicada específicamente a problemas de optimización. Combina técnicas clásicas de procesamiento de lenguaje (análisis morfológico y gramatical) con LLMs para identificar qué partes de un texto corresponden a variables, parámetros, restricciones, etc.
- **Sistema de Ramamonjison et al.**: propone un pipeline de dos etapas. Primero construye una representación intermedia (similar a lo que hace NER4OPT, identificando entidades), y luego usa esa representación para generar la formulación matemática formal. Está basado en el modelo **BART** (un modelo tipo transformer encoder-decoder).
- **Competencia NL4OPT** (organizada en NeurIPS 2022): estableció dos subtareas oficiales: (1) reconocer entidades semánticas dentro del texto del problema, y (2) generar la representación/formulación final a partir de esas entidades. El equipo ganador de la segunda subtarea (Gangwar & Kani) dividió el proceso en dos pasos: primero encontrar las relaciones entre las entidades ya identificadas, y luego formular el problema; también se apoyó en BART.
- **Almonacid**: es el primer trabajo, según los autores, enfocado específicamente en generar modelos de **programación por restricciones (CP)** —y no solo de programación lineal (LP), que era lo predominante hasta entonces—. Traduce directamente a **MiniZinc** en un solo paso (sin subtareas intermedias explícitas) y agrega un mecanismo automático que intenta **arreglar errores de compilación** del código generado, ejecutándolo, detectando fallos, y pidiéndole al LLM que los corrija, en un ciclo iterativo.

Un punto que los autores destacan como diferenciador de su propuesta: casi todo el trabajo previo (NL4OPT, Ramamonjison, etc.) apunta a generar modelos de **programación lineal (LP)**, que es un formalismo más restringido. Este paper apunta explícitamente a **programación por restricciones (CP)**, que permite expresar una gama mucho más amplia de problemas combinatorios (no solo funciones y restricciones lineales).

---

## 3. El framework propuesto: de LN a modelo CP (Figura 1)

El pipeline se organiza en **4 subtareas centrales** más **2 subtareas adicionales** que sirven para "cerrar el ciclo" con el usuario (hacer el sistema iterativo e interactivo, no de una sola pasada).

### Subtareas centrales

1. **NER4OPT (reconocimiento de entidades)**: dado el texto en lenguaje natural, se deben extraer automáticamente las entidades semánticas relevantes: qué son los **parámetros** (valores fijos dados en el problema), qué son las **variables** (las incógnitas a determinar), cuáles son sus **dominios** (los valores posibles que pueden tomar), cuáles son las **restricciones** expresadas textualmente, y cuál es el **objetivo** (qué se busca maximizar o minimizar, si aplica).

   Los autores explican por qué esta tarea es más difícil que el NER "clásico" (el que se usa, por ejemplo, para detectar nombres de personas o lugares en un texto):
   - **Dependencia multi-oración**: una misma entidad (por ejemplo, una variable) puede mencionarse en una oración y su restricción asociada aparecer varias oraciones después; hay que mantener el contexto a través de todo el texto.
   - **Alta ambigüedad**: una misma palabra o frase puede referirse a un parámetro en un contexto y a una variable en otro, dependiendo de cómo esté redactado el problema.
   - **Régimen de pocos datos (low-resource)**: no existen grandes corpus anotados de "problemas de optimización en lenguaje natural con sus entidades etiquetadas", y anotar manualmente este tipo de datos es costoso porque requiere conocimiento experto tanto de lenguaje como de modelado matemático.
   - **Incertidumbre aleatoria inherente**: incluso para un experto humano, a veces la frase del problema es ambigua por naturaleza (no es un problema de falta de información del modelo, sino que el enunciado mismo admite más de una interpretación razonable).

2. **REL (extracción de relaciones)**: una vez identificadas las entidades, hay que determinar **cómo se relacionan entre sí**. Por ejemplo: qué variables están dentro del "scope" (alcance) de cada restricción particular, cómo se vinculan ciertos valores numéricos del texto con su tipo de parámetro correspondiente (por ejemplo, saber que "10 kg" corresponde al parámetro de "capacidad máxima"), determinar el dominio real de cada variable, e identificar exactamente el alcance/scope de cada restricción (a qué variables aplica cada restricción textual). Esta subtarea está inspirada en el primer paso del sistema ganador de Gangwar & Kani en NL4OPT.

3. **Formulation (formulación)**: con la lista de entidades ya etiquetadas y sus relaciones ya identificadas, se debe construir la **formulación formal** del problema como un problema de restricciones: definir explícitamente el conjunto de variables, sus dominios, el conjunto de restricciones matemáticas/lógicas, y la función objetivo (si el problema la tiene).

4. **Translation (traducción a código)**: la formulación formal (que hasta este punto es más bien conceptual/matemática) debe traducirse a **código ejecutable**, siguiendo la sintaxis específica del lenguaje de modelado elegido. En este paper, el lenguaje objetivo es **CPMpy** (una librería de modelado de restricciones en Python).

### Subtareas para cerrar el ciclo (loop)

5. **Fixing the output (corrección del código generado)**: una vez generado el código, se debe **compilar y/o ejecutar**. Si hay errores (de sintaxis, de tipos, etc.), el sistema debe detectarlos automáticamente y usar al LLM para corregirlos, repitiendo este ciclo de "compilar → detectar error → corregir" hasta obtener código libre de errores. Esta idea está inspirada directamente en el mecanismo de Almonacid, y los autores citan evidencia de que los LLMs son particularmente efectivos corrigiendo bugs de código cuando se les da el mensaje de error como contexto.

6. **Refining the model (refinamiento del modelo con el usuario)**: una vez que el modelo compila y se ejecuta correctamente, se le presenta al usuario el modelo final junto con la(s) solución(es) encontradas. Si el problema resulta **insatisfacible** (no tiene solución), el usuario puede solicitar un **MUS (Minimal Unsatisfiable Subset / Subconjunto Insatisfacible Mínimo)** — es decir, el subconjunto más pequeño posible de restricciones que, tomadas juntas, hacen que el problema no tenga solución — y/o pedir explicaciones sobre por qué no hay solución. A partir de esta retroalimentación, el usuario puede interactuar con el sistema para refinar/ajustar el modelo (agregar, quitar o modificar restricciones) de forma iterativa.

---

## 4. Técnicas de prompt engineering discutidas (Sección 3)

El paper repasa varias técnicas de ingeniería de prompts que son relevantes para implementar las subtareas anteriores:

- **Roles and goals (roles y objetivos)**: consiste en fijar, mediante el "system prompt" (el mensaje de configuración inicial del LLM), un rol específico para que el modelo actúe como un experto en la tarea. Por ejemplo: *"Asume que eres un experto en optimización combinatoria..."*. Esto orienta las respuestas del modelo hacia el vocabulario y el estilo de razonamiento esperado.

- **Few-shot learning (aprendizaje con pocos ejemplos)**: se incluyen en el prompt algunos ejemplos resueltos de la tarea (input → output deseado). Además de ayudar a que el modelo entienda mejor la tarea en sí, sirve para **especificar el formato exacto de salida** que se espera (por ejemplo, el formato en que deben listarse las entidades extraídas).

- **Chain-of-thought / CoT (cadena de pensamiento)**: técnica que consiste en pedirle al modelo que descomponga un problema de varios pasos en **pasos intermedios explícitos** de razonamiento, en lugar de saltar directo a la respuesta final. El paper menciona que la variante "zero-shot CoT" (sin ejemplos, simplemente instruyendo al modelo a "pensar paso a paso") es particularmente eficiente en tareas de tipo simbólico, como lo es la formulación de modelos matemáticos.

- **Tree of Thoughts (árbol de pensamientos)**: en vez de seguir una única cadena lineal de razonamiento, se explora el espacio de posibles soluciones/razonamientos como un **árbol**, permitiendo retroceder (backtracking) cuando una rama de razonamiento no lleva a buen puerto, generando así múltiples salidas alternativas entre las cuales luego se elige la mejor.

- **Plan-and-Solve (planificar y resolver)**: técnica que separa el proceso en dos fases claramente diferenciadas: (1) primero se le pide al modelo que **diseñe un plan**, dividiendo la tarea global en subtareas, sin intentar resolver nada todavía; (2) luego, en una segunda fase, se ejecutan esas subtareas siguiendo el plan ya trazado.

---

## 5. Niveles de abstracción para evaluar el sistema (Sección 4)

Para poder medir qué tan bien funciona el sistema frente a distintos grados de "ayuda explícita" que el usuario da en su enunciado, los autores definen **4 niveles de abstracción**, todos ilustrados usando la misma familia de problema base: una variante del clásico problema de la **mochila (Knapsack)** con 5 ítems.

1. **Nivel 1 (más explícito / baseline)**: el enunciado menciona directamente el **nombre del problema clásico** (por ejemplo, "Knapsack", "TSP" — problema del viajante, "graph coloring" — coloreo de grafos). Además, las variables y restricciones ya están identificadas usando terminología técnica clara (tokens reconocibles). Este es el nivel más fácil y sirve como punto de referencia.

2. **Nivel 2**: se **omite el nombre del problema clásico** (ya no se dice "esto es un Knapsack"), pero el enunciado sigue describiendo explícitamente, con lenguaje técnico, cuáles son las variables, las restricciones y los parámetros.

3. **Nivel 3**: se elimina además el **léxico propio del modelado** — es decir, ya no aparecen palabras como "constraint" (restricción), "variable", "domain" (dominio), etc. El problema se describe en términos más cotidianos, aunque todavía de forma bastante estructurada.

4. **Nivel 4 (más abstracto / más realista)**: es el nivel más difícil y el más cercano a cómo una persona común, sin ningún conocimiento de optimización, describiría el problema de forma completamente natural. En este nivel, los **parámetros pueden quedar implícitos y no numéricos** — es decir, el sistema debe **inferir** valores razonables a partir del contexto, no solo extraerlos literalmente del texto.

### Ejemplo ilustrativo (Example 1 del paper)

Se usa un problema de "armar una maleta para un viaje" con 5 ítems: esquís (7 kg), ropa de abrigo (4 kg), botas (3 kg), un libro (1 kg) y un paraguas (2 kg), con un límite de peso de 10 kg para la maleta.

En el **Nivel 4**, el enunciado no entrega valores numéricos explícitos de "utilidad" o "importancia" de cada ítem — simplemente los describe de forma natural (por ejemplo, mencionando que uno quiere llevar cosas para esquiar, pero también no quiere pasar frío, etc.). El sistema debe **inferir** qué tan importante es cada ítem. En el ejemplo mostrado, el sistema terminó infiriendo los siguientes pesos de utilidad: w = [1, 2, 3, 4, 2] para esquís, ropa de abrigo, botas, libro y paraguas respectivamente — priorizando, por ejemplo, el libro por sobre los esquís, una inferencia razonable dado el contexto del enunciado.

---

## 6. Ejemplo de uso del sistema completo (Sección 5)

Los autores implementan una prueba de concepto usando **GPT-3.5**, aplicando las técnicas de prompt engineering descritas en cada una de las subtareas del pipeline, y generando código final en **CPMpy**.

Se muestran dos casos extremos como demostración:

- **Figura 2 — Nivel 1**: el clásico problema de Knapsack, con n=5 ítems, pesos (weights) = [2, 3, 7, 4, 1], utilidades (utilities) = [2, 3, 1, 2, 3], y límite (limit) = 10.
- **Figura 3 — Nivel 4**: el problema de la maleta de vacaciones descrito arriba, sin valores explícitos de utilidad.

**En ambos casos, el modelo final generado es correcto y el código efectivamente compila y se ejecuta sin errores.**

En particular, para el caso de Nivel 4 (el más difícil), el sistema logra con éxito:
- Extraer correctamente todas las entidades relevantes del texto.
- Conectar correctamente cada variable con su peso (weight) correspondiente.
- Construir correctamente la restricción de tipo "knapsack" (que la suma de pesos de los ítems seleccionados no supere el límite).
- **Inferir valores de utilidad razonables** para poder construir la función objetivo, a pesar de que esos valores no estaban dados explícitamente en el texto original.

---

## 7. Conclusión y trabajo futuro

El paper concluye reafirmando que este framework de 4 (+2) subtareas es un camino prometedor para lograr la "Holy Grail 2.0": permitir que cualquier usuario, sin conocimientos de modelado formal, pueda obtener un modelo de restricciones correcto simplemente describiendo su problema en lenguaje natural.

Como líneas de trabajo futuro se mencionan:
- Explorar el uso de **otros LLMs** además de GPT-3.5 (se menciona específicamente LLaMA).
- Desarrollar **métodos especializados por cada subtarea** individual (en vez de usar un único LLM genérico para todo el pipeline).
- Incorporar más **conocimiento de dominio** mediante distintas estrategias: prompt tuning, fine-tuning, y in-context learning.
- Explorar específicamente **(soft) prompt tuning**, técnica que —según citan— ha demostrado superar en desempeño al few-shot learning en otras tareas.
- Mejorar la **interacción con el usuario** en el ciclo de corrección de errores, de modo de minimizar la cantidad de interacciones necesarias para llegar a un modelo correcto y satisfactorio.

**Referencia clave del paper a recordar:** Freuder, E., *"In pursuit of the holy grail"*, ACM Computing Surveys, 1996 — es el origen de la metáfora del "Santo Grial" que da nombre y motivación central a todo el trabajo.
