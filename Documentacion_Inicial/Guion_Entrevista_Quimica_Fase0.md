# Guion de entrevista — Química farmacéutica (Fase 0)

**Sesión:** Entrevista de descubrimiento (antes de wireframes)  
**Stakeholder:** Química farmacéutica — origen del proyecto, experta de dominio, validadora  
**Duración:** 75 minutos (máximo 90)  
**Formato:** Videollamada o presencial  
**Fecha:** [YYYY-MM-DD]  
**Facilitador:** Stakeholder Manager  
**Observadores:** Product Owner + UX Designer (escuchan; no debaten diseño ni arquitectura)  
**Opcional:** CTO solo si hay que aclarar un límite de alcance; no conduce la sesión

Este guion **no** es la demo del prototipo. Esa sesión viene después (días 9-10 de Fase 0).  
Hoy el objetivo es entender el trabajo real, los dolores y las reglas que ella usa en la práctica, para que UX no diseñe a ciegas.

---

## 1. Objetivo de la sesión

Salir con evidencia suficiente para:

1. Mapear el flujo actual de receta en papel (emisión → entrega → surtido → archivo).
2. Documentar dolores de médico, farmacia, paciente y auditoría.
3. Confirmar cómo opera cada fracción **en la práctica**, sin tratar su respuesta como dictamen legal.
4. Alinear expectativas: qué SIRES sí cubre, qué no, y en qué orden.
5. Identificar médicos, farmacéuticos y farmacias que puedan validar la maqueta después.

**Criterio de éxito:** el equipo puede dibujar los 3 portales sin inventar pasos, y hay un registro escrito de lo que ella confirmó, corrigió o dejó abierto.

---

## 2. Brief interno (10 min antes, sin ella)

Leer en voz alta. No improvisar estos puntos.

### Lo que sí vamos a decir

| Tema | Cómo decirlo |
|---|---|
| Alcance | "El sistema resuelve el ciclo completo de la receta. El expediente clínico es un proyecto distinto." |
| e.firma | "Es requisito legal. Sin e.firma no hay receta válida. Acompañaremos al médico en el proceso." |
| Convenios | "Hoy el sistema funciona con declaración del médico y evidencia documental. Cuando se obtengan convenios, se podrá activar validación en línea sin rediseñar todo." |
| Orden de fracciones | "En la maqueta vamos a ver las seis fracciones. En el primer sistema real empezamos por las menos restringidas (IV, V y VI), para probar el flujo completo. Las controladas se diseñan, pero no se operan hasta tener criterio de autoridad." |
| Esta fase | "Hoy no hay sistema todavía. Primero entendemos su operación. En unos días le mostramos pantallas para que las critique." |
| Tiempos | "No vamos a comprometer fechas hoy. Primero validamos la maqueta con usted." |

### Lo que no vamos a hacer

- Prometer expediente clínico, CIE-11, hospitales, IMSS/ISSSTE o integración con el sistema de una cadena.
- Prometer consulta en línea a COFEPRIS, SAT o SEP.
- Tratar un número de cajas, vigencia o surtidos como "ya es ley" si ella lo dice de memoria: se anota como **práctica / hipótesis**, no como regla a programar.
- Debatir arquitectura, nube, costos o código.
- Corregirla frente al grupo. Si hay contradicción con la arquitectura, se anota y se escala después.

### Hipótesis que esta entrevista debe confirmar o romper

