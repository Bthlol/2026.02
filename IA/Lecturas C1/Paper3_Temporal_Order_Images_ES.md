# Paper 3 — "Inferring Temporal Order of Images From 3D Structure"
### Resumen detallado en español (no es traducción literal del texto original; es una explicación exhaustiva sección por sección para fines de estudio)

**Autores:** Grant Schindler, Frank Dellaert (Georgia Institute of Technology), Sing Bing Kang (Microsoft Research, Redmond).

---

## 1. Problema abordado

El paper aborda un problema poco convencional: dado un **conjunto de fotografías de una ciudad, totalmente desordenadas** (sin metadatos de fecha confiables, tomadas a lo largo de un periodo que puede abarcar hasta 100 años), determinar automáticamente el **orden temporal** en el que fueron tomadas.

La clave del método es razonar sobre la **persistencia de estructuras visibles** en las imágenes — típicamente, edificios —, ya que un edificio solo puede ser visible en un rango continuo de tiempo (desde que se construye hasta que eventualmente se demuele), y esa propiedad de "continuidad" es la que permite inferir un orden.

El problema completo se formula como un **CSP (Constraint Satisfaction Problem)**. Los autores destacan que este es, hasta donde saben, el **primer método propuesto** en la literatura para resolver este problema específico de ordenamiento temporal de fotografías no ordenadas usando estructura 3D.

---

## 2. Trabajo relacionado

- El primer paso del pipeline del paper (recuperar estructura y cámaras a partir de imágenes desordenadas) sigue el mismo camino conceptual que el trabajo de Snavely et al., conocido como **"Photo Tourism"**, que también trabaja con colecciones grandes de fotos no ordenadas de un mismo lugar.

- Se menciona un trabajo relacionado de Ge y D'Zmura sobre **SFM (structure from motion) variante en el tiempo**: la diferencia importante es que ese trabajo previo usa **secuencias de imágenes ya ordenadas** de objetos en movimiento (por ejemplo, un video), mientras que este paper trabaja específicamente con una colección de imágenes **completamente desordenada**, tanto en el sentido espacial (no se sabe desde dónde se tomó cada foto) como en el sentido temporal (no se sabe cuándo se tomó cada una) — ambos aspectos deben inferirse.

- Sobre el razonamiento temporal en general, se menciona una genealogía de ideas:
  - El primer trabajo relevante sobre razonamiento temporal formal es el del **álgebra de intervalos de Allen** (1983), que formaliza relaciones lógicas entre intervalos de tiempo (por ejemplo, "antes de", "durante", "se superpone con", etc.).
  - Posteriormente, Dechter, Meiri y Pearl proponen las **redes de restricciones temporales**, que formulan explícitamente este tipo de problemas como un CSP general, siendo aplicadas históricamente sobre todo en problemas de **planificación/scheduling de tareas**.
  - Más adelante, otros trabajos introducen la noción de **incertidumbre** en este tipo de modelos, relajando el requisito estricto de que todas las restricciones deban satisfacerse simultáneamente (permitiendo soluciones "aproximadas" cuando no existe una solución exacta).

---

## 3. Pipeline completo del método (Figura 2)

El método propuesto se organiza en 5 etapas secuenciales:

1. **Detección y correspondencia de características (features)** entre las distintas imágenes — en este trabajo, este paso se realiza de forma **manual**.
2. **SFM (Structure From Motion)** — este paso sí es **automático**, y a partir de las correspondencias de features recupera tanto la **nube de puntos 3D** de la escena como la **pose (posición y orientación) de cada cámara** que tomó cada foto.
3. **Razonamiento de visibilidad** — este es uno de los dos focos centrales y originales del paper.
4. **Ordenamiento temporal mediante CSP** — el segundo foco central del paper.
5. **Visualización de un modelo 4D** (un modelo 3D que además varía en el tiempo, combinando las 3 dimensiones espaciales con la dimensión temporal recién inferida).

---

## 4. Razonamiento de visibilidad (Sección 4)

### 4.1 Clasificación de visibilidad

Dado que ya se conocen las matrices de proyección P₁, P₂, ..., Pₙ de cada una de las n cámaras (obtenidas en la etapa de SFM), cada punto 3D individual Xᵢ (de un total de m puntos reconstruidos) puede clasificarse, para cada imagen Iⱼ, en una de estas categorías:

