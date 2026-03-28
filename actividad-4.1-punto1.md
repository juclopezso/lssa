# Actividad 4.1 — Punto 1: Clasificación de componentes

## Criterio de distinción

- **Software**: existe como programa ejecutable, sin forma física dedicada
- **Hardware**: dispositivo de cómputo físico dedicado con firmware/software embebido
- **Físico**: elemento que opera principalmente mediante principios mecánicos, eléctricos o electromagnéticos, sin lógica de cómputo significativa

---

## T1 — Presentation

| Componente | Tipo | Justificación |
|---|---|---|
| Portal | Software | Aplicación web para pasajeros — se ejecuta como servicio de software |
| Dashboard | Software | Interfaz de monitoreo para operadores — aplicación de software |
| Driver Interface | Hardware | Display físico instalado en la cabina del tren (DMI — Driver Machine Interface). Es un dispositivo embebido con pantalla táctil dedicada, no una app en un navegador |

## T2 — Communication

| Componente | Tipo | Justificación |
|---|---|---|
| API Gateway | Software | Componente de software que enruta peticiones entre capas de presentación y lógica |

## T3 — Logic

| Componente | Tipo | Justificación |
|---|---|---|
| passengers-ms | Software | Microservicio de dominio |
| routes-ms | Software | Microservicio de dominio |
| trains-ms | Software | Microservicio de dominio |
| position-time-ms | Software | Microservicio operacional |
| tickets-ms | Software | Microservicio operacional |
| MAS (Movement Authority Service) | Software | Servicio que autoriza movimiento de trenes — lógica de negocio crítica ejecutada como software |

## T4 — Data

| Componente | Tipo | Justificación |
|---|---|---|
| passengers-db | Software | Base de datos |
| routes-db | Software | Base de datos |
| trains-db | Software | Base de datos |
| position-time-db | Software | Base de datos |
| tickets-db | Software | Base de datos para el microservicio tickets-ms — almacena información de boletos/reservas |
| Data Lake | Software | Almacén de datos operacionales e históricos agregados |

## T5 — Physical

| Componente | Tipo | Justificación |
|---|---|---|
| Onboard Unit (OBU) | Hardware | Computador embarcado a bordo del tren — dispositivo de cómputo físico dedicado (EVC — European Vital Computer) |
| Train Sensor | Físico | Dispositivos físicos (odómetros, acelerómetros, radar doppler) que detectan velocidad y posición mediante principios mecánicos/eléctricos |
| Brake Actuator | Físico | Actuador mecánico/neumático que ejecuta el frenado sobre el tren |
| Balise | Físico | Transponder pasivo instalado en la vía — dispositivo electromagnético sin capacidad de cómputo propia, se activa al paso del tren |

---

# Actividad 4.1 — Punto 2: Conectores por tier

## T1 — Presentation

| Conector | Componentes | Información | Dirección |
|---|---|---|---|
| Consulta de rutas/horarios | Portal → API Gateway | Solicitudes de búsqueda de rutas, horarios y disponibilidad | Portal → API Gateway |
| Visualización de estado de trenes | API Gateway → Dashboard | Posiciones, velocidades, estados operacionales de trenes en tiempo real | API Gateway → Dashboard |
| Comandos del maquinista | Driver Interface ↔ Onboard Unit (T5) | Confirmaciones de autorización de movimiento, selección de nivel ETCS, ingreso de datos del tren | Bidireccional |

## T2 — Communication

| Conector | Componentes | Información | Dirección |
|---|---|---|---|
| Enrutamiento de peticiones de presentación | API Gateway → Microservicios (T3) | Peticiones HTTP/REST de los clientes hacia los servicios de lógica de negocio | API Gateway → T3 |
| Respuestas de servicios | Microservicios (T3) → API Gateway | Datos de respuesta (JSON) con información de pasajeros, rutas, trenes, tickets | T3 → API Gateway |

## T3 — Logic

| Conector | Componentes | Información | Dirección |
|---|---|---|---|
| Consulta de posición para autorización | position-time-ms → MAS | Datos de posición y tiempo de trenes para calcular autorizaciones de movimiento | position-time-ms → MAS |
| Validación de tren en ruta | trains-ms → routes-ms | Datos del tren (características, capacidades) para validar compatibilidad con una ruta asignada | trains-ms → routes-ms |
| Emisión de autorización de movimiento | MAS → Onboard Unit (T5) | Movement Authority: distancia permitida, velocidad máxima, punto de frenado | MAS → OBU |

