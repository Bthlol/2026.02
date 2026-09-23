# Apunte de estudio — Certamen 1: Redes de Computadores (INF-256)

Cubre Capítulo 1 (Introducción a Internet) y Capítulo 2 (Capa de Aplicación),
organizado con la misma lógica que usa el profesor en sus certámenes:
muchas preguntas de V/F y alternativas que ponen a prueba matices y
generalizaciones sospechosas ("siempre", "nunca", "todo dispositivo...").
Al final hay una sección dedicada a diseccionar ese tipo de afirmaciones.

---

# CAPÍTULO 1 — Introducción a Internet

## 1. ¿Qué es Internet? (dos descripciones)

**Descripción por componentes (la vista "de cableado"):**
- **Hosts / End Systems**: computadores, smartphones, tablets, TV, smartwatches — cualquier dispositivo que corre aplicaciones.
- **Enlaces de comunicación**: fibra, cobre, radio, satélite. Se caracterizan por su **ancho de banda** (bandwidth).
- **Packet switches**: routers y switches — reenvían paquetes.
- **Ruta (path)**: secuencia de enlaces y packet switches entre origen y destino.
- Internet es la **"red de redes"**: una red interconectada de ISPs.
- **Protocolos**: controlan el envío/recepción de información (TCP, IP, HTTP, 802.11).

**Descripción por servicios (la vista "de infraestructura"):**
- "Una infraestructura que proporciona servicios a las aplicaciones."
- Los End-Systems ofrecen una **interfaz de socket**: las reglas que le dicen a un programa cómo pedirle a Internet que entregue datos a otro programa en otro sistema final.

## 2. ¿Qué es un protocolo?

> "Un protocolo define el formato y el orden de los mensajes enviados y
> recibidos entre las entidades de la red, y las acciones tomadas en la
> transmisión y recibo de un mensaje."

Analogía humana: pedir la hora a alguien tiene un protocolo implícito
(saludo → pregunta → respuesta). Un servidor web y un navegador siguen
el mismo tipo de "guion" con HTTP.

## 3. Network Edge vs. Network Core

| | Network Edge | Network Core |
|---|---|---|
| Qué es | Hosts (clientes y servidores) + redes de acceso | Routers interconectados — "red de redes" |
| Función | Ejecutan las aplicaciones | Reenvían paquetes (packet switches) |
| Nota importante | Los servidores suelen estar en **data centers** | **NO** son servidores — son el conjunto de routers/switches que interconectan, no DNS ni servidores de contenido |

**Trampa clásica de examen:** decir que el Network Core "está compuesto por
servidores que soportan características de Internet, como DNS o
servidores de videojuegos" es **FALSO** — esos servidores son parte del
**Network Edge** (son hosts/end-systems). El Network Core es routers y
enlaces que interconectan, no aplicaciones.

## 4. Redes de acceso (cómo el Edge se conecta al primer router)

| Tecnología | Infraestructura | Velocidad típica | Nota |
|---|---|---|---|
| **DSL** | Compañía telefónica local, vía DSLAM | ~2.5 Mbps up / 24 Mbps down (asimétrico) | Distancia corta (5-15 km) |
| **Cable (HFC)** | TV por cable, vía CMTS | ~2 Mbps up / 30 Mbps down (asimétrico) | Medio **compartido** entre 500-5000 hogares; usa FDM para separar datos/TV |
| **FTTH** | Fibra hasta el hogar, vía ONT/OLT | Potencialmente Gbps | Splitter combina <100 hogares en una fibra compartida |
| **Ethernet** | LAN, cobre par trenzado | 100 Mbps usuarios / 1-10 Gbps servidores | Desempeño simétrico |
| **WiFi (802.11)** | LAN inalámbrica | hasta 54 Mbps | Radio de alcance de decenas de metros |
| **Celular (3G/4G/LTE)** | Telefonía celular | 1-10 Mbps | Decenas de km desde la estación base |

## 5. Medios físicos

- **Guiados** (la onda va dentro de un medio sólido): cobre par trenzado, cable coaxial, fibra óptica.
- **No guiados** (la onda se propaga por el aire/espacio): radio terrestre, satélite.

| Medio | Velocidad | Notas |
|---|---|---|
| Par trenzado | 10 Mbps - 10 Gbps | Barato, usado en LAN y DSL |
| Coaxial | Decenas de Mbps | Común en TV por cable |
| Fibra óptica | Cientos de Gbps | Inmune a interferencia electromagnética, baja atenuación, hasta 100 km sin repetidor — "pieza fundamental de los enlaces en Internet" |
| Radio terrestre | Variable | Afectado por pérdida de trayectoria, shadow fading, multipath fading |
| Satélite | Variable | Se usa donde no hay otro acceso |

