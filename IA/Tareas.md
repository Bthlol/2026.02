# Modelo del Problema de Transporte

El **Problema de Transporte** consiste en determinar la forma óptima de distribuir un bien homogéneo desde un conjunto de **orígenes** (fábricas, plantas, centros de producción) hacia un conjunto de **destinos** (bodegas, clientes, centros de consumo), minimizando el costo total de transporte, respetando la disponibilidad de cada origen y los requerimientos de cada destino.

A continuación se detalla el modelo siguiendo el siguiente orden: primero los **Índices**, luego **Constantes/Parámetros** (deben conocerse antes de escribir variables y restricciones), **Variables**, **Función Objetivo** y **Restricciones**.

---

## 1. Índices

- $i = 1, 2, \dots, m$ : índice que representa cada **origen**.
- $j = 1, 2, \dots, n$ : índice que representa cada **destino**.

---

## 2. Constantes / Parámetros

Son los datos conocidos a priori, fijos, que caracterizan el problema:

| Símbolo | Definición |
|---|---|
| $m$ | Número total de orígenes. |
| $n$ | Número total de destinos. |
| $c_{ij}$ | Costo unitario de transportar una unidad del bien desde el origen $i$ hasta el destino $j$. |
| $a_i$ | Oferta (disponibilidad, capacidad) del origen $i$, es decir, la cantidad máxima de unidades que puede despachar. |
| $b_j$ | Demanda (requerimiento) del destino $j$, es decir, la cantidad de unidades que necesita recibir. |

**Condición de equilibrio (caso balanceado):**

$$
\sum_{i=1}^{m} a_i = \sum_{j=1}^{n} b_j
$$

Si esta igualdad no se cumple, el problema se denomina **no balanceado**, y se equilibra introduciendo un origen o destino ficticio con costos nulos.

---

## 3. Variables de Decisión

$$
x_{ij} = \text{cantidad de unidades del bien a transportar desde el origen } i \text{ hasta el destino } j, \quad \forall i=1,\dots,m; \; \forall j=1,\dots,n
$$

Estas son las decisiones controlables del modelo: cuánto enviar por cada ruta $(i,j)$.

---

## 4. Función Objetivo

Minimizar el **costo total de transporte**, que corresponde a la suma de los costos unitarios ponderados por las cantidades transportadas en cada ruta:

$$
\min Z = \sum_{i=1}^{m} \sum_{j=1}^{n} c_{ij} \, x_{ij}
$$

---

## 5. Restricciones

**a) Restricciones de oferta (disponibilidad de cada origen):**

La cantidad total enviada desde cada origen $i$ no puede superar (o debe igualar, en el caso balanceado) su disponibilidad:

$$
\sum_{j=1}^{n} x_{ij} \leq a_i \qquad \forall i = 1,\dots,m
$$

*(en el caso balanceado esta restricción se cumple con igualdad: $\sum_{j=1}^n x_{ij} = a_i$)*

**b) Restricciones de demanda (requerimiento de cada destino):**

La cantidad total recibida en cada destino $j$ debe satisfacer (al menos, o exactamente) su demanda:

$$
\sum_{i=1}^{m} x_{ij} \geq b_j \qquad \forall j = 1,\dots,n
$$

*(en el caso balanceado: $\sum_{i=1}^m x_{ij} = b_j$)*

**c) Restricción de no negatividad:**

No tiene sentido físico transportar cantidades negativas:

$$
x_{ij} \geq 0 \qquad \forall i=1,\dots,m; \; \forall j=1,\dots,n
$$

*(Nota: aunque el modelo se plantea como Programación Lineal continua, la estructura particular de la matriz de restricciones — totalmente unimodular — garantiza que toda solución básica factible óptima resulta automáticamente entera cuando $a_i$ y $b_j$ son enteros, lo que conecta este modelo con la Optimización Combinatoria, ya que la solución óptima corresponde a un flujo entero sobre una red.)*

---

## Modelo Completo (resumen)

$$
\begin{aligned}
\min \quad & Z = \sum_{i=1}^{m} \sum_{j=1}^{n} c_{ij} \, x_{ij} \\[4pt]
\text{s.a.} \quad & \sum_{j=1}^{n} x_{ij} \leq a_i, & \forall i = 1,\dots,m \\[4pt]
& \sum_{i=1}^{m} x_{ij} \geq b_j, & \forall j = 1,\dots,n \\[4pt]
& x_{ij} \geq 0, & \forall i,j
\end{aligned}
$$

