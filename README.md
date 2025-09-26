# Luz Verde, Luz Roja — Arena Antropomorfa

Este proyecto contiene un minijuego web inspirado en el reto "luz verde, luz roja" de *El juego del calamar*. Todo está contenido en un único archivo HTML listo para abrirse en cualquier navegador moderno.

## Ejecución rápida

1. Clona o descarga este repositorio.
2. Abre el archivo [`red_light_green_light_single_file_web_game_index.html`](./red_light_green_light_single_file_web_game_index.html).
   - Puedes hacer doble clic sobre él o arrastrarlo a una pestaña de tu navegador.
   - Opcionalmente levanta un servidor local con `python3 -m http.server 8000` y visita <http://localhost:8000/red_light_green_light_single_file_web_game_index.html>.

## Características principales

- Menú holográfico inicial con selección entre tres corredores antropomorfos: perro, gato y topo.
- Carrera contra dos rivales IA con cuenta atrás de inicio, temporizador total y cambio de luz verde/roja aleatorio.
- Capacidad de empujar a tus contrincantes cuando estén cerca para desequilibrarlos.
- Diseño visual con pista futurista, muñeca vigilante reactiva y tablero de posiciones.
- Controles pensados para teclado y pantallas táctiles.

## Controles

| Acción | Teclado | Pantalla táctil |
| ------ | ------- | ---------------- |
| Avanzar | `W` o `↑` | Botón "Avanzar" |
| Cambiar de carril | `A`/`←` o `D`/`→` | Toca el lado izquierdo/derecho de la pista |
| Empujar rival | Barra espaciadora | Botón "Empujar" |
| Salir de la ronda | `Esc` | — |

\* El toque corto en los laterales del lienzo cambia tu carril.

## Objetivo

Arranca solo cuando la muñeca muestre la luz verde, detente por completo con la luz roja y llega a la línea de meta antes de que el cronómetro llegue a cero. Si te mueves durante el rojo o el tiempo se agota, la ronda termina. ¡Elimina a tus rivales con empujes estratégicos y consigue la victoria!