## 6. Packet Switching vs. Circuit Switching

### Packet Switching (lo que usa Internet)
- **Store-and-forward**: el router debe recibir el **paquete completo** antes de retransmitirlo al siguiente enlace. Toma `L/R` segundos transmitir un paquete de tamaño `L` a una tasa `R`.
  - Ejemplo del apunte: con `L=7.5 Mbits`, `R=1.5 Mbps`, un salto (one-hop, sin propagación) tarda `L/R = 5s`; punto a punto con 2 saltos serían `2L/R`.
- Los paquetes usan la **capacidad total** del enlace al transmitirse (no una porción reservada).
- **Ventajas**: más usuarios pueden compartir la red, bueno para tráfico en ráfagas, sin configuración previa, más barato de implementar.
- **Desventaja**: posible congestión excesiva (no hay garantía de recursos).
- **Funciones clave del Network Core**:
  - **Forwarding**: mover un paquete de la entrada a la salida correcta *dentro de un router*.
  - **Routing**: determinar la *ruta completa* desde origen a destino (algoritmos globales).

### Circuit Switching (redes telefónicas tradicionales)
- Recursos **reservados** de extremo a extremo antes de transmitir (ej. TDM: multiplexación por división de tiempo, con "ranuras" o slots).
- Rendimiento **garantizado**, pero recursos dedicados **sin compartir** — quedan inactivos en silencios.
- **Ejemplo numérico típico**: 24 ranuras TDM sobre una línea de 1.536 Mbps → cada ranura = 64 kbps. Transmitir 640.000 bits toma `640000/64000 = 10s`, más 0.5s de tiempo de establecimiento = **10.5s total**. (Este cálculo es independiente del número de enlaces porque no se considera el delay de propagación.)

## 7. Estructura de Internet (cómo se conectan los ISP)

Evolución lógica (cada alternativa resuelve el problema de la anterior):
1. Conectar cada ISP de acceso con todos los demás → no escala.
2. Conectar cada ISP de acceso a un **ISP de tránsito global** → funciona, pero si es negocio viable aparecen competidores.
3. Para conectar a esos competidores entre sí: **IXP (Internet Exchange Points)**. También surgen **redes regionales**.
4. **Multi-homing**: un ISP se conecta a más de un proveedor, por redundancia.
5. **Proveedores de contenido** (Google, Microsoft, Akamai) arman su **propia red privada** para acercar contenido al usuario, muchas veces sin pasar por ISPs regionales de Tier-1.
6. En el centro: pocas redes **Tier-1** (Level 3, Sprint, NTT, Tata, etc.), con cobertura nacional/internacional.

**Trampa clásica:** "Los IXP satisfacen la necesidad de poder conectar ISP globales" — el rol de los IXP es conectar **ISPs competidores entre sí** (para intercambiar tráfico sin depender de un tránsito pagado), no específicamente "ISP globales" como concepto separado. Hay que leer la frase con cuidado: el IXP nace justamente porque *ya existen múltiples* ISP de tránsito global compitiendo, y necesitan un punto neutral donde interconectarse.

## 8. Performance: Delay, Loss, Throughput

*(Ver el diagrama "Las cuatro fuentes de delay" más arriba en la conversación.)*

### Las 4 fuentes de delay (por cada router/nodo que atraviesa un paquete)

| Componente | Símbolo | Fórmula / causa | Orden de magnitud |
|---|---|---|---|
| Procesamiento | `d_proc` | Chequeo de bits, determinar enlace de salida | microsegundos |
| Encolamiento | `d_queue` | Tiempo esperando en el buffer de salida; depende del **nivel de congestión** | variable |
| Transmisión | `d_trans` | `d_trans = L/R` (L=tamaño del paquete en bits, R=ancho de banda del enlace) | milisegundos |
| Propagación | `d_prop` | `d_prop = d/s` (d=distancia física, s≈2×10⁸ m/s en el medio) | milisegundos |

**Delay total de nodo = `d_proc + d_queue + d_trans + d_prop`**

### Analogía de la caravana (para entender transmisión vs. propagación)
- Autos = bits, caravana = paquete, cabina de peaje = router.
- El tiempo que tarda la caravana en "empujarse" completa a través de la cabina = tiempo de **transmisión**.
- El tiempo que tarda un auto en viajar entre cabinas = tiempo de **propagación**.
- Son procesos **independientes**: es posible que el primer auto llegue a la segunda cabina antes de que el último auto salga de la primera (si la velocidad de propagación es alta y el servicio de la cabina es lento) — esto ilustra que el paquete se puede estar transmitiendo en un enlace mientras el principio del paquete ya se está propagando.

