# Matrices de Decisión Ponderada — Ban100

Complementa a [`mejora-arquitectura.md`](mejora-arquitectura.md) sección 2.2. Metodología: guía paso a paso, sección 2.1 (marco de 8 pasos) — no disponible como archivo en este repositorio de equipo; ver la copia del taller en `clase/guia_paso_a_paso_opportunities_solutions.md` del repositorio del curso.

Se documentan las dos brechas priorizadas cuya solución exigía elegir entre varias alternativas de fondo (no solo "hacerlo o no"): el balanceador único (INF-03, Horizonte 1) y el Core Bancario Legacy (INF-01, Horizonte 3). Las demás brechas del Horizonte 1 (STR-05, STR-07, NOR-03, NOR-06, NOR-07) tienen una solución técnica prácticamente única una vez priorizadas (p. ej. agregar cabeceras CSP no admite alternativas de fondo que valga la pena comparar), por lo que no requieren esta matriz — criterio explícito de la guía ("una brecha real casi siempre tiene más de una forma de cerrarla" no significa que *todas* la necesiten formalmente, solo aquellas con una decisión arquitectural de fondo).

**Nota de alcance:** si el docente pide una matriz por cada brecha priorizada, el equipo puede replicar esta misma estructura para las demás usando [`plantillas/plantilla_matriz_decision.md`](../plantillas/plantilla_matriz_decision.md) — el patrón queda establecido en los dos ejemplos siguientes.

---

# Matriz 1 — Balanceador de Carga Global (INF-03)

## Brecha que se decide
**INF-03** — Balanceador de Carga Global en instancia única (SPOF perimetral en el centro de datos principal de Bogotá). Evidenciado en Taller 4, Tabla de Diagnóstico Técnico Priorizado, ítem 3 (Prioridad Alta) y en Taller 6, checklist ítem 8 (DRP "Parcial").

## Paso 1 — Problema en términos de impacto
Si el balanceador único falla, **la totalidad del tráfico de los 250.000+ clientes de Ban100 queda sin enrutamiento**: los canales App Móvil, Portal Web y el módulo Bre-B quedan inaccesibles al mismo tiempo. Cada minuto de caída incumple el SLA de disponibilidad del 99.99% que la Circular Externa 008 de 2023 de la SFC exige para canales digitales, y en el caso específico de Bre-B, deja mesadas pensionales y transferencias inmediatas sin liquidar ante el Banco de la República — con el riesgo reputacional agravado de tratarse, en su mayoría, de población pensionada dependiente de esos pagos.

## Paso 2 — Último momento responsable
El equipo ya clasificó esta brecha como *Quick Win* del **Horizonte 1 (0-3 meses)** en la matriz de brechas. Tomando esa ventana como referencia:

| Dato | Valor |
|---|---|
| Cierre del Horizonte 1 (compromiso ya fijado por el equipo) | Mes 3 desde el inicio del plan de implementación (Taller 9) |
| Tiempo que necesita la opción más probable (aprovisionamiento multi-región + pruebas de conmutación) | 6-8 semanas |
| **Último momento responsable** (fin de H1 − tiempo necesario) | **Debe decidirse dentro de las primeras 4-6 semanas del Horizonte 1**, no al final |
| Consecuencia de decidir tarde | Si se decide en el mes 3, ya no hay tiempo de probar el failover antes de que H1 se dé por cerrado, y la brecha se arrastra a H2 sin haberse cerrado realmente |

## Paso 3 — Criterios, pesos y escala
Pesos propuestos por el equipo técnico como borrador — **pendientes de validación con el área de Riesgo Operacional / Infraestructura de Ban100** antes de la entrega final, tal como exige la guía (los pesos los valida el negocio, no solo el equipo técnico):

| Criterio | Peso | Qué significa 5 | Qué significa 1 |
|---|---|---|---|
| Disponibilidad lograda | 40 % | Cero interrupción perceptible ante la falla del balanceador (RTO ≈ 0) | La falla sigue dejando la plataforma completamente inaccesible |
| Tiempo de implementación (cabe en H1) | 25 % | Menos de 4 semanas | Más de 10 semanas (no cabe en H1 sin retrasar el resto del horizonte) |
| Costo adicional mensual | 20 % | Menos de USD 300/mes | Más de USD 1.500/mes |
| Complejidad operativa nueva | 15 % | Nada nuevo que operar de forma continua | Requiere sincronización y pruebas continuas entre regiones |

