# Resumen completo — 3 papers de Programación por Restricciones (CP)

---

# PAPER 1 — "Holy Grail 2.0: From Natural Language to Constraint Models"

**Autores:** Dimos Tsouros, Hélène Verhaeghe (KU Leuven), Serdar Kadıoğlu (Fidelity Investments / Brown University), Tias Guns (KU Leuven). *Position paper.*

## Idea central
Cita de Eugene Freuder (1996): *"Constraint programming represents one of the closest approaches computer science has yet made to the Holy Grail of programming: the user states the problem, the computer solves it"*. El paper propone la **"Holy Grail 2.0"**: usar LLMs para ir de una descripción en lenguaje natural (LN) directamente a un modelo de programación por restricciones (CP), eliminando la necesidad de que el usuario conozca el formalismo (MiniZinc, CPMpy, Essence).

## Trabajos previos citados
- **NER4OPT** [Dakle et al., 3]: formaliza el subtask de reconocimiento de entidades para optimización, combinando técnicas clásicas (morfológicas/gramaticales) con LLMs.
- Sistema de [15] (Ramamonjison et al.): dos pasos (representación intermedia tipo NER4OPT → formulación formal), basado en **BART**.
- **Competencia NL4OPT** (NeurIPS 2022): dos tareas (reconocer entidades semánticas + generar representación). El sistema ganador de la segunda subtarea [7] (Gangwar & Kani) dividió el proceso en: encontrar relaciones entre entidades → formular. También usa BART.
- [1] (Almonacid): primer approach enfocado en **CP** (no solo LP), traduciendo a **MiniZinc**, con enfoque de un solo paso + proceso automático de arreglo (fixing) de errores de compilación.
- Los trabajos anteriores se enfocaban mayormente en **programación lineal (LP)**; este paper apunta a **CP**.

## El framework propuesto (Figura 1): 4 subtareas principales + 2 de cierre del loop
1. **NER4OPT**: extraer entidades semánticas (parámetros, variables, dominios, restricciones, objetivo). Difiere del NER clásico por: dependencia multi-oración, alta ambigüedad, régimen de pocos datos con alto costo de anotación, e incertidumbre aleatoria inherente.
2. **REL (relaciones)**: identificar relaciones entre entidades extraídas (qué variables están en el scope de cada constraint, vincular valores con tipos de parámetros, encontrar dominios de variables, identificar scopes de constraints). Inspirado en el primer paso de [7].
3. **Formulation**: formular el problema como constraint problem de manera formal, dado la lista de entidades etiquetadas y su relación.
4. **Translation**: traducir la formulación a código, siguiendo la sintaxis del lenguaje de modelado deseado (en este paper: **CPMpy**).

Pasos adicionales para "cerrar el loop":
5. **Fixing the output**: compilar/ejecutar el código, detectar errores y corregirlos automáticamente en loop hasta tener código libre de bugs (inspirado en [1]; LLMs son efectivos en bug-fixing según [19]).
6. **Refining the model**: presentar el modelo final y solución(es) al usuario; el usuario puede pedir **MUS (Minimal Unsatisfiable Subset)** y/o explicaciones si no hay solución; interacción para refinar el modelo.

## Técnicas de prompt engineering usadas/discutidas (Sección 3)
- **Roles and goals**: fijar el rol del LLM vía system prompt (ej: "Assume you are a combinatorial optimization expert...").
- **Few-shot learning**: dar ejemplos de la tarea; también sirve para especificar el formato de salida requerido.
- **Chain-of-thought (CoT)**: descomponer problemas multi-paso en pasos intermedios; el zero-shot CoT es eficiente en tareas simbólicas.
- **Tree of Thoughts**: explora el espacio de soluciones como un árbol, con backtracking, generando salidas alternativas para elegir la mejor.
- **Plan-and-Solve**: dos pasos separados — (1) diseñar un plan dividiendo la tarea sin resolverla, (2) ejecutar las subtareas según el plan.