- **Observed (observado)**: existe efectivamente una medición uᵢⱼ (una correspondencia 2D detectada) para el punto Xᵢ dentro de la imagen Iⱼ.
- **Out of view (fuera de campo)**: al proyectar el punto 3D usando la cámara correspondiente (xᵢⱼ = Pⱼ · Xᵢ), el resultado cae **fuera** de los límites del campo de visión de esa cámara (definidos simplemente por el ancho y el alto en píxeles de la imagen).
- **Missing (faltante) u Occluded (ocluido)**: la proyección del punto sí cae **dentro** del campo de visión de la cámara, pero **no existe** una medición correspondiente en esa imagen. Este es el caso ambiguo: hay que determinar, mediante un análisis adicional, si el punto está realmente ausente de la escena en ese momento (missing) o si simplemente está tapado por otro objeto (occluded).

### 4.2 Razonamiento sobre oclusión

Se parte del supuesto de que los puntos 3D reconstruidos corresponden a un **muestreo disperso** (no denso) de la superficie de estructuras sólidas (edificios). El método está inspirado en el trabajo de Faugeras et al. y en la técnica de triangulación "image-consistent" de Morris y Kanade.

El procedimiento concreto es el siguiente:
1. Para cada imagen Iⱼ, se calcula la **triangulación de Delaunay** de todos los puntos que fueron medidos en esa imagen.
2. Se testean las caras (triángulos) resultantes de esa triangulación contra **todos** los puntos que fueron observados en **todas** las demás imágenes, buscando identificar cuáles triángulos actúan consistentemente como **oclusores** — es decir, triángulos que, cada vez que deberían haber bloqueado la visión de una medición (según la geometría), efectivamente la bloquearon (nunca fallaron en hacerlo).
3. Para decidir si un punto específico Xᵢ está missing u occluded en una imagen específica Iⱼ: se traza un **segmento de línea** desde el centro de proyección Oⱼ de la cámara Cⱼ hasta el punto 3D Xᵢ en cuestión.
   - Si ese segmento **intersecta** alguno de los triángulos previamente identificados como oclusores, entonces el punto se clasifica como **occluded** (está oculto por una estructura sólida que sí existía en ese momento, simplemente no se ve porque algo la tapa).
   - Si el segmento **no intersecta** ningún triángulo oclusor, el punto se clasifica como **missing**, lo cual se interpreta como evidencia de que la estructura correspondiente a ese punto **todavía no existía** (o ya no existía) cuando se tomó esa fotografía.

### 4.3 Construcción de la matriz de visibilidad V

Toda esta información se condensa en una **matriz de visibilidad V**, de tamaño **m × n** (m = cantidad de puntos 3D, n = cantidad de imágenes), donde cada entrada v_ij se define como:

- **v_ij = +1** si el punto Xᵢ fue observado en la imagen Iⱼ.
- **v_ij = −1** si el punto Xᵢ está clasificado como missing en la imagen Iⱼ.
- **v_ij = 0** si el punto Xᵢ está out of view (fuera de campo) u occluded (ocluido) en la imagen Iⱼ.

Esta matriz V es el insumo central que alimenta la siguiente etapa del método (el CSP de ordenamiento).

---

## 5. Formulación del CSP y algoritmo de resolución (Sección 5)

### Suposiciones del modelo

- Cada punto 3D Xᵢ está asociado a algún edificio o estructura física de la escena.
- Los edificios se construyen en algún momento T_A, existen durante un periodo de tiempo finito, y eventualmente pueden ser demolidos en un momento T_B, potencialmente para dar paso a una nueva estructura distinta en ese mismo lugar.
- Se asume que los edificios **nunca se demuelen y luego se reemplazan por una réplica idéntica** de sí mismos (esta suposición es importante porque, si ocurriera, rompería la lógica de "continuidad" en la que se basa el modelo).

### La restricción central del modelo (la única restricción formal)

La idea clave, formalizada como restricción del CSP, es la siguiente: en **cualquier fila** de la matriz V (es decir, mirando la historia de visibilidad de un único punto Xᵢ a través de todas las imágenes, una vez que esas imágenes están ordenadas correctamente), **un valor −1 (missing) no puede aparecer entre dos valores +1 (observed)**.

Dicho de otra forma: nunca deberíamos observar, en el orden temporal correcto, que una estructura **aparece** (+1), luego **desaparece por completo, no solo ocluida** (−1), y luego **vuelve a aparecer** (+1) más adelante — porque eso violaría la suposición de que las estructuras persisten de forma continua una vez construidas y no "reaparecen" tras haber sido genuinamente ausentes. (Nótese que esta restricción no prohíbe que un punto esté momentáneamente oculto —valor 0— entre dos valores +1, porque la oclusión no implica que la estructura haya dejado de existir).

Un ordenamiento de las columnas (imágenes) de la matriz V se considera **válido** si, para **todas** las filas simultáneamente, se cumple esta restricción de no tener un −1 "atrapado" entre dos +1.