| ID | Hipótesis del equipo | Si se confirma | Si se rompe |
|---|---|---|---|
| H1 | El médico emite bajo presión de consulta y necesita un flujo corto | UX prioriza rapidez | Hay un paso de revisión que no podemos omitir |
| H2 | La farmacia vive el fraude de recetas duplicadas / fotocopiadas | El rechazo de "ya surtida" es central en la maqueta | El dolor real es otro (ilegibilidad, folios, libros) |
| H3 | Un QR por receta (no por medicamento) es suficiente para el mostrador | Se mantiene la decisión ya cerrada | Hay que documentar por qué pediría un QR por renglón y escalar a CTO |
| H4 | Surtido parcial es cotidiano (no todo se entrega de una vez) | La maqueta debe mostrar saldo por renglón | El flujo se simplifica a todo-o-nada |
| H5 | Fracciones IV, V y VI son el mejor primer alcance operativo | Se refuerza el MVP | Ella exige I/II/III para que "valga la pena" → expectativa a alinear, no a ceder |
| H6 | La farmacia no puede surtir sin conexión a internet | Se confirma P-07 | Hay que documentar el escenario offline y escalar |
| H7 | Cancelar una receta ya emitida es un caso real | Se incluye en maqueta | Se deja fuera del prototipo de Fase 0 |
| H8 | Ella puede abrir puertas con médicos y farmacias | Se agenda validación 2 | El piloto queda como riesgo de adopción |

---

## 3. Materiales

- Este guion (impreso o en pantalla del facilitador).
- Plantilla de captura (sección 10), abierta en otra ventana por el Product Owner.
- Reloj visible. El UX Designer toma notas de flujos y frases textuales.
- **No** mostrar Figma todavía. Si pide "cómo se vería", responder: "eso es la siguiente sesión; hoy queremos su operación real".
- Datos: **ningún paciente, médico ni farmacia reales**. Si menciona casos, pedir que los anonimice.

---

## 4. Roles en la sala

| Rol | Habla | Hace |
|---|---|---|
| Stakeholder Manager | Sí — conduce | Abre, cierra, traduce, corta desvíos, gestiona expectativas |
| Product Owner | Solo si el facilitador le pasa la palabra | Captura decisiones, impacto y acciones |
| UX Designer | Solo para pedir un detalle de flujo | Dibuja el flujo actual; anota fricciones |
| CTO | Solo si ella pregunta "¿se puede hacer X?" | Responde alcance sí/no, sin diseño técnico |

Regla: una voz a la vez. Si el equipo discute entre sí, el facilitador corta.

---

## 5. Guion minuto a minuto

### Bloque 0 — Apertura (5 min)

**Decir:**

> Gracias por el tiempo. Esta reunión es para entender cómo funciona hoy la receta en la práctica, desde que el médico la escribe hasta que la farmacia la surte y queda el registro.  
> No vamos a mostrarle un sistema todavía. En la siguiente sesión sí: pantallas que usted pueda tocar y criticar.  
> Vamos a grabar / tomar notas para no perder nada. No usaremos nombres de pacientes ni datos reales.  
> Si algo es confidencial o no quiere que quede por escrito, lo decimos y lo marcamos.  
> ¿Le parece bien que empecemos?

Pedir confirmación explícita de notas/grabación.

**Cerrar alcance en 30 segundos:**

> SIRES cubre la receta electrónica: emitir, firmar, llevar a farmacia, surtir una sola vez lo que corresponde, y dejar rastro auditable. No es expediente clínico. No sustituye a COFEPRIS. Hoy no hay convenio con autoridad: el médico declara y anexa evidencia. Eso lo diremos también a los médicos, sin ambigüedad.

Si ella empuja expediente, hospitales o "que valide COFEPRIS en línea":

> Lo anoto porque es importante. Para este primer sistema nos quedamos en el ciclo de la receta. Si mezclamos expediente ahora, retrasamos lo que usted ya identificó como el dolor urgente. Lo revisamos con el responsable de producto después de esta sesión, no lo prometemos aquí.

---

### Bloque 1 — Contexto de ella (8 min)

Objetivo: saber desde dónde habla (mostrador, regulación, red de contactos).

1. ¿Cuál es su rol hoy y con qué tipo de farmacia o consultorio trabaja más?
2. En una semana típica, ¿cuánto de su tiempo toca recetas, libros de control o auditoría?
3. ¿El problema que le hizo proponer este sistema cuál es, en una frase?
4. Si SIRES existiera mañana, ¿quién ganaría primero: el médico, la farmacia, el paciente o el auditor? ¿Por qué?
5. ¿A quién más tendríamos que convencer para que esto se use de verdad?

