# Bitácora · Unidad 3: Fuerzas

## Geometrías en fuerza

- **Instrumento publicado:** [https://jimenisaa30.github.io/test-Unidad-3/](https://jimenisaa30.github.io/test-Unidad-3/)
- **Repositorio:** [jimenisaa30/test-Unidad-3](https://github.com/jimenisaa30/test-Unidad-3)
- **Narrativa:** un instrumento visual de mandalas cinéticos. Las partículas no representan criaturas: forman espirales, estrellas y corazones que se deforman mediante fuerzas físicas durante una interpretación en vivo.

---

## 1. Instrumento funcional y publicado

El instrumento se publica en GitHub Pages y contiene dos modos. **LAB** muestra los parámetros, pruebas y marcador del atractor. **PERFORMANCE** oculta la interfaz y el cursor, aunque el mouse sigue controlando el centro del vórtice.

| Control | Acción |
| --- | --- |
| Mouse | Mueve el centro del atractor y del vórtice. |
| `Q` | Activa o desactiva gravedad. |
| `W` / `E` | Activa repulsión / atracción radial. |
| `R` / `T` | Activa o desactiva vórtice / resistencia del aire. |
| `1–5` | Ajusta radio, tamaño, amortiguamiento, velocidad máxima y potencia. |
| `6` | Alterna espiral, estrella y corazón con transición suave. |
| `C` | Genera una paleta cromática aleatoria. |
| `Espacio` | Dispara un pulso temporal de fuerza. |
| `P` / `Enter` | Alterna LAB-PERFORMANCE / reinicia la forma actual. |

### Espiral en modo LAB

![Captura de pantalla 2026-08-27 230124](<./evidencias/Captura de pantalla 2026-08-27 230124.png>)

La espiral rosada confirma que una forma reconocible puede construirse con miles de partículas. También se observan el panel de LAB, el atractor y los parámetros de fuerza.

### Estrella en modo PERFORMANCE

![Captura de pantalla 2026-08-27 234643](<./evidencias/Captura de pantalla 2026-08-27 234643.png>)

La estrella verde demuestra el modo performativo: no hay panel visible y la forma es la protagonista de la escena.

---

## 2. Mapa del sistema

```text
estado de partículas → fuerzas → aceleración → velocidad → posición → render
```

| Elemento | Archivo y componentes | Función |
| --- | --- | --- |
| Estado | `src/simulation/createSimulation.js`: `positionBuffer`, `velocityBuffer`, `targetBuffer` | Guarda posición, velocidad y destino de cada partícula en GPU. |
| Formas | `createSimulation.js`: inicializadores de espiral, estrella y corazón | Distribuye partículas en mandalas y genera destinos para las transiciones. |
| Fuerzas | `createSimulation.js`: bloque `force` | Suma gravedad, radial, vórtice, resistencia del aire y pulso. |
| Integración | `createSimulation.js`: actualización de `v` y `p` | Aplica Euler semiimplícito, limita velocidad y mantiene las partículas en el espacio de trabajo. |
| Render | `createSimulation.js`: `SpriteNodeMaterial`, `InstancedMesh` | Dibuja sprites circulares con color dependiente de la velocidad. |
| Parámetros | `src/simulation/parameters.js` | Define los uniforms modificables en tiempo real. |
| Controles | `src/main.js` | Conecta teclado, mouse, pulso y modos con la simulación. |
| Interfaz | `src/ui/labPanel.js`, `src/styles.css` | Muestra sliders, pruebas, botones y el estilo visual. |

---

## 3. Ficha de fuerzas

| Fuerza | Modelo y control | Decisión de diseño | Resultado esperado |
| --- | --- | --- | --- |
| Gravedad | `F = (0, -g, 0)` · `Q` | Magnitud limitada para no borrar la figura de inmediato. | La forma cae hacia abajo. |
| Atracción radial | `F = k · dirección / distancia²` · `E` | Se usa *softening* para evitar una fuerza infinita cerca del centro. | Las partículas convergen hacia el cursor. |
| Repulsión radial | `F = -k · dirección / distancia²` · `W` | Invierte la fuerza radial sin cambiar su arquitectura. | La composición se expande desde el cursor. |
| Vórtice | `F = k · (eje Z × dirección radial)` · `R` | El centro se desplaza con el mouse y puede combinarse con atracción. | Las partículas giran alrededor del cursor. |
| Resistencia del aire | `F = -c · v` · `T`, `3` | El aire se modela como fluido que se opone a la velocidad. | El movimiento pierde energía y se estabiliza. |
| Pulso | Ganancia temporal de las fuerzas · `Espacio` | Aumenta la energía expresiva sin cambiar la estructura de fuerzas. | La respuesta física se vuelve momentáneamente más intensa. |

---

## 4. Registro de pruebas

| Prueba | Predicción | Observación documentada | Evidencia |
| --- | --- | --- | --- |
| Inercia y forma inicial | La figura conserva su distribución si no hay una fuerza dominante. | La espiral mantiene una estructura continua antes de una perturbación fuerte. | [Captura de pantalla 2026-08-27 230124](<./evidencias/Captura de pantalla 2026-08-27 230124.png>) |
| Atracción radial | Las partículas se concentran en el atractor. | La nube se acumula alrededor del centro, confirmando una dirección centrípeta. | [Captura de pantalla 2026-08-27 233602](<./evidencias/Captura de pantalla 2026-08-27 233602.png>) |
| Resistencia del aire | Un amortiguamiento alto reduce la energía. | La captura registra drag activo y una nube con movimiento contenido. | [Captura de pantalla 2026-08-27 233147](<./evidencias/Captura de pantalla 2026-08-27 233147.png>) |
| Vórtice y atracción | La combinación produce giro orbital. | La espiral conserva curvatura y círculos concéntricos alrededor del centro de interacción. | [Captura de pantalla 2026-08-28 000423](<./evidencias/Captura de pantalla 2026-08-28 000423.png>) |
| Estrella | Las partículas deben ocupar una figura de puntas reconocibles. | En PERFORMANCE aparece una estrella de múltiples puntas formada por partículas verdes. | [Captura de pantalla 2026-08-27 234643](<./evidencias/Captura de pantalla 2026-08-27 234643.png>) |
| Corazón y potencia | La forma sigue identificable mientras la fuerza es visible. | El corazón se reconoce y la concentración central evidencia atracción radial con potencia elevada. | [Captura de pantalla 2026-08-28 000850](<./evidencias/Captura de pantalla 2026-08-28 000850.png>) |
| Pulso performativo | El pulso debe producir una respuesta más energética. | Se observa una dispersión amplia y contrastes dorados en PERFORMANCE. | [Captura de pantalla 2026-08-28 122352](<./evidencias/Captura de pantalla 2026-08-28 122352.png>) |

### Evidencia de depuración

![Captura de pantalla 2026-08-27 231415](<./evidencias/Captura de pantalla 2026-08-27 231415.png>)

Esta captura registra una etapa donde no se visualizaban partículas. Los ejes y la interfaz sí cargaban, lo cual permitió aislar el problema en el render de partículas, corregirlo y recuperar la simulación visible. La imagen es evidencia del proceso de prueba, depuración y corrección.

---

## 5. Score visual

La obra usa las fuerzas como coreografía: una forma reconocible se contrae, gira, cae, se expande y cambia de color según las acciones de quien interpreta.

1. Iniciar con **espiral** en LAB para presentar los parámetros.
2. Cambiar la paleta con `C`, activar vórtice con `R` y desplazar el centro con el mouse.
3. Usar `Espacio` para crear un primer acento energético.
4. Cambiar con `6` a **estrella**, entrar a PERFORMANCE con `P` y combinar atracción con vórtice.
5. Cambiar a **corazón**, aumentar amortiguamiento y cerrar con un pulso final.

| Momento | Evidencia | Lectura visual |
| --- | --- | --- |
| Espiral inicial | ![Captura de pantalla 2026-08-28 000423](<./evidencias/Captura de pantalla 2026-08-28 000423.png>) | Espiral concéntrica sobre violeta: inicio calmo y legible. |
| Estrella en performance | ![Captura de pantalla 2026-08-27 234643](<./evidencias/Captura de pantalla 2026-08-27 234643.png>) | Forma radial verde sin panel: el gesto visual toma el protagonismo. |
| Pulso final | ![Captura de pantalla 2026-08-28 122352](<./evidencias/Captura de pantalla 2026-08-28 122352.png>) | Campo dorado expandido: momento de mayor energía y densidad. |

---

## 6. Bitácora de IA

| Decisión | Cambio aplicado | Resultado |
| --- | --- | --- |
| Reemplazar criaturas por geometrías cinéticas. | Se implementaron espiral, estrella y corazón como distribuciones reales de partículas. | Las capturas confirman que cada forma puede ser reconocida y transformada. |
| Mantener el flujo físico del proyecto. | Se preservó `estado → fuerzas → integración → render`; solo se añadieron inicializadores, uniforms y controles. | El mapa del sistema muestra las responsabilidades de cada archivo. |
| Modelar gravedad, atracción, repulsión, vórtice y aire. | Las fuerzas se incorporaron al compute shader y se expusieron como controles. | Las pruebas muestran acumulación, giro, expansión y amortiguamiento. |
| Convertir la simulación en instrumento performativo. | Se añadieron teclado, paletas, modo PERFORMANCE, cursor oculto y pulso. | Las evidencias de PERFORMANCE muestran una escena limpia y reactiva. |
| Corregir fallos de visualización. | Se simplificó el camino de render y se descartaron combinaciones de buffers incompatibles. | La captura sin partículas conserva el registro del error; las siguientes documentan la solución. |

---

## 7. Autoevaluación ponderada

La siguiente autoevaluación propone una valoración alta: el instrumento funciona, está publicado y cuenta con evidencia visual amplia. El descuento se debe a que las pruebas se documentan con imágenes y no con un video continuo de toda la performance.

| Criterio | Ponderación | Puntaje obtenido | Justificación |
| --- | ---: | ---: | --- |
| Instrumento funcional y publicado | 20 % | 19 / 20 | El enlace público incluye LAB y PERFORMANCE; las capturas documentan sus formas y controles. |
| Mapa del sistema y comprensión técnica | 15 % | 14 / 15 | Se identifican estado, fuerzas, integración, render, parámetros, controles e interfaz. |
| Fichas de fuerzas y decisiones de diseño | 20 % | 18 / 20 | Se describen cinco fuerzas y el pulso con modelos, límites y efectos esperados. |
| Registro de pruebas | 15 % | 13 / 15 | Se documentan inercia, atracción, aire, vórtice, formas y pulso con capturas. |
| Score visual y controles performativos | 15 % | 14 / 15 | Existe una secuencia de interpretación con paleta, transiciones, pulso y PERFORMANCE. |
| Bitácora de IA y proceso iterativo | 15 % | 14 / 15 | Se explican decisiones, correcciones de render y resultados de pruebas. |
| **Total** | **100 %** | **92 / 100** | **Desempeño alto; la principal mejora futura es registrar la performance completa en video.** |

Nota final: 4.6

## Reflexión final

La unidad permitió comprender que las fuerzas son herramientas de composición y no solo ecuaciones. Atracción, repulsión y vórtice modifican la identidad de una misma figura, mientras la resistencia del aire controla su energía. La decisión más efectiva fue convertir cada forma en una distribución de partículas, de modo que participa realmente en el sistema físico. En una versión futura se añadirá un registro audiovisual continuo y controles de audio para relacionar el pulso con la música.