*Por qué esos pesos:* la disponibilidad pesa más porque es el requisito regulatorio explícito (SFC 99.99%) y el que más directamente protege a los pensionados; tiempo pesa casi igual porque el equipo ya comprometió esta brecha en H1; costo y complejidad pesan menos porque Ban100 ya opera infraestructura cloud multi-región para otros componentes (CDN, WAF), por lo que el salto de madurez operativa es menor que para un banco sin esa base.

**Criterio eliminatorio:** toda opción con Disponibilidad menor a 3 se descarta, sin importar su total — no tendría sentido cerrar esta brecha con algo que no resuelve el SPOF.

## Paso 4 — Opciones
| Opción | Descripción |
|---|---|
| **A** | Balanceador Global Cloud **activo-activo** multi-región (Anycast DNS, health checks continuos, conmutación < 30s) — la que el equipo ya preseleccionó en la lluvia de ideas |
| **B** | Balanceador **activo-pasivo** con failover automático: segunda instancia en espera que toma el tráfico si la principal cae (ventana de conmutación de 30-60s) |
| **C** | Mantener el balanceador único y reforzar solo el monitoreo/alertas tempranas (opción radicalmente distinta: aceptar el riesgo, reducir el tiempo de detección) |

## Paso 5 — Consejo consultado
*(Roles y estimaciones a validar formalmente con el equipo real de Infraestructura de Ban100 y el proveedor cloud antes de la entrega — se anotan aquí como supuestos de trabajo del equipo, siguiendo el mismo tratamiento que la guía da a los datos de RedExpress.)*

| A quién | Qué aportaría (estimación de trabajo) | Qué opción afecta |
|---|---|---|
| Infraestructura / DevOps (Taller 4, ya documentado) | El esquema activo-activo requiere DNS global con health checks — mismo patrón que ya usan para el CDN/WAF | A |
| Proveedor cloud (estimación) | Segunda instancia activa-activo ≈ USD 450-600/mes; activo-pasivo ≈ USD 250/mes; monitoreo reforzado ≈ USD 50/mes | A, B, C |
| Oficial de Cumplimiento SFC (a partir del checklist Taller 6, ítem 8) | El DRP "parcial" ya señalado en el checklist exige conmutación automatizada, no solo monitoreo — descarta implícitamente C como cierre de la brecha normativa NOR-02 | Todas |

## Paso 6 — Puntajes con justificación
| Opción | Disponibilidad | Tiempo | Costo | Complejidad |
|---|---|---|---|---|
| A · Activo-activo | **5** — sin ventana de conmutación perceptible, cumple 99.99% | **3** — 6-8 semanas, ajustado pero cabe en H1 si se decide temprano | **3** — USD ~500/mes cae en rango medio | **3** — requiere health checks y pruebas de balanceo entre regiones |
| B · Activo-pasivo | **4** — conmuta sola, pero con ventana de 30-60s (no cero) | **4** — 4-6 semanas, más holgado | **4** — USD ~250/mes, más barato | **4** — un componente nuevo con failover simple |
| C · Único + monitoreo | **1** — no elimina el SPOF, solo acorta el tiempo de detección | **5** — menos de 1 semana | **5** — USD ~50/mes | **5** — nada nuevo que operar |

**Totales ponderados** (pesos del negocio: 40/25/20/15):

| Opción | Cálculo | Total |
|---|---|---|
| A · Activo-activo | 5×0,40 + 3×0,25 + 3×0,20 + 3×0,15 | **3,80** |
| B · Activo-pasivo | 4×0,40 + 4×0,25 + 4×0,20 + 4×0,15 | **4,00** |
| C · Único + monitoreo | 1×0,40 + 5×0,25 + 5×0,20 + 5×0,15 | **2,40** — **descartada por criterio eliminatorio** (Disponibilidad = 1) |

**Sensibilidad** (escenario alternativo: costo prioritario, 40 % costo / 20 % disponibilidad / 20 % tiempo / 20 % complejidad):

| Escenario de pesos | Total A | Total B | ¿Gana la misma opción? |
|---|---|---|---|
| Pesos del negocio (40/25/20/15) | 3,80 | **4,00** | B |
| Costo prioritario (20/20/40/20) | 3,40 | **4,00** | B (se mantiene) |

