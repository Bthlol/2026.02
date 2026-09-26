# Apuntes: Lean UX

## 1. Ingeniería de Facilidad de Uso (IFD)
- No es una actividad puntual antes de la entrega: es un **conjunto de actividades a lo largo de todo el ciclo de vida del producto**.
- Un solo producto con mala interfaz puede dañar seriamente la reputación de una empresa de software.

## 2. Lean UX: definición y agenda
Lean UX es una **metodología de diseño UX** basada en la colaboración entre tres roles clave (engranajes que giran juntos):
- **Equipo UX**
- **Dueño del Producto**
- **Equipo de desarrollo**

Ideas centrales (agenda del curso):
1. Reducir la incertidumbre a través del proceso de diseño.
2. Cliente y usuario son claves.
3. La mejora continua se logra probando hipótesis.

Actividades core de Lean UX:
- Conocer a los usuarios (sus necesidades).
- Priorizar para reducir el malgasto de esfuerzos.
- Centrarse en objetivos concretos de usuarios.
- Evaluar (testing) ideas y procesos.
- Observar, aprender y ajustar con ciclos rápidos (**pensar–hacer–verificar**).

## 3. Objetivo y principios (inspirados en Agile)
**Medir por metas, no por resultado** (ej: no medir "crear un servicio de registro" sino "incrementar el número de servicios contratados").

Los 4 valores (adaptados del Manifiesto Ágil):
- Individuos e interacción **por sobre** procesos y herramientas.
- Software funcionando **por sobre** documentación exhaustiva.
- Colaboración con el cliente **por sobre** negociación de contratos.
- Responder al cambio **por sobre** seguir un plan.

## 4. Principios generales de equipo
- **Equipos multifuncionales y colaborativos**: ingenieros, administradores de producto, diseñadores de interacción/visuales, marketing, encargados de contenido, QA.
- **Equipo pequeño, dedicado y colocado**: menos de 10 personas, un solo proyecto, mismo lugar físico → facilita comunicación, foco y camaradería.
- **Equipo orientado a solución de problemas**: el foco está en apoyar al negocio, no en implementar funciones por implementarlas.

## 5. Principios de cultura
- **Moverse desde la duda hacia la certidumbre**: todo es suposición hasta que se demuestre lo contrario.
- **Logros de metas, no resultados**: funciones y servicios son "resultados"; el cambio en el comportamiento del usuario es la verdadera "meta". Lean UX se mide por metas.
- **Remover desperdicios**: todo lo que no contribuye a las metas se considera desperdicio.
- **Entendimiento compartido**: el equipo entiende el qué y el porqué gracias al tiempo compartido trabajando en el producto, el espacio y los usuarios.
- **No hay estrellas ni gurús**: se busca cohesión y colaboración entre todos los miembros.
- **Permiso para fallar**: entorno seguro para experimentar; se acepta que algunas ideas no prosperen.

## 6. Principios del proceso
- **Trabajar en pequeñas porciones para mitigar riesgos**: dividir el trabajo permite testear hipótesis de a poco y reducir el riesgo de falla.
- **Descubrimiento continuo**: involucrar al cliente en el proceso de desarrollo, usando métodos cuantitativos y cualitativos para entender qué hace el usuario y por qué.
- **"Salir del edificio de la compañía"**: el éxito o fracaso depende del cliente/usuario, así que hay que ir a trabajar directamente con ellos.
- **Externalizar el trabajo**: compartir el trabajo con el equipo inspira ideas y genera entendimiento mutuo.
- **Hacer el análisis mediante testeo**: no tiene sentido debatir una idea en teoría, es mejor testearla con usuarios reales.
- **Salir del negocio de los entregables**: el foco está en las metas del negocio, no en la documentación adjunta.

## 7. El proceso de Lean UX (ciclo)
Ciclo circular de 4 etapas que se repite continuamente:

**Metas, Supuestos, Hipótesis → Diseñarlo → Crear PMV → Investigar y Aprender → (vuelve al inicio)**

### 7.1 Metas, supuestos e hipótesis
**Tipos de supuestos:**
- **Metas del negocio**: métricas de éxito y definición de "listo".
- **Usuarios**: la gente a la que se le está solucionando un problema.
- **Metas de usuarios**: metas de éxito, emocionales, de experiencia o de largo plazo.
- **Funciones**: cambios del producto y mejoras relevantes para usuarios o el negocio.

**Refinamiento de supuestos:**
- Cada miembro del equipo (incluyendo clientes) completa los supuestos individualmente.
- Se comparten en una mesa redonda.
- Se coleccionan, organizan y priorizan (post-it / pizarra, separados por tema).
- No se busca acuerdo total; se capturan incluso opiniones contrarias.
- Técnicas: **lluvia de ideas**, uso de **pizarra y post-it**; es esencial hacerlo al inicio del proyecto.