## Niveles de abstracción (Sección 4)
Para evaluar el sistema, se definieron 4 niveles usando el mismo problema base (ejemplo de mochila/Knapsack con 5 ítems):
1. **Nivel 1**: nombre del problema explícito (ej. "Knapsack", "TSP", "graph coloring"), variables/restricciones ya identificadas con tokens. Es el baseline.
2. **Nivel 2**: se omite el nombre del problema, pero sigue describiendo explícitamente variables, restricciones y parámetros.
3. **Nivel 3**: se elimina el léxico conocido de modelado (palabras como "constraint", "variable", "domain").
4. **Nivel 4**: el más abstracto — parámetros implícitos, no numéricos; el más cercano a cómo lo describiría un humano sin conocimiento de optimización.

**Ejemplo usado (Example 1)**: problema de maleta/vacaciones con 5 ítems (esquí=7kg, ropa abrigada=4kg, botas=3kg, libro=1kg, paraguas=2kg; límite=10kg). En L4 no se dan utilidades numéricas explícitas — el LLM debe **inferir** valores de importancia (el sistema infirió w = [1,2,3,4,2] para ski, ropa, botas, libro, paraguas respectivamente).

## Ejemplo de uso del sistema (Sección 5)
Usa **GPT-3.5** con prompt engineering en cada subtarea, traduciendo a **CPMpy**. Se muestran los casos extremos: Figura 2 (nivel 1, knapsack clásico con n=5, weights=[2,3,7,4,1], utilities=[2,3,1,2,3], limit=10) y Figura 3 (nivel 4, maleta de vacaciones). En **ambos casos el modelo resultante es correcto y el código funciona**. En el caso L4 el sistema logra: extraer entidades correctamente, conectar variables con pesos, construir correctamente la restricción tipo knapsack, e **inferir valores de utilidad** para construir la función objetivo.

## Conclusión y trabajo futuro
Explorar otros LLMs (LLaMA), métodos especializados por subtarea, más conocimiento de dominio en prompt tuning/fine-tuning/in-context learning, (soft) prompt tuning (se cita que supera al few-shot learning), y mejorar la interacción con el usuario para minimizar interacciones necesarias para corregir errores.

**Referencia clave a recordar:** Freuder, "In pursuit of the holy grail", ACM Computing Surveys, 1996 — origen de la cita "Holy Grail".

---

# PAPER 2 — "GHOST: A Combinatorial Optimization Framework for Real-Time Problems"

**Autores:** Florian Richoux, Alberto Uriarte, Jean-François Baffier. IEEE Trans. on Computational Intelligence and AI in Games, Vol. 8, No. 4, Dic. 2016.

## Objetivo
Presentar GHOST, un framework de optimización combinatoria en **C++11** (licencia GNU GPL v3) para que desarrolladores de IA en juegos RTS (real-time strategy) modelen y resuelvan cualquier problema formulado como **CSP/COP**, con resultados en decenas de milisegundos.

## Contexto: familias de problemas RTS (Ontañón et al.)
Tres niveles de abstracción, de mayor a menor:
1. **Strategy**: decisión de alto nivel (todo el conjunto de unidades/edificios).
2. **Tactics**: implementación de la estrategia (posicionamiento, movimiento, timing de grupos de unidades).
3. **Reactive control**: implementación de tácticas (moverse, apuntar, disparar, huir/kiting) — foco en una unidad específica.

GHOST se prueba con **un problema por cada familia**, usando como testbed **StarCraft: Brood War** (juego con 10 millones de copias vendidas, popular en Corea del Sur; velocidad "normal"=14.96 frames lógicos/segundo, "fastest"=23.81 fps, 1 frame = 42ms en modo más rápido; bots deben calcular en <55ms/frame o pierden si superan ese límite en ≥200 frames).

## Definiciones formales CSP/COP
- **CSP** = tupla **(V, D, C)**: V = conjunto de variables, D = dominio (conjunto de valores), C = conjunto de restricciones. Una restricción c ∈ C es un predicado k-ario c: V^k → {⊤,⊥}.
- **COP** = tupla **(V, D, C, f)**, igual que CSP más una función objetivo f a minimizar/maximizar.
- Diferencia CSP vs COP: en CSP todas las soluciones son equivalentes (basta hallar una, ej. Sudoku); en COP algunas soluciones son mejores que otras.
- Dos familias de algoritmos: (1) **búsqueda en árbol/completos** (backtracking, forward checking — exploran todo el espacio); (2) **metaheurísticas/incompletos** (movimientos locales, más adecuados para problemas de tamaño industrial en tiempo razonable, pero no pueden probar optimalidad ni insatisfacibilidad).

