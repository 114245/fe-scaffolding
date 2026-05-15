# AGENTS.md — Proyecto Frontend (HTML + Tailwind CSS + JavaScript)

## Stack y entorno

- **HTML**: semántico, WCAG 2.2 AA
- **CSS**: Tailwind CSS 3 (utility-first) + CSS custom solo donde Tailwind no alcanza
- **JavaScript**: ES6+, módulos ES nativos, sin frameworks de componentes
- **Build tool**: Vite 5
- **Node.js**: versión LTS

## Setup inicial

```bash
# Instalar dependencias
npm install

# Si hay errores masivos o node_modules corrupto
rm -rf node_modules && npm install
```

## Comandos de desarrollo frecuentes

```bash
# Servidor de desarrollo con hot reload
npm run dev

# Build de producción
npm run build

# Preview del build de producción
npm run preview
```

## Estructura de carpetas

```
proyecto/
  index.html
  package.json
  vite.config.js
  tailwind.config.js
  postcss.config.js
  src/
    main.js          ← punto de entrada y orquestador único
    state.js         ← estado global (si aplica)
    api/
      [recurso].js   ← fetch encapsulado por recurso
    logic/
      [dominio].js   ← lógica pura, sin DOM, sin fetch
    ui/
      [vista].js     ← render y manipulación del DOM, sin lógica de dominio
    styles/
      main.css       ← @tailwind directives + variables CSS en :root
  public/
    [assets estáticos]
```

## Configuración mínima del proyecto

```json
// package.json
{
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "devDependencies": {
    "vite": "^5.0.0",
    "tailwindcss": "^3.4.0",
    "autoprefixer": "^10.4.0",
    "postcss": "^8.4.0"
  }
}
```

```js
// tailwind.config.js
export default {
  content: ["./index.html", "./src/**/*.{js,html}"],
  theme: { extend: {} },
  plugins: [],
};
```

```css
/* src/styles/main.css — estructura mínima */
@tailwind base;
@tailwind components;
@tailwind utilities;

*,
*::before,
*::after {
  box-sizing: border-box;
}

:root {
  --color-primary: #[valor];
  --color-accent: #[valor];
  --color-surface: #[valor];
  --color-text: #[valor];
  --color-error: #[valor];
  --transition-fast: 150ms ease;
  --transition-base: 300ms ease;
}

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

## Convenciones de nomenclatura

| Elemento           | Convención       | Ejemplo             |
| ------------------ | ---------------- | ------------------- |
| Archivos JS        | kebab-case       | `mazo-service.js`   |
| Funciones          | camelCase        | `calcularPuntaje()` |
| Constantes         | UPPER_SNAKE_CASE | `MAX_CARTAS_MANO`   |
| Clases CSS custom  | kebab-case       | `.carta--activa`    |
| Atributos `data-*` | kebab-case       | `data-celda-id`     |

## Convenciones de código

- `logic/` nunca importa de `ui/` ni referencia el DOM
- `ui/` nunca contiene lógica de dominio
- `api/` encapsula todos los `fetch` — ningún `fetch` disperso en otros módulos
- `main.js` es el único punto de orquestación
- `const` por defecto; `let` solo cuando hay reasignación; `var` prohibido
- Módulos ES con `import`/`export` en todos los archivos

## Persistencia

- `localStorage`: preferencias de interfaz que deben sobrevivir entre sesiones
- `sessionStorage`: estado temporal de la sesión actual
- **Nunca** guardar contraseñas, tokens JWT ni datos sensibles en Storage

## Verificación: qué debe funcionar antes de considerar el trabajo terminado

- `npm run dev` sin errores en consola
- Layout verificado en mobile (≤640px), tablet (768px) y desktop (≥1024px)
- Navegación por teclado funcional en todos los elementos interactivos
- Los estados de carga, éxito y error son visibles en la UI para cada operación fetch
- Las preferencias y el ranking persisten al recargar la página

## Archivos que no se deben modificar manualmente

- `dist/` — carpeta generada por Vite build, nunca editar
- `node_modules/` — nunca editar; regenerar con `npm install`

## Recursos adicionales

- **Rules**: ver `.agent/rules/frontend-code-rules.md` — prohibiciones concretas de HTML, JS y CSS
- **Skill**: ver `.agent/skills/frontend-dev-workflow/SKILL.md` — flujo completo de análisis, diseño, codificación y verificación
- **Metodología**: ver `METODOLOGIA.md` — cómo preparar prompts y trabajar con el agente en problemas nuevos