### La tarea del CSP

La tarea a resolver consiste en encontrar un **reordenamiento de las columnas** de la matriz V (es decir, encontrar el orden correcto de las imágenes) tal que el patrón de visibilidad de **cada fila** (cada punto 3D) sea consistente con la restricción recién descrita. Ese reordenamiento de columnas que resulta válido **es, precisamente, el orden temporal inferido** de las fotografías.

### Algoritmos usados para resolver el CSP

- **Backtracking (búsqueda exhaustiva en profundidad)**: el algoritmo exacto más directo. Se va asignando una imagen a la posición 1, luego otra imagen a la posición 2, y así sucesivamente, **podando** (descartando tempranamente) cualquier rama de búsqueda que ya viole alguna restricción antes de completarse. El problema de este enfoque es que resulta **computacionalmente intratable** para colecciones grandes de imágenes, porque el tamaño total del espacio de soluciones posibles es **n! (n factorial)**, donde n es la cantidad de imágenes — un crecimiento extremadamente rápido.

- **Local Search (búsqueda local) — Sección 5.1**: para superar la limitación anterior, se propone un algoritmo de búsqueda local. Funciona así:
  1. Se parte de un **orden aleatorio** inicial de las columnas (imágenes).
  2. En cada paso, se evalúan **movimientos locales** posibles: concretamente, **intercambiar la posición de dos imágenes** (o incluso de dos *grupos* de imágenes/columnas), buscando siempre el movimiento que logre **reducir** el número total de restricciones violadas respecto al estado actual.
  3. Si en algún punto **no existe ningún movimiento** disponible que mejore la situación actual (un mínimo local), el algoritmo simplemente **reinicia** el proceso completo desde un nuevo orden aleatorio.
  - Esta técnica de búsqueda local está inspirada explícitamente en un trabajo de Sosic y Gu, que aplicaron una idea similar al clásico problema de las **n-reinas**, logrando resolver instancias de hasta **3 millones de reinas en menos de 60 segundos** — una demostración de que la búsqueda local puede escalar extraordinariamente bien en ciertos problemas de tipo combinatorio/permutación.

### 5.2 Propiedades de las soluciones

- **Puede haber múltiples soluciones válidas simultáneamente**. Esto ocurre cuando existen **r "eras"** distintas (con r menor que n, la cantidad de imágenes) durante las cuales coexiste exactamente la misma combinación de estructuras visibles — en ese caso, las imágenes que pertenecen a una misma "era" pueden **intercambiarse libremente entre sí** sin que eso viole ninguna restricción, porque no hay ninguna estructura que distinga el orden relativo dentro de esa era.
- Adicionalmente, existe siempre una **segunda clase completa de soluciones**: el **orden temporal invertido**. Cualquier ordenamiento que sea válido, si se invierte por completo (se lee de atrás para adelante), **también** resulta ser una solución válida según las restricciones definidas (porque la restricción de "no −1 entre dos +1" es simétrica respecto a la dirección del tiempo). Este problema de ambigüedad direccional se resuelve de forma pragmática **fijando arbitrariamente** que cierta imagen específica debe ubicarse en la primera mitad del ordenamiento final — un truco conceptualmente análogo a lo que se hace en SFM tradicional, donde se fija arbitrariamente una cámara en el origen del sistema de coordenadas para eliminar la ambigüedad de posición global.

### 5.3 Manejo de incertidumbre

Un punto importante del método es que los CSP permiten manejar de forma **implícita** los errores de clasificación (por ejemplo, un punto que en realidad debería estar "observed" pero fue mal clasificado como "missing"), simplemente **relajando** el requisito de que **todas** las restricciones deban satisfacerse simultáneamente. Para esto, se modifica el algoritmo de búsqueda local para que, en lugar de buscar exclusivamente una solución perfecta, devuelva el mejor ordenamiento encontrado tras una cantidad fija de iteraciones de búsqueda — es decir, el orden que **satisface la mayor cantidad de restricciones posible** (aunque no logre satisfacerlas todas). Esto sacrifica la garantía formal de validez absoluta, pero a cambio gana **aplicabilidad práctica** frente a datos reales, que inevitablemente contienen errores.

