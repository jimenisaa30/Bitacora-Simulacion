# Autoevaluación: Unidad 7 · Tipografía jugable

**Proyecto:** Play-Doh, un sistema tipográfico jugable
**Organización:** Play-Doh (Hasbro)
**Atributo de identidad:** la plastilina como material que se moldea, se descubre y toma forma con las manos
**Contexto de uso:** portada web o pantalla interactiva en un evento
**Tecnología:** Three.js, un solo archivo `index.html`, funciona en tiempo real en el navegador

---

## Resumen del sistema

El jugador no recibe instrucciones ni interfaz. Descubre qué hacer actuando sobre las letras y la plastilina.

| Escena | Qué ve | Qué puede hacer | Respuesta del sistema |
|---|---|---|---|
| 1. Palabra | "dough" negro sobre fondo blanco | Clic sobre las letras | Las letras **u, g** caen; **d, o, h** se resisten (vibran) y quedan formando "doh" |
| 2. Bolitas | Tres bolitas llegan rodando, todo en blanco y negro | Clic en la pantalla | Cada clic devuelve parte del color; al **5.º clic** se revela el rojo, blanco y azul y el fondo amarillo |
| 3. Molde | Cae un molde con la silueta de nube del logo y "Play-Doh" invertido | Arrastrar bolitas y molde; tocar el molde | Las bolitas chocan y se deforman; el molde se levanta al arrastrarlo; un toque lo estampa |
| 4. Resultado | Las bolitas están dentro del molde y lo cubren lo suficiente | Tocar la insignia | Se forma el logo (borde azul, base roja, letras blancas) con un rebote gelatinoso |

### Modelo y reglas

- **Estados:** `letters → balls → reveal → mold → play → press → won`.
- **Letras:** cada letra es un objeto 3D independiente. Se marcan como "se queda" (`KEEP = [0, 1, 4]`) o "se elimina".
- **Color:** el lienzo parte en escala de grises. Cada clic sube la saturación en 1/5 (`CLICKS_TO_REVEAL = 5`).
- **Bolitas:** son círculos en 2D con radio distinto. Tienen fricción, rebote en los bordes y colisión entre ellas con un solape permitido, porque la plastilina es blanda. Un resorte de aplastamiento da el efecto de masa blanda y la rotación depende de la distancia recorrida (rueda sin deslizar).
- **Molde:** se mueve arrastrándolo. Un toque sin arrastrar lo estampa (distancia de movimiento menor a 0,4).
- **Condición de éxito:** el centro de las tres bolitas debe estar dentro de la silueta y deben cubrir al menos `COVERAGE_NEEDED = 28%` del área interior del molde.
- **Si falla:** las bolitas se aplastan y el jugador puede reacomodar e intentar de nuevo.

---

## Evaluación por criterio

### 1. Cumplimiento del encargo: **25 / 25**

El sistema usa las letras del nombre de la organización como elemento gráfico central. Las letras de "dough" se eliminan hasta formar "doh", y el molde es la palabra "Play-Doh" en 3D. El jugador interviene sobre las letras con sus acciones, y todo corre en tiempo real en la web. Tampoco hay puntuación, niveles ni condición de victoria explícita, como pide el encargo.

Evidencias:
- El juego completo está en un único `index.html` con Three.js y la fuente incrustados, sin dependencia de internet.
- Se probó el flujo completo (las cuatro escenas, el caso de fallo y el caso de éxito) en un navegador automatizado, y no hubo errores en consola.
- Los colores y la silueta del molde se basan en el logo original (rojo, blanco, azul y fondo amarillo).

### 2. Comprensión y verificación: **10 / 25**

Puedo describir el modelo, sus siete estados y sus reglas (ver tabla y lista de arriba), y sé qué parámetro controla cada comportamiento. Sin embargo, no tengo todavía evidencia suficiente de haber **predicho y verificado** de forma sistemática qué pasa al modificar una regla. Por eso me doy un puntaje bajo en este criterio.