## Algoritmo interno de GHOST: Adaptive Search (Codognet & Diaz)
- Se eligió por ser, según los autores, una de las metaheurísticas más rápidas conocidas.
- GHOST es **mono-objetivo** (decisión pragmática: los solvers multiobjetivo son más lentos y difíciles de implementar). Para maximizar f, se usa 1/f o −f.
- **Arquitectura (Fig. 1)**: dos loops anidados. Loop externo de **optimización** (azul) contiene el loop interno de **satisfacción** (rojo).
  - Parámetro **x** (obligatorio) = timeout de satisfacción en μs.
  - Parámetro **y** (opcional) = timeout de optimización total en μs; si no se da, y = 10x.
  - El loop de optimización repite n veces el loop de satisfacción, recibe m≤n soluciones válidas, aplica postprocess de satisfacción, calcula el costo con la función objetivo, guarda la de menor costo.
  - Se compara con un tipo de muestreo Monte Carlo (no es Monte Carlo Tree Search).
  - Tercer parámetro: **tabú list** de longitud |V|−1 (óptimo encontrado empíricamente; no requiere tuning).
- **5 clases principales en C++**: Variable, Domain, Constraint, Objective, Solver.
- Dos tipos de usuario objetivo: **casual user** (solo instancia el problema ya codificado, resuelve con `solve`, en 5 líneas de C++) y **developer user** (implementa nuevos problemas heredando de las clases base, sin modificar el solver).

## Problema 1 — Reactive Control: Target Selection
- **Modelo CSP**: Variables = grupo de unidades propias. Dominio = grupo de unidades enemigas. Restricción: cada unidad viva lista para disparar debe apuntar a un enemigo vivo dentro de su rango.
- Se demostró (Furtak & Buro) que este problema pertenece a **PSPACE**.
- Se consideran propiedades: tamaño de unidad (small/medium/large) y tipo de daño (concussive/normal/explosive) → **Tabla I: matriz de eficiencia de daño**: Small: 100/100/50%; Medium: 50/100/75%; Large: 25/100/100% (según tamaño del objetivo × tipo de daño).
- **2 funciones objetivo probadas**: MaxDamage (maximizar daño infligido en el frame actual) y MaxKill (maximizar unidades enemigas muertas en el frame actual).
- Setup experimental: 4 líneas de unidades Terran (5 marines; 2 Goliaths+2 Vultures; 2 Siege Tanks modo tanque+2 Ghosts; 1 Siege Tank modo siege) en enfrentamiento espejo, simulador propio.
- **Tabla II — resultados (100 simulaciones):**
  - 3ms: MaxDamage → 98 wins, 1 draw, 1 loss (avg. 2.8 unidades vivas, 237.9 HP prom. GHOST; oponente 1.0 unidad, 12.0 HP). MaxKill → 94 wins, 5 draws, 1 loss (3.0 unidades, 250.8 HP; oponente 1.0, 3.0 HP).
  - 5ms: MaxDamage → 99 wins, 1 draw, 0 losses (2.6 unidades, 231.9 HP; oponente 0,0). MaxKill → 96 wins, 4 draws, 0 losses (2.6 unidades, 233.6 HP; oponente 0,0).
- Limitaciones simuladas NO consideradas: curación/reparación/regeneración de escudo, terreno alto/bajo, disparos aéreos con probabilidad de fallo, fuego amigo, splash único de firebat.

