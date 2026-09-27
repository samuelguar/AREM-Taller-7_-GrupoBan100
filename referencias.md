# Referencias e Investigación Complementaria — Ban100

**Curso:** Arquitectura Empresarial — Universidad de La Sabana
**Equipo:** Santiago Barrera Rueda · Krish Purmessur Moros · Samuel Guerrero Arcos
**Taller:** 7 — Opportunities & Solutions
**Objetivo:** respaldar con patrones y normativa reales cada una de las 12 iniciativas de mejora y las 2 matrices de decisión ponderada de `mejora-arquitectura.md` y `matrices-decision.md`, evitando soluciones "de manual" sin sustento.

---

## 1. Patrones de arquitectura y modernización

| # | Fuente | Aplicación al proyecto |
|---|---|---|
| 1 | Fowler, M. (2004). *StranglerFigApplication*. martinfowler.com. https://martinfowler.com/bliki/StranglerFigApplication.html | Sustenta la **Idea #3 / Paquete 3** (descomposición del Core Bancario Legacy). El patrón original de Fowler es exactamente el que se propone: envolver el sistema legado con una fachada y migrar dominio por dominio (cartera, luego captaciones) sin *downtime*, en vez de un reemplazo "Big Bang" — confirma la Opción A de la Matriz 2 (INF-01). |
| 2 | Debezium Community (2024). *Debezium Documentation — Change Data Capture*. https://debezium.io/documentation/ | Sustenta la **Idea #2** (CDC en tiempo real). Debezium lee el *transaction log* (WAL) del motor transaccional y publica los cambios en Kafka sin tocar el código de la aplicación origen, tal como se describe en la sección 3.1 del informe (captura en <500 ms). |
| 3 | Kleppmann, M. (2017). *Designing Data-Intensive Applications*, cap. 11 "Stream Processing". O'Reilly Media. | Contextualiza por qué CDC + streaming resuelve la brecha INF-02 (batch nocturno de 12h) mejor que acortar la ventana del batch: elimina la clase de problema en vez de mitigarla. |
| 4 | Cloud Native Computing Foundation (2023). *Kubernetes Documentation — Multi-cluster / Federation*. https://kubernetes.io/docs/ | Sustenta la **Idea #5** (KubeFed) para paridad de microservicios entre Bogotá y Medellín, base de la Idea #4 (Bre-B activo-activo) y del Paquete 3. |

## 2. Autenticación y seguridad (Taller 5 — STRIDE)

| # | Fuente | Aplicación al proyecto |
|---|---|---|
| 5 | FIDO Alliance & W3C (2021). *Web Authentication: An API for accessing Public Key Credentials Level 2 (WebAuthn)*. W3C Recommendation. https://www.w3.org/TR/webauthn-2/ | Sustenta la **Idea #11** (FIDO2/WebAuthn) como mitigación de STR-1 (Spoofing/deepfakes): la autenticación se basa en criptografía de llave pública anclada al hardware del dispositivo, no en biometría transmitida por red, lo que elimina la superficie de ataque de inyección de video sintético. |
| 6 | ISO 20022 Registration Authority (SWIFT). *ISO 20022 — pacs.008.001 (FIToFICstmrCdtTrf)*. https://www.iso20022.org/ | Define el formato del mensaje que se propone firmar digitalmente (Idea/control STR-2) en el flujo de Bre-B; ancla técnicamente la referencia a `pacs.008` usada en el diagrama de secuencia de la sección 3.1. |
| 7 | W3C (2013). *XML Signature Syntax and Processing (XML-DSig) Version 1.1*. W3C Recommendation. https://www.w3.org/TR/xmldsig-core1/ | Sustenta el mecanismo técnico de STR-2 (firma de payload XML-DSig con certificados X.509 clase 3) citado en la matriz de controles (sección 3.3). |
| 8 | OWASP Foundation (2024). *Content Security Policy Cheat Sheet*. https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html | Sustenta la **Idea #9** (cabeceras CSP/HSTS estrictas) como mitigación de STR-5/NOR-6 (XSS y *web skimming*); incluye la recomendación de desplegar primero en modo `Report-Only`, adoptada explícitamente en la tabla de riesgos de la sección 4. |
| 9 | Microsoft (2024). *Dynamic Data Masking — SQL Server documentation*. https://learn.microsoft.com/sql/relational-databases/security/dynamic-data-masking | Referencia técnica de la **Idea #7** (DDM) para enmascarar saldos y mesadas pensionales sin modificar el código de los microservicios consumidores, tal como se describe en la sección 3.3 (control STR-4/NOR-4). |