**Escuchar:** si su frase de dolor no coincide con "doble surtido / papel / auditoría", el Product Owner reescribe el problema de negocio.

---

### Bloque 2 — Flujo actual del médico (12 min)

Objetivo: pasos reales de emisión. UX necesita la secuencia, no la norma.

Pedir que narre **un caso reciente** (anonimizado).

6. Cuando un médico receta hoy, ¿qué hace paso a paso? (recetario, receta libre, sistema del consultorio, WhatsApp, foto)
7. ¿Cuánto tarda, más o menos, desde que decide el medicamento hasta que el paciente sale con la receta?
8. ¿Qué datos no pueden faltar para que la farmacia acepte la receta?
9. ¿El paciente se lleva el original, una copia, o ambas cosas?
10. ¿Qué errores ve más seguido en recetas de médicos? (letra, dosis, cédula, fecha, fracción equivocada)
11. ¿Los médicos con los que trabaja ya tienen e.firma? ¿La usan? ¿Qué tanto les cuesta?
12. Si un médico no tiene e.firma, ¿qué pasa hoy? ¿Y qué debería pasar en SIRES, en su opinión?

**Frase de alineación, si sale "que entre sin e.firma":**

> Entiendo la fricción. En SIRES, sin e.firma no hay receta válida: es el sello que da valor legal. Lo que sí podemos diseñar es acompañar al médico para que el trámite no se sienta como un muro. ¿Qué parte de ese trámite le parece más pesada?

**Para UX, insistir si no quedó claro:**

- ¿El paciente está presente mientras receta?
- ¿Usa computadora, teléfono o papel?
- ¿Receta varios medicamentos en la misma hoja?

---

### Bloque 3 — Flujo actual de la farmacia (12 min)

Objetivo: el mostrador con cola.

13. Un paciente llega con receta. ¿Qué hace el personal, minuto a minuto?
14. ¿Quién valida que la receta sea auténtica? ¿Cómo se dan cuenta de una copia o un reuso?
15. ¿Qué los hace rechazar una receta? Déme los 3 motivos más frecuentes.
16. ¿Surtido parcial: es común que falte un medicamento o una caja y el paciente regrese después?
17. Cuando hay surtido parcial, ¿qué le queda al paciente y qué se queda la farmacia?
18. ¿En algún caso la farmacia **retiene** la receta y no se la devuelve? ¿En cuáles fracciones, según su práctica?
19. Si se va el internet o el sistema de la farmacia, ¿siguen surtiendo recetas controladas o se detienen?
20. ¿La farmacia firma algo al surtir (libro, sello, e.firma, firma manuscrita)?

**Seguimiento obligatorio a P-05 / P-07 (en lenguaje de negocio):**

- "Si el sistema exigiera estar en línea para surtir, ¿eso es aceptable o rompería el turno?"
- "¿Quién, en la farmacia, debería quedar registrado como responsable del surtido: el farmacéutico de mostrador, el responsable sanitario, o ambos?"

No pedir e.firma de farmacia como requisito. Preguntar la práctica y anotar.

---

### Bloque 4 — Fracciones en la práctica (15 min)

Objetivo: entender diferencias operativas. **No cerrar reglas de motor.**

Decir antes de preguntar:

> Voy a preguntar por las seis fracciones. Lo que nos cuente lo tomamos como su experiencia de operación, no como texto legal cerrado. Legal lo confirma después un dictamen y, cuando corresponda, COFEPRIS. Si algo es "así se hace" vs "así dice el reglamento", ayúdenos a distinguirlo.

Recorrer I → VI. Para cada una, las mismas 6 preguntas. Si el tiempo aprieta, priorizar **IV, V, VI** (candidatas a operación real) y luego **I, II, III** (se maquetan bloqueos).