Se enumeran explícitamente los distintos **tipos de errores de clasificación** que pueden ocurrir en la práctica:
- Puntos que en realidad deberían haberse observado, pero terminan clasificados como missing debido a un **fallo en la detección de features** (por ejemplo, por regiones dañadas o de mala calidad en fotografías históricas antiguas).
- Puntos que están ocluidos por **objetos que no fueron modelados** en la reconstrucción 3D (por ejemplo, árboles, niebla, u otros elementos que no forman parte de la nube de puntos), y que por eso terminan etiquetados incorrectamente como missing en lugar de occluded.
- Puntos que **realmente están ocluidos**, pero que el algoritmo de detección de oclusión falla en identificar correctamente como tales (debido a pequeños errores en la estimación geométrica de SFM), y que por eso terminan también etiquetados incorrectamente como missing.
- Puntos que **realmente están missing** (la estructura genuinamente no existía aún o ya no existía), pero que son explicados incorrectamente como occluded por el algoritmo.

### 5.4 Segmentación de estructuras

Una vez que las columnas (imágenes) de la matriz V ya fueron reordenadas correctamente según el CSP, se procede además a reordenar las **filas** de esa misma matriz, agrupando juntos aquellos puntos 3D que comparten exactamente las mismas fechas inferidas de aparición (T_A) y desaparición (T_B). Esta agrupación tiene un efecto directo: **segmenta** la nube de puntos completa en estructuras individuales distintas, aprovechando el hecho de que múltiples puntos 3D suelen provenir físicamente de un mismo edificio (y por lo tanto comparten su ciclo de vida completo).

Sobre cada uno de estos grupos de puntos ya segmentados, se calcula el **casco convexo 3D (3D convex hull)** correspondiente, obteniendo así una geometría sólida aproximada para cada estructura individual. Esa geometría luego se **textura**, proyectando los triángulos resultantes sobre cada una de las imágenes disponibles para obtener su apariencia visual.

---

## 6. Experimentos y resultados (Sección 6)

El dataset utilizado consiste en fotografías de una ciudad, recolectadas a lo largo de un periodo que va desde **1897 hasta 2006**. La detección y correspondencia de características se hizo de forma manual; el resto del pipeline (SFM en adelante) es automático.

### Experimento 1
- 6 imágenes, 56 puntos 3D reconstruidos (mostrado en la Figura 7 del paper original).
- Las fotos fueron elegidas deliberadamente porque ofrecen vistas particularmente claras de la escena, sin puntos mal clasificados — por lo tanto, se garantiza que existe una **solución exacta** al problema.
- Se usó búsqueda exhaustiva por **backtracking**. De las **6! = 720** ordenaciones posibles en total, se encontró que exactamente **24 de ellas satisfacen todas las restricciones** simultáneamente. Estas 24 soluciones resultan ser, en esencia, pequeñas variaciones de un único orden "base": las imágenes 1 y 2 pueden intercambiarse libremente entre sí sin violar restricciones, las imágenes 4, 5 y 6 también pueden intercambiarse libremente entre sí, y además toda la secuencia completa puede invertirse (por la ambigüedad direccional mencionada antes).
- El tiempo total de búsqueda fue de **menos de 1 segundo**.

### Experimento 2
- 20 imágenes, 92 puntos 3D reconstruidos (Figura 8 del paper original).
- Este conjunto sí contiene puntos mal clasificados, debido principalmente a oclusiones causadas por árboles y por edificios que no fueron modelados explícitamente en la reconstrucción 3D.
- Dado que no se espera encontrar una solución exacta perfecta, se recurre al algoritmo de **local search**, ejecutando **1000 iteraciones** en busca del orden que viole la menor cantidad posible de restricciones.
- El tamaño del espacio de soluciones en este caso es de **20! ≈ 2.4 × 10^18** — completamente inabordable mediante backtracking exhaustivo, lo que justifica el uso de local search.
- El mejor orden encontrado por el algoritmo termina violando restricciones en **15 de las 92 filas** totales de la matriz de visibilidad (es decir, la gran mayoría de los puntos, 77 de 92, sí quedan perfectamente consistentes con el orden hallado).

### Experimento 3 (sintético)
- Se genera una escena completamente **sintética** (no real), con **484 puntos 3D** distribuidos de forma aleatoria en el espacio, y **30 cámaras** dispuestas formando un círculo alrededor de la escena.
- A cada punto 3D se le asigna aleatoriamente una fecha de aparición y una fecha de desaparición; cada una de las 30 cámaras "captura" su fotografía en una única fecha, también asignada de forma aleatoria.
- El espacio de soluciones en este caso es **30! = 2.65 × 10^32**, un número astronómicamente mayor al del experimento 2, por lo que necesariamente se requiere local search.
- El algoritmo logra encontrar una solución que **no viola ninguna restricción** (es decir, una solución perfecta) en apenas **26 movimientos locales** a partir de la inicialización aleatoria, completando la búsqueda en **menos de un minuto** de cómputo total, y sin necesidad de reiniciar el proceso de búsqueda ni una sola vez — esto se explica porque, al ser un experimento sintético controlado, **ningún punto está mal clasificado**, lo que hace que el problema sea mucho "más fácil" de resolver que el caso real del Experimento 2.