**Lectura honesta del resultado:** con los pesos tal como quedaron redactados arriba, **B (activo-pasivo) gana por margen estrecho sobre A** en ambos escenarios — a diferencia del ejemplo de RedExpress de la guía, donde A ganaba. La diferencia (0,20 puntos) está *dentro* del margen de empate que la guía define (<0,3 puntos), así que la decisión no se puede tomar solo por el total.

## Paso 7 — Decisión
**Decisión:** implementar el esquema **activo-activo multi-región (Opción A)**, aunque el puntaje ponderado puro favorece a B por un margen de empate técnico. **Razón para no seguir el total automáticamente:** el checklist normativo del Taller 6 (ítem 8) ya calificó el DRP como "Parcial" exigiendo explícitamente conmutación *automatizada* con RTO agresivo — el activo-pasivo (30-60s) sigue dejando una ventana de indisponibilidad para transacciones Bre-B, que el Banco de la República trata como timeout de cámara de compensación. **Trade-off aceptado:** se acepta ~1-2 semanas más de implementación y ~USD 250-350/mes adicionales a cambio de eliminar por completo la ventana de conmutación, que es lo que realmente cierra la brecha normativa NOR-02, no solo la técnica INF-03. **Alternativas descartadas:** C, por fallar el criterio eliminatorio de disponibilidad; B, porque aunque gana en el puntaje puro, no cierra la brecha normativa asociada (NOR-02) que exige RTO cercano a cero para Bre-B — queda registrada como opción de respaldo si el presupuesto de A no se aprueba.

## Paso 8 — Reevaluación
Se revisita esta decisión si: (a) el proveedor cloud no puede garantizar el SLA de conmutación < 30s dentro del presupuesto aprobado, (b) tras 6 meses en producción el volumen real de incidentes de conmutación es significativamente menor al esperado (podría justificar bajar a B en la siguiente ronda de optimización de costos), o (c) cambia la exigencia regulatoria de RTO en una actualización de la Circular 008 SFC.

---

# Matriz 2 — Core Bancario Legacy Monolítico (INF-01)

## Brecha que se decide
**INF-01** — Core Bancario Legacy monolítico on-premise, sin APIs nativas, migración a microservicios incompleta. Evidenciado en Taller 4 (Tabla de Diagnóstico Técnico Priorizado, ítem 1, Prioridad Crítica) y en la ficha de caracterización de Taller 0 ("Problema #3: Rigidez del Core Monolítico e Integraciones Punto a Punto").

## Paso 1 — Problema en términos de impacto
Cada vez que Ban100 necesita integrar un nuevo servicio (Bre-B, banca abierta, un nuevo producto de microcrédito), el equipo de TI debe construir una integración punto a punto contra el Core, un proceso que hoy toma en promedio **6 meses**. Mientras la banca abierta colombiana avanza hacia interoperabilidad obligatoria y Bre-B ya exige integración en tiempo real, cada mes de retraso deja a Ban100 fuera de nuevos ecosistemas de pago y expuesto a perder cuota de mercado frente a competidores con núcleos más flexibles — no es un problema estético de arquitectura, es un problema de velocidad comercial.

## Paso 2 — Último momento responsable
A diferencia del balanceador, aquí no hay una fecha de campaña externa; el "último momento responsable" es de **dependencia de proyecto**, no de calendario: el Paquete 2 (H2, 3-6 meses) ya depende de que el Core exponga eventos en tiempo real vía CDC (INF-02), y el Paquete 3 completo (H3) depende de que el Core exponga las primeras APIs de dominio.

| Dato | Valor |
|---|---|
| El Paquete 2 (CDC hacia Kafka/MDM) necesita que el Core exponga sus logs transaccionales de forma estable | Debe estar decidido antes de iniciar H2 |
| Tiempo que toma solo *decidir y validar la estrategia* (piloto de un primer dominio estrangulado) | 4-6 semanas |
| **Último momento responsable** | Debe decidirse **antes de cerrar el Horizonte 1**, para que el piloto de estrangulamiento corra en paralelo con las quick wins de H1 sin bloquear el inicio de H2 |

