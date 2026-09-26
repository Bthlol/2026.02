# Paper 2 — "GHOST: A Combinatorial Optimization Framework for Real-Time Problems"
### Resumen detallado en español (no es traducción literal del texto original; es una explicación exhaustiva sección por sección para fines de estudio)

**Autores:** Florian Richoux, Alberto Uriarte, Jean-François Baffier.
**Publicación:** IEEE Transactions on Computational Intelligence and AI in Games, Vol. 8, No. 4, Diciembre 2016.

---

## 1. Introducción y motivación

El paper presenta **GHOST**, un framework de código abierto (licencia GNU GPL v3) escrito en **C++11**, pensado para que desarrolladores de inteligencia artificial en videojuegos —en particular juegos de estrategia en tiempo real (RTS)— puedan **modelar cualquier problema como un CSP o un COP** y obtener soluciones de buena calidad en **decenas de milisegundos**, tiempo compatible con las restricciones de tiempo real de un motor de juego.

La motivación central es que los juegos RTS presentan constantemente problemas de decisión combinatoria complejos (a qué enemigo dispararle, cómo posicionar edificios, en qué orden construir cosas) que deben resolverse **muy rápido y de forma repetida**, frame a frame, por lo que los solvers exactos tradicionales (que pueden tardar segundos, minutos u horas) son inviables.

### Las tres familias de problemas en juegos RTS (según Ontañón et al.)

Se identifican tres niveles de abstracción en la toma de decisiones dentro de un RTS, de mayor a menor nivel:

1. **Strategy (estrategia)**: decisiones de más alto nivel, que involucran la totalidad de las unidades y edificios del jugador (por ejemplo, qué construir y en qué orden para llegar a determinada composición de ejército).
2. **Tactics (tácticas)**: la implementación concreta de una estrategia — por ejemplo, cómo posicionar, mover y sincronizar (timing) grupos de unidades para ejecutar un plan táctico.
3. **Reactive control (control reactivo)**: la implementación más de bajo nivel de las tácticas, centrada en una **unidad individual**: moverse, apuntar, disparar, huir/hostigar (kiting), etc.

GHOST se evalúa con **un problema representativo de cada una de estas tres familias**, usando como banco de pruebas el juego **StarCraft: Brood War**. Se menciona que este juego vendió más de 10 millones de copias y es particularmente popular en Corea del Sur. Datos técnicos relevantes del motor del juego: a velocidad "normal" corre a 14.96 cuadros (frames) lógicos por segundo, mientras que en modo "fastest" (el más rápido) corre a 23.81 fps, lo que equivale a que **1 frame dura 42 milisegundos**. Los bots que juegan StarCraft deben tomar sus decisiones en menos de 55 ms por frame; si superan ese límite de tiempo en 200 o más frames a lo largo de la partida, se los penaliza (pueden perder la partida directamente).

---

## 2. Definiciones formales: CSP y COP

- **CSP (Constraint Satisfaction Problem / Problema de Satisfacción de Restricciones)** se define formalmente como una tupla **(V, D, C)**, donde:
  - V = conjunto de variables del problema.
  - D = dominio, es decir, el conjunto de valores posibles que pueden tomar las variables.
  - C = conjunto de restricciones. Cada restricción individual c perteneciente a C es un **predicado k-ario**, es decir, una función que toma k variables y devuelve verdadero (⊤) o falso (⊥) según si esas variables satisfacen o no la restricción: c: V^k → {⊤, ⊥}.

- **COP (Constraint Optimization Problem / Problema de Optimización de Restricciones)** se define como una tupla **(V, D, C, f)** — exactamente igual que un CSP, pero agregando una **función objetivo f** que debe minimizarse o maximizarse.

- **Diferencia clave entre CSP y COP**: en un CSP puro, todas las soluciones válidas son consideradas equivalentes entre sí — basta con encontrar **una** solución que satisfaga todas las restricciones (el ejemplo típico es el Sudoku: cualquier solución válida es igual de "buena" que otra). En cambio, en un COP, dentro del conjunto de soluciones válidas, **algunas son mejores que otras** según el valor que tomen en la función objetivo.