### La razón `La/R` (intensidad de tráfico) y el encolamiento
- `L`: tamaño del paquete. `R`: capacidad del enlace. `a`: tasa promedio de llegada de paquetes.
- **`La/R ≈ 0`**: poco delay de cola en promedio.
- **`La/R ≈ 1`**: delay de cola grande en promedio.
- **`La/R > 1`**: llega más "trabajo" del que se puede procesar → delay **infinito** en promedio (el sistema es inestable).

### Packet Loss
- El buffer que precede al enlace tiene **capacidad finita**.
- Los paquetes que llegan a una cola **llena** se descartan (se pierden).
- Un paquete perdido puede retransmitirse (por el nodo anterior o por el origen) **o no** — depende del protocolo de capa superior (TCP retransmite, UDP no).

### Throughput
- Velocidad (bits/tiempo) a la que se transfieren bits entre emisor y receptor.
- Puede ser **instantáneo** (en un punto del tiempo) o **promedio** (sobre un período más largo).
- **Bottleneck link**: el enlace de la ruta que restringe el throughput de extremo a extremo.
- Con múltiples enlaces en serie de capacidades `R1, R2, ..., Rn`, el throughput end-to-end es `min(R1, R2, ..., Rn)`.
- Con conexiones compartiendo un enlace de acceso: `throughput por conexión ≈ min(Rc, Rs, R/10)` donde `Rc`=capacidad de acceso del cliente, `Rs`=capacidad de acceso del servidor, y `R/10` es una aproximación del reparto si 10 conexiones comparten el enlace troncal.
- En la práctica, el cuello de botella suele estar en `Rc` o `Rs` (el "último kilómetro"), porque el **núcleo de Internet está sobreaprovisionado**.

## 9. Principios arquitectónicos: capas

### Por qué capas
- Permiten manejar sistemas complejos: estructura explícita que identifica y relaciona piezas.
- Modularización → facilita mantenimiento y actualización.
- Un cambio en la implementación de una capa es **transparente** para el resto (mientras se respete la interfaz/servicio que ofrece).

### Internet Protocol Stack (5 capas) — la que usa Internet en la práctica

| Capa | Función | Ejemplos |
|---|---|---|
| **Aplicación** | Mensajes entre procesos de aplicación | HTTP, SMTP, FTP, DNS |
| **Transporte** | Transferencia de datos **proceso a proceso** | TCP, UDP |
| **Red (Network)** | Enrutamiento de datagramas de origen a destino | IP, protocolos de ruteo |
| **Enlace (Link)** | Transferencia de datos entre elementos **vecinos** de la red | Ethernet, 802.11 (WiFi), PPP |
| **Física** | Bits "en el cable" | — |

### ISO/OSI (7 capas) — modelo de referencia más "académico"
Agrega dos capas entre Transporte y Aplicación que el modelo de Internet no usa explícitamente:
- **Sesión**: sincronización, checkpointing, recuperación de datos intercambiados.
- **Presentación**: permite que las aplicaciones interpreten el significado de los datos (ej. cifrado, compresión).

**Trampa clásica:** confundir capas. Los **routers operan principalmente en
la capa de RED** (miran la dirección IP para hacer forwarding), no en la
capa de aplicación. Decir "los routers operan principalmente en la capa
de aplicación" es **FALSO**.

### Encapsulación
*(Ver el diagrama de arriba en la conversación.)* Cada capa, al bajar,
**envuelve** los datos de la capa superior agregando su propio
encabezado: Datos de aplicación → Segmento (TCP/UDP) → Datagrama (IP) →
Frame (Enlace). En el receptor, el proceso es inverso: cada capa quita
su encabezado antes de pasar los datos hacia arriba.

---

# CAPÍTULO 2 — Capa de Aplicación

## 1. Creando una aplicación de red

Solo se necesita software en los **end-systems** — nunca hace falta
implementar nada en los dispositivos del network-core (routers). Esto es
justamente lo que permite que cualquiera pueda crear una app nueva sin
pedirle permiso a la infraestructura de Internet.

## 2. Arquitecturas: Cliente-Servidor vs. P2P