---

# Modelo de Generación de Columnas: Problema de Corte de Stock (Cutting Stock Problem)

El **Problema de Corte de Stock** (Cutting Stock Problem, formulación de **Gilmore-Gomory**, 1961) es uno de los ejemplos clásicos y originales con el que nació la técnica de Generación de Columnas.

---

## 0. Descripción del problema

Una fábrica dispone de **rollos (o barras) de material en un ancho estándar fijo** $W$ (ej. bobinas de papel, planchas de acero, rollos de tela). Los clientes solicitan **piezas más pequeñas** de distintos anchos $w_i$ en ciertas cantidades $d_i$ (demanda). Se debe decidir **cómo cortar cada rollo estándar** (qué combinación de piezas sacar de él) para satisfacer toda la demanda, **minimizando el número total de rollos utilizados** (y por lo tanto minimizando el desperdicio de material).

---

## 1. ¿Qué representa una "columna" en este problema?

Aquí **una columna no es una ruta ni una secuencia temporal**, sino un **patrón de corte (cutting pattern)**: una forma específica de cortar un rollo estándar en piezas más pequeñas.

$$
\text{Columna } j = \text{patrón de corte } j = (a_{1j}, a_{2j}, \dots, a_{mj})
$$

donde $a_{ij}$ = número de piezas de tipo $i$ que se obtienen al aplicar el patrón de corte $j$ sobre un rollo estándar.

**Ejemplo concreto:** si el rollo estándar mide $W=100$ cm, y se necesitan piezas de $30$, $45$ y $50$ cm, un patrón de corte podría ser "3 piezas de 30 cm" (usa $90$ de los $100$ cm, con $10$ cm de desperdicio), lo cual se representa como la columna $(3,0,0)$. Otro patrón podría ser "1 pieza de 50 + 1 pieza de 45" → columna $(0,1,1)$ (con $5$ cm de desperdicio). Cada combinación factible (que no exceda el ancho $W$) es una columna candidata.

**El número total de patrones de corte factibles crece exponencialmente** con el número de tipos de piezas, por lo que —igual que con los pairings en Crew Scheduling— **no se pueden enumerar todos explícitamente**: se generan solo bajo demanda mediante el subproblema de pricing.

---

## 2. Problema Maestro (Master Problem)

### Índices

- $i = 1,\dots,m$ : índice de **tipos de piezas** solicitadas.
- $j = 1,\dots,n$ : índice de **patrones de corte (columnas)** factibles, con $n$ implícitamente muy grande.

### Constantes / Parámetros

| Símbolo | Definición |
|---|---|
| $W$ | Ancho (o largo) del rollo estándar disponible. |
| $w_i$ | Ancho de la pieza tipo $i$ solicitada. |
| $d_i$ | Demanda (cantidad requerida) de piezas tipo $i$. |
| $a_{ij}$ | Número de piezas de tipo $i$ que produce el patrón de corte $j$ (entero $\geq 0$). |

Cada patrón $j$ debe ser **factible**, es decir, respetar el ancho del rollo:

$$
\sum_{i=1}^m w_i\, a_{ij} \leq W
$$

### Variables de Decisión

$$
x_j = \text{número de rollos estándar a cortar usando el patrón } j \qquad (x_j \in \mathbb{Z}_{\geq 0}), \quad \forall j=1,\dots,n
$$

*(a diferencia del Crew Scheduling, aquí $x_j$ no es binaria sino **entera no negativa**, porque un mismo patrón de corte se puede aplicar a varios rollos)*

### Función Objetivo

Minimizar el número total de rollos estándar utilizados:

$$
\min \; Z = \sum_{j=1}^{n} x_j
$$

### Restricciones

**a) Satisfacción de la demanda** (la cantidad total de piezas tipo $i$ producidas, sumando entre todos los patrones usados, debe cubrir la demanda):

$$
\sum_{j=1}^{n} a_{ij}\, x_j \geq d_i \qquad \forall i = 1,\dots,m
$$

**b) Integralidad:**

$$
x_j \in \mathbb{Z}_{\geq 0} \qquad \forall j = 1,\dots,n
$$