## Problema 2 — Tactics: Wall-in
- Objetivo: construir un muro con edificios para cerrar/estrechar la entrada de una base.
- Dos propiedades de edificios: **build size** (w,h en build tiles) y **real size** (w_p,h_p en píxeles, con w_p≤32×w). Un build tile = 32 píxeles.
- **"Significant gap"**: espacio suficiente para que pase un Zergling (16×16 píxeles, la unidad más pequeña de StarCraft).
- Basado en el modelo de Richoux et al. [7] (que a su vez extiende el primer modelo CSP de Certicky [16]).
- **Modelo CSP**: Variables = edificios de la raza del jugador. Dominio = posiciones posibles alrededor del chokepoint. Restricciones: **Overlap** (no solapar entre edificios), **Buildable** (no solapar tiles no-construibles), **NoHoles** (sin huecos del tamaño de un build tile o mayor), **StartingTargetTile** (exactamente un edificio en tile de inicio dado y uno en tile objetivo dado — puede ser el mismo).
- **3 funciones objetivo** (todas a minimizar): **Building** (nº de edificios), **Gap** (nº de gaps significativos), **TechTree** (nivel tecnológico requerido — profundidad en el árbol tecnológico del edificio más avanzado del muro; ej. Command Center=profundidad 0, Barracks=1, Factory=2...).
- **Tabla III** — resultados sobre **48 chokepoints extraídos de 7 mapas de StarCraft**, promedio de 100 corridas por chokepoint, cada llamada de GHOST dura 150ms:
  - Building: satisfaction run 4.05 → optimization run 2.56 (98.04% resuelto).
  - Gap: 1.32 → 0.03 (97.50% resuelto). Con objetivo Gap: 4680 de 4800 muros encontrados (97.50%), de los cuales 4527 son "perfectos" sin gaps significativos (96.73% de los muros hallados son perfectos).
  - TechTree: 1.99 → 1.35 (97.54% resuelto).
  - Comparado con el solver anterior [7]: % de problemas resueltos subió de 95-96% a 97-98%; con objetivo Building, edificios promedio bajó de 2.65 a 2.56; gaps significativos de 0.05 a 0.03; nivel tecnológico de 1.56 a 1.35.
- **Tabla IV** — % de muros encontrados por mapa: Python 100%, HeartbreakRidge 100%, CircuitBreaker 99%, Benzene 99%, Aztec 97%, Andromeda 96%, Fortress 90% (peor resultado — falla ocasionalmente en un chokepoint que solo admite solución con dos edificios de tamaño 3×2).
- **Comparación con Certicky [16] (usa solver Clingo) — Tabla V**, promedio de 20 corridas, limitado a 2 barracks + 4 supply depots:
  - Chokepoint estrecho (65px): GHOST 46.8ms vs Clingo 362.8ms.
  - Chokepoint ancho (250px): GHOST 33.5ms vs Clingo 408.8ms.
  - **GHOST es al menos 7.8 veces más rápido que Clingo.**

## Problema 3 — Strategy: Build Order (BO)
- Un BO plan = serie de acciones con timing específico para lograr una meta (mezcla de edificios/unidades/upgrades/investigación).
- Se modela como **problema de permutación** en CSP (biyección de variables al dominio; cambiar el valor de una variable = intercambiarlo con otra).
- **Modelo CSP**: Variables = todas las acciones necesarias para alcanzar la meta. Dominio = orden de las acciones. Restricción: cada dependencia de una acción α debe ocurrir antes que α (las dependencias pueden ser recursivas, ej: Air Weapons Upgrade nivel 2 requiere nivel 1, que requiere Cybernetics Core, que requiere Gateway).
- Implementación centrada en la raza **Protoss**. Única función objetivo implementada: **minimizar el makespan del BO**.
- Postprocesamiento de optimización (sin postproceso de satisfacción): si el usuario pidió n unidades de tipo U producidas por edificio tipo B, y pidió m<n edificios B, GHOST intenta construir más edificios B para acortar el makespan.
- **Simulador integrado** (a diferencia de target selection, aquí se necesita simular economía/producción sin combate). Se ajustaron parámetros respecto a Churchill & Buro [3] tras analizar replays de pro-gamers coreanos:
  1. Tiempo para ir a construir algo: **74 frames** (vs 96 en [3]).
  2. Tiempo de vuelta a recolectar minerales tras construir: **60** (vs 0 en [3]).
  3. Tiempo de base a parche mineral para empezar a minar: **74** (vs 0 en [3]).
  4. Tiempo de un trabajador para cambiar de mineral a gas: **74** (vs 0 en [3]).
  5. Tasa de recolección de minerales: **0.045 mineral/trabajador/frame** (igual que [3]).
  6. Tasa de recolección de gas: **0.077 gas/trabajador/frame** (vs 0.07 en [3]).
  - El simulador siempre produce trabajadores hasta saturación (**24 trabajadores por base**) y mantiene el suministro para nunca quedar "supply blocked".