- **Dos grandes familias de algoritmos** para resolver estos problemas:
  1. **Algoritmos de búsqueda en árbol / completos**: como backtracking o forward checking. Exploran (de forma más o menos inteligente, con poda) **todo** el espacio de búsqueda, lo cual permite garantizar que si existe una solución, la van a encontrar, y también permite probar que un problema no tiene solución (insatisfacibilidad) o que una solución es óptima.
  2. **Metaheurísticas / algoritmos incompletos**: basados en movimientos locales sobre una solución candidata. Son mucho más adecuados para problemas de tamaño "industrial" quese deben resolver en tiempos razonables, pero tienen la desventaja de que **no pueden garantizar** ni que la solución encontrada sea óptima ni, si no encuentran ninguna solución, que el problema realmente sea insatisfacible.

---

## 3. El motor de GHOST: el algoritmo Adaptive Search

GHOST está construido sobre el algoritmo **Adaptive Search**, propuesto originalmente por Codognet y Diaz. Los autores lo eligieron porque, según argumentan, es una de las metaheurísticas más rápidas conocidas en la literatura.

- **GHOST es mono-objetivo**: solo puede optimizar una única función objetivo a la vez. Esta es una decisión de diseño pragmática de los autores: los solvers multiobjetivo son en general más lentos y considerablemente más difíciles de implementar correctamente. Si se necesita maximizar en lugar de minimizar una función f, simplemente se optimiza 1/f o bien −f.

- **Arquitectura de dos loops anidados (Figura 1)**:
  - Existe un **loop externo de optimización** (representado en azul en el diagrama del paper) que **contiene dentro de sí** un **loop interno de satisfacción** (representado en rojo).
  - El loop de satisfacción tiene un parámetro obligatorio **x**, que corresponde al **timeout** (tiempo límite) de satisfacción, medido en microsegundos.
  - Existe un parámetro opcional **y**, que es el timeout total del loop de optimización, también en microsegundos. Si el usuario no especifica y, por defecto se usa **y = 10x**.
  - Mecánicamente, el loop de optimización ejecuta **n veces** el loop de satisfacción interno. De esas n ejecuciones, se obtienen **m ≤ n** soluciones que efectivamente satisfacen todas las restricciones (soluciones válidas). Sobre esas soluciones válidas se aplica un post-procesamiento de satisfacción, se calcula el costo de cada una usando la función objetivo, y finalmente se conserva la solución de **menor costo** encontrada.
  - Los autores comparan este proceso con una especie de **muestreo tipo Monte Carlo** (aclarando explícitamente que **no** se trata de Monte Carlo Tree Search, que es un algoritmo distinto y más conocido en el contexto de juegos).
  - Existe un tercer parámetro interno: el largo de una **lista tabú**, fijado en **|V| − 1** (donde |V| es la cantidad de variables). Este valor fue determinado empíricamente por los autores como el óptimo, y no requiere ajuste (tuning) adicional por parte del usuario del framework.

- **Estructura del código**: GHOST está organizado en **5 clases principales** en C++: `Variable`, `Domain`, `Constraint`, `Objective` y `Solver`.

- **Dos perfiles de usuario objetivo**:
  - **Usuario casual (casual user)**: simplemente instancia un problema que ya viene implementado/codificado dentro del framework, y lo resuelve llamando al método `solve`. Los autores destacan que esto puede lograrse en tan solo **5 líneas de código C++**.
  - **Usuario desarrollador (developer user)**: implementa **nuevos problemas** heredando de las clases base del framework (Variable, Constraint, Objective, etc.), sin necesidad de modificar el núcleo del solver en sí.

---

## 4. Problema 1 (Reactive Control): Target Selection — Selección de objetivo

Este es el problema representativo de la familia de "control reactivo": decidir, para cada una de las unidades propias, a qué unidad enemiga dispararle.

- **Modelo CSP**:
  - Variables = el grupo de unidades propias (las unidades del jugador que representa GHOST).
  - Dominio = el grupo de unidades enemigas (los posibles objetivos a los que se puede apuntar).
  - Restricción: toda unidad propia que esté viva y en condiciones de disparar debe estar apuntando a una unidad enemiga que también esté viva y que se encuentre dentro de su rango de ataque.

- Se menciona un resultado teórico de Furtak y Buro: este problema de selección de objetivo pertenece a la clase de complejidad **PSPACE**.

