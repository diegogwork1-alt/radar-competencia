# LEEME — `radar-competencia`

> Paquete **onboarding**. Esto es lo que hay que tener en cuenta **antes** de usar la skill.
> Las instrucciones de trabajo están en `SKILL.md`; esto son las condiciones y los límites.

## Qué hace

Radar de competencia de Flowboost vía la Biblioteca de Anuncios de Meta (ads_library_search). Se dispara AL TERMINAR EL BRIEF de un cliente: toma los competidores directos/indirectos del brief, busca sus anuncios ACTIVOS, y arma un informe de ángulos, hooks y anuncios "ganadores" (los que llevan más tiempo corriendo).

## Antes de empezar necesitás

- Los competidores del brief.

## Lo que NO se puede hacer

- ⛔ Nombrar competidores en el copy de los anuncios (regla legal).

## Ojo con esto

- Se dispara al terminar el brief. Los 'ganadores' son los anuncios que llevan más tiempo corriendo.

## Accesos que toca

Google Drive del cliente (solo lectura salvo entregables), Biblioteca de Anuncios de Meta (pública, solo lectura), VPS por SSH.

## Reglas de la casa (valen para todas las skills)

- **Todo el texto para clientes en español de España** (tú/vosotros). Nunca voseo ni LATAM.
- **No se inventa nada**: cifras, testimonios, fechas, garantías o casos. Lo que falte se marca `[FALTA]` y se pide.
- **Las fechas salen del reloj del sistema** (`date +%d/%m/%Y`), nunca de memoria.
- **Los ficheros de un cliente van a `~/Desktop/CLIENTES/<cliente>/`**, nunca sueltos en Descargas.
- **El Drive del cliente es de SOLO LECTURA**, salvo los entregables en su subcarpeta correcta. No se mueve, borra ni renombra nada.
- **Nunca se sube un `.md` crudo al Drive del cliente**: se convierte a Google Doc.
- **Nunca se teclean contraseñas, claves de API ni tokens**, aunque te los den. Los pone Diego.
- **Para avisar a Diego se usa `avisar.py`** (`--nivel urgente|aviso|info`), no un mensaje suelto que nadie lee.

---

*Generado el 10-09-2026 desde el sistema de Flowboost. Se regenera con `gen_leeme.py`; no editar a mano.*
