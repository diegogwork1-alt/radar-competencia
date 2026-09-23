# radar-competencia

Skill de Claude Code de **Flowboost**: radar de competencia vía la **Biblioteca de Anuncios de Meta**.

Toma los competidores de un cliente (los del brief + los que descubre por keywords y búsqueda web), busca sus anuncios **activos**, abre cada creativo para juzgarlo de verdad, separa los **ganadores** (los que llevan semanas corriendo) del ruido, y entrega un informe con ángulos, hooks, tipo de funnel, descartados y oportunidades.

## Empezar aquí

- **[INSTALACION.md](INSTALACION.md)** — cómo montarla en tu ordenador con tus propias cuentas (Claude, Meta, Drive). **Empieza por aquí.**
- **[LEEME.md](LEEME.md)** — condiciones, límites y reglas de la casa antes de usarla.
- **[SKILL.md](SKILL.md)** — las instrucciones que lee Claude (el método completo).
- **[references/anatomia-de-montaje.md](references/anatomia-de-montaje.md)** — cómo convertir un vídeo ganador ajeno en una spec de montaje reutilizable.

## Instalación rápida

```bash
git clone https://github.com/diegogwork1-alt/radar-competencia.git ~/.claude/skills/radar-competencia
```

Luego, en Claude Code: `/radar-competencia`.

Requisito que más falla: **una cuenta publicitaria de Meta activa** en la cuenta conectada; sin ella `ads_library_search` devuelve error. Detalle en [INSTALACION.md](INSTALACION.md).
