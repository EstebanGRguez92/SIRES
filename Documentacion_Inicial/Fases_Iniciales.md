# Cómo arrancar SIRES: Maqueta → POC → MVP

> **Nota de vigencia (9-oct-2026):** este documento se corrige para alinearse con
> `SIRES_Arquitectura_Tecnica_v1_2.md` y con `.cursor/rules/current-phase.mdc`. Los cambios
> se marcan con **[CORREGIDO]**. El roadmap de etapas técnicas de la v1.2 (§22) usa
> "Etapa A-F" para no confundirse con las "Fases 0/1/2" de este documento, que son las fases
> de producto (Maqueta → POC → MVP).

Analisis **no empieces por código de producción**. Empieza por una **maqueta clickeable** para validar con la química farmacéutica, luego una **POC funcional con datos mock**, y solo entonces un **MVP mínimo** con una fracción end-to-end.

Construir infraestructura AWS, e.firma real y bitácora inmutable antes de validar la idea con la stakeholder es quemar presupuesto sin validación.

---

## Contexto

Tienes una idea validada por una química farmacéutica, un alcance claro (recetas electrónicas), un equipo básico, y un objetivo nacional. Pero **no tienes validación visual con el usuario final**, ni has probado el flujo completo con alguien que realmente receta o dispensa.

Antes de invertir en Aurora, S3 Object Lock, e.firma y CDK, necesitas tres cosas:

1. **Validar la idea visualmente** con la química.
2. **Validar el flujo end-to-end** con un caso real (aunque sea mock).
3. **Validar la viabilidad técnica** de las piezas críticas (firma, QR, concurrencia).

Cada validación corresponde a una fase distinta.

---

## Análisis

### Fase 0 — Maqueta visual (1-2 semanas)

**Objetivo:** que la química vea, toque y critique el sistema antes de invertir un peso en infraestructura.

**Alcance:**
- Wireframes de baja fidelidad en papel o Figma.
- Prototipo clickeable navegable (no funcional).
- Tres flujos: médico emite, farmacia dispensa, admin audita.
- Datos mock: pacientes ficticios, medicamentos reales, QR de ejemplo.

**Herramientas recomendadas:**
- **Figma** (gratis para equipo pequeño): prototipo interactivo.
- **Excalidraw** (gratis): bocetos rápidos colaborativos.
- **Penpot** (open source): alternativa a Figma si quieren soberanía.

**Qué mostrar a la química:**
- Login con e.firma (simulado).
- Selección de paciente.
- Selección de medicamento por fracción (se maquetan las 6 fracciones; los topes exactos
  por fracción son ejemplos ilustrativos **pendientes de confirmar con la química**,
  nunca se presentan como ley cerrada).
- Validación de reglas con un ejemplo ilustrativo, marcado como tal en pantalla.
- Firma (botón "Firmar con e.firma").
- **QR generado: uno por receta** (no uno por medicamento/renglón).
- Vista de farmacia: escaneo QR → validación → dispensación.
- Vista admin: bitácora de eventos (el sistema **detecta y alerta** una alteración, no
  "bloquea" una modificación física del registro histórico).

**[CORREGIDO] Orden de fracciones para el MVP real (Fase 2):**
- **Fracciones IV, V y VI primero.** Son las menos restringidas (sin folio COFEPRIS, sin
  retención obligatoria generalizada, sin especialidad). Se valida el flujo técnico completo
  con ellas.
- **Fracciones I, II y III se maquetan y se incluyen en la Fase 0 (y opcionalmente en el
  guion de demo del POC) solo para validar el flujo visual y los bloqueos.** No se
  construyen como operación real hasta que exista el criterio de COFEPRIS (P-01 de la
  arquitectura v1.2) y el equipo llegue a esa etapa del roadmap.
- Ejemplo de reglas de Fracción IV a confirmar con la química antes de programarlas:
  receta con cédula profesional del médico, vigencia de 6 meses, hasta 3 surtidos, sin
  obligación de retener la receta en los primeros dos surtidos. Esto es información de
  referencia, no un requisito ya validado para codificar.