| | Cliente-Servidor | P2P |
|---|---|---|
| Servidor | Siempre activo, IP **permanente**, suele estar en un data center (para escalar) | No requiere un servidor siempre activo |
| Cliente | Se comunica con el servidor (a veces a través de un tercero); puede tener IP **dinámica** | Los end-systems ("peers") se comunican directamente entre sí, de forma arbitraria |
| Escalabilidad | Requiere más servidores/infraestructura para escalar | **Self-scalability**: cada nuevo miembro trae recursos (pero también demanda) |
| Estabilidad de conexión | Servidor estable | Los peers se conectan **intermitentemente** y cambian de IP → más difícil de gestionar |

## 3. Comunicación de procesos

- **Proceso**: programa ejecutándose dentro de un host.
- Mismo host → los procesos se comunican vía IPC (definida por el SO).
- Hosts distintos → se comunican intercambiando **mensajes** a través de la red.
- **Proceso cliente**: inicia la comunicación. **Proceso servidor**: espera ser contactado.

### Sockets
- Los procesos envían/reciben mensajes **exclusivamente** a través de un socket.
- Analogía: el socket es una **puerta**. El proceso empuja el mensaje por la puerta y confía en que la infraestructura de transporte lo entregue al socket del otro lado.
- Un socket es la **interfaz entre la capa de aplicación y la capa de transporte** (el API entre esas dos capas).

### Direccionamiento de procesos
- Un host tiene una dirección **IP** (identificador único del dispositivo).
- La IP **sola no basta** para identificar un proceso, porque en un mismo host pueden correr muchos procesos a la vez.
- El identificador completo de un proceso = **IP + número de puerto**.
- Puertos conocidos: HTTP=80, SMTP=25, SSH=22, DNS=53.

**Trampa clásica:** "todo dispositivo tiene una dirección IP de 32 bits" es
una generalización **FALSA** — esa es la longitud de **IPv4**; IPv6 usa
128 bits. Cualquier afirmación absoluta sobre "todo dispositivo" con un
tamaño de dirección fijo es sospechosa.

## 4. ¿Qué servicio de transporte necesita una app?

Cuatro dimensiones a evaluar:

| Dimensión | Ejemplo de app exigente | Ejemplo de app tolerante |
|---|---|---|
| **Confiabilidad de datos** | Transferencia de archivos, transacciones web (requieren 100% confiable) | Audio (puede tolerar algo de pérdida) |
| **Throughput** | Multimedia (requiere un mínimo para ser efectiva — apps "sensibles al ancho de banda") | Apps "elásticas": usan cualquier throughput que consigan |
| **Timing** | Telefonía IP, juegos interactivos (requieren poco delay) | — |
| **Seguridad** | Cifrado, integridad, autenticación | — |

**La más sensible al packet loss**, entre las opciones típicas de examen,
es el **streaming de video en tiempo real** — no porque no le importe
perder datos, sino porque no puede permitirse el delay que generaría
retransmitir (a diferencia de la descarga de un archivo, que sí puede
esperar).

### TCP vs. UDP

| | TCP | UDP |
|---|---|---|
| Confiabilidad | Transporte **fiable** | **No** garantiza confiabilidad |
| Control de flujo | Sí (el emisor no abruma al receptor) | No |
| Control de congestión | Sí (el emisor infiere si la red está sobrecargada) | No |
| Conexión | **Orientado a conexión**: requiere configuración previa entre cliente y servidor | Sin conexión |
| Garantías que NO da (ninguno de los dos) | timing, throughput mínimo, seguridad | timing, throughput mínimo, seguridad |

**Trampa clásica:** "si un paquete se pierde en una comunicación UDP, se
retransmite automáticamente" es **FALSO** — UDP no ofrece ningún
mecanismo de retransmisión; eso es responsabilidad de la aplicación si
la necesita.

## 5. Protocolos de capa de aplicación

- Definen: **tipos de mensajes** (ej. request/response), **sintaxis** (qué campos hay y cómo se delimitan), **semántica** (qué significa cada valor de cada campo), y las **reglas** de cuándo y cómo se envían/responden los mensajes.
- **Protocolos abiertos**: definidos en RFCs, permiten interoperabilidad (HTTP, SMTP).
- **Protocolos privados**: propietarios (Skype, Teams, Zoom).

## 6. HTTP — HyperText Transfer Protocol

- Protocolo de **capa de aplicación**, definido en RFC 1945 y RFC 2616.
- Arquitectura cliente-servidor: navegador = cliente, web server (Apache, Nginx, IIS) = servidor.
- HTTP define **solo** el protocolo de comunicación — no cómo el navegador interpreta/renderiza la página.
- Una página web = un HTML base + objetos referenciados (imágenes, etc.), cada objeto direccionable por una URL.
- **Usa TCP** como transporte: el cliente abre la conexión TCP primero, luego intercambian mensajes vía sus sockets.
- El servidor **no guarda estado** de peticiones anteriores → **stateless protocol**.

