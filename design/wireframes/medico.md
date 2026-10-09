# Portal del médico

Usuario: médico con e.firma, en consultorio, entre un paciente y el siguiente.
Meta de la maqueta: emitir una receta de fracción IV, V o VI en pocos pasos, y ver con claridad por qué una fracción I, II o III se bloquea.
Presión: menos de un minuto en el camino feliz. El paciente puede estar delante.

Acceso simulado. La llave privada no se "envía": en la maqueta el médico elige un archivo falso en su propia pantalla y escribe una contraseña de teatro. Nada de eso sale de la pantalla.

---

## M-00 Acceso

Objetivo: entrar sabiendo que está firmando con e.firma, aunque en esta fase sea simulado.

```
┌──────────────────────────────────────────────────────────┐
│ SIRES                                                    │
│ Receta electrónica                                       │
│                                                          │
│  Portal del médico                                       │
│                                                          │
│  Esta entrada es una simulación.                         │
│  La llave no sale de esta pantalla.                      │
│                                                          │
│  Certificado (.cer)   [ Elegir archivo ficticio ]        │
│  Llave privada (.key) [ Elegir archivo ficticio ]        │
│  Contraseña de la llave [ ************ ]                 │
│                                                          │
│  [ Entrar ]                                              │
│                                                          │
│  ¿Farmacia?  ¿Administración?                            │
└──────────────────────────────────────────────────────────┘
```

Estados:

- Vacío: los tres campos en blanco. El botón Entrar no responde hasta que hay archivo, llave y contraseña (en la maqueta, cualquier archivo ficticio basta).
- Loading: "Comprobando el certificado…" con el botón deshabilitado.
- Error: "No pudimos leer ese certificado. Revisa que sean los archivos de tu e.firma." Sin códigos.
- Éxito: pasa a M-01. Aviso breve: "Entrada simulada. No es una firma del SAT."

Decisión: un solo paso de acceso, sin registro largo en el camino de la demo. El alta del médico (M-08) existe, pero no bloquea la demo.

---

## M-01 Inicio

Objetivo: ver las recetas de hoy y empezar otra en un toque.

```
┌──────────────────────────────────────────────────────────┐
│ SIRES · Médico          Dra. Ana López (ficticia)  Salir │
├──────────────────────────────────────────────────────────┤
│ [ + Nueva receta ]                                       │
│                                                          │
│ Hoy                                                      │
│ ┌────────────────────────────────────────────────────┐   │
│ │ Luis Hernández · Emitida · Fracción IV             │   │
│ │ Operación real · Amoxicilina                       │   │
│ │ QR listo                                           │   │
│ └────────────────────────────────────────────────────┘   │
│ ┌────────────────────────────────────────────────────┐   │
│ │ Borrador sin paciente · sin firmar                 │   │
│ │ [ Seguir ]  [ Descartar ]                          │   │
│ └────────────────────────────────────────────────────┘   │
│                                                          │
│ [ Ver todas ]                                            │
└──────────────────────────────────────────────────────────┘
```

Estados:

- Vacío: "Todavía no hay recetas hoy." Un solo botón: Nueva receta.
- Loading: tres tarjetas en gris.
- Error: "No pudimos cargar tus recetas. Reintenta." Las recetas ya abiertas no se pierden.
- Éxito: lista como arriba.

Descartar un borrador pide confirmación: "Este borrador no tiene validez. ¿Lo eliminas?" Una receta ya emitida no se borra desde aquí.

---

## M-02 Paciente de esta receta

Objetivo: identificar a la persona de la receta con los datos mínimos. No es un expediente.

```
┌──────────────────────────────────────────────────────────┐
│ Nueva receta · 1 de 3 · Paciente              [ Cancelar ]│
├──────────────────────────────────────────────────────────┤
│ CURP     [ HEGLxxxxxxHDFRRR00 ]   (ficticio)             │
│ Nombre   Luis Hernández García                           │
│          Se llenó al reconocer la CURP de ejemplo.       │
│                                                          │
│ [ Seguir a medicamentos ]                                │
└──────────────────────────────────────────────────────────┘
```

Estados:

- Vacío: solo el campo CURP.
- Loading: "Buscando…"
- Error: "Esa CURP no tiene el formato esperado." No se inventa un paciente.
- Éxito: nombre de ejemplo y botón Seguir.

Si la CURP de ejemplo no existe en la maqueta, un enlace secundario: "Capturar paciente ficticio" (nombre y CURP). Sigue siendo dato de la receta, no una ficha clínica.

Decisión: un campo primero. Edad, dirección y teléfono no aparecen hasta que la química diga que la receta los necesita.

---

## M-03 Medicamentos

Objetivo: armar los renglones, ver la fracción al momento y corregir antes de firmar.

```
┌──────────────────────────────────────────────────────────┐
│ Nueva receta · 2 de 3 · Medicamentos                     │
│ Luis Hernández · ficticio                                │
├──────────────────────────────────────────────────────────┤
│ Buscar medicamento  [ amoxicilina____________ ]          │
│                                                          │
│ Resultados                                               │
│ · Amoxicilina 500 mg cápsulas     Fracción IV            │
│   Operación real                                         │
│ · Clonazepam 2 mg tabletas        Fracción II            │
│   Solo maqueta                                           │
│                                                          │
│ Renglón 1                                                │
│ Amoxicilina 500 mg · Fracción IV · Operación real        │
│ Cantidad   [ 1 ] cajas                                   │
│ Indicación [ 1 cápsula cada 8 horas por 7 días ]         │
│                                                          │
│ ┌────────────────────────────────────────────────────┐   │
│ │ Ilustrativo, por confirmar con la química.         │   │
│ │ Ejemplo de fracción IV: vigencia 6 meses,          │   │
│ │ hasta 3 surtidos.                                  │   │
│ └────────────────────────────────────────────────────┘   │
│                                                          │
│ [ + Otro medicamento ]                                   │
│                                                          │
│ [ Revisar receta ]                                       │
└──────────────────────────────────────────────────────────┘
```

Una receta puede mezclar renglones. Si se mezclan fracciones distintas, la maqueta lo permite y lo marca: "Esta receta junta fracciones distintas. Confirmar con la química si eso debe permitirse." No se decide aquí.

El QR todavía no existe. Se genera al emitir, uno para toda la receta.

### Bloqueo en la misma pantalla (no es otra página)

Ejemplo de demo, fracción II, solo maqueta:

```
┌──────────────────────────────────────────────────────────┐
│ Clonazepam 2 mg · Fracción II · Solo maqueta             │
│ Cantidad   [ 3 ] cajas                                   │
│                                                          │
│ ┌────────────────────────────────────────────────────┐   │
│ │ No se puede continuar con 3 cajas.                 │   │
│ │                                                        │
│ │ Ilustrativo, por confirmar con la química.         │   │
│ │ En esta maqueta, la fracción II acepta hasta       │   │
│ │ 2 cajas por receta.                                │   │
│ │                                                        │
│ │ [ Dejar en 2 cajas ]                               │   │
│ └────────────────────────────────────────────────────┘   │
│                                                          │
│ [ Revisar receta ]   ← apagado mientras el renglón falle │
└──────────────────────────────────────────────────────────┘
```

El médico entiende qué falló, qué número usa la maqueta y cómo seguir. No ve "error" ni un código.

Otros bloqueos, mismo patrón:

| Situación | Mensaje |
|---|---|
| Fracción I sin folio cargado | "Falta el folio de COFEPRIS. El sistema no lo asigna. Cárgalo para continuar." Enlace a M-07. Chip: Solo maqueta. |
| Fracción III, V o VI sin tope definido | "Aún no hay tope para esta fracción. Ilustrativo, por confirmar con la química." El botón Revisar sigue apagado si la cantidad está vacía. |
| Cantidad 0 o vacía | "Indica cuántas cajas." |

Estados de M-03:

- Vacío: buscador y ningún renglón. Revisar apagado.
- Loading de búsqueda: "Buscando en el catálogo…"
- Error de catálogo: "No encontramos ese nombre. Prueba con el genérico."
- Éxito: renglón aceptado, leyenda ilustrativa visible, Revisar activo.

---

## M-04 Revisión (lo que se va a firmar)