- Se modelan propiedades de las unidades relevantes para el daño: el **tamaño** de la unidad objetivo (pequeño/mediano/grande — small/medium/large) y el **tipo de daño** del atacante (concussive/normal/explosive). Estos se combinan en la **Tabla I: matriz de eficiencia de daño**, que indica qué porcentaje del daño nominal efectivamente se aplica según la combinación tamaño-tipo:
  - Unidad pequeña (small): 100% contra daño concussive, 100% contra daño normal, 50% contra daño explosive.
  - Unidad mediana (medium): 50% / 100% / 75% respectivamente.
  - Unidad grande (large): 25% / 100% / 100% respectivamente.

- **Dos funciones objetivo probadas**:
  - **MaxDamage**: maximizar el daño total infligido a los enemigos durante el frame actual.
  - **MaxKill**: maximizar la cantidad de unidades enemigas que mueren durante el frame actual.

- **Setup experimental**: se enfrentan 4 líneas de unidades Terran contra una composición espejo idéntica: 5 marines; 2 Goliaths + 2 Vultures; 2 Siege Tanks (en modo tanque) + 2 Ghosts; y 1 Siege Tank en modo siege (asediado). Se usa un simulador propio construido por los autores.

- **Tabla II — resultados sobre 100 simulaciones**, comparando ambas funciones objetivo a dos timeouts distintos:
  - **Timeout de 3ms**:
    - MaxDamage: 98 victorias, 1 empate, 1 derrota. En promedio quedan vivas 2.8 unidades con 237.9 HP promedio del lado de GHOST, contra 1.0 unidad y 12.0 HP del oponente.
    - MaxKill: 94 victorias, 5 empates, 1 derrota. Promedio: 3.0 unidades vivas con 250.8 HP para GHOST, contra 1.0 unidad y 3.0 HP del oponente.
  - **Timeout de 5ms**:
    - MaxDamage: 99 victorias, 1 empate, 0 derrotas. Promedio: 2.6 unidades vivas con 231.9 HP para GHOST, contra 0 unidades y 0 HP del oponente.
    - MaxKill: 96 victorias, 4 empates, 0 derrotas. Promedio: 2.6 unidades vivas con 233.6 HP para GHOST, contra 0 y 0 del oponente.

- **Limitaciones del simulador** (elementos que explícitamente NO se modelaron): curación y reparación de unidades, regeneración de escudos, diferencias de terreno alto/bajo, disparos aéreos con probabilidad de fallar, fuego amigo (friendly fire), y el daño en área (splash) exclusivo del Firebat.

---

## 5. Problema 2 (Tactics): Wall-in — Construcción de muro

Este es el problema representativo de la familia "táctica": construir con edificios un muro que bloquee (o al menos estreche significativamente) la entrada natural a la base del jugador, para dificultar el paso de unidades enemigas.

- Cada edificio tiene dos propiedades de tamaño relevantes: el **build size** (tamaño de construcción, en unidades llamadas "build tiles", medido en ancho × alto) y el **real size** (tamaño real en píxeles, w_p y h_p, donde w_p es como máximo 32 veces el ancho en build tiles). Un **build tile equivale a 32 píxeles**.

- Se define el concepto de **"significant gap"** (hueco significativo): un espacio lo suficientemente grande como para que pase a través de él un **Zergling**, que es la unidad más pequeña del juego (mide 16×16 píxeles). Si el muro deja un hueco de ese tamaño o mayor, se considera que el muro "falla" en su propósito.

- El modelo utilizado está basado en el trabajo previo de Richoux et al., que a su vez extiende el primer modelo CSP para este problema propuesto por Certicky.

- **Modelo CSP**:
  - Variables = los edificios disponibles de la raza que está jugando el usuario.
  - Dominio = todas las posiciones posibles alrededor del chokepoint (punto de estrangulamiento/entrada angosta del mapa) donde se podría colocar cada edificio.
  - Restricciones:
    - **Overlap**: los edificios no pueden solaparse entre sí.
    - **Buildable**: los edificios no pueden solaparse con tiles del mapa que no son construibles (por ejemplo, agua, terreno inválido).
    - **NoHoles**: no puede quedar ningún hueco del tamaño de un build tile o mayor entre los edificios que conforman el muro.
    - **StartingTargetTile**: debe existir exactamente un edificio colocado sobre un tile de inicio predeterminado, y exactamente un edificio sobre un tile objetivo predeterminado (pudiendo, en algunos casos, ser el mismo edificio el que cumpla ambos roles).

