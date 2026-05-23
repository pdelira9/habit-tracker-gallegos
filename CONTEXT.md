# CONTEXT.md — Habit Tracker

Registro de ediciones manuales (no generadas por un agente según spec/plan/ADRs).

## 2026-05-22 — Bloque 1.3 (cierre de bloqueadores en spec)

- **Archivo:** `spec.md`
- **Qué:** Se añadieron secciones "Decisiones de producto" y criterios de aceptación verificables; se precisó auth email/password, campos del hábito, archivado vs borrado, check-in solo hoy con desmarcar, reglas de racha diaria/semanal, push por hábito y aislamiento entre usuarios.
- **Por qué:** El prompt 1.3 exige cerrar bloqueadores antes de invocar al agente arquitecto; la spec v0 original dejaba esas decisiones abiertas (crítica 1.2).

## 2026-05-22 — Spec del agente arquitecto (post 3.2)

- **Archivos:** `.claude/agents/arquitecto.spec.md`, `.claude/agents/arquitecto.md`
- **Qué:** Cierre de preguntas 16–18: no inventar features; no `plan.md` ni `src/`; sí puede tocar el repo si el humano lo pide explícitamente (por defecto solo chat). Resto de decisiones del cuestionario 3.2 integradas en la spec.
- **Por qué:** Pasar de spec provisional a artefacto invocable (3.3) sin ambigüedad en no-goals.

## 2026-05-22 — Integración con GitHub

- **Archivos:** merge con `origin/main`; `insumos/spec-github-v1.1.md` conserva la spec v1.1 que estaba en GitHub.
- **Qué:** Se mantuvo `spec.md` (v0 + bloqueadores cerrados), `AGENTS.md` completo, `.gitignore` ampliado y README con nombre del repo.
- **Por qué:** Unificar historial local del curso con [habit-tracker-gallegos](https://github.com/pdelira9/habit-tracker-gallegos) sin perder trabajo de ningún lado.

## 2026-05-22 — Spec v0 en repo

- **Archivo:** `spec.md`, `insumos/spec-repo-detallada.md`
- **Qué:** Reemplazo de la spec detallada del repo por la Spec v0 del autor; respaldo de la versión larga en `insumos/spec-repo-detallada.md`.
- **Por qué:** Alinear el producto con la spec del estudiante antes del Bloque 1.