- **Validación (Tabla VI)**: se comparó el simulador contra el pro-gamer Protoss coreano "Bisu" en los primeros 1900 frames (80s) — resultados muy similares, con leve ventaja al simulador porque su producción de probes es casi perfecta.
- **Dataset de experimentos**: 3647 BOs en total de replays (Synnaeve, refinado por Glen Robertson): 768 PvP, 2043 PvT, 836 PvZ. Cada llamada a GHOST dura **30ms**, corrido 10 veces por BO.
- **Tabla VII — makespan promedio (humanos vs GHOST), en frames:**
  - Techo 10000 frames — All: humanos 9794, GHOST 9250, %Solved 94.4, ganancia 544 frames. PvP: 9727/9078/95.0/649. PvT: 9861/9378/93.9/483. PvZ: 9692/9097/95.0/595. **Allpro (solo pro-gamers): 9605/8916/96.3/689.**
  - Techo 7800 frames — All: 7726/7332/98.8/394. PvP: 7630/7249/99.3/381. PvT: 7800/7564/98.3/236. PvZ: 7626/6841/99.7/785. **Allpro: 7485/7179/100/306.**
- También se comparó contra 8 replays de pro-gamers top (Protoss: Bisu, BeSt, Violet, Cure; Zerg: Jaedong, sAviOr; Terran: Flash) — **GHOST supera a los pro-gamers Protoss por 689 frames (BOs de 10000 frames) y 306 frames (BOs de 7800 frames)**.
- Comparación con Churchill & Buro [3] (branch and bound): su método computa el 90% de las veces BOs con el mismo makespan que pro-gamers en ~3.735s (para BOs de hasta 249s = 5928 frames) → ratio CPU-tiempo/makespan = 1.5%. GHOST computa en promedio BOs con makespan de 9250 frames en 30ms → en modo más rápido, 9250 frames ≈ 388s → **ratio CPU-tiempo/makespan = 0.007%**.
- Computación: **20ms para satisfacción, 30ms para optimización global** → cabe dentro de un solo frame de StarCraft en modo más rápido.
- Razón de por qué el BO es "más fácil" que el wall-in: se modela como problema de permutación, reduciendo drásticamente la complejidad combinatoria.

## Comparación con solvers estado del arte (Sección VI) — problema de Resource Allocation
- Problema: dado stock fijo de minerales/gas/supply, ¿qué unidades entrenar para maximizar **DPS (damage per second)** total en tierra? Es una instancia del **problema de la mochila multidimensional (multidimensional knapsack)** con 3 dimensiones (una por tipo de recurso). El knapsack normal es NP-completo; la versión multidimensional es aún más dura: **no existe un PTAS eficiente desde 2 dimensiones (salvo P=NP)**.
- Solvers comparados: **Opturion CPX** y **Gecode** (algoritmos completos, deterministas — parsean MiniZinc). Gecode ganó todas las medallas de oro en el MiniZinc Challenge 2008-2012; desde 2013 Opturion CPX es el solver dominante (2 oros+2 bronces en 2013; 4 platas en 2014; 2 oros+1 plata+1 bronce en 2015). También se comparó con **Oscar/CBLS** (metaheurística que también parsea MiniZinc).
- Instancias: **20000 minerales, 14000 gas, 380 supply**, para razas Zerg, Protoss y Terran.
- **Tabla VIII — resultados**:
  - **Zerg**: DPS óptimo=11400.00. Opturion CPX: 200ms. Gecode: 99580ms (99.58s). GHOST (100 runs): 59% de las veces encuentra el óptimo, en 80ms, DPS medio=11387.70 (99.89% del óptimo). Oscar/CBLS (10 runs): DPS medio=11400.00, mejor DPS=11400.00, runtime medio=562ms.
  - **Protoss**: óptimo=4916.38. Opturion CPX: 1620ms (Gecode no encontró solución tras 6h). GHOST: 54% óptimo en 1300ms, DPS medio=4907.70 (99.82% del óptimo). Oscar/CBLS: DPS medio=3445.21 (70.08% del óptimo), mejor DPS=4480.00 (91.12% del óptimo), runtime=1200ms.
  - **Terran**: óptimo=6632.73. Opturion CPX: **3h19min** (!). Gecode: no encontró solución. GHOST: 53% óptimo en 130ms, DPS medio=6619.30 (99.80% del óptimo). Oscar/CBLS: DPS medio=3035.73 (45.77% del óptimo), mejor DPS=5767.55 (86.96% del óptimo), runtime=1390ms.