### Conexiones no persistentes vs. persistentes

| | No persistente | Persistente |
|---|---|---|
| Objetos por conexión TCP | Como máximo **uno** | Varios objetos en una sola conexión |
| Cierre | Se cierra tras recibir el objeto | Se mantiene abierta (con timeout) |
| Descarga de varios objetos | Requiere **múltiples** conexiones TCP (a veces en paralelo) | Un solo RTT extra para todos los objetos referenciados |

**Tiempo de respuesta HTTP no persistente** = `2×RTT + tiempo de transmisión del archivo`
(1 RTT para establecer la conexión TCP + 1 RTT para la solicitud/primeros bytes + tiempo de transmitir el archivo).

**Definición de RTT**: tiempo que le toma a un paquete pequeño viajar del
cliente al servidor y de vuelta (round trip).

**Trampa clásica sobre RTT** (factores que SÍ o NO contribuyen): el RTT se
compone de tiempo de **propagación** + tiempo de **transmisión** + tiempo
de **encolamiento** en cada salto. El **tamaño del archivo solicitado**
NO es parte del RTT en sí — afecta el tiempo de transmisión del archivo
completo (que se suma aparte), pero el RTT como concepto se mide con un
paquete pequeño.

### Códigos de estado HTTP

| Código | Significado |
|---|---|
| **200 OK** | Solicitud exitosa |
| **301 Moved Permanently** | El objeto se movió, nueva ubicación en el mensaje |
| **400 Bad Request** | El servidor no entendió el mensaje |
| **404 Not Found** | Documento no encontrado |
| **505** | Versión HTTP no soportada |

**Trampa clásica:** "un código 404 indica un error del lado del
servidor" es **FALSO** — los códigos 4xx son errores del **cliente**
(la petición estaba mal, o el recurso no existe); los errores del
servidor son de la familia **5xx**.

### HTTP/2
- Objetivo: reducir latencia permitiendo **multiplexar** consultas y respuestas HTTP sobre **una sola conexión TCP**.
- **No cambia** métodos, códigos de estado ni campos de cabecera — cambia **cómo se formatean y transmiten** los datos.
- Introduce una **Binary Framing Layer**: los mensajes se parten en frames binarios pequeños que se intercalan en la misma conexión (más eficiente que ASCII, pero más difícil de inspeccionar a simple vista — se necesita algo como Wireshark).
- Permite que **múltiples requests y responses** se entrelacen en paralelo, sin bloquearse entre sí ("no head-of-line blocking" a nivel de HTTP).

**Trampa clásica:** "HTTP/2 permite múltiples flujos de datos
concurrentes en una sola conexión TCP" es la afirmación **correcta**
sobre la diferencia HTTP/1.0 vs 2.0 — no es que HTTP/1.0 use binario y
2.0 ASCII (es al revés: 1.0 es ASCII, 2.0 introduce framing binario), ni
que 2.0 "solo permita IPv6" (no tiene relación con eso), ni que HTTP/1.0
soporte conexiones persistentes por defecto en *todas* sus versiones
(HTTP/1.0 es no-persistente por defecto; recién HTTP/1.1 lo cambia).

### Cookies
Cuatro componentes del mecanismo: (1) línea de cabecera en la respuesta
HTTP que crea la cookie, (2) línea de cabecera en la siguiente solicitud
que la reenvía, (3) el archivo de cookie se guarda en el host del
usuario (gestionado por el navegador), (4) puede además guardarse en una
base de datos en el backend del sitio. Usos: autorización, carritos de
compra, recomendaciones, estado de sesión (ej. correo web).

### Web Caches (Proxy Server)
- Entidad de red que satisface una solicitud HTTP **en nombre del** servidor de origen, guardando copias de los objetos más solicitados.
- El navegador manda **todas** sus solicitudes al Web Cache. Si el objeto está en caché (**hit**), lo devuelve directo; si no (**miss**), lo pide al servidor de origen, lo guarda, y luego lo entrega al cliente.
- Normalmente lo instala el **ISP** (universidad, empresa, ISP residencial).
- Beneficios: reduce tiempo de respuesta al cliente, reduce tráfico, y reduce accesos al servidor de origen.
- *(Este es exactamente el concepto que resolvieron en el certamen anterior con `webcache.py` — TTL, MISS/HIT, y la cabecera `Connection: close` para saber cuándo terminó la respuesta.)*

## 7. Correo Electrónico

