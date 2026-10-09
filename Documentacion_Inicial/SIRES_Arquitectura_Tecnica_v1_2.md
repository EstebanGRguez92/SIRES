# SIRES — Diseño de Arquitectura Técnica

**Versión:** 1.2 (APROBADA como base el 9-oct-2026; los ADRs y pendientes P-xx siguen abiertos)
**Fecha:** 9 de octubre de 2026
**Reemplaza a:** `SIRES_Arquitectura_Tecnica_v1_1` (docx/pdf, que internamente decía "Versión 1.0")
**Estatus:** Propuesta técnica. No es especificación de construcción, ni certificación de cumplimiento.

> **Naturaleza del documento.** Este documento diseña la arquitectura; no declara que SIRES sea
> legalmente conforme. La conformidad con NOM-004, NOM-024, Art. 226 LGS y la ley vigente de
> protección de datos personales se demuestra con evidencia y dictamen jurídico, no por usar este stack.

---

## 0. Control de cambios v1.1 → v1.2

### 0.1 Decisiones de dirección incorporadas (9-oct-2026)

| # | Decisión | Efecto |
|---|---|---|
| D1 | Se reabren Go, OpenSearch y Valkey: **autorizado** | Se retiran del MVP (§6, §19) |
| D2 | Validez legal por fracción: sin dictamen aún. Hay contacto directo con directivos de COFEPRIS | Se abre acción de consulta formal por escrito (§24, P-01) |
| D3 | Región de DR (pilot light) puede estar fuera de México **si los datos pueden regresar a México** | Se documenta como principio, sujeto a dictamen (§16, ADR-006) |

> Nota sobre D2: el contacto directo con COFEPRIS es una ventaja, pero **no sustituye un criterio
> por escrito**. Sin oficio o respuesta formal, la validez de la receta electrónica para
> Fracciones I–III sigue siendo un supuesto, no un hecho.

### 0.2 Corrección de los 11 errores técnicos de v1.1

| # | Error en v1.1 | Corrección en v1.2 | Sección |
|---|---|---|---|
| 1 | La LCO del SAT se usaba como lista de revocación, con caché diario | La LCO se elimina como control de revocación. Se usa estado de revocación del certificado (OCSP/CRL) con TTL máximo y fail-closed | §9.3 |
| 2 | `kms:Decrypt` "para validar la e.firma" | Verificar firmas solo usa la llave pública. KMS se reserva para datos personales, con llaves separadas por clase | §13.1, §13.2 |
| 3 | Orden de emisión: COMMIT en BD y luego S3; el sello NOM-151 no aparecía | Archivo de evidencia y sello pasan por Outbox; la receta no es surtible hasta que la evidencia esté archivada | §9.4 |
| 4 | Reglas validadas solo antes de firmar | El servidor construye el documento, el cliente firma ese hash y el servidor **revalida las reglas** al recibir la firma | §9.2, §9.4 |
| 5 | Tres versiones incompatibles de la máquina de estados | Una sola máquina de estados; caducidad derivada; retención como atributo de la dispensación | §8.1 |
| 6 | Control de surtido por receta, no por medicamento | Control por renglón (`prescription_item`) con restricciones únicas y `CHECK` de cantidad acumulada | §11.2 |
| 7 | `UNIQUE(curp)` imposible con cifrado de columna | Índice ciego (HMAC con llave KMS separada) | §11.1, §13.1 |
| 8 | Cadena de hashes sin ancla externa; cadena global serializa escrituras | Cadena por agregado + puntos de control firmados y anclados en S3 Object Lock en otra cuenta | §12 |
| 9 | Límite de 50 req/min por IP bloquea cadenas con IP compartida | Límite por credencial de sucursal en la aplicación; WAF solo con umbral alto por IP | §13.3 |
| 10 | "Purgar la .key de la RAM" como garantía | Se retira la garantía; se documentan mitigaciones y riesgo residual | §9.5 |
| 11 | SLO 99.9% mensual junto con RTO de 4 h | El SLO excluye eventos de desastre declarados; el RTO aplica solo a desastre | §21.2 |

### 0.3 Alineación con restricciones del proyecto

| Restricción | Qué cambió |
|---|---|
| 1. Solo recetas, sin expediente clínico ni CIE-11 | Se elimina HL7 CDA R2, búsqueda de diagnósticos y cifrado de "diagnósticos". El paciente no es usuario del sistema. |
| 2. Farmacias privadas | Sin cambios. |
| 5. Cumplimiento sin certificación ISO | Se retira "aseguramos que cumpla las NOM" (v1.1 §11.5). Se habla de mapeo de requisitos y evidencia. |
| 6–7. AWS, mx-central-1, una sola nube | Sin cambios. Servicios por confirmar en la región (§15.2). |
| 8. e.firma obligatoria para médicos | Se diseña el acceso del médico con e.firma (§13.2). Para farmacia, la e.firma **no** es requisito de proyecto; queda abierto (P-05). |
| 9. Prioridad OPEX y serverless | Se retiran OpenSearch, ElastiCache y EventBridge; Aurora Serverless v2; sin componentes siempre encendidos salvo Fargate mínimo. |
| 10. Equipo con experiencia básica | Un solo lenguaje (TypeScript) en MVP. Sin servicios en Go ni Rust. |
| Nunca: consultas en línea a COFEPRIS/SAT/SEP sin convenio | Se eliminan los circuit breakers hacia SEP/RENAPO/COFEPRIS; validación declarativa con evidencia (§14). La verificación de revocación SAT queda condicionada a dictamen (P-03). |
| Nunca: reglas de fracción hardcodeadas | Se mantiene y refuerza con versionado por receta (§11.3). |

---

## 1. Resumen ejecutivo

SIRES es un sistema transaccional crítico de recetas electrónicas para México. **No** es un
expediente clínico. Sus garantías son:

1. **No repudio**: la receta la firma el médico con su e.firma, en su navegador. La `.key` nunca llega al servidor.
2. **Cero doble surtido**: la garantía vive en PostgreSQL (lock transaccional + restricción única + idempotencia), no en un caché.
3. **Reglas por fracción** (Art. 226 LGS) parametrizadas y versionadas, nunca hardcodeadas.
4. **Bitácora verificable matemáticamente**, con anclaje externo.
5. **Evidencia inmutable** (S3 Object Lock).

Arquitectura: un **monolito modular en TypeScript** (NestJS) sobre ECS Fargate, Aurora PostgreSQL
Serverless v2 como única autoridad de estado, S3 Object Lock para evidencia, SQS para trabajo
asíncrono. Next.js para los portales. **Sin Go, sin OpenSearch, sin Valkey, sin EventBridge en el MVP.**