- Explicación de las diferencias: para Zerg la estrategia óptima es trivial (solo entrenar Zerglings, mejor ratio DPS/costo), para Terran también es trivial (solo Firebats), pero para Protoss se requiere combinar Zealots y Dark Templars. El espacio de búsqueda Terran es mucho más amplio (~1.57×10^20 configuraciones) vs Zerg (~1.55×10^13) y Protoss (~1.33×10^12) — Terran tiene 9 unidades relevantes contra tierra vs 6 en Zerg/Protoss.
- Nota: Oscar/CBLS usa múltiples cores; GHOST es secuencial. Experimentos en Intel i7 quad-core, 4GB RAM, Ubuntu 14.04 64-bit.
- **Conclusión general de esta sección: GHOST supera tanto a solvers completos (Opturion CPX, Gecode) como a la metaheurística Oscar/CBLS, tanto en calidad de solución como en tiempo de ejecución.**

## Conclusión general del paper
Buscar la solución óptima absoluta puede no ser la mejor estrategia para juegos RTS (confirmado por los tiempos de los solvers completos); metaheurísticas rápidas dan una solución "suficientemente óptima" en decenas de milisegundos, y si no es suficientemente buena, se puede re-ejecutar en el siguiente frame. Trabajo futuro: mecanismo de pausa/resume, usar múltiples cores (Adaptive Search es paralelizable — cita [9] con speedups casi lineales sobre 8192 cores), e investigar un nuevo formalismo CSP para manejar incertidumbre eficientemente (el framework actual no está bien adaptado a información incompleta/incierta).

---

# PAPER 3 — "Inferring Temporal Order of Images From 3D Structure"

**Autores:** Grant Schindler, Frank Dellaert (Georgia Institute of Technology), Sing Bing Kang (Microsoft Research, Redmond).

## Problema
Ordenar temporalmente una colección de fotos no ordenadas (de una ciudad, tomadas a lo largo de hasta 100 años) razonando sobre la **persistencia de estructuras visibles** (edificios). Se formula como un **CSP**. Se destaca que es el **primer método conocido** para resolver este problema de ordenamiento.