Lo que sí hice:
- Identifiqué los parámetros que controlan la experiencia: `CLICKS_TO_REVEAL`, `COVERAGE_NEEDED`, `KEEP`, radios de las bolitas y colores.
- Comprobé que un cambio de tamaño de las bolitas (de 1,9 / 1,6 / 1,3 a 1,55 / 1,3 / 1,05) hacía viable acomodarlas dentro del molde, y que bajar la cobertura de 35 % a 28 % permitía ganar con ese tamaño.

Lo que me falta para mejorar este criterio:
- Registrar en la bitácora una tabla de pruebas del tipo *"cambié X → esperaba Y → observé Z"* para cada parámetro.
- Probar casos límite, como arrastrar el molde hasta el borde de la pantalla o reacomodar muchas veces seguidas.

### 3. Interacción e identidad: **20 / 25**

Las acciones están pensadas para que el jugador descubra la idea de la marca: la plastilina se moldea. Las letras se quitan para formar "doh" (la masa), el color aparece con el gesto de tocar, y el molde le da forma a la plastilina como en la vida real. Hay descubrimiento por ensayo y error: nada se explica, y el sistema responde con vibración, rebote, aplastamiento o color.

Evidencias:
- Las letras que no se pueden quitar vibran, lo que le indica al jugador que "no son para quitar".
- El molde se levanta al ser arrastrado y baja al estamparse, con una sacudida de cámara.
- Las bolitas se aplastan cuando el molde cae sobre ellas sin éxito.

Por qué no es 25:
- Aún **no contrasté** lo observado con pruebas de otras personas. La presentación grupal exige invitar a alguien a jugar antes de explicar, y ese resultado todavía no está registrado.
- No se sabe aún si el jugador descubre por sí solo que debe tocar el molde para estamparlo, ni si entiende por qué debe dar 5 clics.

### 4. Resolución tipográfica y experiencia de juego: **25 / 25**

La composición y el ritmo sostienen la intención: el blanco y negro inicial pasa a un amarillo saturado cuando se revela el color, la palabra tiene peso visual, y cada escena tiene su momento de lectura (la pausa antes de que lleguen las bolitas, el momento en que cae el molde).

Evidencias:
- La palabra "Play-Doh" del molde se compone letra por letra con tamaños, giros y desniveles distintos, como en el logo original, y se invierte para actuar como sello.
- Las respuestas perceptibles son claras: rodar, aplastar, vibrar, rebotar, sacudir la cámara, y migajas de plastilina al ganar.
- La interfaz es inexistente a propósito.

Limitación reconocida:
- La fuente es Helvetiker Bold (la que incluye Three.js), no la fuente redondeada original de Play-Doh. Se acerca por la forma de las letras y el biselado, pero no es idéntica.

---

## Puntaje total y nota

| # | Criterio | Puntaje | Máximo |
|---|---|---|---|
| 1 | Cumplimiento del encargo | 25 | 25 |
| 2 | Comprensión y verificación | 10 | 25 |
| 3 | Interacción e identidad | 20 | 25 |
| 4 | Resolución tipográfica y experiencia de juego | 25 | 25 |
| | **Total** | **80** | **100** |

**Conversión a nota (escala de 1 a 5):**

```
nota = (puntaje total / 100) × 5
nota = (80 / 100) × 5 = 4.0
```

### **Nota de autoevaluación: 4.0 / 5.0**

---

## Plan de mejora antes de la presentación

1. **Criterio 2:** documentar pruebas "cambio de regla → predicción → observación" en la bitácora.
2. **Criterio 3:** dejar que al menos dos compañeros jueguen sin explicación, y anotar qué intentaron, qué descubrieron y dónde se quedaron atascados.
3. Con esos resultados, ajustar el juego (por ejemplo, una pista visual si nadie descubre que el molde se toca) y recalcular esta autoevaluación.

## Referentes

- Ji Lee, [*Word as Image*](https://pleaseenjoy.com/word-as-image/): visualizar el significado de una palabra usando solo sus letras.
- [Unidad 7 · Actividad 03](https://juanferfranco.github.io/simulacion-2026-20/units/unit7/#actividad-03-presentaci%C3%B3n-grupal): criterios de autoevaluación, de 0 a 25 puntos cada uno.