Principio rector: una sola fuente de verdad; todo lo demás (caché, búsqueda, CDN, mensajería) es
derivado y prescindible.

---

## 2. Alcance

### 2.1 Dentro del alcance
- Portal de médico: emisión y firma de recetas.
- Portal de farmacia: validación por QR y dispensación.
- Administración: catálogo, políticas por fracción, usuarios, auditoría.
- Motor de reglas por fracción (vigencia, topes, retención, número de surtidos).
- Libro de Control Digital firmable y auditable (modelo por definir, ADR-007).
- Verificación pública mínima de autenticidad de una receta (datos expuestos por definir, ADR-008).

### 2.2 Fuera de alcance
- Expediente clínico, CIE-11, diagnósticos, historia clínica.
- HL7 CDA R2 (el documento firmado es un documento de receta, no un documento clínico).
- Hospitales, IMSS, ISSSTE.
- Paciente como usuario del sistema. El paciente es un dato de la receta.
- Integraciones en línea con SEP, COFEPRIS, SAT o RENAPO sin convenio firmado.

### 2.3 No cerrado
- Validez legal de la receta electrónica por fracción (P-01).
- Contrato, APIs y SLA con dependencias gubernamentales: no existen; no se diseña sobre ellos.
- Modelo jurídico de retención.

---

## 3. Drivers

| Driver | Implicación técnica |
|---|---|
| Datos en México | Región primaria mx-central-1; matriz de ubicación de cada copia (§16) |
| Integridad de receta | ACID + estados explícitos + auditoría append-only + evidencia WORM |
| No repudio | Firma en cliente; verificación en servidor; sello de tiempo NOM-151 |
| Cero duplicidad | Lock en PostgreSQL + `UNIQUE` + idempotencia |
| Reglas por fracción | Políticas en tablas versionadas, referenciadas por receta |
| Sin convenios gubernamentales | Validación declarativa con evidencia y estados de verificación |
| Equipo básico | Un lenguaje, pocos servicios, todo administrado |
| OPEX / serverless | Sin componentes siempre encendidos innecesarios |

---

## 4. Principios

| Principio | Aplicación |
|---|---|
| Fuente de verdad única | Aurora PostgreSQL decide el estado legal. Ningún caché, índice o CDN decide. |
| Seguridad por diseño | La `.key` no se almacena ni se transmite. |
| Modularidad antes que fragmentación | Monolito modular; extracción solo con criterio medible (§7.3). |
| Eventos para desacoplar, no para sustituir ACID | Estado en transacción; evento vía Outbox. |
| Fail closed | Ante incertidumbre sobre firma, revocación, estado o integridad, se rechaza. |
| Idempotencia | Toda operación reintentable lleva `Idempotency-Key`. |
| Trazabilidad | `correlation_id`/`trace_id`, actor, resultado, hashes. |
| Reglas versionadas | Cada receta guarda la versión de política aplicada. |
| Evidencia, no declaración | Cada requisito normativo mapea a una prueba verificable. |
| Simplicidad operable | Si el equipo no puede operarlo sin un héroe único, no entra. |

---

## 5. Contexto

Actores: médico, farmacia (farmacéutico/responsable sanitario), administrador SIRES, verificador
público (consulta mínima). Dependencias externas en el MVP: **proveedor de sello NOM-151 (PSC)** y
**servicio de estado de revocación de certificados** (sujeto a P-03). Nada más.

```
 MÉDICO / FARMACIA / ADMIN / VERIFICADOR
                |
              HTTPS (TLS 1.3 + HSTS)
                |
        AWS WAF (regional, sobre ALB)
                |
               ALB
                |
      ECS Fargate (>=2 tareas, 2 AZ)
      NestJS modular (API + workers)
                |
     +----------+-----------+------------------+
     |          |           |                  |
 Aurora PG   S3 Object   SQS + DLQ        PSC NOM-151
 Serverless  Lock        (outbox,         (externo)
 v2          (evidencia) reintentos)
     |
 KMS / Secrets Manager / CloudTrail
```

---

## 6. Arquitectura lógica

```
+---------------------------------------------------------------+
| EXPERIENCIA   Next.js / TypeScript                            |
| Médico | Farmacia | Administración | Verificación pública     |
|   (módulo de firma aislado en origen/ruta dedicada)           |
+-------------------------------+-------------------------------+
                                | HTTPS / TLS 1.3
+-------------------------------v-------------------------------+
| PERÍMETRO   AWS WAF (regional) | ALB                          |
+-------------------------------+-------------------------------+
                                |
+-------------------------------v-------------------------------+
| NestJS MODULAR (un solo despliegue, un solo lenguaje)         |
| identity | practitioner | patient | catalog | regulatory      |
| prescription | dispensing | pharmacy | audit | evidence       |
| integration (adaptadores inactivos hasta convenio)            |
| shared: auth, idempotency, errors, observability, crypto      |
+-------+------------------------+------------------------------+
        |                        |
+-------v--------+     +---------v---------+
| Aurora PG      |     | S3 Object Lock    |   SQS + DLQ (workers
| estado + audit |     | evidencia WORM    |   dentro del mismo
+----------------+     +-------------------+   código base)
```

**Retirado del MVP (decisión D1):**

| Componente v1.1 | Motivo | Reemplazo | Criterio para reintroducir |
|---|---|---|---|
| Go Crypto Service | Verificar firmas solo usa llave pública; Node lo hace nativo | Módulo `crypto` en NestJS | Auditoría de seguridad exige aislamiento de proceso/permisos |
| Go Dispensing Service | 50 TPS es bajo para NestJS+PG | Módulo `dispensing` con frontera limpia | p95 de dispensación > objetivo con BD sana y tras optimización |
| OpenSearch | Catálogo pequeño; diagnósticos fuera de alcance; costo fijo 24/7 | Búsqueda en PostgreSQL (`pg_trgm` + full-text) | Latencia de búsqueda de catálogo p95 > 300 ms con índices correctos |
| ElastiCache Valkey | Locks e idempotencia ya en PG; LCO no aplica | Ninguno | Métrica demostrada de carga de lectura que PG no absorba |
| EventBridge | Sin múltiples consumidores en MVP | SQS directo vía Outbox | Más de un consumidor independiente de eventos |

---

## 7. Arquitectura de aplicación

### 7.1 Frontend — Next.js + TypeScript
- Portales: médico, farmacia, administración, verificación pública.
- Módulo de firma **aislado**: ruta/origen dedicado, CSP estricta, sin scripts de terceros, Subresource Integrity, `Trusted Types` cuando aplique.
- Prohibido persistir `.key`, contraseña o material secreto en `localStorage`, `IndexedDB`, cookies, logs o telemetría.
- Separación: UI, cliente API, sesión, formulario de receta, componente de firma.

