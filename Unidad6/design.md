# Autoevaluación: presentación del performance

**Curso:** Simulación 2026-20 · **Unidad 6:** Agentes autónomos · **Actividad 04**
**Instrumento:** *Colonia de murciélagos*, una bandada con flocking dentro de un icosaedro wireframe sobre un fondo de flow field.

> Los campos marcados con `TODO` son datos que solo tú tienes (pieza musical, enlaces, capturas). Complétalos antes de la sesión 4.

---

## 1. Resumen de puntajes

| # | Criterio | Puntaje | Máximo |
|---|----------|:-------:|:------:|
| 1 | Cumplimiento del encargo | **25** | 25 |
| 2 | Comprensión y verificación | **18** | 25 |
| 3 | Diseño e intención | **25** | 25 |
| 4 | Interpretación humana | **25** | 25 |
| | **Total** | **93** | **100** |

### Conversión a nota (escala de 0 a 5)

```text
nota = total / 100 × 5
nota = 93 / 100 × 5 = 4.65
```

**Nota de la autoevaluación: 4.65 / 5.0** (93 %)

---

## 2. Descripción del instrumento

| Elemento | Qué es | Algoritmo |
|----------|--------|-----------|
| Murciélagos (50 por defecto, de 5 a 150) | Agentes que vuelan dentro del icosaedro | Steering behaviors y flocking (separación, alineación, cohesión) |
| Icosaedro de 20 caras | Límite del entorno. Solo se ve el wireframe y rota de forma constante | Las 20 caras funcionan como paredes reales que empujan a los agentes |
| Fondo rojo y morado | Partículas que dejan estelas y consultan un campo de ángulos | Flow field con ruido de valor |

Tecnología: HTML, JavaScript, Three.js (r128) para la escena 3D y Canvas 2D para el flow field. Enlace al instrumento: `TODO`

---

## 3. Criterios

### 3.1 Cumplimiento del encargo: 25 / 25

**Por qué:**

- Usa tecnología web: una sola página HTML que corre en el navegador.
- Funciona en tiempo real: un único bucle con `requestAnimationFrame` actualiza la bandada, el campo y el render en cada cuadro.
- Respeta la restricción de algoritmos: solo hay steering behaviors, flocking y flow fields. No hay análisis de audio ni secuencias automáticas.
- Ocupa toda la ventana y permite ocultar la interfaz (tecla `H`) para presentar sin elementos en pantalla.
- Pieza musical elegida: `TODO`

**Evidencias:** enlace al instrumento `TODO` · video del ensayo `TODO` · captura en pantalla completa `TODO`

### 3.2 Comprensión y verificación: 18 / 25

**Qué perciben los agentes y cómo calculan su acción**

| Agente | Qué percibe | Cómo calcula la acción |
|--------|-------------|------------------------|
| Murciélago | Vecinos a menos de 3.2 unidades (alineación y cohesión), vecinos a menos de 1.2 (separación), caras del icosaedro, y el mouse a menos de 4 unidades | Suma de fuerzas: alineación (×0.9), cohesión (×0.5), separación (×1.8), atracción suave al centro (×0.12), fuerza del mouse (hasta 30) y empuje de las paredes (×25 según cuánto las penetra). La velocidad se limita entre 1.8 y 4.2 |
| Partícula del flow field | El ángulo del campo en su posición | Avanza 2.2 px por cuadro en esa dirección. El ángulo es `ruido × 4π + dirección`, con dos octavas de ruido (70 % y 30 %) que se desplazan lentamente en el tiempo |

**Predicción y verificación al mover un parámetro**

| Parámetro | Lo que predigo | Cómo lo verifico |
|-----------|----------------|------------------|
| Tamaño de las curvas | El campo usa `1/tamaño` como escala: más tamaño, curvas más amplias y suaves | Mover el slider y observar las estelas |
| Dirección de las curvas (`1`) | Suma un ángulo fijo a todo el campo: el patrón no cambia, solo hacia dónde fluye | Pulsar `1` y ver cómo el flujo gira 45° |
| Tecla `A` | Invierte el signo de la fuerza del mouse: atrae o repele | Acercar el mouse a la bandada en cada modo |
| Tamaño del icosaedro | Las paredes se acercan o se alejan y cambia la densidad de la bandada | Reducirlo y ver que la bandada se aprieta y rebota más |
| Tecla `V` | Aumenta la velocidad de giro de las paredes, que empujan más a los agentes | Pulsar `V` y ver el efecto en la bandada |
| Cantidad de murciélagos | Más agentes, más fuerza de separación local y bandada más densa | Mover el slider |

Evidencia de las pruebas: `TODO`

**Qué me faltó para llegar a 25**

- Los radios y pesos del flocking están fijos en el código. No los puedo mover en vivo ni los justifiqué con una serie de pruebas que cambie un valor a la vez.
- La bandada y el flow field son dos capas independientes: los murciélagos no leen el campo. La relación entre ellas es de composición visual y de control, no física, y me falta defender bien esa decisión.
- No tengo registradas las predicciones contra los resultados con capturas o video.

### 3.3 Diseño e intención: 25 / 25

**Por qué:**

- **Flocking** para los murciélagos, porque una colonia que se mueve junta sin líder es el comportamiento emergente que quiero mostrar.
- **Flow field** para el fondo, porque da una textura orgánica de fluido que contrasta con el movimiento de agentes.
- **Icosaedro** como entorno: un contenedor geométrico que rota y condiciona a los agentes desde afuera.
- **Paleta limitada** a morado, vino tinto, rojo, gris y negro, con orden aleatorio, para sostener una atmósfera oscura y tensa.

**Relación con la pieza musical** (ejemplo, ajústalo a tu pieza):

| Pasaje | Intención | Intervención |
|--------|-----------|--------------|
| `TODO` | Calma, la bandada se mueve unida | Giro lento, curvas amplias |
| `TODO` | Tensión creciente | Subir el giro (`V`), más curvas (`U`), acelerar el color |
| `TODO` | Dispersión o ataque | Mouse en modo repele (`A`), `R` para reiniciar |

Score visual: `TODO`

### 3.4 Interpretación humana: 25 / 25

**Por qué:** todos los cambios los decide la persona durante la ejecución, escuchando la música y mirando el sistema. No hay análisis de audio ni secuencia automática.

| Tipo de control | Controles |
|-----------------|-----------|
| Entorno | Tamaño del icosaedro, giro del icosaedro (`V`), reinicio al azar (`R`) |
| Flow field | Velocidad del color, tamaño de las curvas, cantidad de curvas (`U`), dirección (`1` y slider), color del fondo (`C`) |
| Agentes | Cantidad de murciélagos, mouse que atrae o repele (`A`), color de la colonia (`M`) |
| Presentación | Ocultar o mostrar la interfaz y el cursor (`H`); el mouse sigue actuando |

Cada control tiene un efecto perceptible y se puede operar con teclado, de modo que se puede interpretar en vivo sin buscar sliders. Evidencia de un ensayo conduciendo la pieza: `TODO`

---

## 4. Cierre

Mi mayor fortaleza es la conducción en vivo: los controles son claros y cada uno tiene un efecto visible. Lo que más debo trabajar es poder explicar y defender cada parámetro del sistema con pruebas registradas, que es lo que explica los 18 puntos del criterio 2.
