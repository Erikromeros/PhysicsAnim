# Propuesta Técnica: Coleo y Doma — Edición Ética y Cómica

Este documento consolida la visión de **Coleo** como un juego de "Acrobacia Deportiva y Humor" en lugar de uno de confrontación física. El objetivo es que el jugador sienta la adrenalina de la velocidad, pero que cualquier "daño" o maltrato sea redirigido de forma cómica hacia el jinete (Karma).

---

## 1. Filosofía de "Cero Maltrato"

Para alertar sobre el maltrato animal y cambiar el tono del juego, implementaremos:
- **El Toro Atleta:** No es una víctima asustada; es un profesional del show con actitud competitiva (puede usar accesorios como bandas en la frente o gafas de sol).
- **Aviso Legal Cómico:** Al iniciar cada turno, un pequeño mensaje tipo caricatura dice: *"Ningún polígono fue herido en la realización de esta acrobacia"*.
- **Bienestar al 100%:** El toro siempre se levanta rápido, se sacude el polvo y hace una "pose de victoria" para dejar claro que está bien.

---

## 2. Mecánica de Coleo: "Show de Acrobacia"

### A. Físicas de "Pinball & Airbags"
- **La Manga Inflable:** Las barandas tienen sistemas de seguridad exagerados (Airbags gigantes con el logo del juego) que se despliegan al impacto.
- **Toro de Goma:** El toro tiene una física elástica. Al ser "coleado", rueda como una bola de boliche elástica, rebotando en las paredes sin fricción ni daño.
- **Partículas de "Impacto Suave":** En lugar de nubes de polvo secas, al chocar salen partículas de confeti, estrellas de colores y globos de texto con *"¡BOING!"* o *"¡ZAP!"*.

### B. Sistema de Karma Slapstick (Riesgo para el Jinete)
- **El Contra-Tirón:** Si el jugador jala de la cola con demasiada fuerza bruta o mal timing, el toro jala de vuelta.
- **Consecuencias Cómicas:**
    - El jinete sale disparado como por una catapulta.
    - Termina colgado de una rama de árbol mientras su caballo sigue corriendo.
    - Cae en una montaña de paja o un charco de lodo con un sonido sordo de "splat".
    - El jinete queda con la animación de "pajaritos volando" (mareado).

---

## 3. Mecánica de Doma: "Baile de Resistencia"

- **Input de Ritmo:** La doma no es "someter" al caballo, es "seguirle el paso".
- **Visuales de Error:** Si el balance falla, el jinete se resbala de forma ridícula por el costado del caballo y queda arrastrado (estilo comedia muda) antes de soltarse.
- **Recompensa Cómica:** Al ganar, el caballo y el jinete hacen un "High-Five" o un baile sincronizado.

---

## 4. Sistemas de Control y Ética (HUD)

- **Comisionado de Bienestar (Pájaro Árbitro):** Un personaje recurrente (un pájaro gracioso con gorra y silbato) que aparece en el HUD:
    - **Tarjeta Amarilla:** Por movimientos bruscos.
    - **Tarjeta Roja:** Si el "Medidor de Karma" se llena. Cierre inmediato del turno por "Conducta Antideportiva".
- **Castigo Progresivo (Los Tomates):** Si el jugador persiste en ser agresivo, el público le lanza tomates virtuales a la pantalla, bloqueando su visión hasta que "limpie" su actitud.

---

## 5. Plan para el Pasante (Technical Side)

Para que el pasante pueda ejecutar esto, el BDD incluirá:
1. **Material de "Goma":** Shader de elasticidad para el Toro.
2. **Componente de "Karma":** Lógica que detecta la fuerza del jalón y activa el lanzamiento del Jinete.
3. **Data Table de Choques:** Una lista de 10 muertes/caídas cómicas diferentes para que no sea repetitivo.

---

## Preguntas para el Usuario

1. **¿Estilo Visual?:** ¿Te gustaría que el Toro sea un personaje único con nombre (ej. "Toro-Beto") para que el jugador cree un vínculo con él?
2. **¿Muerte Cómica?:** ¿Aprobamos el sistema de "lanzamiento por los aires" como la penalización principal por maltrato?

> [!TIP]
> **Ventaja de Negocio:** Este enfoque hace que el juego sea apto para todas las edades (PEGI 3 / ESRB E) y evita controversias, mientras mantiene la adrenalina del deporte llanero.

---
*Propuesta generada por Antigravity para Coleo Project.*