- **Tres funciones objetivo** (las tres se buscan **minimizar**):
  - **Building**: minimizar la cantidad total de edificios usados para formar el muro.
  - **Gap**: minimizar la cantidad de "gaps significativos" (huecos por los que podría pasar un Zergling).
  - **TechTree**: minimizar el nivel tecnológico requerido, entendido como la profundidad en el árbol tecnológico del edificio más avanzado que forma parte del muro (por ejemplo: Command Center tiene profundidad 0, Barracks profundidad 1, Factory profundidad 2, y así sucesivamente — cuanto más "avanzado" el edificio necesario, mayor su profundidad).

- **Tabla III — resultados**, obtenidos sobre un conjunto de **48 chokepoints extraídos de 7 mapas distintos de StarCraft**, promediando 100 corridas por cada chokepoint, donde cada llamada individual a GHOST se limitó a 150 ms:
  - Con objetivo Building: el promedio de edificios usados pasa de 4.05 (en la corrida de satisfacción, antes de optimizar) a **2.56** (tras la fase de optimización), logrando resolver el 98.04% de las instancias.
  - Con objetivo Gap: el promedio de gaps pasa de 1.32 a **0.03**, resolviendo el 97.50% de las instancias. En términos absolutos: de 4800 intentos de muro, se encontraron 4680 muros válidos (97.50%), y de esos, **4527 resultaron ser muros "perfectos"** (sin ningún gap significativo) — es decir, el 96.73% de los muros efectivamente hallados son perfectos.
  - Con objetivo TechTree: el promedio de nivel tecnológico pasa de 1.99 a **1.35**, resolviendo el 97.54% de las instancias.
  - **Comparación con el solver anterior de Richoux et al.**: el porcentaje de problemas resueltos subió de un rango de 95-96% al nuevo rango de 97-98%. Con el objetivo Building específicamente, el promedio de edificios bajó de 2.65 (versión anterior) a 2.56 (GHOST); los gaps significativos bajaron de 0.05 a 0.03; y el nivel tecnológico promedio bajó de 1.56 a 1.35. En todos los aspectos, GHOST mejora al solver previo.

- **Tabla IV — porcentaje de muros exitosos encontrados, desglosado por mapa**: Python 100%, HeartbreakRidge 100%, CircuitBreaker 99%, Benzene 99%, Aztec 97%, Andromeda 96%, y **Fortress 90%** (el peor resultado de todos los mapas). Los autores explican que en el mapa Fortress falla ocasionalmente porque hay un chokepoint particular que solo admite solución usando dos edificios de tamaño 3×2, una combinación más difícil de encontrar para el algoritmo.

- **Comparación con Certicky (que usa el solver Clingo) — Tabla V**: promedio de 20 corridas, limitando el problema a usar únicamente 2 Barracks y 4 Supply Depots (para poder comparar en igualdad de condiciones con el trabajo previo):
  - Chokepoint estrecho (65 píxeles de ancho): GHOST tarda **46.8 ms**, mientras que Clingo tarda **362.8 ms**.
  - Chokepoint ancho (250 píxeles de ancho): GHOST tarda **33.5 ms**, mientras que Clingo tarda **408.8 ms**.
  - Conclusión de esta comparación: **GHOST es al menos 7.8 veces más rápido** que el enfoque basado en Clingo.

---

## 6. Problema 3 (Strategy): Build Order (BO) — Orden de construcción

Este es el problema representativo de la familia "estrategia": decidir un **plan de build order**, es decir, una serie de acciones (construir edificios, entrenar unidades, investigar mejoras/upgrades) con un timing específico, que permita alcanzar cierta meta (por ejemplo, cierta composición de ejército) lo más rápido posible.

- Se modela como un **problema de permutación** dentro del framework CSP: es decir, las variables deben mapearse de forma **biyectiva** al dominio (cada valor del dominio se asigna exactamente a una variable, y viceversa). En este tipo de modelado, "cambiar el valor de una variable" en realidad significa **intercambiar su valor con el de otra variable** (un swap), para mantener siempre la biyección.