**Lo que NO se hace en esta fase:**
- Nada de código de producción.
- Nada de AWS.
- Nada de e.firma real.
- Nada de base de datos.

**Costo:** prácticamente cero. Solo tiempo de diseño.

**Entregable:** link de Figma navegable + video de 3 minutos.

---

### Fase 1 — POC funcional con datos mock (3-4 semanas)

**Objetivo:** probar que el flujo completo funciona técnicamente, con datos ficticios y sin e.firma real.

**Alcance:**
- Frontend Next.js con los tres portales mínimos.
- Backend NestJS con un módulo funcional: recetas.
- Base de datos PostgreSQL local (Docker) o Supabase.
- Datos mock: 10 médicos, 20 pacientes, 50 medicamentos.
- Motor de reglas por fracción funcionando, para Fracciones IV, V y VI (operación real
  del POC). I, II y III siguen siendo solo maqueta/flujo, no se construyen en POC.
- QR generado y validado (sin criptografía real), uno por receta.
- **[CORREGIDO] Sin Redis/Redlock.** La v1.2 retira Redis/Valkey del MVP y no lo acepta
  como autoridad de cero duplicidad. El POC prueba el mecanismo real desde el inicio:
  `SELECT ... FOR UPDATE` + restricción `UNIQUE` + `Idempotency-Key`, solo con PostgreSQL.
- Sin e.firma real: firma simulada con certificado de prueba autofirmado. **La prueba de
  concepto criptográfica con `.key` reales del SAT (riesgo R2 de la arquitectura) corre en
  paralelo durante esta fase, a cargo del Security Engineer, sin datos reales de pacientes
  y sin que la `.key` toque ningún servidor** — es la única excepción a "e.firma real
  solo hasta el MVP", porque es el riesgo más crítico del proyecto y conviene probarlo pronto.

**Lo que se prueba:**
- Flujo médico: emitir receta de Fracción IV (o V/VI).
- Flujo farmacia: escanear QR y dispensar.
- Flujo concurrencia: dos farmacias intentan dispensar simultáneamente → solo una gana,
  por el lock transaccional en PostgreSQL, no por Redis.
- Motor de reglas: rechazar una cantidad fuera de política (ejemplo a confirmar con la
  química, no un número de ley asumido).
- **[CORREGIDO] Máquina de estados:** la del documento de arquitectura v1.2 §8.1
  (`BORRADOR → EMITIDA → PARCIALMENTE_SURTIDA → SURTIDA_TOTAL`, con `CANCELADA` como
  transición adicional). La retención y la caducidad **no son estados**: la retención es
  un atributo de cada registro de dispensación y la caducidad se calcula contra la fecha.

**Herramientas:**
- **Docker Compose** para levantar todo local.
- **Next.js + NestJS** como ya definimos.
- **PostgreSQL** en Docker.
- **Prisma** como ORM (curva de aprendizaje baja).
- **Supabase** como alternativa si no quieren Docker.

**Lo que NO se hace en esta fase:**
- Nada de AWS.
- Nada de e.firma real.
- Nada de S3 Object Lock.
- Nada de Lambda@Edge.
- Nada de KMS.

**Costo:** prácticamente cero. Solo tiempo de desarrollo.

**Entregable:** demo local ejecutable con `docker compose up` + video de 5 minutos.

**Recomendación:** esta POC es lo que le presentas a la química **después** de la maqueta. Ya no es solo visual: es funcional. Ella puede emitir una receta ficticia y ver el QR generado.

---

### Fase 2 — MVP mínimo end-to-end (8-12 semanas)

**Objetivo:** primer sistema con infraestructura real, una sola fracción funcional, listo para piloto controlado con farmacias aliadas.

**Alcance:**
- Infraestructura AWS Mexico Central.
- e.firma real (mecanismo final definido en ADR-001; puede o no requerir Wasm).
- S3 Object Lock para bitácora.
- Aurora PostgreSQL (Serverless v2).
- SQS + DLQ.
- KMS.
- **[CORREGIDO] Fracciones IV, V y VI end-to-end desde el inicio del MVP**, no solo IV.
  Las tres comparten el mismo nivel de restricción (sin folio, sin especialidad, sin
  dependencia de convenios gubernamentales) y validan el mismo flujo técnico; no hay razón
  para escalonarlas entre sí.