## T4 — Data

| Conector | Componentes | Información | Dirección |
|---|---|---|---|
| Lectura/escritura de datos de dominio | Microservicios (T3) ↔ Bases de datos (T4) | Operaciones CRUD sobre datos de pasajeros, rutas, trenes, tickets | Bidireccional |
| Ingesta al Data Lake | Bases de datos (T4) → Data Lake | Datos históricos y operacionales agregados para análisis | DBs → Data Lake |

## T5 — Physical

| Conector | Componentes | Información | Dirección |
|---|---|---|---|
| Telemetría de sensores | Train Sensor → Onboard Unit | Velocidad, posición, aceleración, estado de ruedas | Train Sensor → OBU |
| Comando de frenado | Onboard Unit → Brake Actuator | Órdenes de frenado (servicio o emergencia) calculadas por el EVC | OBU → Brake Actuator |
| Lectura de baliza | Balise → Onboard Unit | Perfil de vía, autorización de movimiento, información de señalización | Balise → OBU (unidireccional) |



# Actividad 4.1 — Punto 3: Conexion entre capas

Los conectores intranivel, como la comunicación entre microservicios en la capa lógica (T₃), implican un alto grado de interacción horizontal, lo que incrementa la complejidad del sistema debido a la necesidad de coordinación, consistencia de datos y manejo de fallos distribuidos.

Por otro lado, los conectores internivel, como la interacción entre la capa de presentación (T₁), comunicación (T₂) y lógica (T₃), reflejan una arquitectura en capas que favorece la separación de responsabilidades, la mantenibilidad y la escalabilidad. Sin embargo, estos conectores también pueden introducir latencia y puntos críticos de fallo, como el API Gateway.

En el caso de sistemas críticos, los conectores internivel entre la lógica (T₃) y el sistema físico (T₅) implican requisitos estrictos de confiabilidad, disponibilidad y tiempo real, ya que afectan directamente la operación segura del sistema.

## Conectores intra-tier (dentro del mismo nivel)

| Tier | Conector | Componentes | Tipo |
|---|---|---|---|
| T1 | Sincronización de estado operacional | Dashboard ↔ Driver Interface (vía OBU) | Intra-tier (indirecta) |
| T3 | Coordinación entre microservicios | passengers-ms ↔ tickets-ms | Intra-tier |
| T3 | Consulta de posición para autorización | position-time-ms → MAS | Intra-tier |
| T3 | Validación de tren en ruta | trains-ms → routes-ms | Intra-tier |
| T4 | Ingesta al Data Lake | Bases de datos → Data Lake | Intra-tier |
| T5 | Telemetría de sensores | Train Sensor → Onboard Unit | Intra-tier |
| T5 | Comando de frenado | Onboard Unit → Brake Actuator | Intra-tier |
| T5 | Lectura de baliza | Balise → Onboard Unit | Intra-tier |

## Conectores cross-tier (entre niveles)

| Conector | Componentes | Tiers | Tipo |
|---|---|---|---|
| Peticiones de usuario | Portal / Dashboard (T1) → API Gateway (T2) | T1 → T2 | Cross-tier |
| Enrutamiento a servicios | API Gateway (T2) → Microservicios (T3) | T2 → T3 | Cross-tier |
| Persistencia de datos | Microservicios (T3) ↔ Bases de datos (T4) | T3 ↔ T4 | Cross-tier |
| Emisión de Movement Authority | MAS (T3) → Onboard Unit (T5) | T3 → T5 | Cross-tier |
| Reporte de posición | Onboard Unit (T5) → position-time-ms (T3) | T5 → T3 | Cross-tier |
| Interfaz del maquinista | Driver Interface (T1) ↔ Onboard Unit (T5) | T1 ↔ T5 | Cross-tier |

## Implicaciones arquitectónicas

### Intra-tier

- **Determinismo**: los componentes dentro de T5 (sensores, OBU, actuadores) deben operar con tiempos de respuesta deterministas para garantizar la seguridad — un retraso en la orden de frenado puede ser catastrófico.
- **Autonomía de supervivencia**: el tren (T5) puede operarse autónomamente en caso de fallo de conexión con capas superiores. El OBU tiene lógica suficiente para aplicar frenado de emergencia y mantener operación segura sin depender de T3.
- **Cohesión y confianza**: los componentes dentro de un mismo tier comparten un nivel de confianza alto. En T5, el OBU confía en los datos del Train Sensor sin necesidad de validación externa. En T3, los microservicios se comunican dentro de una misma red de confianza.

