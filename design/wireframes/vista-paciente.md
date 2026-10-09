# Vista de la receta para el paciente

El paciente no tiene portal ni contraseña. Esta vista es la hoja que el médico muestra o imprime, y la página mínima si alguien apunta la cámara al QR fuera de la farmacia.

Qué datos salen en esa página pública sigue abierto (decisión pendiente del proyecto). La maqueta enseña poco a propósito.

---

## P-01 Hoja que se muestra o se imprime

Se abre desde "Mostrar al paciente" en la receta emitida. Sin menú del médico. Letra grande.

```
┌──────────────────────────────────────────────────────────┐
│ Receta electrónica                                       │
│                                                          │
│  ┌──────────┐                                            │
│  │          │                                            │
│  │   QR     │   Un código para toda la receta            │
│  │          │                                            │
│  └──────────┘                                            │
│                                                          │
│  Presenta este código en la farmacia.                    │
│                                                          │
│  Paciente    Luis Hernández García                       │
│  Médico      Dra. Ana López                              │
│  Fecha       9 oct 2026                                  │
│                                                          │
│  1. Amoxicilina 500 mg cápsulas                          │
│     1 caja                                               │
│     1 cápsula cada 8 horas por 7 días                    │
│     Fracción IV                                          │
│                                                          │
│  Vigencia de ejemplo: 6 meses.                           │
│  Ilustrativo, por confirmar con la química.              │
│                                                          │
│  [ Imprimir ]                                            │
└──────────────────────────────────────────────────────────┘
```

El QR se dibuja una vez. Si hay dos medicamentos, se listan abajo del mismo código.

Estados:

- Esperando evidencia: la hoja ya muestra el QR y añade "La farmacia podría pedirte esperar unos segundos."
- Lista para surtir: esa frase desaparece.
- Cancelada o surtida total: el médico no usa esta hoja para un surtido nuevo. Si la abre para consulta, un sello grande dice "Cancelada" o "Ya surtida".

---

## P-02 Si el paciente escanea el QR con su teléfono

No es el surtido. Es una página pública de ejemplo, sin sesión.

```
┌──────────────────────────────────────────────────────────┐
│ SIRES                                                    │
│                                                          │
│ Esta receta está emitida.                                │
│ Preséntala en la farmacia para el surtido.               │
│                                                          │
│ Código de ejemplo SR-2041                                │
│ Vigente en la maqueta                                    │
│                                                          │
│ La farmacia verá el detalle al escanearla en su portal.  │
└──────────────────────────────────────────────────────────┘
```

A propósito no se listan medicamento, dosis ni nombre del paciente en esta página. Qué sí debe verse en público se confirma con la química y con la decisión de verificación pública. Hasta entonces, la maqueta prefiere de menos.

Si la receta no se puede surtir, la página dice solo: "Esta receta no está disponible para surtir." Sin explicar si fue por saldo, cancelación o fecha, para no dar detalle de más a quien tenga el código.

---

## Decisiones

| Decisión | Por qué |
|---|---|
| Sin cuenta de paciente | El paciente no es usuario del sistema. |
| Un QR en la hoja, medicamentos en lista | El código es de la receta completa. |
| Página pública con poco texto | Todavía no está cerrado qué puede ver cualquiera que tenga el código. |
| Sello grande si ya no sirve | Evita que alguien haga fila con una hoja cancelada sin enterarse. |

## Qué preguntarle a la química

1. ¿La hoja impresa lleva dosis e indicación, o solo el código y el nombre del medicamento?
2. ¿Qué puede leer una persona cualquiera que escanee el QR en la calle?
3. ¿Hace falta el nombre de la farmacia, o la receta se surte en cualquiera del piloto?
