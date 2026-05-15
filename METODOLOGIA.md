# METODOLOGIA.md — Cómo trabajar con el agente en un problema Frontend

Esta guía es el complemento humano de los tres archivos de configuración del agente:

- `AGENTS.md` — setup y convenciones del proyecto
- `rules/frontend-code-rules.md` — prohibiciones concretas de HTML, JS y CSS
- `skills/frontend-dev-workflow.md` — flujo completo de análisis a verificación

Las reglas técnicas viven en esos archivos. Este documento te enseña a **activarlas correctamente** cuando llegás a un problema nuevo y necesitás resolverlo con el agente de forma eficiente.

**La regla de oro: vos tomás las decisiones de diseño, el agente las ejecuta.**

---

## Qué se evalúa (pesos de referencia)

| Criterio                                      | Peso | Qué mirar concretamente                                         |
| --------------------------------------------- | ---- | --------------------------------------------------------------- |
| Modelado del dominio y reglas                 | 20%  | Módulos `logic/` sin DOM, entidades bien definidas              |
| Lógica y validaciones                         | 20%  | Validaciones antes de aplicar acciones, feedback de errores     |
| Máquina / decisiones automáticas              | 15%  | Al menos dos niveles de dificultad diferenciables y explicables |
| HTML semántico, CSS responsive, accesibilidad | 15%  | Sin div-itis, mobile/tablet/desktop, contraste, labels          |
| Render visual                                 | 10%  | Animaciones, feedback visual, estados de UI claros              |
| Consumo de API y manejo de estado             | 10%  | fetch con `response.ok`, `try/catch`, carga/éxito/error         |
| Defensa oral                                  | 10%  | Podés explicar cada decisión técnica y modificar en vivo        |

**Conclusión práctica**: el 60% del puntaje es lógica + dominio + validaciones + accesibilidad. El render visual es solo el 10%. No invertir el tiempo al revés.

---

## Paso 1 — Análisis en papel (antes de abrir la computadora)

Con lápiz sobre el enunciado, marcá:

| Qué buscar                             | Para qué sirve                                            |
| -------------------------------------- | --------------------------------------------------------- |
| **Sustantivos** del dominio            | Módulos `logic/`                                          |
| **Verbos** / acciones del usuario      | Eventos a manejar, funciones a implementar                |
| "Ranking", "historial", "preferencias" | Persistencia → `localStorage` vs `sessionStorage`         |
| "API REST", "fetch", "JSON local"      | Módulo `api/` + estados de UI obligatorios                |
| Reglas de validación                   | Lógica pura en `logic/`                                   |
| "Animación", "feedback visual"         | Puntos de animación que el evaluador va a mirar           |
| Niveles de dificultad                  | Estrategia de la máquina → al menos dos ramas de decisión |

Al terminar este paso debés poder responder:

- ¿Qué módulos `logic/` necesito y qué hace cada uno?
- ¿Qué módulos `ui/` necesito?
- ¿Qué va a `localStorage`? ¿Qué va a `sessionStorage`?
- ¿Qué llama a la API y cuándo?
- ¿Cuáles son las 3-4 validaciones más importantes?
- ¿Qué animo y cuándo?

---

## Paso 2 — Redactar el prompt

```
Ejecutá el skill frontend-dev-workflow.md sobre el siguiente problema.

## Enunciado
[Transcribir textualmente. No resumir ni interpretar.]

## Mi análisis previo (confirmar antes de codificar)

Módulos logic/ identificados (sin DOM):
- logic/[nombre].js: [responsabilidad]

Módulos ui/ identificados:
- ui/[nombre].js: renderiza [qué]

Persistencia:
- localStorage: [qué datos] porque [razón]
- sessionStorage: [qué datos] porque [razón]

API:
- Recurso: [descripción]
- Estados de UI: carga / éxito / error en [dónde]

Validaciones clave del dominio:
- [Regla 1]: validar antes de [acción]
- [Regla 2]: rechazar con feedback visible si [condición]

Puntos de animación:
- [Evento 1]: [qué se anima y cómo]
- [Evento 2]: [qué se anima y cómo]

Niveles de dificultad de la máquina:
- Fácil: [estrategia]
- Difícil: [estrategia]

Dirección visual propuesta:
- Tono: [editorial / playful / industrial / luxury / etc.]
- Paleta: [descripción breve]
- Tipografía: [fuente display] + [fuente cuerpo]

## Instrucción
Ejecutá el Paso 1 del skill (análisis) con mi análisis previo como base.
Confirmá, corregí o ampliá antes de continuar al Paso 2.
```