**Supuestos de negocio** (preguntas guía, ejemplos):
- ¿Qué necesitan mis clientes? ¿Cómo se resuelven esas necesidades?
- ¿Quiénes son mis clientes iniciales?
- ¿Cuál es el valor N°1 que buscan? ¿Qué beneficios adicionales obtienen?
- ¿Cómo adquiriré clientes? ¿Cómo generaré dinero?
- ¿Cuál es mi mayor riesgo de producto y cómo lo resolveré?
- ¿Qué cambios de comportamiento indicarán éxito?

**Supuestos de usuario** (preguntas guía):
- ¿Quién es el usuario?
- ¿Cómo entra el producto a su vida o trabajo?
- ¿Qué problemas resuelve? ¿Cuándo y cómo se usa?
- ¿Qué funciones son importantes? ¿Cómo debe verse y comportarse el producto?

**Formulando hipótesis (formato estándar):**
> "Creemos que **[una aseveración es verdadera]**. Sabremos que estamos en lo cierto cuando obtengamos la siguiente respuesta del mercado: **[retroalimentación cualitativa]** y/o **[retroalimentación cuantitativa]** y/o **[cambio en un indicador clave de desempeño]**."

Para crear hipótesis se requiere combinar:
- Metas de negocio a cumplir
- Usuarios a servir
- Metas de usuario que los motivan
- Funcionalidades que pueden servir en esa situación

**Métricas de negocio — "StartUp Metrics for Pirates" (AARRR):**
| Etapa | Rol del usuario | Pregunta clave |
|---|---|---|
| **A**dquisición | Visitante (Suspect) | ¿Podemos atraer clientes a la funcionalidad/producto? |
| **A**ctivación | Usuario (Lead) | ¿Podemos lograr que la usen? |
| **R**etención | Potencial cliente (Prospect) | ¿Podemos hacer que la usen de nuevo? |
| **R**eferencia | Cliente (Customer) | ¿Podemos hacer que los clientes la recomienden? |
| **R**evenue (Ingresos) | Enamorado (Reference) | ¿Podemos hacer que nos paguen por esta funcionalidad? |

**Resultado, meta e impacto (distinción importante):**
- **Resultado (output)**: funciones que se diseñan, implementan y liberan.
- **Meta (outcome)**: cambio en el mundo/comportamiento que se espera tras crear el resultado.
- **Impacto**: medidas del nivel de salud del negocio.

### 7.2 Artefacto Personas
- Herramienta para representar lo aprendido en investigación.
- Promueve el entendimiento compartido en el equipo y recuerda que **el equipo no es el usuario**.

**Estructura de la plantilla (4 cuadrantes):**
| Nombre, rol + dibujo | Demografía / características inolvidables |
|---|---|
| Comportamiento | Necesidades, obstáculos, deseos |

**Creación de personas:**
- Lluvia de ideas y discusión de equipo para llegar a **3-4 personas**.
- Diferenciar más por **roles** que por demografía.
- Completar la plantilla para cada una.
- Compartir fuera del equipo para validar inicialmente.
- **Validar externamente** al iniciar la investigación: ¿Existen realmente? ¿Tienen las necesidades/obstáculos supuestos? ¿Valoran la solución al problema?

### 7.3 Metas de usuarios (3 niveles, con ejemplos)
1. ¿A qué trata de llegar el usuario? (ej: "quiero comprar un celular")
2. ¿Cómo quiere sentirse durante/después del proceso? (ej: "sentir que obtuve el celular que necesito a buen precio")
3. ¿Cómo lleva el producto al usuario a su meta de vida o sueño? (ej: "sentirme experto en tecnología y ser respetado por ello")

### 7.4 Funciones
- Se buscan funciones que permitan a los usuarios llegar a sus metas.
- Distinción clave: función que **sirve** las necesidades del usuario y la empresa vs. función "cool" que no aporta valor real al negocio.
- Técnica: lluvia de ideas.

### 7.5 Tabla de hipótesis (síntesis)
Se organiza todo lo anterior en una tabla con 4 columnas, pegando post-it en cada casillero:

| Creemos que: |||
|---|---|---|---|
| Lograremos... | ...si este usuario... | ...puede lograr... | ...con esta función |
| [meta de negocio] | [Persona] | [meta de usuario] | [función] |

- Es normal encontrar vacíos (metas sin funciones asociadas, funciones sin valor) → llenarlos o **podarlos**.
- **7-10 filas** es un buen punto de partida.
- Si varias soluciones sirven a la vez, conviene refinarlas para concentrarse en una sola función.

### 7.6 Priorización de hipótesis
- Se prioriza según **matriz de Valor vs. Riesgo** (Alto/Bajo Valor × Alto/Bajo Riesgo).
- Meta: priorizar hipótesis según su nivel de riesgo junto con el valor que generan.