| # | Pregunta por fracción |
|---|---|
| A | ¿Qué medicamentos le vienen a la cabeza como ejemplo cotidiano? |
| B | ¿La receta se queda en la farmacia o regresa con el paciente? |
| C | ¿Cuántas veces se puede surtir y en cuánto tiempo, según ustedes operan? |
| D | ¿Hay tope de cantidad / cajas que ustedes aplican aunque el médico ponga más? |
| E | ¿Pide folio especial, recetario especial, o especialidad del médico? |
| F | ¿Qué pasa si alguien intenta surtirla otra vez en otra farmacia? |

**Cierre del bloque (no negociable):**

21. Si tuviéramos que enseñar el sistema primero con un grupo de medicamentos, ¿con cuáles empezaría usted y por qué?
22. Si empezamos por antibióticos y similares (menos controlados) y dejamos estupefacientes y psicotrópicos para después, ¿eso le parece responsable o le parece que "no sirve"?

Si responde que sin I/II/III no tiene valor:

> Tiene sentido que el valor grande esté en lo controlado. Igual, si empezamos por ahí sin el criterio de autoridad, el riesgo es construir algo que luego haya que desarmar. La maqueta sí va a mostrar esas fracciones y los bloqueos, para que usted los vea. Lo que no haremos es operarlas de verdad hasta tener eso por escrito. ¿Le parece un camino aceptable?

Anotar la respuesta literal. Eso es un riesgo de relación, no una orden de alcance.

**Especialidad (P-01, en lenguaje claro):**

23. Para psicotrópicos, ¿hoy exigen que el médico sea de cierta especialidad, o basta la cédula profesional?

---

### Bloque 5 — Paciente, receta y QR (6 min)

24. El paciente, ¿qué necesita entender de la receta para que no lo rechacen en la farmacia?
25. ¿Está acostumbrado a mostrar algo en el celular, o sigue siendo papel sí o sí?
26. Un solo código para toda la receta (aunque traiga varios medicamentos): ¿le funciona en el mostrador o complicaría el surtido parcial?
27. ¿Qué datos de la receta **no** deberían verse si alguien solo escanea el código? (por ejemplo en un portal público de verificación)

Confirmar, sin abrir debate técnico, la decisión ya cerrada: **un QR por receta**. Si ella pide uno por medicamento, anotar el motivo operativo y escalar a CTO + Product Owner. No comprometer el cambio en la sesión.

---

### Bloque 6 — Auditoría, libros y autoridad (8 min)

28. Cuando llega una auditoría o una revisión, ¿qué les piden ver primero?
29. El libro de control: ¿lo llevan en papel, Excel, sistema de la cadena? ¿Quién lo firma y cada cuánto?
30. ¿Qué les ha fallado de ese libro en la práctica (atraso, tachaduras, extravío, diferencia vs existencias)?
31. Si el sistema armara el libro solo a partir de lo surtido, ¿qué tendría que poder exportar o imprimir para que a usted le sirva?
32. ¿Tiene contacto o experiencia previa con COFEPRIS / autoridad estatal en este tema? ¿Algo que debamos saber antes de una consulta formal?

No pedir que ella "consiga el visto bueno de COFEPRIS". Si ofrece el contacto:

> Eso puede ser muy valioso. No vamos a hablar de un sistema ya listo ni a pedir validación en línea. Si hay una consulta, será formal y por escrito, con lo que sí y lo que no hacemos. Lo coordinamos; no improvisamos un mensaje.

---

### Bloque 7 — Expectativas, alcance y red (7 min)