## Contexto y trabajo relacionado
- El enfoque temprano (SFM — structure from motion) sigue el mismo camino que [11] (Snavely et al., "Photo Tourism").
- Diferencia con trabajo de SFM variante en el tiempo [Ge & D'Zmura, 7]: ese trabajo usa secuencias de imágenes **ordenadas** de objetos en movimiento; este paper trabaja con una colección **no ordenada** (ni espacial ni temporalmente).
- Primer trabajo de razonamiento temporal: álgebra de intervalos de Allen [1] (1983); luego redes de restricciones temporales de Dechter, Meiri, Pearl [3] (formulan el problema como CSP general, usado en task scheduling); incertidumbre introducida después por [4,5,2] relajando el requisito de satisfacción total de restricciones.

## Pipeline completo (Figura 2)
1. Detección y correspondencia de features (**manual** en este trabajo).
2. SFM (**automático**) → recupera puntos 3D y poses de cámara.
3. **Razonamiento de visibilidad** (foco del paper).
4. **Ordenamiento temporal vía CSP** (foco del paper).
5. Visualización de modelo 4D (3D + tiempo).

## 4. Razonamiento de visibilidad

**4.1 Clasificación de visibilidad**: dado proyecciones conocidas P₁..ₙ para cada cámara C₁..ₙ, cada punto 3D Xᵢ se clasifica en cada imagen Iⱼ como:
- **observed** (existe medición uᵢⱼ para Xᵢ en Iⱼ).
- **out of view** (la proyección xᵢⱼ = Pⱼ·Xᵢ cae fuera del campo de visión de la cámara, definido por ancho/alto de la imagen).
- **missing** u **occluded** (proyección dentro del campo de visión pero no hay medición — se requiere distinguir cuál de las dos es).

**4.2 Oclusión**: se asume que los puntos 3D corresponden a un muestreo disperso de la superficie de estructuras sólidas. Método inspirado en [6] (Faugeras et al.) y en la triangulación "image-consistent" de [9] (Morris & Kanade): para cada imagen Iⱼ se calcula la **triangulación de Delaunay** de los puntos medidos, y se testean las caras/triángulos contra todos los puntos observados en todas las imágenes para encontrar **oclusores** (triángulos que nunca fallaron en bloquear una medición que debería haber sido observada). Para determinar si un punto Xᵢ está missing u occluded en Iⱼ: se traza un segmento de línea desde el centro de proyección Oⱼ de la cámara Cⱼ hasta el punto 3D Xᵢ; si el segmento OⱼXᵢ intersecta algún triángulo oclusor, se clasifica **occluded**; si no, se clasifica **missing** (indicando que el punto no existía cuando se tomó la imagen).

**4.3 Matriz de visibilidad V** (tamaño m×n, m=puntos, n=imágenes):
- v_ij = **+1** si Xᵢ es observado en Iⱼ
- v_ij = **−1** si Xᵢ está missing en Iⱼ
- v_ij = **0** si Xᵢ está out of view u occluded en Iⱼ

## 5. Formulación CSP y algoritmo

**Suposiciones del modelo**: cada punto Xᵢ está asociado a un edificio; los edificios se construyen en un momento T_A, existen por un tiempo finito, y pueden ser demolidos en T_B para dar paso a otros; los edificios nunca se demuelen y se reemplazan por una réplica idéntica.

**Restricción formalizada (única restricción del modelo):** en cualquier fila de V, **un valor −1 no puede ocurrir entre dos valores +1** — es decir, nunca debemos observar que una estructura aparece, desaparece y vuelve a aparecer (salvo por oclusión, que se marca como 0, no −1). Los ordenamientos válidos son aquellos que no violan esta restricción para ninguna fila.

**Tarea CSP**: reordenar las **columnas** de V para que el patrón de visibilidad de cada punto (fila) sea consistente con esta restricción → ese reordenamiento de columnas = el orden temporal inferido de las imágenes.

### Algoritmos de resolución
- **Backtracking** (exhaustivo, profundidad primero): asigna imagen a posición 1, luego a posición 2, etc., podando ramas que violan restricciones. Intratable computacionalmente para muchas imágenes porque el espacio de soluciones tiene **n! soluciones** (factorial en el número de imágenes).
- **Local Search** (Sección 5.1): inicia con un orden aleatorio de columnas; en cada paso evalúa movimientos locales — **intercambiar la posición de dos imágenes o de dos grupos de imágenes** (columnas), buscando siempre reducir el número de restricciones violadas. Si no hay movimiento que mejore, se reinicializa con un nuevo orden aleatorio. Técnica inspirada en la aplicación de local search al problema de las n-reinas para **3 millones de reinas en menos de 60 segundos [12]** (Sosic & Gu).

**5.2 Propiedades de las soluciones**: puede haber múltiples soluciones válidas — si hay **r eras** (r<n) en las que coexisten distintas combinaciones de estructuras, hay más de una solución (imágenes de la misma era pueden intercambiarse sin violar restricciones). Además, existe una **segunda clase de soluciones con el tiempo invertido** (cualquier orden válido, invertido, también satisface las restricciones) — se resuelve fijando arbitrariamente que cierta imagen debe estar en la primera mitad del ordenamiento (análogo a fijar una cámara en el origen en SFM).

**5.3 Manejo de incertidumbre**: los CSP pueden manejar implícitamente clasificaciones erróneas relajando el requisito de que se satisfagan TODAS las restricciones — se modifica el algoritmo de local search para que devuelva el orden que **satisface más restricciones que cualquier otro** tras una cantidad fija de búsqueda (ya no se garantiza validez absoluta, pero se gana aplicabilidad al mundo real). Tipos de errores de clasificación posibles:
- Puntos que deberían haberse observado se clasifican como missing (por fallo de detección de features o regiones dañadas de imágenes históricas).
- Puntos ocluidos por objetos no modelados (árboles, niebla) etiquetados falsamente como missing.
- Puntos realmente ocluidos que fallan en ser bloqueados por la geometría de oclusión (errores de estimación SFM), etiquetados falsamente como missing.
- Puntos realmente missing explicados falsamente como occluded.

**5.4 Segmentación de estructura**: una vez reordenadas las columnas, se reordenan también las **filas** de V para agrupar puntos 3D que comparten fechas de aparición T_A y desaparición T_B (esto segmenta la nube de puntos en estructuras distintas, ya que múltiples puntos 3D provienen del mismo edificio físico). Se calculan **cascos convexos 3D (3D convex hulls)** de cada grupo de puntos para obtener geometría sólida aproximada, que luego se textura proyectando triángulos en cada imagen.

## 6. Experimentos y resultados
Dataset: imágenes de una ciudad recolectadas entre **1897 y 2006**. Detección/correspondencia manual; SFM y el resto automático.

1. **Experimento 1**: 6 imágenes, 56 puntos 3D (Figura 7). Fotos elegidas a propósito con vistas claras (sin puntos mal clasificados) → **solución exacta garantizada**. Búsqueda backtracking exhaustiva: de las **6! = 720** ordenaciones posibles, **24 satisfacen todas las restricciones** (todas son variaciones menores del mismo orden: imágenes 1 y 2 intercambiables, 4/5/6 intercambiables, y toda la secuencia puede invertirse). Tiempo de búsqueda: **menos de 1 segundo**.

2. **Experimento 2**: 20 imágenes, 92 puntos 3D (Figura 8). Contiene puntos mal clasificados por oclusiones de árboles y edificios no modelados. No se espera solución exacta → se usan **1000 iteraciones** de local search para hallar el orden que viola menos restricciones. Espacio de soluciones: **20! ≈ 2.4×10^18**. El orden hallado viola restricciones en **15 de las 92 filas** de la matriz de visibilidad.

3. **Experimento 3 (sintético)**: escena sintetizada con **484 puntos 3D** distribuidos aleatoriamente y **30 cámaras** colocadas en círculo alrededor. Cada punto tiene fecha aleatoria de aparición/desaparición; cada cámara captura en una única fecha aleatoria. Espacio de soluciones: **30! = 2.65×10^32** (necesita local search). Se encuentra una solución que no viola ninguna restricción en solo **26 movimientos locales** desde la inicialización aleatoria, en **menos de un minuto** de cómputo, sin necesidad de reinicializar la búsqueda (porque ningún punto está mal clasificado en las imágenes sintéticas).

4. Se construye además un **modelo 3D variante en el tiempo (4D)** a partir de las 6 imágenes del experimento 1 (Figura 10), mostrando la escena en 4 momentos distintos desde el mismo punto de vista, generado automáticamente a partir de correspondencias 2D en 6 imágenes no ordenadas.

## 7. Discusión
- Costo computacional principal: calcular el número de restricciones violadas por un ordenamiento dado, que **crece linealmente con m** (número de puntos) **y n** (número de imágenes). Además, en cada paso de local search, el número de ordenamientos evaluados crece con **n²** (hay (n)(n−1)/2 formas de elegir dos imágenes a intercambiar).
- La cantidad de cómputo varía **inversamente** con el número de soluciones válidas: cuando hay muchas soluciones (como en el problema de n-reinas de [12], donde el número de soluciones crece con el tamaño del tablero), la inicialización aleatoria suele estar cerca de una solución válida y el problema se resuelve rápido. Cuando hay pocas o ninguna solución exacta (como en el experimento de 20 imágenes), se requieren muchas iteraciones.
- **Naturaleza abstracta de las fechas inferidas**: el método no puede inferir fechas absolutas de construcción/demolición (ej. "1902 a 1966"), solo puede inferir que una estructura existió "desde la Imagen 1 hasta la Imagen 13" (según posición en el orden inferido). Los humanos infieren fechas absolutas usando conocimiento adicional (edificios conocidos, estilo de autos, ropa de las personas en la foto) — esto sugiere que se necesitaría **machine learning** para asignar fechas absolutas.

## 8. Conclusión
Los CSP proveen un framework poderoso para resolver problemas de ordenamiento temporal en visión por computadora; es el primer método conocido para este problema específico. El mayor obstáculo para un sistema totalmente automatizado sigue siendo la variedad de errores de clasificación (missing/occluded); con más puntos mal clasificados, la calidad de la segmentación de estructura decrece (menos puntos comparten exactamente las mismas fechas). Trabajo futuro: extender el razonamiento de oclusión para manejar oclusores no modelados explícitamente en la escena (como árboles).