### 7.7 Diseñarlo (Diseño Colaborativo)
- La **co-creación** entre diseñadores y no-diseñadores enriquece el diseño y genera confianza.
- Equipo de **5-8 personas**, con un bloque de tiempo dedicado.

**Proceso de 5 pasos:**
1. Definición del problema y restricciones (15-45 min)
2. Generación individual de ideas (10 min, **divergir**)
3. Presentación y crítica (3 min por persona)
4. Iteración y refinamiento en pares (10 min, **emerger**)
5. Generación de la idea del equipo (45 min, **converger**)

**Sistema de diseño** (para consistencia):
- Guías de estilo, bibliotecas de patrones, guía de marca, bibliotecas de activos (assets).
- Beneficios: mayor consistencia, mayor calidad, menores costos.

### 7.8 Crear PMV (Producto Mínimo Viable)
- Pregunta clave: ¿esa táctica producirá la meta deseada? ¿Con la menor cantidad de trabajo, cómo podemos comprobarlo?

**PMV para entender el valor de la idea:**
- Ir al grano: distilar la idea a su valor principal (dejar para después menús, recuperación de contraseña, etc.)
- Usar un claro llamado a la acción (call to action): las personas demuestran que valoran algo cuando muestran intención de usarlo o pagar por ello.
- Priorizar despiadadamente.
- Medir comportamiento y hablar con los usuarios.
- No reinventar la rueda.

**PMV para entender la implementación:**
- Funcional: crear un escenario realista integrando el PMV al resto de la app.
- Integrado con la analítica existente.
- Consistente con el resto de la app (para minimizar el sesgo hacia la novedad).

**Curva de la Verdad**: el aprendizaje real (verdad) aumenta progresivamente según el tipo de validación usada, de menor a mayor fidelidad:
**Fantasía → Conversación → Test de papel → Prototipo → Producto en vivo**

**Ejemplos de tipos de PMV:**
- **Landing page**: permite estimar demanda con una propuesta de valor clara y un botón de acción (mide conversión).
- **Función falsa** ("botón a ninguna parte"): cuando el costo de implementación es alto, se simula la función y se muestra un mensaje de "próximamente".
- **Mago de Oz**: se ofrece un PMV que parece completamente funcional, pero detrás hay un humano respondiendo manualmente (ej: "Alexa, busca calorías de la empanada" y una persona entrega el resultado).

**Prototipos:**
- Son una aproximación de la experiencia; deben ser **clickeables o tapeables**.
- Tipos: en papel, mockup de baja fidelidad, mockup de alta fidelidad, prototipos codificados con datos reales.

### 7.9 Investigar y Aprender
- Es momento de testear el PMV.
- La investigación es **continua y colaborativa**.
- **Descubrimiento colaborativo**: enfoque que envía a todo el equipo fuera del edificio para encontrarse y aprender directamente de los clientes.

**Pasos del proceso de investigación:**
1. Revisar preguntas, supuestos, hipótesis y PMV; decidir como equipo qué se quiere aprender.
2. Decidir con quién se conversará/observará.
3. Crear un guion de entrevistador.
4. Separar al equipo en pares, mezclando roles.
5. Armar cada par con la versión del PMV y enviarlos al encuentro con el cliente/usuario.
6. Un miembro entrevista mientras el otro toma notas.
7. Empezar con preguntas, conversación y observación.
8. Mostrar el PMV más tarde en la sesión y dejar que el cliente interactúe.
9. Tomar notas de la retroalimentación.
10. Cambiar roles cuando el entrevistador esté listo.
11. Al final, pedir contactos de otras personas que puedan dar retroalimentación valiosa.

- Las conversaciones regulares con clientes/usuarios **minimizan el tiempo** entre la creación de hipótesis, el diseño experimental y la retroalimentación.

**Ejemplo de calendario semanal de investigación:**
| Lunes | Martes | Miércoles | Jueves | Viernes |
|---|---|---|---|---|
| Empezar reclutamiento de usuarios; decidir qué testear | Refinar lo que será testeado | Refinar lo testeado; escribir guion del test; finalizar reclutamiento | **Testing Day**; revisar hallazgos con todo el equipo | Planificar el siguiente test |

## 8. Resumen del ciclo completo
El proceso de Lean UX es **iterativo y cíclico**, no lineal:

```
Metas, Supuestos, Hipótesis
        ↓
    Diseñarlo
        ↓
    Crear PMV
        ↓
Investigar y Aprender
        ↓
(vuelve a Metas, Supuestos, Hipótesis)
```

Cada vuelta del ciclo reduce incertidumbre, valida (o invalida) hipótesis con evidencia real de usuarios, y permite ajustar el producto con el menor desperdicio de esfuerzo posible.
