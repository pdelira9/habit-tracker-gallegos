# AGENTS.md — Habit Tracker

Contrato del repositorio para agentes (arquitecto, diseñador, qa, reviewer, implementer) y para cualquier desarrollador humano.

## Stack (cerrado)

- Next.js 15 con App Router, TypeScript en modo estricto, Tailwind.
- Supabase: Postgres + Auth (email y contraseña).
- Deploy en Vercel.
- shadcn/ui permitido; prohibido Material UI, Chakra y librerías de componentes pesadas equivalentes.
- La spec de producto vive en `spec.md`; el stack no se renegocia en ADRs.

## Convenciones TypeScript

- Mantener `strict`; prohibido `any` salvo justificación explícita en `CONTEXT.md`.
- Tipos explícitos en modelos de dominio, props públicas y respuestas de API.
- Validar entradas de usuario en cliente y en servidor cuando haya persistencia.

## Estructura de carpetas esperada

```
/
├── spec.md, plan.md, AGENTS.md, CONTEXT.md, README.md
├── .claude/agents/       # definiciones de agentes
├── .claude/skills/       # skills reutilizables
├── docs/adr/             # Architecture Decision Records
├── insumos/              # material de referencia del curso (no código de producto)
├── supabase/migrations/  # migraciones Postgres (cuando exista el proyecto Supabase)
└── src/                  # aplicación Next.js (solo después de plan aprobado)
    ├── app/              # rutas App Router
    ├── components/
    └── lib/              # cliente Supabase, helpers
```

## Política de commits

- Un commit = una unidad funcional verificable; mensaje claro con prefijo (`feat:`, `fix:`, `docs:`, `chore:`).
- Prohibido mezclar features no relacionadas ni commits del tipo "implement everything".
- Nada queda sin commitear al cerrar una unidad de trabajo.

## Flujo git (gitflow)

- `main`: rama estable; no se commitea en el día a día del curso.
- `develop`: rama de integración; todo el trabajo mergea aquí.
- Desde `develop`, abrir ramas tipadas: `feat/`, `docs/`, `chore/`, `fix/` (una rama por unidad de trabajo).
- Al cerrar la unidad: merge a `develop`. `release/*` y `hotfix/*` reservadas para deploy.
- Pasos de solo lectura (inventarios, críticas, compuertas) no crean rama ni commit.

## Regla de CONTEXT.md

- Toda edición manual de código (no generada por un agente) se documenta aquí: archivo, cambio y motivo.
- Sin entrada, se asume autoría de agente según `spec.md`, `plan.md` y ADRs en `docs/adr/`.

## Prohibiciones explícitas

- Escribir o modificar código de la app sin `plan.md` aprobado.
- Usar `any` sin justificación registrada en `CONTEXT.md`.
- Material UI, Chakra u otras librerías de componentes pesadas.
- Tests automatizados (unit, integración, e2e) dentro del alcance de este proyecto.
- Trabajo sin versionar en git o commits directos a `main` fuera de lo acordado.

## Decisiones cerradas vs libres

| Cerrado (no renegociar sin cambiar spec) | Libre dentro de spec y ADRs |
|------------------------------------------|-----------------------------|
| Stack Next + Supabase + Vercel + TS strict + Tailwind | Detalle de tablas, RLS, índices |
| Gitflow y commits atómicos | Rutas exactas salvo las citadas en spec |
| Sin tests automatizados ni UI libraries pesadas | Componentes internos y organización bajo `src/` |
| Plan antes de código | Estrategia push (service worker, VAPID) acotada a "recordatorios básicos" |
| Reglas de dominio ya en `spec.md` (archivo, racha, check-in hoy) | Textos de error y detalles de UI no especificados |

Ante conflicto entre un agente y este archivo, prevalece `AGENTS.md` para proceso y `spec.md` para producto.