## 3. Normatividad colombiana (Taller 6 — Cumplimiento)

| # | Fuente | Aplicación al proyecto |
|---|---|---|
| 10 | Congreso de la República de Colombia. *Ley 1581 de 2012, por la cual se dictan disposiciones generales para la protección de datos personales*. Diario Oficial No. 48.587. | Fundamento legal directo de NOR-1 (derechos ARCO) e Idea #6 (módulo de autoservicio de privacidad); también sustenta NOR-7 (retención de biometría) y la Idea #10 (TTL de 30 días), bajo el principio de necesidad y temporalidad que cita el propio diagnóstico. |
| 11 | Congreso de la República de Colombia. *Ley 527 de 1999, por medio de la cual se define y reglamenta el acceso y uso de los mensajes de datos, del comercio electrónico y de las firmas digitales*. Diario Oficial No. 43.673. | Fundamento legal de la **Idea #12** (pagaré desmaterializado con firma digital), citada en la sección 2.1 y en el control STR-3 de la sección 3.3. |
| 12 | Depósito Centralizado de Valores de Colombia — DECEVAL S.A. *Desmaterialización y custodia electrónica de títulos valores*. https://www.deceval.com.co/ | Sustenta la ruta de custodia y radicación automática vía API propuesta para el pagaré electrónico (Idea #12), como alternativa real y ya operativa en el mercado de valores colombiano al proceso manual con checkbox web actualmente en riesgo (STR-3). |
| 13 | Superintendencia Financiera de Colombia. *Circular Externa 007 de 2018, requerimientos mínimos de seguridad y calidad en el manejo de información a través de medios y canales de distribución de productos y servicios*. | **Nota de verificación para el equipo:** esta es la circular vigente que regula específicamente disponibilidad, continuidad y seguridad de canales digitales — no la Circular Externa 008 de 2023 citada en `mejora-arquitectura.md` y `matrices-decision.md` (esa circular trata sobre el Sistema de Control Interno, no sobre continuidad operativa). **Se recomienda que el equipo corrija esta referencia antes de la entrega final**, o la sustente si el docente entregó un checklist propio del Taller 6 que use ese número. |
| 14 | Banco de la República (2025). *Bre-B: Sistema de Pagos Inmediatos e Interoperables de Colombia*. https://www.banrep.gov.co/ | Fuente primaria del sistema de pagos inmediatos que Ban100 debe integrar; confirma la operación 24/7 con confirmación en segundos que motiva el requisito de RTO < 1s de la Idea #4 y el Paquete 3. |

## 4. Marco metodológico del curso

| # | Fuente | Aplicación al proyecto |
|---|---|---|
| 15 | The Open Group (2022). *TOGAF® Standard, 10th Edition — Architecture Development Method (ADM), Phase E: Opportunities & Solutions*. https://www.opengroup.org/togaf | Marco metodológico oficial que estructura este taller (diagnóstico → lluvia de ideas → priorización por horizontes H1/H2/H3 → matrices de decisión). |
| 16 | MinTIC Colombia. *Marco de Referencia de Arquitectura Empresarial (MRAE) v3.0*. https://www.mintic.gov.co/arquitecturati/ | Marco de referencia nacional citado en la portada de `mejora-arquitectura.md` como entregable oficial de la Fase E. |

---

## Nota sobre el uso de IA en esta investigación

Esta lista de referencias fue construida con apoyo de Claude (Anthropic), que buscó y verificó cada fuente contra el contenido real de la página o documento citado (no se inventó ningún título, autor o URL). El equipo debe:

- [ ] Verificar que cada enlace abra y corresponda al contenido citado antes de la entrega.
- [ ] Resolver la inconsistencia señalada en la referencia #13 (Circular Externa 007 de 2018 vs. 008 de 2023) con el docente o con el checklist real del Taller 6.
- [ ] Añadir cualquier fuente adicional que el equipo haya consultado por su cuenta y que no esté aquí registrada.

_Este documento hace parte de la entrega oficial del Taller 7 (Opportunities & Solutions) del curso Arquitectura Empresarial — Universidad de La Sabana._