- Mecanismo **asíncrono**.
- Componentes: **User Agents** (lectores de correo: Outlook, Thunderbird), **Mail Servers**, **SMTP**.
- Cada destinatario tiene un **buzón** en un mail server; el servidor autentica usuarios y mantiene una **cola de mensajes salientes**.

### SMTP [RFC 2821]

- Protocolo de capa de aplicación para correo, **usa TCP**, puerto **25**.
- Arquitectura cliente/servidor: el **cliente** SMTP corre en el servidor de correo del **remitente**; el **servidor** SMTP corre en el servidor de correo del **destinatario**.
- Tres fases: **handshaking**, **transferencia de mensajes**, **cierre**.
- Los mensajes deben estar en **ASCII de 7 bits**.
- Usa **conexiones persistentes**.
- El servidor SMTP usa **CRLF.CRLF** para determinar el final del mensaje.

**Flujo típico (Alice → Bob):**
1. Alice redacta el mensaje en su User Agent.
2. El User Agent de Alice lo envía a su servidor de correo, que lo pone en cola.
3. El **cliente** SMTP (del lado de Alice) abre una conexión TCP con el servidor de correo de Bob.
4. El cliente SMTP envía el mensaje por esa conexión.
5. El servidor de correo de Bob pone el mensaje en el buzón de Bob.
6. Bob invoca su User Agent para leerlo.

### Comparación SMTP vs. HTTP (pregunta MUY típica)

| | HTTP | SMTP |
|---|---|---|
| Modelo | **pull** (el cliente pide el objeto) | **push** (el remitente empuja el mensaje) |
| Transporte | TCP | TCP |
| Interacción | comando/respuesta en **ASCII**, con códigos de estado | comando/respuesta en **ASCII**, con códigos de estado |
| Empaquetado de objetos | cada objeto en su **propio** mensaje de respuesta | **varios** objetos en un solo mensaje de varias partes |