- **Modelo CSP**:
  - Variables = todas las acciones necesarias para alcanzar la meta especificada por el usuario.
  - Dominio = el orden (posiciones) en que esas acciones pueden ejecutarse.
  - Restricción: toda acción α que tenga una dependencia (por ejemplo, un edificio que requiere otro edificio previo, o una mejora que requiere un nivel anterior de esa misma mejora) debe programarse **después** de que esa dependencia ya se haya cumplido. Estas dependencias pueden ser **recursivas**: por ejemplo, el "Air Weapons Upgrade nivel 2" requiere haber completado primero el nivel 1, que a su vez requiere tener construido un Cybernetics Core, que a su vez requiere tener un Gateway.

- La implementación se centra específicamente en la raza **Protoss**. La única función objetivo implementada para este problema es **minimizar el makespan** (el tiempo total, en frames, que toma completar todo el plan de build order).

- Hay un **post-procesamiento de optimización** (a diferencia de los problemas anteriores, aquí no hay post-procesamiento de satisfacción): si el usuario solicitó, por ejemplo, n unidades de cierto tipo U que se producen desde un edificio de tipo B, pero solo pidió construir m edificios B (con m menor que n), GHOST puede decidir construir edificios B adicionales si eso permite acortar el makespan total del plan.

- **Simulador integrado**: a diferencia del problema de Target Selection (que usaba un simulador de combate), aquí se necesita simular la **economía y producción** del juego (sin combate involucrado). Los autores ajustaron varios parámetros del simulador respecto al trabajo previo de Churchill y Buro, luego de analizar replays de jugadores profesionales coreanos:
  1. Tiempo que toma a un trabajador ir hasta el lugar de construcción: **74 frames** (en el trabajo de Churchill y Buro este valor era 96).
  2. Tiempo de vuelta del trabajador a recolectar minerales después de terminar de construir: **60 frames** (antes: 0).
  3. Tiempo desde la base hasta un parche de mineral para empezar a minarlo: **74 frames** (antes: 0).
  4. Tiempo que toma a un trabajador cambiar de recolectar minerales a recolectar gas: **74 frames** (antes: 0).
  5. Tasa de recolección de minerales: **0.045 minerales por trabajador por frame** (igual que en el trabajo previo).
  6. Tasa de recolección de gas: **0.077 de gas por trabajador por frame** (antes: 0.07).
  - Adicionalmente, el simulador siempre produce trabajadores automáticamente hasta alcanzar la **saturación de 24 trabajadores por base**, y siempre construye suministro (supply) adicional preventivamente para nunca quedar "supply blocked" (bloqueado por falta de población disponible).

- **Validación del simulador (Tabla VI)**: se comparó el comportamiento del simulador contra el jugador profesional coreano de Protoss apodado "Bisu", durante los primeros 1900 frames (80 segundos) de una partida real. Los resultados fueron muy similares entre el simulador y el jugador humano, con una leve ventaja para el simulador, explicada porque su producción de probes (trabajadores) es prácticamente perfecta (sin los pequeños errores/demoras que comete un humano).

- **Dataset de experimentos**: se usó un conjunto de **3647 build orders en total**, extraídos de replays reales (dataset original de Synnaeve, refinado posteriormente por Glen Robertson). Se desglosan en 768 partidas Protoss vs Protoss (PvP), 2043 Protoss vs Terran (PvT), y 836 Protoss vs Zerg (PvZ). Cada llamada individual a GHOST duró **30 ms**, y se corrió 10 veces por cada build order del dataset.

- **Tabla VII — makespan promedio (en frames), comparando jugadores humanos vs GHOST**, separado en dos "techos" de duración considerados:
  - **Techo de 10000 frames**:
    - Todos los BOs: humanos promedian 9794 frames, GHOST promedia 9250 frames; se resolvió el 94.4% de los casos; ganancia promedio de 544 frames a favor de GHOST.
    - PvP: humanos 9727 / GHOST 9078 / 95.0% resueltos / ganancia de 649 frames.
    - PvT: humanos 9861 / GHOST 9378 / 93.9% / ganancia de 483 frames.
    - PvZ: humanos 9692 / GHOST 9097 / 95.0% / ganancia de 595 frames.
    - **Solo considerando pro-gamers (Allpro)**: humanos 9605 / GHOST 8916 / 96.3% resueltos / **ganancia de 689 frames**.
  - **Techo de 7800 frames**:
    - Todos los BOs: humanos 7726 / GHOST 7332 / 98.8% resueltos / ganancia de 394 frames.
    - PvP: 7630 / 7249 / 99.3% / ganancia de 381 frames.
    - PvT: 7800 / 7564 / 98.3% / ganancia de 236 frames.
    - PvZ: 7626 / 6841 / 99.7% / ganancia de 785 frames.
    - **Allpro**: 7485 / 7179 / 100% resueltos / **ganancia de 306 frames**.