33. En la maqueta que va a ver, ¿qué tres cosas tienen que estar sí o sí para que usted diga "van por buen camino"?
34. ¿Qué sería un fracaso, aunque las pantallas se vean bonitas?
35. ¿Cuánto tiempo puede dedicarnos en estas dos semanas? (proponer: 1 sesión de descubrimiento hoy, 1 de maqueta de ~45 min, 1 de ajustes si hace falta)
36. ¿Conoce 1 o 2 médicos y 1 o 2 farmacéuticos que puedan ver la maqueta después, sin compromiso?
37. ¿Hay alguna farmacia que podría ser piloto más adelante, solo para hablar, sin firmar nada ahora?

**Chequeo de expectativas (preguntar aunque ya se hayan tocado):**

38. ¿Esperaba ver un sistema funcionando en esta reunión, o le parece bien empezar por pantallas?
39. ¿Hay algo que usted asume que SIRES va a hacer y no hemos mencionado?

Si menciona fechas, inversión o "para el mes que entra":

> Lo anoto. Fechas las cierra el responsable técnico y de producto después de que usted apruebe la maqueta. Hoy no le voy a dar una fecha que después no podamos cumplir.

---

### Bloque 8 — Cierre (5 min)

**Resumir en voz alta (máximo 5 puntos).** Ejemplo de estructura:

> Si entendí bien: el dolor principal es [X]; el médico hoy hace [Y]; la farmacia se traba en [Z]; el surtido parcial [sí/no] es cotidiano; y para usted el primer valor visible es [W]. ¿Corrijo algo?

Pedir:

40. ¿Hay algo que no preguntamos y deberíamos haber preguntado?
41. ¿Puede enviarnos (sin datos personales) una receta de ejemplo tachada / un recetario en blanco / una foto de libro de control, si le resulta cómodo?

**Compromisos a ofrecer, y solo estos:**

- Minuta de esta sesión en 24 horas.
- Siguiente sesión: revisión de maqueta (wireframes), fecha tentativa [YYYY-MM-DD].
- No vamos a escribir sistema real ni usar datos reales en esta fase.

**Agradecer y cerrar.** No "ya casi lo tenemos". Cerrar con: "la siguiente vez usted critica pantallas; nosotros no avanzamos a construir hasta que usted las apruebe".

---

## 6. Señales de desalineación (cortar con tacto)

| Lo que dice | Riesgo | Respuesta |
|---|---|---|
| "También debería llevar el expediente / diagnóstico CIE" | Scope creep | Alcance recetas. Expediente es otro proyecto. |
| "Que consulte cédula en línea / folio COFEPRIS en línea" | Promesa imposible hoy | Declarativo + evidencia; convenio después, sin rediseño. |
| "Si no trae estupefacientes desde el día uno, no me sirve" | Choque con MVP | Maqueta de I–III sí; operación real después de criterio escrito. |
| "Que el médico entre con usuario y contraseña, la e.firma es tediosa" | Rompe no repudio | e.firma obligatoria; diseñamos el acompañamiento. |
| "Que la farmacia surta aunque no haya internet" | P-07 | Lo anotamos como riesgo operativo; no lo prometemos. |
| "¿Para cuándo está listo?" | Expectativa de fecha | Después de aprobar maqueta; no hay fecha hoy. |
| "Yo les paso datos reales de pacientes para que se vea auténtico" | LFPDPPP / datos sensibles | No. Solo ficticios. |
| "Háblenle directo a mi contacto en COFEPRIS esta semana" | Canal informal | Consulta formal, coordinada; no mensaje improvisado. |

---

## 7. Preguntas que NO se hacen en esta sesión

- Detalle de nube, costos AWS, base de datos, hashes, Object Lock.
- "¿Firmamos un ADR?"
- Pedirle que dicte la política numérica final de cada fracción como si fuera ley.
- Pedirle que autorice Fase 1 (eso es después de ver la maqueta).
- Negociar presupuesto o equity.

---

## 8. Cómo tomar notas (Product Owner)

Copiar frases textuales entre comillas. No traducir a jerga en el momento.

Por cada hallazgo, clasificar al vuelo:

| Código | Significado |
|---|---|
| DOLOR | Fricción actual |
| FLUJO | Paso del proceso |
| REGLA-P | Práctica operativa (hipótesis, no ley) |
| REGLA-N | Ella la presenta como norma; igual va a Compliance, no se programa aún |
| FUERA | Fuera de alcance; se registra, no se acepta |
| RED | Contacto o puerta que abre |
| RIESGO | Expectativa o relación en peligro |

Impacto (para la tabla de feedback): Alto / Medio / Bajo según si cambia la maqueta de esta semana.

---

## 9. Después de la sesión (mismo día)

1. Stakeholder Manager envía minuta en el formato de la sección 10, máximo 24 h.
2. Product Owner extrae: dolores priorizados, alcance confirmado, ítems fuera de alcance.
3. UX Designer dibuja el flujo actual (as-is) y la lista de pantallas que faltan.
4. Compliance recibe por separado las REGLA-N; **no entra al diseño visual**.
5. Si hubo presión de alcance o de fechas: escalar a CTO + Product Owner al día siguiente, no "ya lo vimos en la junta".

Entregable de esta entrevista: este registro lleno + flujo as-is.  
No es go/no-go a Fase 1. El go/no-go es después de la maqueta.

---

## 10. Plantilla de minuta (llenar en la sesión)

```
Sesión: Entrevista de descubrimiento Fase 0 — Química farmacéutica
Fecha: [YYYY-MM-DD]
Stakeholder: [Nombre], química farmacéutica — origen del proyecto
Participantes internos: [SM], [PO], [UX], [CTO si aplicó]
Grabación / notas: [sí/no, dónde]

Temas tratados:
1. Contexto y dolor origen
2. Flujo actual médico
3. Flujo actual farmacia
4. Fracciones en la práctica (I–VI)
5. Paciente / QR
6. Auditoría y libros
7. Expectativas, tiempo y red de validación

Feedback recibido:
| Tema | Feedback | Impacto | Acción |
|---|---|---|---|
| | | Alto/Medio/Bajo | |

Decisiones tomadas:
- (solo lo que ella y el equipo acordaron en la sala; no reglas de motor)

Preguntas abiertas:
- 

Compromisos:
| Compromiso | Responsable | Fecha |
|---|---|---|
| Minuta enviada | Stakeholder Manager | +24 h |
| Wireframes 3 portales | UX Designer | [fecha] |
| Sesión de maqueta | Stakeholder Manager | [fecha] |

Riesgos detectados:
- 

Validación de hipótesis:
| ID | Resultado | Nota |
|---|---|---|
| H1 | Confirmada / Rota / Parcial | |
| H2 | | |
| H3 | | |
| H4 | | |
| H5 | | |
| H6 | | |
| H7 | | |
| H8 | | |

Próxima sesión: revisión de maqueta — [fecha]
```

---

## 11. Invitación sugerida (enviar 3-5 días antes)

Asunto: SIRES — conversación sobre cómo recetan y surten hoy (75 min)

> [Nombre]:  
> El [fecha] queremos una conversación de 75 minutos sobre cómo se emite y se surte una receta hoy, con sus dolores reales. No le vamos a mostrar un sistema todavía; eso va en la siguiente reunión, con pantallas para que las critique.  
>  
> No necesitamos datos de pacientes. Si puede tener a la mano (opcionales): un recetario en blanco, un ejemplo de receta anonimizada, o cómo ven el libro de control.  
>  
> Participamos: [nombres]. Cualquier cosa que no quiera que quede por escrito, lo marcamos.  
>  
> Confirme si [fecha y hora] le funciona.

---

## 12. Relación con la siguiente sesión (demo de maqueta)

Esta entrevista alimenta wireframes. La demo de maqueta usa otro guion (flujos: emitir, bloquear por fracción, firmar, QR, surtir, rechazar duplicado, bitácora).

No mezclar las dos. Si en esta entrevista pide "enséñenme pantallas", anotar el deseo y mantener el foco en el flujo actual.
