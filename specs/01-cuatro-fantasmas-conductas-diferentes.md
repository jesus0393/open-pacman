# SPEC 01 — Cuatro fantasmas con conductas diferentes

> **Estado:** Approved
> **Depende de:** Ninguna
> **Fecha:** 2026-10-02
> **Objetivo:** Incorporar cuatro fantasmas con conductas diferenciadas, incluyendo uno que persiga agresivamente a Pac-Man.

## Alcance

**Incluye:**

- Crear cuatro fantasmas con los tipos `hunter`, `ambusher`, `patroller` y `random`.
- Mantener los cuatro fantasmas dentro de la casa al comenzar y al reiniciar una vida.
- Hacer que `hunter` elija la ruta válida que reduzca la distancia hasta Pac-Man.
- Hacer que `ambusher` persiga un objetivo situado cuatro celdas delante de Pac-Man.
- Hacer que `patroller` alterne entre perseguir a Pac-Man durante 5 segundos y regresar al punto `(17, 23)` durante 5 segundos.
- Hacer que `random` elija una dirección válida aleatoria en cada cruce y evite invertir la marcha salvo en callejones.
- Mantener las reglas actuales de velocidad, colisiones, vidas, túneles y reinicio.
- Mantener los cuatro colores actuales de los fantasmas.

**Fuera de alcance (para futuras especificaciones):**

- Modos clásicos de dispersión, persecución temporal y miedo.
- Fantasmas comestibles y puntuación adicional por comer fantasmas.
- Salida escalonada de la casa.
- Cambios visuales, nuevos sprites o animaciones específicas por conducta.
- Aumento de velocidad del fantasma cazador.
- Persistencia, niveles adicionales o nuevas variantes del laberinto.

## Modelo de datos

Los fantasmas existentes se amplían con estado específico de su conducta:

```js
{
  x: 12,
  y: 14,
  dir: 'up',
  speed: 0.1,
  kind: 'hunter',
  mode: 'chase',
  modeFrames: 0,
}
```

- `kind` puede ser `hunter`, `ambusher`, `patroller` o `random`.
- `mode` solo es necesario para `patroller` y alterna entre `chase` y `return`.
- `modeFrames` cuenta actualizaciones dentro del modo actual.
- Los cuatro puntos iniciales son `(12,14)`, `(13,14)`, `(14,14)` y `(15,14)`, asignados en ese orden a `hunter`, `ambusher`, `patroller` y `random`.
- El punto de patrulla es `(17,23)`.
- El objetivo del emboscador se calcula desde la posición y dirección actuales de Pac-Man, cuatro celdas hacia delante.
- Las decisiones se toman únicamente cuando el fantasma está alineado con el centro de una celda.

## Plan de implementación

1. Actualizar `src/js/maze.js` para definir cuatro posiciones iniciales dentro de la casa y asignar los cuatro tipos de fantasma.
2. Ampliar `createGame()` en `src/js/game.js` para inicializar el estado necesario del patrullero sin mutar `MAZE`.
3. Separar en `src/js/game.js` el cálculo de objetivos y la selección de dirección para permitir las conductas cazadora, emboscadora, patrullera y aleatoria, manteniendo el bloqueo de paredes y puertas existente.
4. Implementar en `src/js/game.js` el ciclo del patrullero: 300 actualizaciones persiguiendo y 300 actualizaciones regresando al punto `(17,23)`, usando la misma selección de rutas válidas.
5. Actualizar el reinicio de posiciones en `src/js/game.js` para restablecer la posición, dirección y estado de conducta de cada fantasma.
6. Verificar manualmente el juego mediante `python3 -m http.server -d src 8000`: iniciar una partida, observar las cuatro conductas y comprobar que las colisiones, vidas y pantallas actuales siguen funcionando.

## Criterios de aceptación

- [ ] El juego carga sin errores en la consola.
- [ ] `createGame()` crea exactamente cuatro fantasmas.
- [ ] Los fantasmas comienzan en `(12,14)`, `(13,14)`, `(14,14)` y `(15,14)`.
- [ ] Los cuatro fantasmas pueden salir de la casa y desplazarse por celdas transitables.
- [ ] `hunter` selecciona, entre las rutas válidas, la que reduce la distancia hasta Pac-Man.
- [ ] `ambusher` selecciona rutas hacia un objetivo situado cuatro celdas delante de Pac-Man.
- [ ] `patroller` persigue durante 300 actualizaciones y después intenta volver a `(17,23)` durante 300 actualizaciones.
- [ ] `random` selecciona rutas aleatorias en los cruces y no invierte la marcha cuando existe otra opción válida.
- [ ] Ningún fantasma atraviesa paredes y todos pueden atravesar la puerta de la casa según las reglas actuales.
- [ ] Los fantasmas reaparecen en sus posiciones y estados iniciales después de una colisión que no termina la partida.
- [ ] Las colisiones siguen restando una vida y la partida termina al perder la última vida.
- [ ] Los cuatro fantasmas conservan sus colores actuales.
- [ ] Pac-Man puede ganar al consumir todos los puntos y puede reiniciar desde las pantallas existentes.

## Decisiones

- **Sí:** cuatro arquetipos `hunter`, `ambusher`, `patroller` y `random`. Crean diferencias claras usando la arquitectura existente.
- **Sí:** el cazador prioriza reducir la distancia a Pac-Man. Es agresivo sin requerir una velocidad especial.
- **Sí:** el emboscador apunta cuatro celdas por delante. Es una anticipación visible y sencilla de verificar.
- **Sí:** el patrullero usa `(17,23)` como punto fijo. Es una celda transitable de la zona central inferior y no coincide con el inicio de Pac-Man.
- **Sí:** los cuatro fantasmas comienzan dentro de la casa. Mantiene una presentación coherente con el nivel actual.
- **Sí:** cinco segundos equivalen a 300 actualizaciones a 60 FPS. Evita introducir un sistema de tiempo independiente en esta versión.
- **Sí:** se conservan las velocidades y colores existentes. El cambio se limita a la toma de decisiones de la IA.
- **No:** modos clásicos de Pac-Man. Requieren estados, transiciones y reglas de puntuación propias.
- **No:** un sistema de navegación con búsqueda global. La selección local entre direcciones válidas es suficiente para este laberinto y mantiene el código simple.

## Riesgos

| Riesgo | Mitigación |
| --- | --- |
| El objetivo del emboscador puede quedar detrás de una pared o fuera del laberinto. | Evaluar solo direcciones válidas y usar la posición objetivo para escoger la mejor ruta disponible. |
| El patrullero puede no alcanzar su punto durante un intervalo de 5 segundos. | Cambiar de modo al terminar el intervalo y conservar el punto como objetivo del siguiente regreso. |
| Cuatro fantasmas pueden colisionar dentro de la casa al iniciar. | Usar cuatro celdas consecutivas y mantener la separación inicial actual mediante sus direcciones. |
| La cadencia real de `requestAnimationFrame` puede no ser exactamente 60 FPS. | Documentar que los 300 frames son una aproximación y no introducir aún tiempo basado en reloj. |

## Lo que **no** está incluido en esta especificación

- Modos clásicos de dispersión, miedo o fantasmas comestibles.
- Salida escalonada de la casa.
- Nuevos sprites, animaciones o colores.
- Aumento de velocidad del cazador.
- Persistencia, niveles adicionales o multijugador.