- También se comparó específicamente contra 8 replays de jugadores profesionales de primer nivel mundial (del lado Protoss: Bisu, BeSt, Violet, Cure; del lado Zerg: Jaedong, sAviOr; del lado Terran: Flash). En ese análisis, **GHOST supera consistentemente a los pro-gamers Protoss**, obteniendo un makespan mejor por 689 frames en promedio para BOs largos (techo de 10000 frames) y por 306 frames para BOs más cortos (techo de 7800 frames).

- **Comparación con el método de Churchill y Buro** (que usa branch and bound, un algoritmo completo): ese método logra, el 90% de las veces, encontrar build orders con el mismo makespan que los jugadores profesionales, pero tardando en promedio **3.735 segundos** de cómputo, para BOs de hasta 249 segundos de duración (equivalentes a 5928 frames). Esto da un **ratio de tiempo de cómputo sobre makespan del 1.5%**. En contraste, GHOST computa en promedio build orders con un makespan de 9250 frames en tan solo **30 ms**; considerando que 9250 frames equivalen a aproximadamente 388 segundos en el modo más rápido del juego, el **ratio de GHOST es de apenas 0.007%** — es decir, GHOST usa proporcionalmente muchísimo menos tiempo de cómputo respecto a la duración del plan que genera.

- En cuanto a tiempos de cómputo concretos: GHOST necesita **20 ms para la fase de satisfacción** y **30 ms para la fase completa de optimización**, lo cual encaja perfectamente dentro de la duración de un único frame del juego en modo más rápido (42 ms).

- Los autores explican por qué el problema de Build Order resultó "más fácil" de resolver que el problema de Wall-in a pesar de ser conceptualmente más complejo: el modelarlo como un **problema de permutación** reduce drásticamente el tamaño efectivo del espacio de búsqueda combinatorio que el algoritmo tiene que explorar.

---

## 7. Comparación con solvers del estado del arte (Sección VI): Resource Allocation

Como experimento adicional para validar la calidad de GHOST frente a otros solvers reconocidos (no solo frente a GHOST mismo o a trabajos anteriores específicos de RTS), se plantea un cuarto problema: **Resource Allocation (asignación de recursos)**.

- **Definición del problema**: dado un stock fijo de minerales, gas y supply (población), ¿qué combinación de unidades conviene entrenar para **maximizar el DPS (damage per second / daño por segundo) total en tierra**? Este problema es, formalmente, una instancia del **problema de la mochila multidimensional (multidimensional knapsack)**, con **3 dimensiones** (una dimensión por cada tipo de recurso: minerales, gas y supply). El problema de la mochila clásico (una dimensión) ya es **NP-completo**; la versión multidimensional es todavía más difícil — los autores señalan que, salvo que P=NP, **no existe un PTAS eficiente** (esquema de aproximación en tiempo polinomial) a partir de 2 dimensiones o más.

- **Solvers usados para la comparación**:
  - **Opturion CPX** y **Gecode**: dos solvers **completos y deterministas** (algoritmos de búsqueda exacta), ambos capaces de parsear directamente modelos escritos en **MiniZinc**. Se destaca que Gecode ganó **todas** las medallas de oro del prestigioso MiniZinc Challenge entre 2008 y 2012; a partir de 2013, Opturion CPX pasó a ser el solver dominante de la competencia (obtuvo 2 oros y 2 bronces en 2013, 4 platas en 2014, y 2 oros, 1 plata y 1 bronce en 2015).
  - **Oscar/CBLS**: una metaheurística (no un solver completo) que también es capaz de parsear modelos MiniZinc.

- **Instancias de prueba**: se fijó un stock de **20000 minerales, 14000 gas y 380 de supply**, generando una instancia separada para cada una de las tres razas del juego (Zerg, Protoss y Terran).