---

## Paso 3 — Validar el análisis del agente

| Señal de problema                          | Qué pedirle                                             |
| ------------------------------------------ | ------------------------------------------------------- |
| Un archivo mezcla `logic/` con DOM         | "Separar en `logic/` y `ui/` siguiendo el AGENTS"       |
| `fetch` en `main.js` o en un archivo de UI | "Mover a `api/[recurso].js` encapsulado"                |
| No menciona `response.ok` ni `try/catch`   | "Agregar validación y manejo de error con estado de UI" |
| Solo un nivel de dificultad                | "Describir dos estrategias diferenciables"              |
| Dirección visual genérica                  | "Elegir tono más específico y fuente distintiva"        |
| Sin puntos de animación definidos          | "Identificar al menos 2 momentos de animación"          |

---

## Paso 4 — Iterar con precisión quirúrgica

```
// ✅ Prompt acotado
"En ui/tablero.js hay cálculo de puntaje.
Moverlo a logic/puntaje.js como función pura.
No toques ningún otro archivo."

// ✅ Prompt acotado
"El fetch en api/ranking.js no valida response.ok.
Agregar la validación y try/catch. No toques la función de render."

// ❌ Prompt vago
"Mejorá la arquitectura del código."
```

---

## Paso 5 — Checklist de cierre

- [ ] `logic/` no importa nada de `ui/` ni del DOM
- [ ] `fetch` encapsulado en `api/`, no disperso
- [ ] Sin `var` en ningún archivo
- [ ] Sin `innerHTML` con datos del usuario
- [ ] Toda llamada fetch tiene `response.ok` + `try/catch` + estados de UI
- [ ] `localStorage`/`sessionStorage` con `JSON.stringify`/`JSON.parse`
- [ ] Al menos 2 animaciones significativas implementadas
- [ ] Responsive en mobile, tablet y desktop (verificar con DevTools)
- [ ] `npm run dev` sin errores en consola
- [ ] Releer el enunciado: ¿el programa hace exactamente lo que pedía?

**Preguntas de defensa oral para prepararse:**

- ¿Por qué este código está en `logic/` y no en `ui/`?
- ¿Qué pasa si el fetch falla? Mostrámelo.
- ¿Cómo funciona la delegación de eventos en el tablero?
- ¿Por qué usaste `DocumentFragment`?
- ¿Cómo diferenciás los dos niveles de dificultad de la máquina?
- ¿Qué guardarías en `localStorage` vs `sessionStorage` y por qué?

---

## Flujo completo

```
Enunciado
    │
    ▼
Análisis en papel (lápiz)
  → módulos logic/ y ui/, persistencia, API,
    validaciones, animaciones, dificultad, dirección visual
    │
    ▼
Prompt con plantilla
  → enunciado + tu análisis + instrucción de espera
    │
    ▼
Agente ejecuta Paso 1 del skill (análisis)
    │
¿Hay problemas de arquitectura o diseño?
Sí → prompt acotado   No ↓
    │
    ▼
Agente ejecuta Pasos 2-3 del skill (diseño + código)
    │
    ▼
Verificación manual (Paso 4 del skill)
    │
    ▼
npm run dev → sin errores
    │
    ▼
DevTools: mobile / tablet / desktop
    │
    ▼
Releer enunciado → entregar
```