$$
\begin{aligned}
\min \quad & Z = \sum_{j=1}^{n} x_j \\
\text{s.a.} \quad & \sum_{j=1}^{n} a_{ij}\, x_j \geq d_i, & \forall i=1,\dots,m \\
& x_j \in \mathbb{Z}_{\geq 0}, & \forall j
\end{aligned}
$$

---

## 3. Relajación y precios sombra

Se resuelve el **RMP** (relajación lineal, $x_j \geq 0$ continuo) con un subconjunto reducido de patrones $J'$ (típicamente se inicializa con los $m$ patrones triviales: un patrón por cada tipo de pieza, cortando solo esa pieza repetida tantas veces como quepa en $W$).

Al resolver el RMP se obtienen las variables duales $\pi_i \geq 0$, una por cada restricción de demanda del tipo de pieza $i$. Estas $\pi_i$ representan el **valor marginal (sombra)** de disponer de una unidad adicional de la pieza $i$: cuánto se reduciría el número de rollos usados si se pudiera "obtener gratis" una pieza más de tipo $i$.

---

## 4. Subproblema de Pricing: Problema de la Mochila (Knapsack)

Se busca un nuevo patrón de corte $(a_1,\dots,a_m)$ que tenga **costo reducido negativo**. Como el costo original de cada columna es $c_j = 1$ (usar un rollo cuesta "1 rollo"), el costo reducido es:

$$
\bar{c} = 1 - \sum_{i=1}^{m} \pi_i \, a_i
$$

Se busca minimizar $\bar c$, lo cual equivale a **maximizar** $\sum_i \pi_i a_i$ sujeto a que el patrón sea físicamente factible:

### Variables del Subproblema

$$
a_i = \text{número de piezas de tipo } i \text{ a incluir en el nuevo patrón de corte} \qquad (a_i \in \mathbb{Z}_{\geq 0})
$$

### Función Objetivo del Subproblema

$$
\max \; \sum_{i=1}^{m} \pi_i \, a_i
$$

### Restricción del Subproblema (Knapsack)

$$
\sum_{i=1}^{m} w_i \, a_i \;\leq\; W
$$

$$
a_i \in \mathbb{Z}_{\geq 0} \qquad \forall i
$$

**Interpretación:** cada pieza $i$ tiene un "valor" igual a su precio sombra $\pi_i$ y un "peso" igual a su ancho $w_i$; el rollo estándar es la "mochila" de capacidad $W$. Se busca el patrón de corte que **maximice el valor dual acumulado** sin exceder el ancho del rollo — exactamente la estructura del clásico Problema de la Mochila 0-1/entera.

Si el valor óptimo del subproblema $\sum_i \pi_i a_i^* > 1$, entonces $\bar c = 1-\sum_i\pi_i a_i^* < 0$, y ese patrón se agrega como nueva columna al RMP. Si el óptimo es $\leq 1$, no existe ningún patrón con costo reducido negativo y el RMP relajado es óptimo.

---

## 5. Esquema completo

```
1. Inicializar RMP con patrones triviales (1 solo patrón por tipo de pieza)
2. Resolver RMP (relajación lineal) → obtener duales π_i
3. Resolver Subproblema de Pricing (Knapsack: max Σ π_i·a_i, s.a. Σ w_i·a_i ≤ W)
4. Si valor óptimo del Knapsack > 1 → nuevo patrón con c̄ < 0 → agregarlo, volver a 2
5. Si valor óptimo ≤ 1 → óptimo de la relajación lineal alcanzado
6. Aplicar Branch-and-Price para recuperar soluciones enteras (x_j ∈ ℤ≥0)
```

---

## Comparación con otro ejemplo de Generación de Columnas (Crew Scheduling)

| Elemento | Crew Scheduling | Cutting Stock |
|---|---|---|
| ¿Qué es una columna? | Un **pairing** (secuencia factible de vuelos) | Un **patrón de corte** (combinación de piezas por rollo) |
| Tipo de variable $x_j$ | Binaria $\{0,1\}$ | Entera $\geq 0$ |
| Restricción del Maestro | Set Partitioning ($=1$) | Cobertura de demanda ($\geq d_i$) |
| Subproblema de Pricing | Camino más corto con restricción de recursos (RCSPP) | Problema de la Mochila (Knapsack) |
| Recurso limitante | Jornada/descanso de la tripulación | Ancho $W$ del rollo estándar |

---