**Sobre la afirmación de ejemplo** ("HTTP y SMTP tienen en común que son
orientados a la conexión, usan TCP, y su interacción comando/respuesta
es en ASCII"): **es verdadera** — el apunte lo dice explícitamente
("Ambos tienen interacción de comando/respuesta ASCII, códigos de
estado"). La diferencia real entre ambos NO está en eso, sino en
**pull vs. push** y en cómo empaquetan múltiples objetos.

### Protocolos de acceso al correo (recuperación desde el servidor)

| | POP3 [RFC 1939] | IMAP [RFC 1730] |
|---|---|---|
| Fases | Autorización (`user`, `pass` → `+OK`/`-ERR`) y transacción (`list`, `retr`, `dele`, `quit`) | Más funciones: manipulación de mensajes almacenados en el servidor |
| Modo típico | "Descargar y borrar" (Bob no puede releer el correo si cambia de cliente) | Mantiene **todos** los mensajes en el servidor |
| Organización | No permite carpetas | Permite **organizar mensajes en carpetas** en el servidor |
| Estado entre sesiones | **No tiene estado** entre sesiones | **Mantiene** el estado del usuario a lo largo de la sesión (nombres de carpetas, asignación ID↔carpeta) |
| Variante "descargar y mantener" | Existe, pero deja copias en distintos clientes (no sincronizado) | No aplica (todo vive en el servidor) |

**Trampa clásica (idéntica a la del ejemplo):** "el gran beneficio de
POP3 sobre IMAP es que usa **compresión de datos** para ahorrar espacio
en el servidor" — **FALSO**. POP3 no comprime nada; su forma de "ahorrar
espacio" es literalmente **borrar** el mensaje del servidor tras
descargarlo (modo por defecto), lo cual es justo lo contrario de una
ventaja moderna (Bob pierde acceso al correo desde otro dispositivo). La
compresión no es un concepto que aparezca en la comparación POP3/IMAP en
absoluto.

## 8. DNS — El directorio de Internet

- Traduce **hostname** (ej. `www.usm.cl`, preferido por personas) ↔ **dirección IP** (ej. `200.1.19.11`, preferida por routers).
- Es una **base de datos distribuida y jerárquica**, implementada como una jerarquía de servidores DNS.
- Es un protocolo de **capa de aplicación**: los hosts y servidores de nombres se comunican para resolver nombres — aunque es una función básica de Internet, no vive en el network core.
- Se ejecuta normalmente sobre **UDP, puerto 53** — pero **puede usar TCP** cuando la respuesta es demasiado grande para un datagrama UDP (ej. transferencias de zona, o respuestas con muchos registros).
- Otros servicios de DNS: **alias** (nombres cortos para hosts con nombres complicados), **alias de servidor de correo**, y **distribución de carga** (un mismo hostname con varias IPs, ej. para servidores web replicados).

### Jerarquía de servidores DNS
1. **Root name servers**: contactados cuando el servidor local no puede resolver; administrados por 12 organizaciones independientes.
2. **TLD (Top-Level Domain) servers**: responsables de `.com`, `.org`, `.edu`, y dominios de país (`.cl`, `.uk`, `.ar`). Ej: VeriSign para `.com`.
3. **Authoritative DNS servers**: los propios servidores DNS de cada organización, con el mapeo autorizado de sus hosts.
4. **Local DNS name server**: NO pertenece estrictamente a la jerarquía. Cada ISP tiene uno ("servidor de nombres por defecto"). Actúa como **proxy**: reenvía la consulta a la jerarquía, y mantiene una **caché local** (que puede estar desactualizada).

### Ejemplo de resolución (cliente quiere IP de `www.amazon.com`)
1. Consulta servidor **raíz** → responde con el servidor `.com`.
2. Consulta servidor TLD **.com** → responde con el servidor de `amazon.com`.
3. Consulta servidor **authoritative de amazon.com** → responde con la IP.

### Consulta Iterada vs. Recursiva

| | Iterada | Recursiva |
|---|---|---|
| Quién hace el trabajo | El **cliente** repregunta a cada servidor sucesivo ("no sé esto, pregúntale a este otro") | El servidor contactado **delega** la consulta hacia arriba/abajo por el cliente |
| Carga | Repartida en el cliente/servidor local | Carga pesada en los niveles superiores de la jerarquía si todos piden recursivo |

- Una vez que **cualquier** servidor de nombres aprende un mapeo, lo **cachea**, con un **TTL** que determina cuánto tiempo puede usarse esa entrada antes de expirar.
- Por esto, los servidores TLD suelen estar cacheados en los servidores locales — los servidores raíz casi no se visitan directamente en la práctica.

**Trampa clásica del ejemplo (TTL cerca de expirar + segundo estudiante):**
Si el DNS local **ya tiene** la IP cacheada (aunque el TTL esté por
expirar), un segundo estudiante que consulta el mismo hostname
**minutos después** obtiene la respuesta directo de esa caché → su
ventaja es **menor latencia** (no tiene que ir hasta la jerarquía de
nuevo), siempre que el TTL no haya expirado todavía.

### DNS Records (Resource Records)

Formato: `(Name, Value, Type, TTL)`

| Type | Name | Value |
|---|---|---|
| **A** | hostname | dirección IP |
| **CNAME** | alias | nombre canónico (el "real") |
| **NS** | dominio (ej. `foo.com`) | hostname del servidor de nombres autorizado para ese dominio |
| **MX** | nombre | hostname del servidor de correo asociado |

Ejemplos del apunte: `(foo.com, dns.foo.com, NS)`, `(foo.com,
mail.bar.foo.com, MX)`.

**Trampa clásica:** el flujo típico "¿qué pasa primero al ingresar una
URL?" es: **se consulta la caché local del sistema primero** (no se
manda nada a la red todavía) — recién si no está ahí, se activa el
cliente DNS para consultar al servidor DNS local, y de ahí hacia la
jerarquía si hace falta.

## 9. Aplicaciones P2P

- Sin servidor siempre activo; los end-systems se comunican directo entre sí; conexión **intermitente** y con IP cambiante.
- Ejemplos: distribución de archivos (BitTorrent), streaming (KanKan), VoIP (Skype).
- Características: **self-organization**, alto grado de descentralización, **tolerancia a fallas** (sin punto único de falla), múltiples dominios administrativos, escalabilidad.

### Tiempo de distribución de archivo: Cliente-Servidor vs. P2P

Sea `F` el tamaño del archivo, `u_s` la tasa de subida del servidor,
`u_i` la tasa de subida del peer `i`, `d_min` la tasa de descarga mínima
entre los `N` clientes.

**Cliente-Servidor:**
- El servidor debe enviar `N` copias secuencialmente: tiempo para enviar todas ≥ `NF/u_s`.
- Cada cliente debe descargar una copia completa: tiempo ≥ `F/d_min`.
- **Tiempo total ≥ max(NF/u_s, F/d_min)** — y como se envían secuencialmente, en la práctica domina `NF/u_s` cuando `N` es grande: **crece linealmente con N**.

**P2P:**
- El servidor solo necesita subir **una** copia: `F/u_s`.
- Cada cliente necesita al menos `F/d_min` para descargar su copia.
- Como agregado, se deben descargar `NF` bits entre todos, a una tasa máxima limitada por `u_s + Σu_i` (la capacidad de subida total del sistema, servidor + todos los peers).
- **Tiempo total ≥ max(F/u_s, F/d_min, NF/(u_s + Σu_i))** — la clave es que a medida que `N` crece, también crece la capacidad de subida agregada (cada peer que descarga también puede subir), por lo que P2P **escala mucho mejor** que cliente-servidor.

**Esta es la ventaja central del modelo P2P frente a cliente-servidor:
escalabilidad mediante recursos distribuidos** (cada nuevo peer trae su
propia capacidad de subida, en vez de solo agregar carga a un servidor
fijo).

### Tipos de topología P2P

| Tipo | Características |
|---|---|
| **No estructurada** | Topología sin restricciones; ubicación de datos sin correlación con la topología; usa **controlled flooding** (BFS, random walk, iterative deepening); no escala bien con muchas consultas; ineficiente para contenido poco popular |
| **Super-peers** | Arquitectura híbrida: algunos peers con más recursos hacen de "mini-servidores" (QoS, control de acceso); ej. Napster, Gnutella2, Skype |
| **Estructurada** | Topología definida; cada peer tiene un ID único en un espacio numérico (ej. 128 bits); a cada peer se le asigna un rango de llaves; usa una **DHT (Distributed Hash Table)**; búsqueda eficiente, típicamente en tiempo **logarítmico** respecto al tamaño de la red |

---

# Trampas comunes de examen — cómo detectarlas

Basado en el patrón real del certamen de ejemplo, estas son las señales
de alerta que conviene entrenar el ojo para detectar:

1. **Cuantificadores absolutos ("siempre", "nunca", "todo", "cualquiera")
   casi siempre están puestos ahí para que la frase sea falsa.**
   - "Siempre que la velocidad de llegada sea mayor a la de salida, la
     cola se llenará y habrá pérdida" → depende de la **duración** del
     desbalance y del **tamaño del buffer**; una ráfaga corta no
     necesariamente llena la cola. El "siempre" lo vuelve falso.
   - "Todo dispositivo tiene una IP de 32 bits" → ignora IPv6 (128 bits).
   - "Aunque la cola sea infinita, siempre es posible que haya pérdida" →
     con buffer infinito, no hay pérdida por **desborde** (aunque el
     delay promedio se vuelva infinito si `La/R ≥ 1`); hay que
     distinguir "pérdida por buffer lleno" de otras causas de pérdida.

2. **Confundir capas o entidades de la arquitectura.**
   - Routers en capa de aplicación (son capa de **red**).
   - Network Core como "servidores" (son **routers/packet switches**).

3. **Números de puerto intercambiados.** SMTP=25, HTTP=80, DNS=53,
   SSH=22 — el examen de ejemplo puso deliberadamente "SMTP usa el
   puerto 53" (que es el de DNS) para ver si memorizaste bien.

4. **Atribuir una ventaja real a un mecanismo que no la produce.** POP3
   sí ahorra espacio en el servidor (al borrar tras descargar), pero
   NO por compresión — la causa mecánica importa, no solo la conclusión.

5. **Relaciones de proporcionalidad que no son lineales.** Duplicar la
   capacidad de un enlace no necesariamente reduce la congestión "a la
   mitad" — la relación entre capacidad y delay de cola es no lineal
   (ver la curva de `La/R`), especialmente cerca de `La/R ≈ 1`.

6. **Errores de cliente vs. servidor en códigos HTTP.** Familia 4xx =
   error del **cliente** (petición mal formada o recurso inexistente),
   familia 5xx = error del **servidor**.

7. **Confundir "quién tiene la ventaja" en un cambio de arquitectura.**
   Ej.: en DNS con TTL a punto de expirar, la ventaja del *segundo*
   consultante es **menor latencia** (por la caché), no algo relacionado
   con seguridad o ancho de banda.

---

# Cómo estudiar esto para la parte práctica de mañana

Este apunte es la mitad **teórica**. Para la parte **práctica** (rellenar
código de sockets TCP/UDP), ya tienes el material separado que armamos
antes: el mapa de conexiones del laboratorio, los ejercicios de práctica
con código real, y la comparación con el certamen anterior (Web Cache).
Vale la pena repasar en particular la sección de "Servicio de transporte
según la app" de este capítulo (arriba) junto con esos ejercicios: ahí
es donde teoría y práctica se cruzan — por ejemplo, entender *por qué*
HTTP usa TCP y *por qué* el heartbeat del laboratorio usa UDP son la
misma pregunta ("¿qué garantías necesita esta aplicación?") vista desde
dos ángulos distintos del curso.