- **Sin ElastiCache ni Lambda@Edge** (retirados en v1.2 por simplicidad operativa y OPEX).

**¿Por qué IV, V y VI primero?**
- No requieren folios COFEPRIS.
- No requieren permiso especial.
- No dependen de especialidad verificada.
- Permiten validar todo el flujo técnico sin la complejidad normativa de I, II y III.
- Fracciones I, II y III **no entran a operación real** hasta tener el criterio escrito de
  COFEPRIS (pendiente P-01 de la arquitectura v1.2); antes de eso, solo existen en la
  maqueta y en el guion de demo, nunca con datos ni dispensación real.

**Lo que se prueba:**
- Un médico real emite una receta real.
- Un farmacéutico real la dispensa.
- La bitácora queda inmutable.
- El QR funciona en el Edge.
- La concurrencia funciona con lock transaccional en PostgreSQL + restricción única
  (sin Redlock; ver corrección en Fase 1).

**Costo:** ~$200-500 USD/mes en AWS con poco volumen.

**Entregable:** sistema en producción limitada con 5-10 médicos y 3-5 farmacias piloto.

---

## Opciones evaluadas

| Opción | Tiempo | Costo | Riesgo | Valor para la química |
|---|---|---|---|---|
| **Solo maqueta Figma** | 1-2 semanas | $0 | Muy bajo | Alto (visual, validable) |
| **Maqueta + POC local** | 4-6 semanas | $0 | Bajo | Muy alto (funcional, validable) |
| **Maqueta + POC + MVP Fracciones IV/V/VI** | 12-18 semanas | ~$500/mes | Medio | Definitivo (sistema real) |
| **MVP completo todas las fracciones** | 9-12 meses | ~$2,500/mes | Alto | Prematuro sin validación |
| **Infraestructura completa primero** | 6 meses | ~$5,000/mes | Muy alto | Contraproducente |

---

## Decisión recomendada

**Arrancar con Fase 0 (maqueta) inmediatamente, seguida de Fase 1 (POC local) antes de tocar AWS.**

Justificación:
- Costo cero en infraestructura.
- Validación temprana con la química.
- Aprendizaje rápido del equipo.
- Riesgo mínimo.
- Permite ajustar el diseño antes de comprometer arquitectura.
- La POC local se convierte en la base del MVP sin desperdicio.

**Regla:** no se escribe una línea de código AWS hasta que la química apruebe la POC local.

---

## Plan de ejecución concreto

### Semana 1-2 — Maqueta visual

| Día | Actividad | Responsable |
|---|---|---|
| 1-2 | Entrevista con la química: flujos reales, dolores, expectativas | Product Owner |
| 3-5 | Wireframes de los 3 portales | **UX Designer** (Frontend Lead apoya) |
| 6-8 | Prototipo clickeable en Figma | **UX Designer** (Frontend Lead apoya) |
| 9-10 | Presentación a la química y ajustes | CTO + PO |

**Entregable:** link Figma + video 3 min.

---

### Semana 3-6 — POC local

| Semana | Actividad | Responsable |
|---|---|---|
| 3 | Setup: repo, Docker Compose, estructura NestJS + Next.js | Tech Lead |
| 4 | Módulo recetas: emisión, motor de reglas, máquina de estados | Tech Lead + Frontend |
| 5 | Módulo farmacia: escaneo QR, dispensación, lock transaccional PostgreSQL | Tech Lead + Backend |
| 6 | Módulo auditoría: eventos, hash chain, vista admin | Data Engineer + Frontend |

**Entregable:** repo con `docker compose up` funcional + video 5 min.

---

### Semana 7-8 — Validación con la química

| Día | Actividad |
|---|---|
| 1-2 | Demo presencial o virtual con la química |
| 3-4 | Ajustes según feedback |
| 5 | Decisión: continuar a MVP o iterar POC |

---

## Qué mostrarle a la química (guion de demo)

