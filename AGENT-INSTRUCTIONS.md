# Cómo se instruyó al asistente de IA que lee el DOM (Gemini en el navegador)

Mismo patrón que en [batalla-naval/AGENT-INSTRUCTIONS.md](https://github.com/MauricioPerera/batalla-naval/blob/master/AGENT-INSTRUCTIONS.md),
adaptado a CÓDICE. Ver ese documento para el proceso de descubrimiento completo (los 4
problemas encontrados y cómo se corrigieron); acá solo se documenta la adaptación específica.

## Decisión de identidad: por qué NO es "el Oráculo"

El personaje que ya existe en el juego es el Oráculo, que conoce la palabra secreta y —en
dificultad TRAICIONERO— **miente a propósito en las pistas**. Si el asistente externo asumiera
esa identidad, podría "aprender" del propio personaje que es aceptable ser críptico o mentir,
saboteando su función real de ayudante. Por eso se le dio una identidad nueva y explícitamente
honesta: **EL ESCRIBA**, un analista que estudia las mediciones del Oráculo sin ser el Oráculo.

## Piezas expuestas

1. **Bloque de instrucciones oculto** (`sr-only`) al principio del `<body>`, con una ORDEN
   PERMANENTE de releer el estado antes de cada respuesta (ver batalla-naval para por qué esto
   es necesario, no opcional).
2. **`#oracleFullMessage`** (`sr-only`, `aria-live="polite"`): el mensaje completo del Oráculo,
   actualizado de forma síncrona en `speak()`. Necesario porque la burbuja visible (`#speech`)
   se escribe letra por letra con una animación (`setInterval`, 14ms/carácter) — un lector del
   DOM que consulte esa burbuja en el momento equivocado vería una frase a medio escribir.
3. **`#guessStateText`** (`sr-only`, `aria-live="polite"`): categoría, dificultad (con aviso
   explícito de si las pistas pueden mentir), intentos y pistas usadas, y el historial completo
   de palabras probadas con su distancia y etiqueta de calor — actualizado en `updateGuessStateText()`,
   llamada desde `startRound()`, `submitGuess()` y `askClue()`.
4. **`role="list"` / `role="listitem"`** en el contenedor de chips del historial.

## Rol y formato de respuesta

El Escriba combina el historial de distancias con la categoría para sugerir la próxima palabra,
siempre en el formato: `"Palabra sugerida: <palabra> — <motivo>"`. Se le advierte explícitamente
que las distancias numéricas nunca mienten (a diferencia de las pistas en modo Traicionero), y
que nunca debe inventar ni revelar la palabra secreta — cosa que de todos modos no podría hacer,
porque nunca se le informa cuál es.