### Modelo 3D variante en el tiempo (4D)
Además de los tres experimentos de ordenamiento anteriores, los autores construyen un **modelo 3D que varía en el tiempo** (efectivamente 4D: 3 dimensiones espaciales + tiempo), usando las 6 imágenes del Experimento 1 (mostrado en la Figura 10 del paper original). El resultado permite visualizar la misma escena en **4 momentos temporales distintos**, todos desde el mismo punto de vista virtual, generado de forma completamente automática a partir únicamente de las correspondencias 2D encontradas en las 6 imágenes originalmente desordenadas.

---

## 7. Discusión (Sección 7)

- **Costo computacional**: el costo dominante del algoritmo es calcular, para un ordenamiento candidato dado, cuántas restricciones resultan violadas. Este cálculo **crece linealmente tanto con m** (la cantidad de puntos 3D) **como con n** (la cantidad de imágenes). Además, en cada paso individual del algoritmo de local search, la cantidad de posibles ordenamientos candidatos que se evalúan crece de forma **cuadrática con n** (específicamente, existen (n)(n−1)/2 formas distintas de elegir un par de imágenes para intercambiar en cada paso).

- **Relación inversa entre cómputo necesario y cantidad de soluciones válidas**: los autores observan que la cantidad de cómputo requerida varía **de forma inversa** al número de soluciones válidas que efectivamente existen para una instancia dada del problema. Citan como ejemplo el problema de las n-reinas trabajado por Sosic y Gu, donde el número total de soluciones válidas **crece** junto con el tamaño del tablero — en esos casos, una inicialización puramente aleatoria suele terminar relativamente "cerca" de alguna solución válida, y el problema se resuelve rápidamente. En cambio, cuando existen pocas soluciones válidas o ninguna solución exacta en absoluto (como ocurre en el Experimento 2, con datos reales y ruidosos), se requieren muchísimas más iteraciones de búsqueda para converger a un buen resultado.

- **Sobre la naturaleza abstracta de las fechas inferidas por el método**: es importante remarcar una limitación conceptual central del método — este **no puede inferir fechas absolutas** de calendario para la construcción o demolición de un edificio (por ejemplo, no puede concluir algo como "este edificio existió desde 1902 hasta 1966"). Lo único que el método puede inferir es información **relativa**, del tipo "esta estructura existió desde la Imagen 1 hasta la Imagen 13" (haciendo referencia a la posición dentro del orden ya inferido, no a una fecha de calendario real). Los autores señalan que los seres humanos, en cambio, sí son capaces de inferir fechas absolutas aproximadas, pero lo logran apoyándose en **conocimiento adicional externo** a la imagen misma — por ejemplo, reconociendo edificios históricos conocidos, identificando el estilo de los automóviles visibles en la foto, o la moda/vestimenta de las personas fotografiadas. Esto sugiere, según los autores, que incorporar **técnicas de machine learning** entrenadas para reconocer este tipo de pistas visuales sería el camino natural para, en el futuro, poder asignar fechas absolutas reales (y no solo un orden relativo) a este tipo de colecciones fotográficas.

---

## 8. Conclusión (Sección 8)

Los autores concluyen que los **CSP constituyen un framework poderoso y efectivo** para resolver problemas de ordenamiento temporal dentro del campo de la visión por computadora, y remarcan nuevamente que este trabajo constituye, hasta donde tienen conocimiento, el **primer método propuesto** específicamente para este problema de inferir el orden temporal de una colección de fotografías no ordenadas a partir de estructura 3D.

El **mayor obstáculo** que identifican para lograr un sistema completamente automatizado de punta a punta sigue siendo la variedad de **errores de clasificación** posibles entre las categorías missing/occluded (descritos en la Sección 5.3) — cuantos más puntos terminan mal clasificados en una instancia dada, peor resulta la calidad de la **segmentación de estructuras** final (Sección 5.4), ya que se vuelve más difícil encontrar grupos de puntos que compartan exactamente las mismas fechas inferidas de aparición y desaparición.

Como **trabajo futuro**, se propone específicamente extender el razonamiento de oclusión del método para que sea capaz de manejar correctamente **oclusores que no están modelados explícitamente** dentro de la escena reconstruida — el ejemplo recurrente que se menciona es el caso de los **árboles**, que actualmente generan clasificaciones erróneas al no formar parte de la geometría sólida que el sistema es capaz de razonar.
