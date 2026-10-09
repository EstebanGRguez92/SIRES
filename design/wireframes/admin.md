# Portal de administración

Usuario: admin o auditor, en oficina, sin fila de pacientes.
Meta: ver qué pasó, quién lo hizo, y llevarse un reporte. No emite ni surte.

La bitácora se consulta. No se edita. Si algo no cuadra, el sistema avisa. No promete "bloquear una modificación" de un registro que ya no debe poder cambiarse: el mensaje es de detección.

---

## A-00 Acceso

Título: "Administración". Entrada simulada, distinta de la e.firma del médico: usuario y contraseña de ejemplo, más la leyenda "Acceso de oficina. Simulado en esta maqueta."

La química debe confirmar si el auditor entra con e.firma o con usuario de oficina. La maqueta usa usuario de oficina para no mezclarlo con la firma clínica.

---

## A-01 Tablero

Objetivo: orientar. Cuatro números y un salto a la bitácora. Sin gráficas decorativas.

```
┌──────────────────────────────────────────────────────────┐
│ SIRES · Administración                         Salir     │
├──────────────────────────────────────────────────────────┤
│ Hoy (datos de ejemplo)                                   │
│                                                          │
│  12 emitidas     9 surtidas     2 rechazadas     0 avisos│
│                                                          │
│  [ Ver bitácora ]     [ Exportar ]                       │
│                                                          │
│  Avisos                                                  │
│  Ninguno. Si un registro no cuadra, aparecerá aquí.      │
│                                                          │
│  Fracciones en esta maqueta                              │
│  IV, V, VI · Operación real                              │
│  I, II, III · Solo maqueta                               │
└──────────────────────────────────────────────────────────┘
```

Estados:

- Vacío: ceros y la frase "Aún no hay movimientos de ejemplo."
- Loading: números en gris.
- Error: "No pudimos cargar el resumen."
- Éxito: cifras de ejemplo, calculadas con los recorridos de la demo.

"2 rechazadas" abre la bitácora ya filtrada a intentos de surtido que no procedieron.

---

## A-02 Bitácora

Objetivo: una lista cronológica que se puede filtrar sin ser experto.

```
┌──────────────────────────────────────────────────────────┐
│ Bitácora                                                 │
│ Estos registros no se editan.                            │
├──────────────────────────────────────────────────────────┤
│ Desde [ hoy ] Hasta [ hoy ]                              │
│ Tipo [ Todos ▾ ]   Receta [ ________ ]   [ Filtrar ]     │
│                                                          │
│ 10:41  Receta emitida      SR-2041   Dra. Ana López      │
│ 10:44  Surtido registrado  SR-2041   Farmacia Centro     │
│ 10:44  Intento rechazado   SR-2041   Otra sucursal       │
│        Motivo: ya no hay saldo                           │
│                                                          │
│ [ Cargar más ]                                           │
└──────────────────────────────────────────────────────────┘
```

Tipos que la maqueta muestra, en palabras:

- Receta creada como borrador
- Receta emitida
- Evidencia lista
- Surtido registrado
- Intento de surtido rechazado
- Receta cancelada
- Aviso de integridad

Estados:

- Vacío: "No hay registros con ese filtro."
- Loading: filas en gris.
- Error: "No pudimos cargar la bitácora. Reintenta."
- Éxito: filas como arriba. Cada fila abre A-03.

Decisión: el motivo del rechazo se lee en la lista. El auditor no abre cada fila para saber que fue saldo agotado.

---

## A-03 Detalle de un evento

Objetivo: evidencia legible. Los datos técnicos van debajo, con su nombre en claro.

```
┌──────────────────────────────────────────────────────────┐
│ Surtido registrado                                       │
│ 9 oct 2026 · 10:44                                       │
├──────────────────────────────────────────────────────────┤
│ Receta     SR-2041                                       │
│ Qué pasó   Se surtió 1 caja de amoxicilina 500 mg        │
│ Quién      Q.F.B. ejemplo · Farmacia Centro              │
│ Estado de  Pasó de Emitida a Surtida total               │
│ la receta                                                │
│ Nota       En este surtido la farmacia no se quedó       │
│            con la receta.                                │
│            Ilustrativo, por confirmar con la química.    │
│                                                          │
│ Comprobación                                             │
│ Huella del registro     a1b2… (ejemplo)                  │
│ Huella del anterior     9f03… (ejemplo)                  │
│ Encadenado al anterior  sí, en la maqueta                │
│                                                          │
│ [ Volver a la bitácora ]                                 │
└──────────────────────────────────────────────────────────┘
```

No hay botón Editar, Borrar ni Corregir.

"Encadenado al anterior" en la maqueta significa: el ejemplo muestra que cada registro apunta al anterior. La verificación matemática real no existe en Fase 0; el texto lo dice: "Verificación de ejemplo. No es la bitácora real."

---

## A-04 Aviso de integridad

Objetivo: mostrar qué ve el auditor si un registro no cuadra con el anterior. Es una alerta, no una palanca para "bloquear" el pasado.