### 7.2 Backend — NestJS por dominios
```
src/modules/
  identity/ practitioner/ patient/ catalog/ regulatory/
  prescription/ dispensing/ pharmacy/ evidence/ audit/ integration/
src/shared/
  auth/ idempotency/ errors/ observability/ crypto/
```
Reglas de frontera: un módulo solo accede a las tablas de su esquema; la comunicación entre módulos es
por interfaz explícita. Esto es lo que permite extraer un módulo en el futuro.

### 7.3 Criterios de extracción de servicios
Un módulo se extrae solo si se cumple **al menos una**, medida y documentada en un ADR:
1. Requisito de aislamiento de permisos/llaves que un proceso único no puede satisfacer.
2. Escala o latencia fuera de objetivo tras optimización.
3. Ciclo de despliegue independiente justificado por riesgo regulatorio.

Nunca por simetría tecnológica ni por preferencia de lenguaje.

---

## 8. Dominios

| Dominio | Responsabilidad | Persistencia |
|---|---|---|
| Identity & Access | Usuarios, roles, sesiones, vínculo e.firma ↔ usuario | PostgreSQL / IdP (ADR-003) |
| Practitioner | Médico, cédula declarada, estado de verificación | PostgreSQL |
| Patient | Datos mínimos del paciente para la receta | PostgreSQL (cifrado de campo) |
| Catalog | Medicamentos, presentación, fracción | PostgreSQL |
| Regulatory Rules | Políticas por fracción, versionadas | PostgreSQL |
| Prescription | Receta, renglones, firma, estados | PostgreSQL + S3 |
| Pharmacy | Sucursales, usuarios, responsable sanitario | PostgreSQL |
| Dispensing | Surtidos por renglón, retención | PostgreSQL |
| Evidence | Artefactos firmados, sellos | S3 + PostgreSQL |
| Audit | Eventos, cadenas, checkpoints | PostgreSQL + S3 |
| Integration | Adaptadores (inactivos hasta convenio), colas | PostgreSQL + SQS |

### 8.1 Máquina de estados de receta (única, normativa de este documento)

Estados persistidos:

```
BORRADOR ──firma verificada──► EMITIDA ──primer surtido parcial──► PARCIALMENTE_SURTIDA
                                  │                                     │
                                  │ último renglón completo             │ último renglón completo
                                  ▼                                     ▼
                              SURTIDA_TOTAL ◄───────────────────────────┘
     (BORRADOR | EMITIDA | PARCIALMENTE_SURTIDA) ──cancelación autorizada──► CANCELADA
```

Reglas:
- **La caducidad no es un estado que gobierna la decisión.** Se deriva de `fecha_caducidad < now()` y se evalúa en cada dispensación. Un job puede marcar `caducada_at` solo para reportes.
- **La retención no es un estado de la receta.** Es un atributo del registro de dispensación (`retiene_receta`), según la política de la fracción vigente al emitir.
- **Evidencia y sello son dimensiones ortogonales**, no estados de la receta:

| Dimensión | Valores |
|---|---|
| `evidencia_estado` | `PENDIENTE`, `ARCHIVADA` |
| `sello_estado` | `PENDIENTE`, `OBTENIDO`, `FALLIDO` |

- **Una receta es surtible solo si:** estado ∈ {`EMITIDA`, `PARCIALMENTE_SURTIDA`} **y** `now() < fecha_caducidad` **y** `evidencia_estado = ARCHIVADA` **y** hay saldo en al menos un renglón **y** la política lo permite. Política para `sello_estado = PENDIENTE`: ver ADR-001 (propuesta inicial: bloquea surtido en Fracciones I–III; permite ventana acotada en el resto; **por validar**).
- Toda transición la ejecuta un servicio de dominio, valida contexto regulatorio, escribe evento de auditoría y Outbox en la misma transacción. Prohibido `UPDATE` directo del estado desde la UI o desde scripts.
- Cancelación: quién, cuándo y con qué firma → ADR-002 (P-06).

---

## 9. Flujo de prescripción y firma

### 9.1 Formato del documento firmado
No se usa HL7 CDA R2 (fuera de alcance). El formato (XMLDSig sobre XML canónico vs. JSON canónico RFC 8785 con firma CMS separada) se decide en **ADR-001**, con prueba de concepto. Requisito inamovible: el documento firmado debe ser reproducible byte a byte desde el estado guardado.

### 9.2 Principio: el servidor construye, el cliente firma lo que ve
1. El médico captura la receta en un **borrador** (`BORRADOR`, sin validez).
2. El backend evalúa las reglas (profesional verificado, medicamento, fracción, cantidades, vigencia) y **construye el documento canónico**, calcula su hash y lo vincula al borrador (`draft_hash`, `policy_version_id`).
3. El cliente **muestra al médico el contenido que se firmará** (WYSIWYS) y verifica localmente que el hash mostrado corresponde al contenido.
4. El cliente firma ese hash/documento con la e.firma. La `.key` nunca sale del navegador.
5. El backend recibe firma + certificado y **revalida todo**: firma válida sobre el `draft_hash` esperado, borrador sin cambios, reglas aún cumplidas con la versión de política registrada.