Objetivo: el médico lee exactamente lo que firmará. Puede volver atrás. Firmar es el único paso irreversible de la emisión.

```
┌──────────────────────────────────────────────────────────┐
│ Nueva receta · 3 de 3 · Revisión                         │
│ Esto es lo que vas a firmar. Aún no es una receta.       │
├──────────────────────────────────────────────────────────┤
│ Médico     Dra. Ana López · cédula ficticia              │
│ Paciente   Luis Hernández García                         │
│                                                          │
│ 1. Amoxicilina 500 mg cápsulas                           │
│    Fracción IV · Operación real                          │
│    1 caja · 1 cápsula cada 8 horas por 7 días            │
│                                                          │
│ Vigencia de ejemplo: 6 meses desde la firma.             │
│ Ilustrativo, por confirmar con la química.               │
│                                                          │
│ Un solo código QR se creará para toda la receta.         │
│                                                          │
│ [ Corregir ]              [ Firmar con e.firma ]         │
└──────────────────────────────────────────────────────────┘
```

Estados:

- Vacío: no aplica; no se llega sin renglón válido.
- Loading: no aplica en esta pantalla quieta.
- Error: si la regla cambió al volver, se regresa a M-03 con el bloqueo visible. No se firma un texto distinto al revisado.
- Éxito: el botón Firmar abre M-05.

Decisión: Corregir vuelve a medicamentos sin borrar lo capturado.

---

## M-05 Firma simulada

Objetivo: que se sienta como un acto de firma, no como "guardar".

```
┌──────────────────────────────────────────────────────────┐
│ Firmar con e.firma                                       │
│ Simulación. La llave no sale de esta pantalla.           │
│                                                          │
│ Vas a firmar la receta de Luis Hernández,                │
│ 1 medicamento, fracción IV.                              │
│                                                          │
│ Contraseña de la llave  [ ************ ]                 │
│                                                          │
│ [ Cancelar ]     [ Confirmar firma ]                    │
└──────────────────────────────────────────────────────────┘
```

Estados:

- Vacío: contraseña en blanco. Confirmar apagado.
- Loading: "Firmando en este equipo…" El botón no se pulsa dos veces.
- Error: "La contraseña no coincide con la simulación." La receta sigue en borrador.
- Éxito: pasa a M-06. Estado de la receta: Emitida.

Si la persona cierra esta ventana, no hay receta emitida.

---

## M-06 Receta emitida y QR único

Objetivo: entregar un código, imprimirlo o mostrarlo. Dejar claro cuándo ya se puede surtir.

```
┌──────────────────────────────────────────────────────────┐
│ Receta emitida                                           │
│ Estado: Emitida · todavía no surtida                     │
├──────────────────────────────────────────────────────────┤
│  ┌──────────┐                                            │
│  │          │   Un código para toda la receta            │
│  │   QR     │   No hay un código por medicamento.        │
│  │          │                                            │
│  └──────────┘   Folio de ejemplo: SR-2041               │
│                                                          │
│ Luis Hernández · Amoxicilina 500 mg · Fracción IV        │
│ Operación real                                           │
│                                                          │
│ La farmacia podrá surtirla en unos segundos,             │
│ cuando quede guardada la evidencia.                      │
│ [ Esperando evidencia… ]                                 │
│                                                          │
│ [ Mostrar al paciente ]  [ Imprimir ]  [ Nueva receta ]  │
└──────────────────────────────────────────────────────────┘
```

"Mostrar al paciente" abre la vista de paciente (una hoja, sin menú del médico).

Estados:

- Loading: QR visible y la línea "Esperando evidencia…".
- Error: "La receta quedó emitida, pero la evidencia no se guardó. No la surtas todavía. Reintenta el guardado." No se ofrece un segundo QR.
- Éxito: la línea cambia a "Lista para surtir." El QR no cambia.

Decisión: el QR aparece una vez, arriba, nunca repetido junto a cada renglón.

---

## M-07 Folio de fracción I (solo maqueta)

Objetivo: mostrar que el folio lo trae el médico y el sistema no lo fabrica. No es operación real.