### Cross-tier

- **Resiliencia**: la arquitectura en capas permite que el sistema continúe operando ante fallos parciales. Si T1 (presentación) falla, los trenes siguen operando. Si la comunicación T3→T5 se pierde, el OBU aplica políticas de seguridad autónomas.
- **Fronteras de seguridad — Zero Trust**: cada cruce entre tiers constituye una frontera de seguridad. La comunicación T3↔T5 (MAS → OBU) requiere cifrado, autenticación mutua y verificación de integridad. No se asume confianza implícita entre capas.
- **Protocolos de comunicación**: cada frontera cross-tier puede usar protocolos distintos — HTTP/REST entre T1-T2-T3, protocolos de base de datos entre T3-T4, y GSM-R/Euroradio con protocolos seguros entre T3-T5.
- **Escalabilidad**: la separación en tiers permite escalar cada capa de forma independiente. T3 puede escalar horizontalmente (más instancias de microservicios) sin afectar T5, y T4 puede replicar bases de datos sin modificar T3.

---

# Actividad 4.1 — Punto 4: Componentes incompletos o faltantes

## T1 — Presentation

| Componente faltante | Tipo | Justificación |
|---|---|---|
| Mobile App | Software | Aplicación móvil para pasajeros — compra de tickets, consulta de horarios, notificaciones en tiempo real |
| Maintenance Console | Software | Interfaz para equipos de mantenimiento — gestión de alertas, programación de intervenciones sobre trenes y vía |

## T2 — Communication

| Componente faltante | Tipo | Justificación |
|---|---|---|
| Message Broker / Event Bus | Software | Middleware de mensajería asíncrona (ej. Kafka, RabbitMQ) para desacoplar microservicios en T3 y permitir comunicación basada en eventos |
| GSM-R Gateway | Hardware | Punto de entrada a la red de radio ferroviaria GSM-R — traduce comunicación entre el mundo IP (T3) y el protocolo Euroradio hacia los trenes (T5) |
| Load Balancer | Software | Distribuye tráfico entre instancias de microservicios para garantizar disponibilidad y escalabilidad |

## T3 — Logic

| Componente faltante | Tipo | Justificación |
|---|---|---|
| alerts-ms | Software | Microservicio de alertas y notificaciones — gestión de alarmas operacionales, incidentes y emergencias |
| scheduling-ms | Software | Microservicio de planificación — gestión de horarios, asignación de trenes a rutas, planificación de turnos |

## T4 — Data

| Componente faltante | Tipo | Justificación |
|---|---|---|
| tickets-db | Software | Ya identificada anteriormente — base de datos del microservicio tickets-ms |
| alerts-db | Software | Base de datos para persistir alertas, incidentes y su historial de resolución |
| Event Store | Software | Registro inmutable de eventos del sistema — necesario para auditoría, trazabilidad y reconstrucción de estado en un sistema crítico de seguridad |

## T5 — Physical

| Componente faltante | Tipo | Justificación |
|---|---|---|
| Onboard Radio Unit | Hardware | Equipo de radio GSM-R embarcado en el tren — necesario para la comunicación bidireccional en ETCS Nivel 2 y 3 |



# Actividad 4.3 — Reflexión

## Punto 1: Complejidad arquitectónica por tier

La capa que concentra la mayor complejidad arquitectónica es **T3 — Logic**.

### Evidencia cuantitativa

| Tier | Componentes (ref + propuestos) | Conectores intra-tier | Conectores cross-tier |
|---|---|---|---|
| T1 — Presentation | 3 + 2 = 5 | 1 | 3 (→T2, ↔T5) |
| T2 — Communication | 1 + 3 = 4 | 0 | 2 (←T1, →T3) |
| **T3 — Logic** | **6 + 2 = 8** | **3** | **4 (←T2, ↔T4, →T5, ←T5)** |
| T4 — Data | 5 + 3 = 8 | 1 | 1 (↔T3) |
| T5 — Physical | 4 + 1 = 5 | 3 | 3 (↔T1, ←T3, →T3) |