Así, un cliente manipulado no puede firmar contenido distinto del evaluado por las reglas (corrige error #4).

### 9.3 Verificación de certificado y revocación (corrige error #1)
- La **LCO** del SAT es una lista de contribuyentes para CFDI; **no** informa revocación de e.firma. Se elimina como control.
- Controles de verificación en servidor, en este orden, todos fail-closed:
  1. Cadena de certificación hasta la AC raíz del SAT (certificados de confianza versionados en repositorio y aprobados).
  2. Vigencia del certificado a la fecha de firma.
  3. Uso de llave y tipo de certificado (e.firma, no CSD).
  4. **Estado de revocación** por OCSP o CRL de la AC, con TTL máximo definido en ADR-001.
  5. Coincidencia RFC/CURP del certificado con el médico registrado.
- Si el estado de revocación no puede determinarse: **se rechaza la emisión**.
- **Pendiente crítico (P-03):** confirmar si el OCSP/CRL del SAT es de uso público para terceros o requiere convenio. Si requiere convenio, aplica la regla de proyecto de no consultar en línea sin convenio y la alternativa será CRL descargada de publicación oficial pública, definida en ADR-001.

### 9.4 Secuencia de emisión corregida (errores #3 y #4)

```
Médico      Next.js        NestJS (prescription/crypto)      Aurora        SQS/worker      S3 Lock     PSC
  |  captura   |                 |                             |              |              |          |
  |----------->| POST /drafts    |                             |              |              |          |
  |            |---------------->| reglas + documento canónico |              |              |          |
  |            |                 |---- INSERT draft ---------->|              |              |          |
  |            |<-- doc + hash --|                             |              |              |          |
  | revisa y firma localmente (.key no sale del navegador)     |              |              |          |
  |            | POST /drafts/{id}/sign (firma + .cer)         |              |              |          |
  |            |---------------->| verifica firma + revoc. + RFC|             |              |          |
  |            |                 | revalida reglas (policy ver.)|             |              |          |
  |            |                 |== TX: receta EMITIDA ========|             |              |          |
  |            |                 |   + audit_event + hash chain |              |              |          |
  |            |                 |   + outbox(ARCHIVAR_EVIDENCIA, SOLICITAR_SELLO)         |          |
  |            |<-- 201 (receta emitida, NO surtible aún) ------|              |              |          |
  |            |                 |                             |  outbox ---->|              |          |
  |            |                 |                             |              | PutObject    |          |
  |            |                 |                             |              | (clave = sha256, idempotente)
  |            |                 |                             |              |------------->|          |
  |            |                 |                             |              | solicita sello ------->|
  |            |                 |                             |<-- evidencia_estado=ARCHIVADA, sello_estado=OBTENIDO
```

Propiedades:
- No hay estado en el que exista receta emitida sin que el sistema sepa que falta evidencia: la bandera `evidencia_estado = PENDIENTE` lo registra y bloquea el surtido.
- La clave del objeto en S3 es el hash del contenido: reintentar nunca crea duplicados. No se suben objetos con retención antes del COMMIT, por lo que no hay huérfanos con retención irreversible.
- El QR se entrega al médico al emitir, pero la farmacia recibirá "no surtible aún" hasta que la evidencia esté archivada. Se espera que sea cuestión de segundos; se monitorea la latencia (§17).
- El sello NOM-151 se obtiene sobre el hash del documento firmado (modalidad exacta en ADR-001; sellado por lote es opción de ahorro de OPEX, sujeta a dictamen).

### 9.5 Manejo de la `.key` en el navegador (corrige error #10)
**v1.1 prometía que la llave "se destruye de la RAM". Eso no se puede garantizar en un navegador** (recolector de basura, copias intermedias, extensiones).

Lo que sí se compromete:

| Control | Detalle |
|---|---|
| Superficie mínima | Ruta de firma en origen aislado, CSP estricta sin `unsafe-inline`/`unsafe-eval`, sin scripts de terceros, SRI en todos los recursos |
| Vida mínima | El archivo `.key` y la contraseña se leen solo al firmar; se sobrescriben los buffers propios (`fill(0)`) en cuanto termina; no se retienen en estado de UI |
| Llave no extraíble | Siempre que sea posible, importar como `CryptoKey` con `extractable: false` |
| Cero persistencia | Nada en storage, cookies, logs, telemetría ni reportes de error |
| Compatibilidad | El `.key` del SAT es PKCS#8 cifrado; WebCrypto no lo descifra directamente. La PoC decide si basta una librería JS auditada o si se requiere Wasm (ADR-001) |
| Riesgo residual | Malware/extensión maliciosa en el equipo del médico puede capturar la llave. Se documenta, se informa al médico y se mitiga con MFA adicional, alertas y revocación rápida |

---

## 10. Dispensación y anti-duplicidad

### 10.1 Flujo (módulo `dispensing` en NestJS)

```
Farmacia scan QR + Idempotency-Key
   |
   v
BEGIN
  SELECT idempotency (si existe respuesta guardada -> devolverla)
  SELECT * FROM prescription WHERE qr_uuid = ? FOR UPDATE
  validar: estado, caducidad, evidencia ARCHIVADA, sello según política,
           identidad/estado de la farmacia, saldo por renglón
  si no válida -> ROLLBACK -> 409 / 422 (respuesta determinística)
  INSERT dispensing_records + dispensing_items (por renglón)
  UPDATE prescription_items.cantidad_surtida (CHECK <= cantidad)
  UPDATE prescription.estado (por servicio de dominio)
  INSERT audit_event + chain + outbox(RecetaDispensada)
  INSERT idempotency_key + respuesta
COMMIT
   |
   v
200 OK
```

### 10.2 Controles
- Lock de fila sobre la receta (`FOR UPDATE`) con timeout corto de lock (`lock_timeout`).
- Restricciones `UNIQUE` y `CHECK` por renglón (§11.2) como **última línea de defensa independiente del código**.
- `Idempotency-Key` obligatoria; reintento devuelve la misma respuesta.
- Sin Redlock: no garantiza exclusión mutua ante pausas de proceso ni desfase de reloj; no puede ser autoridad legal. La regla de proyecto "Redlock + restricción única" se interpreta como **"lock transaccional + restricción única"**; se formaliza en ADR-002.
- Sin operación offline en farmacia: burn-on-read necesita estado central. Contingencia: surtir en otra farmacia con conectividad. Confirmación de dirección pendiente (P-07).

### 10.3 Outbox
La transacción escribe el evento en `outbox_events`. Un worker lo publica a SQS y lo marca enviado; entrega al-menos-una-vez, consumidores idempotentes. Se monitorea la antigüedad del evento más viejo sin publicar.

---

## 11. Datos

Aurora PostgreSQL con esquemas por dominio (`identity`, `clinical`, `catalog`, `pharmacy`, `audit`). UUID como identificador. `JSONB` solo para metadatos no regulados.

### 11.1 `clinical`

| Tabla | Columnas clave | Controles |
|---|---|---|
| `practitioners` | id, `curp_hash`, `cedula_hash`, cedula cifrada, `estado_verificacion` | `UNIQUE(curp_hash)`, `UNIQUE(cedula_hash)` |
| `patients` | id, `curp_hash`, `curp_cifrado`, nombre cifrado, fecha_nacimiento cifrada, `tipo_identificador` | `UNIQUE(curp_hash)` cuando exista CURP; soporte para extranjeros y recién nacidos sin CURP |
| `prescriptions` | id, practitioner_id, patient_id, `estado`, `fecha_emision`, `fecha_caducidad`, `qr_uuid`, `draft_hash`, `policy_version_id`, `evidencia_estado`, `sello_estado` | `estado` ENUM de §8.1; `UNIQUE(qr_uuid)`; FK a política versionada |
| `prescription_items` | id, prescription_id, medication_id, `cantidad_prescrita`, `cantidad_surtida`, frecuencia, duracion_dias | `CHECK (cantidad_surtida <= cantidad_prescrita)`; FK a catálogo; sin texto libre en dosis |

**Índice ciego (error #7):** el cifrado aleatorio impide `UNIQUE` y búsqueda. Se guarda `curp_hash = HMAC-SHA256(llave_indice, curp_normalizada)`; la llave de índice es **distinta** de la de cifrado, vive en KMS y solo la usa el módulo que maneja pacientes. Permite unicidad y búsqueda exacta; no permite búsqueda parcial (decisión aceptada).

### 11.2 `pharmacy` (error #6)

| Tabla | Columnas clave | Controles |
|---|---|---|
| `dispensing_records` | id, prescription_id, pharmacy_id, pharmacist_id, `numero_surtido`, `retiene_receta`, fecha_surtido, idempotency_key | `UNIQUE(prescription_id, numero_surtido)`; inmutable (sin UPDATE/DELETE) |
| `dispensing_items` | id, dispensing_record_id, prescription_item_id, `cantidad` | `UNIQUE(dispensing_record_id, prescription_item_id)`; `CHECK (cantidad > 0)`; el acumulado se protege con el `CHECK` de `prescription_items` |
| `idempotency_keys` | key, scope, request_hash, response_body, created_at | `UNIQUE(scope, key)`; TTL ≥ 24 h; conflicto de `request_hash` con misma key → 422 |

### 11.3 `catalog`

| Tabla | Columnas clave | Controles |
|---|---|---|
| `medications` | id, sustancia_activa, presentación, `fraccion_id` | FK a fracción; clasificación aprobada por responsable regulatorio |
| `medication_policies` | id, `fraccion_id`, `medication_id` (opcional, para excepciones), max_cajas, dias_vigencia, requiere_retencion, requiere_folio_cofepris, `vigente_desde`, `vigente_hasta`, `version` | Sin solapes de vigencia (restricción de exclusión); inmutable; cambio = nueva versión; cada receta guarda `policy_version_id` |

Soporta excepciones por medicamento o presentación, sin código. Cambios de política siguen su propio ciclo de liberación (datos, con doble aprobación).

### 11.4 `audit`
Ver §12. Incluye `audit_events`, `audit_chain_heads`, `audit_checkpoints`, `outbox_events`, `signed_artifacts`.

### 11.5 Reglas globales
- Cambio de estado y registros asociados en una sola transacción.
- Cifrado de campo (AWS Encryption SDK + KMS) para datos personales del paciente y cédula.
- Ningún dato personal ni clínico en logs.
- Versionado de catálogo y políticas para reproducir el contexto exacto de una receta histórica.
- Esta arquitectura **soporta** el cumplimiento de NOM-004 y NOM-024 en persistencia; la conformidad se demuestra con la matriz de trazabilidad (§25), no se declara.

---

## 12. Auditoría, trazabilidad e inmutabilidad (error #8)

### 12.1 Capas

| Capa | Contenido | Propósito |
|---|---|---|
| Estado transaccional | Receta, dispensación, catálogo | Operación y ACID |
| Audit/Event log | Eventos append-only con cadena de hashes | Detección de alteración |
| Evidencia WORM | Documento firmado, sello, checkpoints | Preservación inmutable |
| Cloud audit | CloudTrail | Auditoría de plataforma |

### 12.2 Cadena por agregado, no global
- Una cadena **por agregado** (receta, farmacia, médico): `current_hash = SHA256(previous_hash || payload_hash || event_id || timestamp)`.
- Evita serializar todas las escrituras de bitácora del sistema; el orden se garantiza con lock por agregado (ya existe en dispensación y emisión).

### 12.3 Anclaje externo (lo que faltaba)
Una cadena dentro de la misma base **no** protege contra un administrador con privilegios altos (puede desactivar triggers y recalcular). Por eso:

1. Cada N minutos un job calcula una **raíz Merkle** de los últimos `current_hash` y la guarda como `audit_checkpoint`.
2. El checkpoint se escribe como objeto en un bucket **S3 Object Lock Compliance en una cuenta AWS separada** ("cuenta de evidencia"), a la que la cuenta de aplicación solo puede escribir (sin lectura ni borrado).
3. Periódicamente (diario) el checkpoint se sella con NOM-151 (sujeto a costo y dictamen).
4. Un job de verificación recalcula las cadenas y las compara con los checkpoints anclados; cualquier discrepancia dispara alerta de seguridad.

### 12.4 Separación de privilegios
- El rol de la aplicación solo tiene `INSERT` sobre `audit.*` (revocados `UPDATE`/`DELETE`/`TRUNCATE`).
- Sin uso diario de superusuario; acceso de emergencia (break-glass) con aprobación, MFA, sesión registrada y alerta automática.
- Los eventos incluyen `correlation_id`, actor, IP, resultado y hashes.

### 12.5 Retención
La duración de Object Lock **no está definida** (P-08). Hasta tener la matriz de retención aprobada:
- Ambientes dev y staging: modo **Governance**.
- Producción: no se activa Compliance sin aprobación jurídica, porque es irreversible.

---

## 13. Seguridad

### 13.1 Cifrado y llaves (error #2)
- En reposo: Aurora, S3 y volúmenes con AES-256 y CMK en KMS, rotación anual automática.
- Llaves separadas por clase de dato:

| Llave | Uso | Quién puede usarla |
|---|---|---|
| `datos-paciente` | Cifrado de campo de datos del paciente | Solo rol del módulo `patient` |
| `indice-ciego` | HMAC de CURP/cédula | Solo rol del módulo `patient`/`practitioner` |
| `evidencia` | Cifrado de S3 de evidencia | Servicio de evidencia |
| `datos-generales` | Aurora, SQS, logs | Servicios base |

- **Verificar una firma no requiere KMS.** No existe permiso `kms:Decrypt` para "validar la e.firma".
- Secretos: AWS Secrets Manager, inyectados en ejecución. Sin `.env` con secretos.
- Dado que hay un solo despliegue de aplicación, el aislamiento por rol IAM se hace a nivel de **política de llave KMS + contexto de cifrado** y no por contenedor. Si se requiere aislamiento por proceso, es criterio de extracción (§7.3).

### 13.2 Identidad y acceso
- **Médico: e.firma obligatoria.** Sin e.firma, sin acceso. Mecanismo propuesto: autenticación por reto firmado (el cliente firma un reto con la e.firma; el servidor verifica y vincula al RFC/CURP registrado). Detalle y proveedor de identidad en **ADR-003**.
- **Farmacia:** autenticación mediante IdP con MFA. La e.firma de farmacia no es requisito del proyecto; si dirección la exige, se abre en P-05.
- Alta de médicos y farmacias: **declarativa con evidencia** (cédula, licencia sanitaria, responsable sanitario) y revisión manual; estados de verificación explícitos. Ver §14.
- RBAC por actor; mínimo privilegio; MFA obligatorio para administradores.
- Roles de ejecución IAM por tarea ECS; acceso privado a servicios AWS por VPC Endpoints donde existan en la región.

### 13.3 Perímetro (error #9)
- **WAF regional sobre el ALB en mx-central-1** (si se usa WAF de CloudFront, su configuración y logs residen fuera de México; ver P-09).
- Reglas gestionadas OWASP.
- Rate limiting:
  - **WAF:** umbral alto por IP (anti-DDoS genérico), no para dispensación fina.
  - **Aplicación:** límite por credencial de sucursal/usuario, que es la unidad real de abuso. Las cadenas de farmacias con una IP de salida no se bloquean entre sí.
- Geo-bloqueo: permitir México; considerar la región de operación real (médicos en el extranjero, soporte). Decisión: **bloquear por defecto fuera de México** con lista blanca para soporte. (La v1.1 permitía todo Norteamérica; se endurece por principio de mínima superficie.)
- TLS 1.3 + HSTS en ALB. Verificar política TLS soportada en el ALB de mx-central-1 y documentar excepciones.

### 13.4 Auditoría de plataforma
CloudTrail a nivel organización, hacia la cuenta de evidencia (Object Lock). Validación de integridad de logs activada.

### 13.5 Protección de datos personales
La ley de datos personales en posesión de particulares fue **sustituida en 2025**; hay que confirmar texto, autoridad y obligaciones vigentes con el dictamen jurídico (P-02). Hasta entonces, se diseña con minimización, aviso de privacidad, cifrado y trazabilidad de acceso.

---

## 14. Integraciones externas (rediseñadas)

La regla del proyecto es no consultar en línea COFEPRIS/SAT/SEP/RENAPO sin convenio firmado. Por tanto, **v1.1 §14.1–14.3 (circuit breakers y colas para SEP/RENAPO/COFEPRIS) se retiran del MVP**.

| Integración | MVP | Cuando exista convenio |
|---|---|---|
| SEP (cédula/especialidad) | Carga declarativa + evidencia (documento) + revisión manual. Estado `VERIFICACION_MANUAL_PENDIENTE/APROBADA/RECHAZADA` | Adaptador sobre el contrato real |
| RENAPO (CURP) | Validación **local de formato y dígito verificador**; sin consulta | Adaptador, validación asíncrona |
| COFEPRIS / folios Fracción I | Carga segura de folio y evidencia por el médico; SIRES **no fabrica ni asume folios** | Adaptador según contrato |
| SAT (e.firma) | Verificación criptográfica local + estado de revocación según P-03 | Revisar con convenio |
| PSC NOM-151 | Contrato comercial con proveedor autorizado; es la única dependencia externa operativa | — |

- Patrón: **adaptador desactivado por feature flag**, con interfaz definida y pruebas contra mock. El circuit breaker genérico (timeout, umbral, half-open) se mantiene como utilidad compartida para el PSC y futuros adaptadores; los umbrales de v1.1 (2.5 s / 30% / 10 s / 60 s) quedan como valores iniciales **a calibrar con pruebas**.
- **Fracción I (v1.1 §14.4, nota editorial resuelta):** carga segura de folios y evidencia; sin regla local que fabrique folios; todo permiso que desbloquee la UI conserva evidencia de origen y vigencia.
- **Especialidad (revisar supuesto):** v1.1 oculta el catálogo de Fracciones II/III si la especialidad no está verificada. No hay evidencia en este análisis de que la LGS exija especialidad para esas fracciones. Se parametriza en las políticas (`requiere_especialidad`) y se confirma con el dictamen (P-01).

### Cola de reintentos (SQS)
Aplica al PSC, outbox y a adaptadores futuros: cola principal + reintentos con retraso creciente + DLQ tras 5 intentos + alarma CloudWatch. Máximo de reintentos y retrasos configurables.

---

## 15. Infraestructura AWS

### 15.1 Componentes MVP

| Servicio | Uso | Criterio OPEX |
|---|---|---|
| ECS Fargate | NestJS (API + workers), Next.js | Mínimo 2 tareas pequeñas en 2 AZ; autoescalado |
| Aurora PostgreSQL Serverless v2 | Estado + audit | Capacidad mínima baja en no-producción; piso fijo en producción |
| S3 Object Lock | Evidencia y checkpoints | Pago por uso |
| SQS + DLQ | Outbox, reintentos | Pago por uso |
| KMS / Secrets Manager | Llaves y secretos | Costo bajo, fijo |
| WAF regional + ALB | Perímetro | Costo fijo moderado |
| CloudWatch + OpenTelemetry | Observabilidad | Controlar retención y volumen |
| CloudTrail | Auditoría de plataforma | Bajo |
| ECR | Imágenes | Bajo |

### 15.2 Servicios por confirmar en mx-central-1
Antes de comprometer diseño, confirmar disponibilidad y funciones en la región: Aurora Serverless v2 (incluyendo comportamiento de capacidad mínima), WAF regional, VPC Endpoints necesarios, KMS (CMK, multi-región), Secrets Manager, SQS, AWS Backup y copia entre regiones, y proveedor de identidad (Cognito u otro). Cada hallazgo se registra en la matriz de servicios (P-09).

### 15.3 Cuentas
AWS Organizations: `management`, `seguridad/evidencia` (Object Lock, CloudTrail), `dev`, `staging`, `prod`. El rol de aplicación en `prod` solo puede escribir en la cuenta de evidencia.

---

## 16. Alta disponibilidad, continuidad y DR

| Componente | Diseño base |
|---|---|
| ECS | ≥ 2 tareas en 2 AZ, despliegue rolling con rollback |
| Aurora | Multi-AZ (instancia lectora/failover automático) |
| S3 | Versioning + Object Lock |
| SQS | Durable con DLQ |
| DR | Estrategia **pilot light** (restauración desde respaldos) |

**Objetivos de recuperación propuestos (por validar en P-04):** RTO 4 h, RPO 15 min para desastre de región. Son objetivos de diseño, no compromiso.

**Residencia y DR (decisión D3):** se admite región de DR fuera de México **si los datos pueden regresar a México** tras el evento. Condiciones de diseño:
- Respaldos cifrados con llave cuyo control permanezca bajo SIRES.
- Plan documentado de retorno a mx-central-1 (runbook y ensayo).
- Matriz de clasificación y ubicación de cada copia, log, respaldo y servicio (P-09).
- Dictamen jurídico previo sobre transferencia internacional de datos personales de salud (P-02). La afirmación de que esto "parece ser una condición para un sistema de salud" no está verificada; se incluye en el dictamen.
- Región concreta, tipo de datos que viajan y retención de copias: **ADR-006**.

---

## 17. Observabilidad y operación

- Logs JSON con UTC, servicio, entorno, `correlation_id`, `trace_id`, `actor_id` pseudonimizado, operación, resultado. Nunca: `.key`, contraseñas, tokens completos, secretos, datos del paciente.
- Métricas: latencia p50/p95/p99, 5xx, 409/422, colas, antigüedad de outbox, DLQ, conexiones a BD, capacidad ACU.
- Trazas con OpenTelemetry de punta a punta.
- Alertas: fallo de verificación de firma, crecimiento de DLQ, outbox atrasado, `evidencia_estado = PENDIENTE` antigua, discrepancia de checkpoint, intento de acceso break-glass, picos de 409/422.
- Señal crítica: **cualquier doble surtido lógico es incidente severidad 1.**

---

## 18. DevSecOps

- GitHub Actions: lint, pruebas, SAST, dependencias, contenedor, IaC, secretos.
- **IaC: Terraform** (decisión v1.1 se conserva). *Nota: la regla del rol DevOps menciona CDK; se unifica una sola herramienta en ADR de plataforma antes de la Fase 1.*
- Despliegue dev → staging → prod con rollback.
- Migraciones expand/contract.
- Feature flags para funcionalidades de riesgo y adaptadores.
- Releases de **datos regulatorios** separados de releases de código, con doble aprobación.
- Revisión obligatoria de cada PR que toque auth, crypto, folios o dispensación.
- Pruebas de concurrencia de dispensación en CI como puerta de calidad.

---

## 19. Stack tecnológico

| Capa | Tecnología | Estado |
|---|---|---|
| Cloud | AWS mx-central-1 | Elegido |
| Frontend | Next.js + TypeScript | Elegido |
| Firma en cliente | Librería JS auditada o Wasm; formato XMLDSig/CMS | **Pendiente PoC (ADR-001)** |
| Backend | NestJS + TypeScript (monolito modular) | Elegido |
| Base de datos | Aurora PostgreSQL Serverless v2 | Elegido (confirmar región) |
| Evidencia | S3 Object Lock | Elegido (retención pendiente) |
| Mensajería | SQS + DLQ | Elegido |
| Runtime | ECS Fargate | Elegido |
| Perímetro | WAF regional + ALB | Elegido (CloudFront opcional para estáticos) |
| Secretos y llaves | Secrets Manager + KMS | Elegido |
| Observabilidad | CloudWatch + OpenTelemetry | Elegido |
| CI/CD | GitHub Actions + ECR + ECS | Elegido |
| IaC | Terraform | Elegido (confirmar vs. CDK) |
| Go, OpenSearch, Valkey, EventBridge | — | **Retirados del MVP** |

---

## 20. Decisiones frente a la propuesta original (documento base)

| Tema | Propuesta original | Decisión |
|---|---|---|
| Cloud | Azure (alt. AWS) | AWS |
| Microservicios | Sí | Monolito modular |
| Event Sourcing | Central | Audit log + Outbox |
| Ledger | Azure SQL Ledger | Aurora PG + S3 Object Lock + checkpoints anclados |
| Redlock | Cero duplicidad | Lock transaccional + `UNIQUE`; Redlock no es autoridad |
| Edge QR | Lambda@Edge | Validación en backend regional |
| Kubernetes | EKS | ECS Fargate |
| Búsqueda | Typesense/ES/OpenSearch | PostgreSQL en MVP |

---

## 21. Requisitos no funcionales

### 21.1 Dimensionamiento
La volumetría **no está confirmada**. Los valores de v1.1 se conservan como **techo de diseño**, no como baseline de MVP:

| Parámetro | Techo de diseño (v1.1) |
|---|---|
| Médicos activos diarios | 10,000 |
| Farmacias conectadas | 20,000 |
| Recetas por mes | ~500,000 |
| Pico dispensación | 50 TPS |

Observación: 500,000 recetas/mes equivalen a menos de 1 transacción por segundo en promedio; 50 TPS es un techo muy holgado. El MVP se dimensiona mucho más abajo y se escala con pruebas de carga.

### 21.2 Metas de servicio (error #11)
Objetivos de diseño, no SLA.

| NFR | Objetivo | Validación |
|---|---|---|
| Disponibilidad API de negocio | ≥ 99.9% mensual, **excluyendo eventos de desastre declarados** | Synthetic monitoring |
| Recuperación ante desastre | RTO 4 h / RPO 15 min (propuesto, P-04) | Simulacros de restauración |
| Latencia de consulta | p95 < 500 ms | Pruebas de carga |
| Escaneo/dispensación | p95 < 300 ms, excluyendo fallas externas | Pruebas de carga y concurrencia |
| Doble surtido | 0 bajo prueba de carrera | Pruebas de concurrencia en CI |
| Integridad | 100% de transacciones críticas con audit event | Reconciliación |
| Bitácora | Verificación de cadena y checkpoints diaria | Job de verificación |
| Seguridad | 0 secretos en repositorios o logs | Secret scanning |
| Trazabilidad | 100% de requests críticas | Auditoría de cobertura |

---

## 22. Roadmap técnico (Etapas A–F)

> Se usa "Etapa" en lugar de "Fase" para no confundirse con las fases de producto
> (Fase 0 Maqueta, Fase 1 POC, Fase 2 MVP) definidas en `Fases_Iniciales.md` y en
> `.cursor/rules/current-phase.mdc`. Las etapas A y B ocurren dentro de la Fase 0/1 de
> producto; la etapa C es el contenido técnico de la Fase 2 (MVP) y arranca por
> **Fracciones IV, V y VI** (menor restricción, sin convenios gubernamentales necesarios).
> Fracciones I, II y III quedan en diseño/maqueta hasta tener el criterio de COFEPRIS.

| Etapa | Contenido | Prerrequisitos |
|---|---|---|
| A. Validación | ADR-001, 002, 003; PoC de firma; consulta formal a COFEPRIS; dictamen jurídico; matriz de servicios en región; cotización PSC | — |
| B. Fundamentos | Cuentas AWS, red, KMS, CloudTrail, Terraform, CI/CD, observabilidad base; módulos `identity`, `practitioner`, `catalog`, `regulatory`, `audit` | ADR-001/003, P-09 |
| C. Fracciones IV/V/VI | Emisión con firma, evidencia, sello; dispensación; Libro de Control, para las fracciones menos restringidas | ADR-002, ADR-004 |
| D. Fracciones II/III | Mismo flujo que C, extendido a psicotrópicos con retención | Dictamen P-01, ADR-004 |
| E. Fracción I | Carga de folios y evidencia del médico | Criterio COFEPRIS |
| F. Escala e integraciones | Pruebas de carga, DR ensayado, hardening, pentest; adaptadores SEP/COFEPRIS/RENAPO/SAT según convenios reales | Volumetría real, convenios firmados |

---

## 23. Riesgos

| # | Riesgo | Impacto | Mitigación | Dueño |
|---|---|---|---|---|
| R1 | Validez legal de receta electrónica por fracción sin criterio escrito | Existencial | Consulta formal a COFEPRIS + dictamen | Dirección / Legal |
| R2 | Formato de firma o `.key` inviable en navegador | Crítico | PoC con `.key` reales en Chrome, Edge, Safari | CTO / Seguridad |
| R3 | Doble surtido | Crítico | Lock + `UNIQUE` + `CHECK` + idempotencia + pruebas de carrera | Tech Lead |
| R4 | Revocación de e.firma no verificable sin convenio | Alto | P-03; fail-closed; CRL pública si aplica | Seguridad |
| R5 | Retención Object Lock mal configurada (irreversible) | Crítico | Governance hasta aprobación; matriz de retención | Legal / CTO |
| R6 | Costo del sello NOM-151 por receta | Alto (OPEX) | Cotización; evaluar sellado por lote | CTO |
| R7 | DR fuera de México vs. residencia de datos | Alto | Dictamen; plan de retorno; ADR-006 | Legal / DevOps |
| R8 | Servicios no disponibles en mx-central-1 | Alto | Matriz de servicios antes de Fase 1 | DevOps |
| R9 | Robo de `.key` por malware/extensión | Alto | CSP, origen aislado, MFA, revocación; riesgo residual documentado | Seguridad |
| R10 | Scope creep hacia lo clínico | Medio | Revisión de alcance en cada ADR | Product Owner |
| R11 | Dependencia de una sola persona | Alto | Un lenguaje, runbooks, revisión cruzada | Tech Lead |
| R12 | Latencia entre emisión y archivo de evidencia bloquea surtido | Medio | Métrica y alerta; reintentos prioritarios | Tech Lead |

---

## 24. Decisiones y validaciones pendientes

Se corrige la numeración de v1.1, que omitía los puntos 1 y 5 aunque el texto los citaba como prioritarios.

| # | Pendiente | Bloquea | Prioridad |
|---|---|---|---|
| P-01 | Criterio **escrito** de COFEPRIS sobre validez de receta electrónica por fracción, libros de control electrónicos, y requisito de especialidad | Fases 2 y 3 | Bloqueante |
| P-02 | Dictamen sobre ley vigente de datos personales, transferencia internacional y conservación | Fases 1, 4, DR | Bloqueante |
| P-03 | Uso permitido del OCSP/CRL del SAT sin convenio | ADR-001 | Bloqueante |
| P-04 | RTO/RPO por módulo | Diseño de DR | Alta |
| P-05 | ¿Firma la farmacia con e.firma? ¿Quién y con qué frecuencia? | ADR-002 | Alta |
| P-06 | Cancelación de receta: quién, cuándo, con qué firma | ADR-002 | Alta |
| P-07 | Confirmar que farmacia no opera offline | ADR-002 | Alta |
| P-08 | Retención legal exacta por tipo de registro | Object Lock Compliance | Alta |
| P-09 | Matriz de servicios en mx-central-1 y clasificación de datos | Fase 1 | Alta |
| P-10 | Costeo (PSC NOM-151, infraestructura) | Presupuesto | Alta |
| P-11 | Origen y gobernanza del catálogo de medicamentos y clasificación por fracción | Fase 1 | Media |
| P-12 | Contenido del QR y datos del portal de verificación pública (confirmado: **1 QR por receta**, ya modelado en `prescriptions.qr_uuid` §11.1; falta definir qué datos expone el portal) | ADR-008 | Media |
| P-13 | Libro de Control Digital: modelo, firma, auditoría | ADR-007 | Media |
| P-14 | Unificar Terraform vs. CDK | Fase 1 | Media |

---

## 25. Matriz de trazabilidad normativa (inicio)

Cada feature debe mapear a un requisito explícito. Esta tabla se completa en el Diseño Detallado.

| Capacidad | Requisito de referencia | Evidencia |
|---|---|---|
| Firma de receta | Validez y no repudio (marco de firma electrónica) | Documento firmado, certificado, sello NOM-151 |
| Conservación | NOM-024 / NOM-151 | Object Lock, matriz de retención |
| Bitácora | NOM-024 | Cadena + checkpoints verificados |
| Reglas por fracción | Art. 226 LGS | `medication_policies` versionadas |
| Protección de datos | Ley de datos personales vigente | Cifrado, aviso, trazabilidad |
| Libro de Control | Reglamento aplicable (por confirmar con P-01) | ADR-007 |

*(La interpretación jurídica de cada obligación corresponde al dictamen especializado.)*

---

## 26. Índice de ADRs

| ADR | Título | Estado | Depende de |
|---|---|---|---|
| ADR-001 | Firma electrónica: formato, manejo de `.key`, revocación, sello | Por redactar (1.º) | P-03 |
| ADR-002 | Máquina de estados y dispensación anti-doble-surtido | Por redactar (2.º) | ADR-001, P-05/06/07 |
| ADR-003 | Identidad y validación sin convenios gubernamentales | Por redactar (3.º) | — |
| ADR-004 | Motor de reglas por fracción | Pendiente | P-01 |
| ADR-005 | Bitácora verificable y anclaje externo | Pendiente | ADR-001 |
| ADR-006 | Residencia de datos y DR | Pendiente | P-02, P-09 |
| ADR-007 | Libro de Control Digital | Pendiente | P-01, P-13 |
| ADR-008 | QR y verificación pública | Pendiente | P-12 |

---

## 26.1 Agente estable: Compliance Analyst

Se incorpora el rol **Compliance Analyst** (`.cursor/rules/compliance-analyst.mdc`), activo en
todas las fases y etapas, no asignado "por fase" como los demás roles de `current-phase.mdc`.
Es dueño de la matriz de trazabilidad normativa (§25) y del mapeo norma → requisito → feature.
No sustituye al dictamen jurídico externo (P-02) ni a la consulta formal a COFEPRIS (P-01); los
prepara, documenta y les da seguimiento. En Fase 0 trabaja en paralelo, fuera del prototipo
visual, sin bloquear al UX Designer.

## 27. Cierre

La v1.2 conserva el núcleo correcto de v1.1 (una sola autoridad transaccional, evidencia inmutable,
firma en cliente, reglas versionadas, monolito modular) y corrige los errores que comprometían el
no repudio, la verificabilidad de la bitácora y la integridad del surtido. Reduce la pila a lo que un
equipo básico puede operar con OPEX controlado. Lo que sigue sin cerrar no es técnico: es el criterio
regulatorio por escrito (P-01), el dictamen sobre datos personales (P-02) y la revocación de e.firma (P-03).