- **Tabla VIII — resultados comparativos**:
  - **Zerg**: el DPS óptimo real es 11400.00. Opturion CPX lo encuentra en 200 ms. Gecode tarda 99580 ms (~99.58 segundos) en encontrarlo. GHOST (promediando 100 corridas) encuentra el óptimo exacto el 59% de las veces, en un tiempo de 80 ms, con un DPS promedio de 11387.70 (equivalente al 99.89% del valor óptimo). Oscar/CBLS (promediando 10 corridas) logra un DPS promedio de 11400.00 (encuentra el óptimo consistentemente), con un tiempo de ejecución promedio de 562 ms.
  - **Protoss**: el óptimo es 4916.38. Opturion CPX tarda 1620 ms; **Gecode no logró encontrar ninguna solución** tras 6 horas de cómputo. GHOST encuentra el óptimo exacto el 54% de las veces, en 1300 ms, con DPS promedio de 4907.70 (99.82% del óptimo). Oscar/CBLS obtiene un DPS promedio de solo 3445.21 (70.08% del óptimo), con su mejor resultado individual llegando a 4480.00 (91.12% del óptimo), en un tiempo promedio de 1200 ms.
  - **Terran**: el óptimo es 6632.73. Opturion CPX tarda **3 horas y 19 minutos** en resolverlo (un tiempo enorme comparado con los otros casos). Gecode nuevamente no logró encontrar ninguna solución. GHOST encuentra el óptimo el 53% de las veces, en apenas 130 ms, con DPS promedio de 6619.30 (99.80% del óptimo). Oscar/CBLS obtiene un DPS promedio de solo 3035.73 (45.77% del óptimo), con su mejor resultado individual de 5767.55 (86.96% del óptimo), en 1390 ms.

- **Explicación de por qué varía tanto la dificultad entre razas**: para Zerg, la estrategia óptima resulta ser trivial (entrenar únicamente Zerglings, que tienen la mejor relación DPS/costo); para Terran, también es relativamente trivial (entrenar únicamente Firebats); pero para Protoss, la solución óptima requiere **combinar** Zealots y Dark Templars en cierta proporción, lo que hace el problema más difícil de resolver de forma consistente. Además, el espacio de búsqueda de Terran es sustancialmente más amplio: aproximadamente **1.57×10^20 configuraciones posibles**, comparado con ~1.55×10^13 para Zerg y ~1.33×10^12 para Protoss — esto se debe a que Terran cuenta con **9 tipos de unidades relevantes** para combate terrestre, frente a solo 6 en Zerg y Protoss.

- Nota metodológica: Oscar/CBLS aprovecha múltiples núcleos de CPU en paralelo, mientras que GHOST corre de forma **secuencial** (un solo núcleo). Los experimentos se realizaron en un equipo Intel i7 de 4 núcleos, con 4GB de RAM, sobre Ubuntu 14.04 de 64 bits.

- **Conclusión de esta sección**: GHOST logra superar tanto a los solvers completos (Opturion CPX y Gecode) como a la metaheurística de referencia Oscar/CBLS, tanto en la **calidad** de las soluciones encontradas como en el **tiempo de ejecución** necesario para obtenerlas — y esto a pesar de correr en un solo núcleo.

---

## 8. Conclusión general del paper

Los autores concluyen que, para juegos RTS, **buscar la solución óptima absoluta puede no ser la estrategia más conveniente en la práctica** — esto queda claramente confirmado por los tiempos de cómputo excesivamente largos que requieren los solvers completos (como se vio con Opturion CPX tardando más de 3 horas en el caso Terran). En cambio, usar metaheurísticas rápidas como Adaptive Search permite obtener una solución "suficientemente buena" (aunque no garantizadamente óptima) en tan solo decenas de milisegundos; y si esa solución no resulta suficientemente buena en la práctica, simplemente se puede **volver a ejecutar el algoritmo en el siguiente frame** del juego, dado lo económico que resulta en tiempo.

**Trabajo futuro propuesto**:
- Implementar un mecanismo de **pausa y reanudación (pause/resume)** de la búsqueda.
- Aprovechar **múltiples núcleos de CPU en paralelo**, dado que el algoritmo Adaptive Search es paralelizable por naturaleza — se cita evidencia de un trabajo previo que logró speedups casi lineales usando hasta 8192 núcleos.
- Investigar un **nuevo formalismo de CSP** capaz de manejar de forma eficiente la **incertidumbre**, ya que el framework actual de GHOST no está bien adaptado a escenarios con información incompleta o incierta (una limitación relevante para juegos RTS reales, donde el jugador no tiene visión completa del mapa ni de las acciones del enemigo).
