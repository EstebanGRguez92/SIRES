# Wireframes de baja fidelidad — convenciones (Fase 0)

Fecha: 9 de octubre de 2026.
Audiencia: química farmacéutica, Product Owner, Frontend Lead.
Estado: propuesta para validar. Sin Figma. Sin código.

Estos bocetos describen pantallas en texto. Sirven para que la química diga qué sobra, qué falta y qué mensaje no entiende, antes de dibujar en Figma.

## Qué se maqueta

Tres portales: médico, farmacia y admin. Más una vista de receta para el paciente (no es un cuarto portal: el paciente no entra al sistema).

Un solo código QR por receta. Si la receta tiene varios medicamentos, el QR sigue siendo uno y cubre todos los renglones.

Las seis fracciones del Artículo 226 se ven en pantalla.

| Fracción | En esta maqueta | En el MVP (operación real) |
|---|---|---|
| IV, V y VI | Flujo completo de emitir y surtir | Sí. Son las candidatas a operación real |
| I, II y III | Flujo y bloqueos, para que la química los vea | No. No se construyen en el POC ni entran a operación hasta el criterio de COFEPRIS |

## Cómo se marcan los números

Cualquier tope, vigencia, número de surtidos o regla de retención que aparezca en pantalla lleva esta leyenda, visible, no en letra pequeña escondida:

> Ilustrativo, por confirmar con la química.

Esos números no son ley cerrada. Son ejemplos para discutir. La química puede tacharlos en la sesión.

Ejemplos que usa la maqueta, todos ilustrativos:

| Fracción | Para qué sirve el ejemplo | Número que se muestra |
|---|---|---|
| II | Bloqueo al pasar de un tope (demo con clonazepam ficticio) | Hasta 2 cajas por receta |
| IV | Vigencia y surtidos repetidos | Vigencia 6 meses; hasta 3 surtidos; sin retención en los dos primeros |
| I | El médico carga un folio; el sistema no lo inventa | Sin tope inventado |
| III, V y VI | Que existan en el catálogo y muestren su ficha | Topes en blanco: "por confirmar" |

## Cómo se habla en pantalla

El médico y el farmacéutico ven palabras de mostrador, no códigos.

| Lo que ve el usuario | Lo que no se muestra como mensaje principal |
|---|---|
| Borrador | `BORRADOR` |
| Emitida | `EMITIDA` |
| Parcialmente surtida | `PARCIALMENTE_SURTIDA` |
| Surtida total | `SURTIDA_TOTAL` |
| Cancelada | `CANCELADA` |
| Esta receta ya se surtió | HTTP 409 |
| Esta receta ya no está vigente | "caducada" como si fuera un estado de la receta |
| En este surtido la farmacia se queda con la receta | "Retenida" como estado de la receta |

La caducidad se calcula con la fecha. No es un estado.
Quedarse con la receta en farmacia es una nota del surtido, no un estado de la receta.

Chips de fracción, siempre juntos al nombre:

- **Operación real** — fracciones IV, V y VI.
- **Solo maqueta** — fracciones I, II y III. Texto de apoyo: "Se muestra para validar el bloqueo. No se construye en la prueba de concepto."

## Datos

Solo personas, cédulas, farmacias y folios ficticios. Medicamentos con nombre real de ejemplo (amoxicilina, clonazepam) están permitidos como catálogo de referencia; los pacientes no.

## Camino que debe entender la química en 3 minutos

1. Médico emite una receta de fracción IV y ve un solo QR.
2. Médico intenta tres cajas de un ejemplo de fracción II y el sistema se lo impide, con la leyenda ilustrativa.
3. Farmacia lee ese QR, surte, y un segundo intento se rechaza con una frase clara.
4. Admin abre la bitácora, ve el evento y exporta.

## Fuera de estas pantallas

Código, Figma, colores finales, AWS, firma real, base de datos y bitácora real.
La especialidad del médico no esconde el catálogo de fracciones II y III: ese ocultamiento es un supuesto por confirmar, no una regla de esta maqueta.