```
┌──────────────────────────────────────────────────────────┐
│ Folio COFEPRIS · Fracción I · Solo maqueta               │
│ Se muestra para validar el bloqueo.                      │
│ No se construye en la prueba de concepto.                │
├──────────────────────────────────────────────────────────┤
│ Esta receta no puede emitirse sin folio.                 │
│ SIRES no asigna folios.                                  │
│                                                          │
│ Número de folio   [ ______________ ]                     │
│ Comprobante       [ Adjuntar archivo de ejemplo ]        │
│                                                          │
│ Ilustrativo, por confirmar con la química.               │
│ El formato del folio y la evidencia se confirman         │
│ con el criterio de COFEPRIS.                             │
│                                                          │
│ [ Volver a la receta ]                                   │
└──────────────────────────────────────────────────────────┘
```

Estados:

- Vacío: sin folio. La receta de fracción I no llega a revisión.
- Loading: "Guardando el comprobante de ejemplo…"
- Error: "Falta el número o el comprobante."
- Éxito: regreso a M-03 con la nota "Folio de ejemplo cargado." Sigue siendo solo maqueta.

---

## M-08 Alta del médico (fuera del camino de la demo)

Objetivo: que la química vea que sin e.firma no hay acceso, y que la cédula se declara, no se consulta en línea.

```
┌──────────────────────────────────────────────────────────┐
│ Alta de médico · simulación                              │
│                                                          │
│ Nombre                                                      │
│ Cédula profesional (declarada)                              │
│ Especialidad (opcional; no oculta fracciones)               │
│ Certificado .cer y llave .key ficticios                     │
│                                                          │
│ [ Enviar a revisión ]                                    │
│                                                          │
│ Queda como "Pendiente de revisión".                      │
│ Nadie entra a emitir hasta que un admin lo apruebe.       │
└──────────────────────────────────────────────────────────┘
```

Nota visible: "No se consulta al SAT ni a la SEP en línea. No hay convenio."

---

## M-09 Detalle de una receta ya emitida

Objetivo: consultar, no reeditar. Cancelar solo con confirmación y dejando el motivo a la vista.

```
┌──────────────────────────────────────────────────────────┐
│ SR-2041 · Emitida · Lista para surtir                    │
│ QR único (el mismo de la emisión)                        │
│                                                          │
│ Renglones, saldo por surtir, surtidos ya hechos.         │
│ Si hubo surtido parcial: "Parcialmente surtida".         │
│ Si se acabó el saldo: "Surtida total".                   │
│                                                          │
│ [ Cancelar receta ]  solo si aún no está surtida total   │
│ Al pulsarlo: "Vas a anular esta receta. No se puede      │
│ deshacer." Motivo obligatorio. Simulación de firma.      │
└──────────────────────────────────────────────────────────┘
```

Una receta surtida total no ofrece Cancelar. Una cancelada se lee completa y no se reabre.

---

## Decisiones de este portal

| Decisión | Por qué |
|---|---|
| Tres pasos: paciente, medicamentos, revisión | El médico está en consulta. Cada paso de más es fricción. |
| El bloqueo vive en el renglón, con la acción para corregirlo | Leer "no se puede" sin salida obliga a adivinar. |
| Leyenda ilustrativa en el mismo bloque que el número | Si el número va solo, la química puede tomarlo por norma. |
| Chip Operación real / Solo maqueta en cada fracción | IV, V y VI no deben confundirse con el teatro de I, II y III. |
| Un QR al emitir, no al agregar medicamentos | El código identifica la receta firmada, no el borrador. |
| "Esperando evidencia" separado del estado Emitida | La receta ya existe, pero la farmacia puede tener que esperar unos segundos. |
| Alta y folio I fuera del camino feliz | La demo de tres minutos es una fracción de operación real. |

## Qué preguntarle a la química

1. ¿CURP y nombre alcanzan para la receta, o falta algo que ella exige ver?
2. ¿Una receta puede mezclar fracciones?
3. ¿El ejemplo de 2 cajas en fracción II y el de 6 meses / 3 surtidos en fracción IV se parecen a lo que ella espera discutir, o los cambiamos antes de Figma?
4. ¿La indicación de cada medicamento es texto libre?
5. ¿Cancelar una receta ya emitida y no surtida es un acto que el médico debe poder hacer?