### Demo 1 — Médico emite receta

1. Login simulado con e.firma.
2. Selecciona paciente ficticio.
3. Busca "Clonazepam" (Fracción II).
4. Intenta poner 3 cajas → sistema bloquea con mensaje claro.
5. Cambia a 2 cajas → sistema acepta.
6. Presiona "Firmar con e.firma" → simulación.
7. QR generado en pantalla.

### Demo 2 — Farmacia dispensa

1. Login simulado de farmacia.
2. Escanea QR (o pega el código).
3. Sistema valida → muestra receta completa.
4. Presiona "Dispensar" → receta pasa a RETENIDA.
5. Intenta dispensar de nuevo → sistema rechaza con HTTP 409.

### Demo 3 — Concurrencia

1. Dos pestañas de farmacia abiertas.
2. Ambas escanean el mismo QR simultáneamente.
3. Solo una gana. La otra recibe "Receta ya dispensada".

### Demo 4 — Admin audita

1. Login de admin.
2. Vista de bitácora con eventos inmutables.
3. Click en un evento → muestra hash, actor, timestamp, IP.
4. Intento de alterar → sistema detecta y bloquea.

---

## Riesgos a vigilar

| Riesgo | Mitigación |
|---|---|
| La química pide features no contempladas | Prototipo rápido en Figma antes de código |
| El equipo se entusiasma y quiere ir directo a AWS | Regla dura: no AWS hasta validar POC |
| La POC se convierte en producción sin refactor | Definir desde el inicio que POC es desechable o refactorizable |
| Se subestima la complejidad de e.firma real | Dejar e.firma real para MVP, no POC |
| Se intenta cubrir todas las fracciones en POC | Solo Fracción II y IV en POC |
| Se olvida validar con farmacia además de médico | Incluir un farmacéutico en la validación |

---

## Herramientas concretas recomendadas

| Necesidad | Herramienta | Costo |
|---|---|---|
| Prototipo visual | Figma | Gratis |
| Bocetos rápidos | Excalidraw | Gratis |
| Alternativa soberana | Penpot | Gratis |
| POC local | Docker Compose | Gratis |
| Frontend | Next.js 15 | Gratis |
| Backend | NestJS 11 | Gratis |
| Base de datos local | PostgreSQL en Docker | Gratis |
| Redis local | Redis en Docker | Gratis |
| ORM | Prisma | Gratis |
| Alternativa sin Docker | Supabase | Gratis tier |
| Repo | GitHub | Gratis |
| Video demo | Loom | Gratis tier |
| Presentación | Google Slides | Gratis |

---

## Próximos pasos

1. **Esta semana:** agendar sesión de entrevista con la química farmacéutica.
2. **Semana 1:** wireframes de los 3 portales en Figma.
3. **Semana 2:** prototipo clickeable + presentación a la química.
4. **Semana 3:** arrancar POC local con Docker Compose.
5. **Semana 6:** demo funcional a la química.
6. **Semana 7:** decisión go/no-go para MVP Fracciones IV/V/VI.

---

## Preguntas abiertas

1. ¿La química farmacéutica tiene acceso a médicos y farmacéuticos que puedan participar en validación?
2. ¿Hay presupuesto para Figma Pro o se queda en gratuito?
3. ¿El equipo tiene experiencia con Docker Compose?
4. ¿La química prefiere ver primero la maqueta o esperar a la POC?
5. ¿Existe alguna farmacia piloto dispuesta a probar la POC sin compromiso?
6. ¿Cuánto tiempo tiene la química disponible para sesiones de validación?

---

## Entregables por fase

| Fase | Entregable | Audiencia |
|---|---|---|
| Maqueta | Link Figma + video 3 min | Química + stakeholders |
| POC | Repo con `docker compose up` + video 5 min | Química + equipo técnico |
| MVP Fracciones IV/V/VI | Sistema en AWS piloto | Química + farmacias aliadas |

---

¿Quieres que arranquemos con el **guion detallado para la entrevista con la química**, o prefieres que el UX Designer diseñe el **wireframe textual de los 3 portales** para pasarlo a Figma?