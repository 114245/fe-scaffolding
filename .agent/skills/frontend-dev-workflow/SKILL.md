---
name: frontend-dev-workflow
description: "Flujo paso a paso obligatorio para análisis, diseño, codificación y verificación manual de problemas en Frontend."
---

# Flujo de desarrollo Frontend

Este skill define el workflow completo que el agente ejecuta ante **cualquier problema frontend**. Los pasos son secuenciales y ninguno se salta. Cada checkpoint marcado con 🔴 requiere confirmación del usuario antes de continuar.

---

## Paso 1 — Análisis del problema 🔴

Antes de crear cualquier archivo, el agente responde estas preguntas y las presenta al usuario:

**Dominio y UI**

- ¿Cuáles son las entidades del dominio? (→ módulos `logic/`)
- ¿Cuáles son los estados posibles de la UI? (carga, éxito, error, vacío, interactuando)
- ¿Cuáles son los flujos de usuario principales?
- ¿Qué datos persisten entre sesiones? (`localStorage` vs `sessionStorage`)
- ¿Hay llamadas a API? (→ módulo `api/` + estados de UI obligatorios)

**Accesibilidad (checklist antes de diseñar)**

- ¿Hay un único `<h1>` por página? ¿La jerarquía de headings es lógica?
- ¿Todos los `<input>` tienen `<label>` vinculado?
- ¿Todas las `<img>` tienen `alt`?
- ¿Hay componentes dinámicos que necesiten atributos ARIA?
- ¿Los elementos interactivos son navegables por teclado?
- ¿El contraste cumple WCAG 2.2 AA (mínimo 4.5:1)?

**Arquitectura de módulos**

- ¿Qué módulos `logic/` se necesitan? (funciones puras, sin DOM)
- ¿Qué módulos `ui/` se necesitan? (render, sin lógica de dominio)
- ¿Qué módulo `api/` se necesita? (fetch encapsulado)
- ¿Hay estado global? (→ `state.js`)

**Rendimiento de renderizado**

- ¿Hay listas con múltiples nodos? → `DocumentFragment` + `replaceChildren()`
- ¿Hay eventos sobre elementos dinámicos? → delegación en el padre
- ¿Hay peticiones con riesgo de race condition? → `AbortController`

Si hay ambigüedades, listarlas, proponer interpretación y esperar confirmación.

---

## Paso 2 — Diseño de arquitectura y dirección visual 🔴

Presentar al usuario antes de codificar:

**Mapa de módulos:**

```
logic/juego.js     → calcularPuntaje(), validarMovimiento() [sin DOM]
api/ranking.js     → obtenerRanking(), guardarPartida()     [sin DOM]
ui/tablero.js      → renderTablero(), actualizarCelda()     [solo DOM]
state.js           → { mazo, mano, turno, puntaje }         [sin DOM, sin fetch]
main.js            → orquesta todo, ata eventos
```

**Dirección visual (declarar explícitamente):**

- **Tono**: elegir uno concreto (editorial, playful, industrial, luxury, brutalist, orgánico, etc.)
- **Paleta**: dominante + acento sharp + variables en `:root`
- **Tipografía**: fuente display (titulares) + fuente cuerpo. No Inter/Roboto/Arial como primera elección.
- **Layout**: Flexbox y/o Grid, mobile-first
- **Animaciones**: identificar al menos 2 momentos significativos (entrada de componentes, feedback de acción)

---

## Paso 3 — Codificación

Generar el código siguiendo las reglas de `rules/frontend-code-rules.md`.

**Plantilla de estructura HTML:**

```html
<!DOCTYPE html>
<html lang="es">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>[Título]</title>
    <link rel="stylesheet" href="/src/styles/main.css" />
  </head>
  <body>
    <header>
      <nav aria-label="Navegación principal">...</nav>
    </header>
    <main>
      <h1>[Título único]</h1>
      <section aria-labelledby="[id]">
        <h2 id="[id]">[Sección]</h2>
      </section>
    </main>
    <footer>...</footer>
    <script type="module" src="/src/main.js"></script>
  </body>
</html>
```