## Paso 3 — Criterios, pesos y escala
| Criterio | Peso | Qué significa 5 | Qué significa 1 |
|---|---|---|---|
| Riesgo operacional (probabilidad de interrupción del servicio en producción) | 35 % | Cero interrupción; el Core sigue operando mientras se migra | Corte de servicio total durante la migración |
| Flexibilidad futura / deuda técnica eliminada | 30 % | El Core queda completamente desacoplado en microservicios independientes | El monolito sigue igual de rígido tras la iniciativa |
| Tiempo hasta el primer valor entregado | 20 % | Menos de 2 meses hasta el primer dominio migrado y en producción | Más de 8 meses sin nada operando en producción |
| Costo total del programa | 15 % | Reutiliza mayormente el equipo y la infraestructura ya existente | Requiere licenciamiento de un core bancario nuevo completo |

*Por qué esos pesos:* riesgo operacional pesa más que en cualquier otra decisión de este taller porque Ban100 **no puede permitirse una interrupción del Core** — ahí vive el 100% de las obligaciones de cartera y depósitos de 250.000 clientes; flexibilidad futura pesa casi igual porque es la razón de ser de esta iniciativa (Problema #3 de la ficha de caracterización); tiempo y costo pesan menos porque esta es la iniciativa de H3 (largo plazo), donde el equipo ya aceptó un horizonte extendido.

**Criterio eliminatorio:** toda opción con Riesgo operacional menor a 3 se descarta — no es negociable poner en riesgo la operación del Core de un banco regulado.

## Paso 4 — Opciones
| Opción | Descripción |
|---|---|
| **A** | **Strangler Fig**: descomposición progresiva mediante API Gateway y capa anti-corrupción, dominio por dominio (cartera, luego captaciones), manteniendo el monolito operando hasta que cada dominio se retire — la opción que el equipo ya preseleccionó |
| **B** | **Reemplazo "Big Bang"** por un core bancario nuevo (vendor de core-as-a-service), con corte y migración de datos en una sola ventana |
| **C** | **Mantener el monolito intacto** y agregar solo una capa de API Gateway / *Backend-for-Frontend* que expone lo mínimo necesario, sin descomponer nada internamente (opción radicalmente distinta: minimizar el cambio) |

## Paso 5 — Consejo consultado
| A quién | Qué aportaría (estimación de trabajo, a validar) | Qué opción afecta |
|---|---|---|
| Equipo de Arquitectura (Taller 4, patrón ya documentado en el informe) | El patrón Strangler Fig permite migrar cartera y captaciones sin retirar el monolito hasta estabilizar cada dominio — coincide con el enfoque ya descrito en el informe de Taller 4 | A |
| Riesgo Operacional (rol de negocio, no consultado aún formalmente) | Un corte "Big Bang" sobre el Core de un banco regulado normalmente exige ventanas de mantenimiento aprobadas por la SFC con semanas de anticipación — eleva el riesgo regulatorio de B | B |
| Producto / Comercial (rol de negocio, no consultado aún formalmente) | Un wrapper sin descomponer (C) seguiría sin resolver la lentitud de 6 meses para lanzar productos nuevos, solo la esconde en el corto plazo | C |

## Paso 6 — Puntajes con justificación
| Opción | Riesgo operacional | Flexibilidad futura | Tiempo a primer valor | Costo |
|---|---|---|---|---|
| A · Strangler Fig | **4** — el monolito sigue operando en paralelo; el riesgo se limita al dominio migrado en cada paso | **5** — cada dominio migrado queda en microservicios independientes | **3** — primer dominio piloto en 2-4 meses | **3** — reutiliza el equipo interno, sin licencia de un core nuevo |
| B · Big Bang | **1** — corte único sobre el 100% de la cartera y depósitos, sin reversa fácil si algo falla | **5** — el core nuevo llega completamente desacoplado desde el día uno | **1** — 8-12 meses de proyecto antes de cualquier corte en producción | **1** — licenciamiento de un core-as-a-service nuevo, el más costoso |
| C · Solo wrapper API | **5** — no se toca la lógica interna del monolito, riesgo mínimo | **1** — el monolito sigue igual de rígido internamente | **5** — semanas, es solo una capa nueva | **5** — el más barato, sin tocar el core |

**Totales ponderados** (pesos del negocio: 35/30/20/15):

| Opción | Cálculo | Total |
|---|---|---|
| A · Strangler Fig | 4×0,35 + 5×0,30 + 3×0,20 + 3×0,15 | **4,05** |
| B · Big Bang | 1×0,35 + 5×0,30 + 1×0,20 + 1×0,15 | **2,20** — **descartada por criterio eliminatorio** (Riesgo operacional = 1) |
| C · Solo wrapper | 5×0,35 + 1×0,30 + 5×0,20 + 5×0,15 | **3,80** |

**Sensibilidad** (escenario alternativo: priorizando velocidad de entrega, 40 % tiempo / 30 % riesgo / 20 % flexibilidad / 10 % costo):

| Escenario de pesos | Total A | Total C | ¿Gana la misma opción? |
|---|---|---|---|
| Pesos del negocio (35/30/20/15) | **4,05** | 3,80 | A |
| Velocidad prioritaria (30 riesgo/20 flex/40 tiempo/10 costo) | 3,60 | **4,10** | **C — se invierte** |

**Lectura honesta:** si el negocio llegara a valorar la velocidad de entrega por encima de la flexibilidad estructural (escenario poco probable dado que el Problema #3 de la ficha de caracterización es explícitamente sobre rigidez, no sobre velocidad de un solo lanzamiento), C se volvería la opción ganadora. Esto es información útil para el comité: la decisión por A depende de que el negocio siga valorando resolver la rigidez de fondo, no solo lanzar más rápido en el corto plazo.

## Paso 7 — Decisión
**Decisión:** iniciar la descomposición mediante **Strangler Fig (Opción A)**, comenzando por el dominio de cartera de libranza (el de mayor volumen transaccional documentado). **Trade-off aceptado:** se sacrifica velocidad de entrega inmediata (3/5, primer dominio en 2-4 meses) y se mantiene el monolito operando en paralelo por más tiempo del que tomaría un wrapper simple, a cambio de eliminar realmente la deuda técnica de fondo en vez de esconderla. **Alternativas descartadas:** B, por fallar el criterio eliminatorio de riesgo operacional (un banco regulado no puede aceptar un corte "Big Bang" sobre su Core); C, porque aunque es más rápida y barata, no resuelve el Problema #3 documentado en la ficha de caracterización — quedaría como solución de contención de muy corto plazo, no como cierre de la brecha INF-01.

## Paso 8 — Reevaluación
Se revisita si: (a) el primer dominio piloto (cartera) supera el plazo de 4 meses sin llegar a producción — señal de que el riesgo operacional real es mayor al estimado en el Paso 6, o (b) tras el piloto, el costo real de mantener ambos sistemas en paralelo (monolito + microservicios) resulta significativamente más alto que lo proyectado, lo que obligaría a acelerar el retiro completo del monolito antes de lo planeado.

---

## Registro de uso de IA (esta sesión)

| Paso | Qué se le pidió a la IA (Claude) | Qué propuso | Qué debe verificar o recalcular el equipo | Qué cambió/decidió el equipo |
|---|---|---|---|---|
| Generar las matrices completas (Pasos 1-8) para INF-03 e INF-01 | Construir el marco de 8 pasos a partir de las brechas ya diagnosticadas en Talleres 4 y 6, sin inventar hallazgos nuevos | Criterios, pesos borrador, puntajes justificados, análisis de sensibilidad y redacción de las 8 secciones | **Los pesos son un borrador técnico, no están validados por el negocio** (ver Paso 3 de cada matriz); los costos y tiempos en dólares/semanas son estimaciones de trabajo, no cotizaciones reales del proveedor cloud ni del equipo de Infraestructura | Pendiente: el equipo debe validar pesos y cifras con roles reales (o con el docente actuando como "negocio") antes de la entrega final |
| Verificación de trazabilidad | — | Cada código de brecha (INF-03, INF-01, NOR-02) citado en estas matrices se contrastó contra `04-infraestructura/informe.md` y `06-normatividad/checklist-cliente.xlsx` del repositorio del equipo | — | Ningún hallazgo nuevo fue inventado; todo cita su taller de origen |
| Confidencialidad | — | — | Ban100 es un caso académico (cliente real del curso, no una entidad con la que el equipo tenga NDA); no se identificó información que requiera anonimización adicional a la ya usada en los talleres previos | — |

- [ ] El equipo puntuó por su cuenta y comparó con lo que propuso la IA (evita el anclaje) — .
- [ ] Los pesos fueron validados por "el negocio" (docente o cliente simulado), no solo por el equipo técnico — .
- [x] Toda cifra o afirmación de la IA quedó marcada explícitamente como estimación a verificar, no como dato confirmado.