```
┌──────────────────────────────────────────────────────────┐
│ Aviso de integridad                                      │
│                                                          │
│ Un registro de ejemplo no coincide con el anterior.      │
│                                                          │
│ Qué hacer: revisar el evento y exportar el aviso.       │
│ El sistema no reescribe el registro.                     │
│                                                          │
│ Evento marcado: Surtido SR-2041 · 10:44                  │
│                                                          │
│ [ Ver evento ]    [ Exportar aviso ]                     │
└──────────────────────────────────────────────────────────┘
```

Este aviso se dispara en la maqueta con un control de demo etiquetado "Simular registro que no cuadra", visible solo en un pie de página de la maqueta: "Control de demostración. No existe en el sistema real."

Así la química ve la alerta sin que parezca una función de oficina para alterar datos.

---

## A-05 Exportar

Objetivo: llevarse un archivo para una revisión, sin armar un reporteador.

```
┌──────────────────────────────────────────────────────────┐
│ Exportar                                                 │
│                                                          │
│ Qué        ( ) Bitácora del periodo                      │
│            ( ) Surtidos (libro de control de ejemplo)    │
│            ( ) Recetas emitidas                          │
│                                                          │
│ Desde      [ 1 oct 2026 ]                                │
│ Hasta      [ 9 oct 2026 ]                                │
│                                                          │
│ Formato    ( ) Hoja de cálculo   ( ) PDF                 │
│                                                          │
│ El archivo usa los mismos filtros que la bitácora.       │
│ Lleva la leyenda: datos de ejemplo, Fase 0.              │
│                                                          │
│ [ Generar archivo ]                                      │
└──────────────────────────────────────────────────────────┘
```

Estados:

- Vacío: nada seleccionado. Generar apagado.
- Loading: "Preparando el archivo…"
- Error: "No se pudo generar. Reintenta." No descarga un archivo a medias.
- Éxito: "Archivo listo" y descarga de ejemplo. El hecho de exportar queda como fila en la bitácora ("Se exportó la bitácora").

---

## A-06 Reglas en pantalla (solo lectura)

Objetivo: que la química vea, juntas, las seis fracciones y la leyenda. No es un editor de normas.

```
┌──────────────────────────────────────────────────────────┐
│ Reglas que muestra la maqueta                            │
│ Solo lectura. Nada de esto está cerrado.                  │
├──────────────────────────────────────────────────────────┤
│ Fracción I   Solo maqueta                                │
│              Pide folio cargado por el médico.           │
│              Topes: por confirmar.                       │
│                                                          │
│ Fracción II  Solo maqueta                                │
│              Ejemplo: hasta 2 cajas.                     │
│                                                          │
│ Fracción III Solo maqueta                                │
│              Topes: por confirmar.                       │
│                                                          │
│ Fracción IV  Operación real                              │
│              Ejemplo: vigencia 6 meses, hasta 3 surtidos,│
│              sin retención en los dos primeros.          │
│                                                          │
│ Fracción V   Operación real                              │
│              Topes: por confirmar.                       │
│                                                          │
│ Fracción VI  Operación real                              │
│              Topes: por confirmar.                       │
│                                                          │
│ En cada fila:                                            │
│ Ilustrativo, por confirmar con la química.               │
└──────────────────────────────────────────────────────────┘
```

Sin botones de guardar. Si la química corrige un número, se anota en el registro de feedback; no se "configura" en esta fase.

---

## A-07 Revisión de altas (secundaria)

Lista corta: médicos y farmacias de ejemplo en "Pendiente de revisión", "Aprobada" o "Rechazada".

Aprobar o rechazar pide motivo. No consulta padrones en línea. Texto fijo: "Revisión manual. Sin convenio con SAT, SEP ni COFEPRIS."

Sirve para que la química vea el alto de la puerta. No entra en el video de 3 minutos.

---

## Decisiones de este portal

| Decisión | Por qué |
|---|---|
| Tablero de cuatro números | El auditor llega a orientar, no a explorar un tablero. |
| Bitácora sin edición | La confianza está en que no haya lápiz. |
| Huella visible, llamada "huella" | "Hash" no le dice nada a la química. El detalle técnico puede vivir en la segunda lectura. |
| Alerta de integridad, no "bloquear cambio" | El sistema detecta que algo no cuadra y lo muestra. No ofrece una herramienta para intervenir el historial. |
| Exportar con tres preguntas | Periodo, tipo y formato cubren la demo. Un diseñador de reportes sería alcance de más. |
| Reglas en solo lectura, con la leyenda en cada fracción | Es el lugar donde la química puede tachar los números ilustrativos de un vistazo. |
| Control "simular registro que no cuadra" marcado como demo | Sin eso, la alerta nunca se ve. Con eso, no parece una función real de alteración. |

## Qué preguntarle a la química

1. ¿El auditor entra con usuario de oficina o también con e.firma?
2. ¿La lista de la bitácora trae los datos que ella pediría en una visita?
3. ¿El libro de control exportado debe verse como el libro que ya llena en papel?
4. ¿Quién más, además de ella, consultaría este portal en un piloto?