Aunque T4 iguala en cantidad de componentes, sus elementos son pasivos (bases de datos, almacenes). T3, en cambio, concentra la mayor cantidad de **conectores activos**: coordina microservicios entre sí (intra-tier), recibe peticiones de T2, persiste en T4, y emite/recibe información crítica de seguridad hacia/desde T5.

### Qué revela sobre las prioridades de diseño

1. **Centralidad de la lógica de negocio**: el sistema prioriza la separación de dominios funcionales (pasajeros, rutas, trenes, tickets, posición-tiempo) en servicios independientes dentro de T3, lo que favorece la mantenibilidad y la escalabilidad horizontal, pero a costa de mayor complejidad en la coordinación.

2. **Seguridad como driver arquitectónico**: la presencia del MAS (Movement Authority Service) en T3 — con conexiones directas a T5 — muestra que la lógica de autorización de movimiento se diseñó como un servicio de software centralizado que media entre el mundo digital y el físico. Esto permite aplicar validaciones, auditoría y control de acceso en un solo punto, pero convierte a T3 en el tier más crítico del sistema.

3. **Trade-off explícito**: mayor complejidad en T3 reduce la complejidad en los demás tiers. T1 se limita a presentar, T2 a enrutar, T4 a persistir y T5 a ejecutar. Esta decisión arquitectónica refleja el patrón de **"smart services, thin layers"**, donde la inteligencia del sistema se concentra deliberadamente en la capa lógica.

## Punto 2: Disponibilidad crítica en la vista C&C

La disponibilidad es más crítica en dos puntos del sistema: el **conector cross-tier T3↔T5** (MAS → OBU / OBU → position-time-ms) y el **Onboard Unit (OBU)** en T5.

### Por qué estos puntos son los más críticos

- **MAS → OBU (Movement Authority)**: si este canal se pierde, el tren deja de recibir autorizaciones de movimiento. En ETCS Nivel 2, donde la comunicación radio GSM-R es continua, una interrupción obliga al tren a aplicar frenado de emergencia al agotar la última autorización recibida. Una indisponibilidad prolongada de este conector detiene el tráfico ferroviario completo en el corredor afectado.
- **OBU → position-time-ms (reporte de posición)**: si el centro de control pierde la posición de un tren, no puede emitir nuevas autorizaciones ni para ese tren ni para los adyacentes, ya que no puede garantizar que los cantones estén libres. El efecto se propaga en cascada.
- **OBU como nodo crítico**: es el único componente que conecta el mundo físico (sensores, actuadores, balizas) con el mundo lógico (MAS, position-time-ms). Si el OBU falla, se pierde simultáneamente la supervisión, el frenado automático y la comunicación — el tren queda sin protección ETCS.

### Cómo la arquitectura aborda (o no) la disponibilidad

| Mecanismo | Presente en la arquitectura | Observación |
|---|---|---|
| Autonomía del OBU | Sí | En caso de pérdida de comunicación con T3, el OBU aplica políticas de seguridad autónomas (frenado de emergencia). Esto es **fail-safe**, no **fail-operational**: el tren se detiene, pero no sigue operando |
| Redundancia del OBU | No visible en la vista C&C | En implementaciones reales ERTMS, el EVC (European Vital Computer) es un sistema redundante 2oo3 (two-out-of-three). La vista de referencia no modela esta redundancia interna |
| Redundancia del MAS | No visible | No se modela replicación de instancias del MAS ni failover. Un fallo del MAS es un punto único de falla para todo el corredor |
| Comunicación dual GSM-R | No visible | El estándar ERTMS especifica dos canales de radio independientes, pero la vista C&C actual modela un solo conector T3↔T5 |
| API Gateway como SPOF | Parcialmente abordado | Se identificó un Load Balancer en componentes faltantes (T2), pero el API Gateway sigue siendo un punto único para toda la comunicación T1→T3 |

### Conclusión

La arquitectura aplica un enfoque **fail-safe** en T5 (el tren se detiene ante falta de información), pero la vista C&C actual no refleja mecanismos de **alta disponibilidad** como redundancia del MAS, replicación de canales de radio o failover entre instancias de microservicios. Esto no significa que ERTMS carezca de estos mecanismos — existen en la especificación real — sino que la vista C&C de referencia los omite, lo que representa una oportunidad de mejora en el modelo.