**Plantilla de módulo `api/`:**

```js
// api/ranking.js
export async function obtenerRanking() {
  const controller = new AbortController();
  try {
    const response = await fetch("/api/ranking", {
      signal: AbortSignal.timeout(5000),
    });
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return await response.json();
  } catch (error) {
    if (error.name === "AbortError")
      throw new Error("Tiempo de espera agotado");
    throw error;
  }
}
```

**Plantilla de uso de fetch con estados de UI:**

```js
async function cargarRanking() {
  mostrarEstadoCarga();
  try {
    const datos = await obtenerRanking();
    renderRanking(datos);
  } catch (error) {
    mostrarEstadoError("No se pudo cargar el ranking.");
  } finally {
    ocultarEstadoCarga();
  }
}
```

**Plantilla de delegación de eventos:**

```js
contenedor.addEventListener("click", (e) => {
  const celda = e.target.closest("[data-celda-id]");
  if (!celda) return;
  manejarClickCelda(celda.dataset.celdaId);
});
```

**Plantilla de renderizado eficiente:**

```js
function renderLista(items, contenedor) {
  const fragment = document.createDocumentFragment();
  items.forEach((item) => {
    const el = crearElemento(item);
    fragment.appendChild(el);
  });
  contenedor.replaceChildren(fragment);
}
```

**Plantilla de animación CSS-first:**

```css
.elemento--entrando {
  animation: entrar 300ms ease forwards;
}
@keyframes entrar {
  from {
    transform: translateY(-16px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}
/* Staggered reveal */
.lista > *:nth-child(1) {
  animation-delay: 0ms;
}
.lista > *:nth-child(2) {
  animation-delay: 80ms;
}
.lista > *:nth-child(3) {
  animation-delay: 160ms;
}
```

```js
// Disparar animación desde JS agregando/quitando clase
elemento.classList.add("elemento--entrando");
elemento.addEventListener(
  "animationend",
  () => {
    elemento.classList.remove("elemento--entrando");
  },
  { once: true },
);
```

**Persistencia:**

```js
// localStorage — preferencias entre sesiones
const guardar = (key, valor) =>
  localStorage.setItem(key, JSON.stringify(valor));
const cargar = (key, fallback) => {
  try {
    return JSON.parse(localStorage.getItem(key)) ?? fallback;
  } catch {
    return fallback;
  }
};
```

---

## Paso 4 — Verificación manual

El frontend no tiene suite de tests automáticos. La verificación es manual y estructurada.

**Estructura y semántica**

- [ ] Un solo `<h1>` en toda la página
- [ ] Sin `<div>` donde habría etiqueta semántica
- [ ] Todo `<input>` tiene `<label>` vinculado
- [ ] Toda `<img>` tiene `alt`
- [ ] Atributos ARIA solo donde el HTML nativo es insuficiente

**JavaScript**

- [ ] Sin `var`
- [ ] Sin `innerHTML` con datos del usuario
- [ ] Sin mutación de arrays/objetos originales
- [ ] Sin listeners individuales en elementos dinámicos
- [ ] Toda operación `fetch` tiene: `response.ok`, `try/catch`, estados de UI
- [ ] Storage con `JSON.stringify`/`JSON.parse`; sin datos sensibles
- [ ] Módulos separados: `logic/` ≠ `ui/` ≠ `api/`

**CSS y diseño**

- [ ] `box-sizing: border-box` global
- [ ] Responsive en mobile (≤640px), tablet (768px) y desktop (≥1024px)
- [ ] Contraste mínimo 4.5:1 para texto normal
- [ ] `:focus-visible` visible en todos los interactivos
- [ ] Al menos 2 animaciones significativas implementadas
- [ ] `prefers-reduced-motion` respetado

**API y estado**

- [ ] Estados de carga, éxito y error visibles en la UI
- [ ] Peticiones cancelables con `AbortController` o `AbortSignal.timeout()`
- [ ] Preferencias y ranking persisten al recargar

**Build**

- [ ] `npm run dev` sin errores en consola
- [ ] Sin errores en DevTools al interactuar con la aplicación
