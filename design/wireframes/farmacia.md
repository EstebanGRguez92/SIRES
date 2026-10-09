# Portal de farmacia

Usuario: farmacéutico en mostrador, con gente esperando.
Meta: leer un QR, entender si se puede surtir, confirmar, y salir. El segundo intento sobre la misma receta debe explicarse en una frase.

La farmacia no crea recetas. No ve el portal del médico.

---

## F-00 Acceso

Misma simulación que el médico, con el título "Portal de farmacia" y el nombre de la sucursal ficticia (Farmacia Centro, sucursal ejemplo).

Error propio: "Esta e.firma no está ligada a una farmacia de ejemplo."

---

## F-01 Mostrador

Objetivo: una acción grande. Escanear o pegar. Nada más en la primera vista.

```
┌──────────────────────────────────────────────────────────┐
│ SIRES · Farmacia Centro (ejemplo)              Salir     │
├──────────────────────────────────────────────────────────┤
│                                                          │
│              [ Escanear código de la receta ]            │
│                                                          │
│              o pegar el código  [ ____________ ]         │
│              [ Buscar ]                                  │
│                                                          │
│  El código es uno por receta.                            │
│  Si trae varios medicamentos, aparecen juntos.           │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

Estados:

- Vacío: como arriba. Es el estado normal entre pacientes.
- Loading: "Leyendo la receta…" a pantalla completa. El botón no se pulsa otra vez.
- Error de lectura: "Ese código no corresponde a una receta. Pide a la persona que lo muestre de nuevo."
- Éxito: abre F-02 con la receta.

Debajo, en segundo plano y con menos peso visual: "Surtidos de hoy" (lista corta). No compite con el botón de escanear.

---

## F-02 Receta leída

Objetivo: decidir en segundos si se surte, cuánto queda y si esta farmacia se queda con el papel o la constancia.

```
┌──────────────────────────────────────────────────────────┐
│ Receta SR-2041                                           │
│ Estado: Emitida · Lista para surtir                      │
├──────────────────────────────────────────────────────────┤
│ Paciente   Luis Hernández García (ficticio)              │
│ Médico     Dra. Ana López                                │
│ Vigencia   hasta el 9 abr 2027                           │
│            Ilustrativo, por confirmar con la química.    │
│                                                          │
│ Renglones                                                │
│ 1. Amoxicilina 500 mg                                    │
│    Fracción IV · Operación real                          │
│    Recetado 1 caja · Ya surtido 0 · Queda 1              │
│    Surtir ahora [ 1 ]                                    │
│                                                          │
│ Este surtido, ejemplo de fracción IV:                    │
│ la farmacia no se queda con la receta.                   │
│ Ilustrativo, por confirmar con la química.               │
│                                                          │
│ [ No surtir ]              [ Confirmar surtido ]         │
└──────────────────────────────────────────────────────────┘
```

Si la fracción de ejemplo pide retención en ese número de surtido, la frase cambia, sin convertirla en estado de la receta:

> En este surtido la farmacia se queda con la receta.
> Ilustrativo, por confirmar con la química.

El farmacéutico no puede quitar esa nota si la regla de ejemplo la exige. No es una casilla opcional.

Surtido parcial: "Surtir ahora" acepta una cantidad menor al saldo. Al confirmar, el estado que verá después es "Parcialmente surtida", y el saldo queda a la vista.

Estados de lectura que no abren el botón Confirmar (pantalla F-04):

- Evidencia aún no lista.
- Caducada por fecha.
- Surtida total.
- Cancelada.
- Código de otra receta inexistente.

---

## F-03 Confirmación

Objetivo: el surtido no ocurre con un toque accidental.

```
┌──────────────────────────────────────────────────────────┐
│ ¿Registrar este surtido?                                 │
│                                                          │
│ SR-2041 · Amoxicilina 500 mg · 1 caja                    │
│ Farmacia Centro · ahora                                  │
│                                                          │
│ No se puede deshacer.                                    │
│                                                          │
│ [ Volver ]                 [ Sí, surtir ]                │
└──────────────────────────────────────────────────────────┘
```

Estados:

- Loading: "Registrando el surtido…" Sí, surtir queda apagado.
- Error de carrera: pasa al mensaje de F-04 "Ya se surtió", aunque esta pantalla haya estado abierta. La otra farmacia ganó.
- Éxito: F-05.

---

## F-04 No se puede surtir

Una sola pantalla, mensaje distinto según el caso. Siempre dice qué pasó y qué hacer. Nunca un código como título.

```
┌──────────────────────────────────────────────────────────┐
│ No se puede surtir                                       │
│                                                          │
│ [ mensaje de la tabla ]                                  │
│                                                          │
│ [ Escanear otra receta ]                                 │
└──────────────────────────────────────────────────────────┘
```

| Caso | Mensaje |
|---|---|
| Ya surtida por completo | "Esta receta ya se surtió por completo. No queda saldo." |
| Otro mostrador se adelantó | "Otra farmacia acaba de surtir esta receta. No queda saldo." |
| Fecha vencida | "Esta receta ya no está vigente. La fecha límite ya pasó." |
| Emitida, evidencia en proceso | "La receta existe, pero todavía no se puede surtir. Espera unos segundos y vuelve a escanear." Botón: Reintentar. |
| Cancelada | "El médico canceló esta receta. No se surte." |
| Parcial, y piden más del saldo | "Solo quedan N cajas de este medicamento." El campo vuelve a F-02 con el máximo marcado. |
| Fracción I, II o III | Se muestra el renglón con chip Solo maqueta y el aviso: "En esta maqueta puedes recorrer el flujo. Estas fracciones no se surten de verdad en la prueba de concepto." |

La nota para el equipo (no va en la pantalla del mostrador): el rechazo de doble surtido corresponde al conflicto de la operación. El farmacéutico no necesita ver "409".

---

## F-05 Surtido registrado

```
┌──────────────────────────────────────────────────────────┐
│ Surtido registrado                                       │
│                                                          │
│ SR-2041                                                  │
│ Estado de la receta: Surtida total                       │
│ o, si quedó saldo: Parcialmente surtida                  │
│                                                          │
│ Amoxicilina 500 mg · 1 caja · hoy                        │
│ Nota del surtido: la farmacia no se queda con la receta. │
│ Ilustrativo, por confirmar con la química.               │
│                                                          │
│ Este registro queda en el libro de control de la         │
│ sucursal. No se edita.                                   │
│                                                          │
│ [ Escanear otra ]                                        │
└──────────────────────────────────────────────────────────┘
```

Estados:

- Éxito: el de arriba. Es el único estado de esta pantalla.
- Si alguien vuelve a escanear el mismo QR y ya no hay saldo: F-04, no una segunda copia de este comprobante.

Decisión: el libro de control no se captura a mano. Esta pantalla es el renglón que quedará en el libro. La química debe decir si el texto le sirve como comprobante de mostrador.

---

## F-06 Libro de hoy (secundario)

Lista de surtidos de la sucursal en el día: hora, receta, medicamento, cantidad, si el surtido retuvo la receta (sí/no) y quién surtió.

Vacío: "Hoy todavía no hay surtidos."
Sin botón de borrar ni de editar.

---

## Decisiones de este portal

| Decisión | Por qué |
|---|---|
| Un botón dominante: escanear | En mostrador no hay tiempo para menús. |
| Pegar el código como alternativa | La demo y el mostrador sin cámara tienen que funcionar. |
| Saldo por renglón, un solo QR | El control es por medicamento; el código es de la receta. |
| Retención escrita como nota del surtido | Si se muestra como estado "Retenida", se contradice la regla del producto y confunde a la química. |
| Confirmación de una frase antes de surtir | El surtido no se deshace. |
| Rechazo en lenguaje de mostrador | "Ya se surtió" se entiende con la fila enfrente. "409" no. |
| Reintentar solo cuando la evidencia va tarde | Es el único rechazo que puede resolverse esperando. |

## Qué preguntarle a la química

1. ¿En mostrador alcanza con escanear, o también buscan por nombre del paciente?
2. ¿El surtido parcial (llevarse una caja y dejar saldo) ocurre en su práctica?
3. ¿La frase "la farmacia se queda con la receta" es la que ella usaría?
4. ¿El comprobante de F-05 debe imprimirse siempre?
5. ¿Quién de la sucursal puede surtir: cualquiera con e.firma de esa farmacia, o solo el responsable sanitario?
