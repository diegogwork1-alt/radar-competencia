# Instalación de `radar-competencia` — guía para Carla

> Esta guía es para montar la skill **en tu propio ordenador y con tus propias cuentas**.
> Nada de aquí usa las cuentas de Diego: tú conectas tu Claude, tu Meta y tu Google Drive.
>
> Las instrucciones de trabajo de la skill están en [`SKILL.md`](SKILL.md) y las condiciones de uso en [`LEEME.md`](LEEME.md). Esta guía solo cubre **la instalación y los accesos**.
>
> Fecha de esta guía: 22/09/2026.

---

## 0. Qué vas a instalar

Una skill de Claude Code que escanea la **Biblioteca de Anuncios de Meta** para ver qué anuncios están corriendo los competidores de un cliente, separa los "ganadores" (los que llevan semanas activos) del ruido, abre cada creativo para juzgarlo de verdad, y entrega un informe con ángulos, hooks, funnels y oportunidades.

Son tres ficheros:

```
radar-competencia/
├── SKILL.md                            # las instrucciones que lee Claude
├── LEEME.md                            # condiciones, límites y reglas de la casa
└── references/
    └── anatomia-de-montaje.md          # cómo descomponer el montaje de un vídeo ganador
```

---

## 1. Requisitos previos

| Qué | Para qué | Obligatorio |
|---|---|---|
| **Claude Code** instalado con tu cuenta de Anthropic | es donde vive la skill | ✅ Sí |
| **Cuenta de Meta Business con al menos UNA cuenta publicitaria ACTIVA** | sin eso la herramienta `ads_library_search` devuelve error, no resultados | ✅ Sí |
| **Conector de Meta Ads** conectado en tu Claude | es quien expone `ads_library_search` | ✅ Sí |
| **Navegador integrado o Claude en Chrome** | abrir las fichas de los anuncios, sacar la antigüedad real y el destino | ✅ Sí |
| **Búsqueda web** activada en Claude | descubrir competidores que el brief no nombra | ✅ Sí |
| `ffmpeg` + `faster-whisper` | descargar y transcribir los vídeos de los anuncios | ⚠️ Recomendado |
| `rclone` con remoto `gdrive:` | entregar el informe en el Drive del cliente | ⚠️ Solo si entregas en Drive |
| Scripts internos de Flowboost (`marcar_etapa.py`, etc.) | marcar la etapa y avisar | ❌ Opcional (ver §6) |

> ⚠️ **El punto de la cuenta publicitaria activa es el que más falla.** La Biblioteca de Anuncios es pública en la web, pero la herramienta `ads_library_search` solo responde a quien tiene al menos una cuenta de anuncios activa en la cuenta de Meta conectada. Si no tienes ninguna, pide a Diego acceso a una cuenta publicitaria de la agencia **desde tu propio usuario de Meta** (Business Manager → Usuarios → Añadir persona), no las credenciales de nadie.

---

## 2. Instalar la skill

### Opción A — para todos tus proyectos (recomendada)

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/diegogwork1-alt/radar-competencia.git ~/.claude/skills/radar-competencia
```

### Opción B — solo para un proyecto concreto

```bash
mkdir -p .claude/skills
git clone https://github.com/diegogwork1-alt/radar-competencia.git .claude/skills/radar-competencia
```

### Comprobar que ha quedado bien

```bash
ls ~/.claude/skills/radar-competencia
```

Tienes que ver `SKILL.md`, `LEEME.md` y `references/`. **`SKILL.md` tiene que estar en la raíz de la carpeta**, no dentro de otra subcarpeta: si al clonar te queda `radar-competencia/radar-competencia/SKILL.md`, mueve el contenido un nivel arriba.

Abre Claude Code y escribe `/radar-competencia`. Si aparece en la lista, está instalada.

> Si ya la tenías y quieres actualizarla:
> ```bash
> git -C ~/.claude/skills/radar-competencia pull
> ```

---

## 3. Conectar tu cuenta de Meta

La skill depende de una sola herramienta de Meta: **`ads_library_search`**.

1. Entra en Claude con **tu** cuenta → **Ajustes → Conectores**.
2. Conecta el conector de **Meta Ads** y autoriza con **tu** usuario de Facebook/Meta.
3. Al autorizar, concede acceso al Business Manager donde tengas la cuenta publicitaria activa.

**Cómo comprobar que funciona** — en un chat nuevo de Claude Code, pide:

> «Busca en la Biblioteca de Anuncios de Meta anuncios de "reformas integrales" en España, límite 5.»

- Si devuelve anuncios con `page_name` y `ad_snapshot_url` → listo.
- Si devuelve un error de tipo *"no active ad account"* → te falta el requisito de la cuenta publicitaria (§1).
- Si dice que no tiene la herramienta → el conector no está conectado o no está activo en esa sesión.

Parámetros que usa la skill: `search_terms`, `countries` (ISO-2, `["ES"]` por defecto), `page_ids`, `ad_active_status`, `limit` (máximo 50).

---

## 4. Navegador

El listado de `ads_library_search` **no da la antigüedad real ni el destino del anuncio**. Eso se saca abriendo la ficha en el navegador, y es la mitad del valor del informe. Necesitas una de las dos:

- **Navegador integrado de Claude** (viene con la app de escritorio): no hay que instalar nada.
- **Claude en Chrome** (extensión): se instala desde la Chrome Web Store y se vincula a tu Claude.

Con cualquiera de las dos, la skill abre:

- la biblioteca completa de un anunciante:
  `https://www.facebook.com/ads/library/?active_status=active&ad_type=all&country=ES&view_all_page_id=<PAGE_ID>`
- la ficha de un anuncio concreto:
  `https://www.facebook.com/ads/library/?id=<AD_ID>`

No hace falta iniciar sesión en Facebook para leer la Biblioteca de Anuncios: es pública.

---

## 5. Vídeo: descarga y transcripción (recomendado)

La **REGLA #0** de la skill es no juzgar un anuncio por el texto que devuelve Meta, sino por el creativo. Para los vídeos, eso significa bajarlos y transcribirlos. Sin esto, la skill sigue funcionando pero te quedas coja en los anuncios de vídeo.

**macOS:**
```bash
brew install ffmpeg
pip3 install faster-whisper
```

**Comprobación:**
```bash
ffmpeg -version | head -1
python3 -c "import faster_whisper; print('faster-whisper OK')"
```

El pipeline que usa la skill (lo hace sola, no tienes que ejecutarlo a mano):
1. Saca el `.mp4` de la ficha del anuncio (o, como alternativa, del descargador de EachSpy).
2. `ffmpeg -i <anuncio>.mp4 -vn -ac 1 -ar 16000 a.wav`
3. Transcribe con faster-whisper en `es`.
4. Analiza ángulo, hook, mecanismo y CTA reales con la transcripción + los fotogramas.

Si un vídeo concreto no se puede descargar, la skill lo marca como *"no se pudo descargar"* y sigue. **Nunca se inventa el pitch a partir del copy.**

---

## 6. Entrega: Drive y scripts internos

La skill, tal y como está escrita, entrega en la estructura de Drive de Flowboost y marca la etapa con scripts internos de la agencia. **Esa parte solo funciona si tienes esos accesos.** Mira qué caso es el tuyo:

### Si trabajas dentro del sistema de Flowboost

1. **rclone con el remoto `gdrive:`** apuntando a tu cuenta de Google con acceso al Drive de clientes:
   ```bash
   brew install rclone
   rclone config      # crear un remoto llamado exactamente "gdrive" de tipo Google Drive
   rclone lsd gdrive: # comprobar que lista carpetas
   ```
   Pide a Diego que te dé acceso al Drive **con tu correo**. No uses la cuenta de nadie.

2. **Los scripts internos** (`marcar_etapa.py`, `estado_cliente.py`, `leer_estado.py`, `avisar.py`) viven en el repositorio de Flowboost, en `Documentos-Flowboost/Estandar-carpetas/`. Si no lo tienes clonado, pídeselo a Diego. Las rutas que aparecen en `SKILL.md` apuntan a `~/Desktop/FLOWBOOST-BACKUP-MAC/…`: si tú lo guardas en otro sitio, tendrás que ajustarlas.

### Si NO tienes esos accesos (instalación suelta)

La skill sigue siendo útil entera: el escaneo, el filtrado, el criterio de ganador y el informe no dependen de Drive. Salta los pasos de entrega y dile a Claude al lanzarla:

> «Sin Drive ni scripts de estado: guarda el informe en `~/Desktop/CLIENTES/<cliente>/Radar-competencia-<fecha>.md`.»

Lo que **no** debes hacer es dejar que dé por hechos pasos que no ha podido ejecutar.

---

## 7. Cómo se usa

En un chat de Claude Code:

```
/radar-competencia
```

o directamente:

> «Haz el radar de competencia de <Cliente>.»

Le tienes que dar (o dejar a mano) el **brief del cliente**: de ahí salen la oferta, el comprador, el mercado y los primeros competidores. Aunque el brief los traiga, la skill **siempre** descubre los que faltan por keywords y búsqueda web — eso no es opcional.

Mercado por defecto: **España** (`countries:["ES"]`). Otro país solo si el brief lo dice.

Lo que recibes: resumen, tabla de anuncios ganadores, ficha por competidor, descartados con su motivo, y oportunidades para el cliente. **Todo con Ad ID y URL de Ad Library** — un informe sin enlaces está mal hecho.

---

## 8. Límites que conviene tener claros desde el primer día

- **No hay métricas.** Gasto, resultados y rendimiento de la competencia no son públicos. La antigüedad del anuncio es el único proxy de que algo funciona: ≥ 21 días = ganador fuerte, 7-20 = candidato, < 7 = test. **No se inventa nada.**
- **No hay barrido semanal automático.** El radar corre dentro de una sesión de chat. El resumen semanal recurrente está pendiente del token de Meta en el VPS; no lo prometas a ningún cliente.
- **No se nombra a la competencia en el copy de los anuncios** (regla legal).
- **De la competencia se copia la gramática, no el material.** Ritmo, estructura, ángulo y gestos de montaje sí; metraje, guion, música o chiste concreto, nunca (ver `references/anatomia-de-montaje.md`).
- **"No hay competidor válido" es un resultado, no un fracaso.** Antes que rellenar el informe con anunciantes de otro modelo de negocio, se dice claramente que el nicho está vacío.
- **Todo el texto para clientes, en español de España** (tú/vosotros). Nunca voseo ni español de Latinoamérica.
- **Las fechas salen del reloj del sistema** (`date +%d/%m/%Y`), nunca de memoria.
- **Nunca se teclean contraseñas, claves de API ni tokens**, aunque te los pasen.

---

## 9. Si algo no va

| Síntoma | Causa más probable | Solución |
|---|---|---|
| `/radar-competencia` no aparece | ruta mal o `SKILL.md` anidado | `ls ~/.claude/skills/radar-competencia/SKILL.md` y reinicia Claude Code |
| Error *"no active ad account"* | tu Meta no tiene cuenta publicitaria activa | pide acceso a una cuenta de anuncios con tu usuario (§1) |
| No encuentra `ads_library_search` | conector de Meta Ads sin conectar | Ajustes → Conectores → Meta Ads |
| Devuelve anuncios pero sin antigüedad ni destino | no está usando el navegador | comprueba el navegador integrado o Claude en Chrome (§4) |
| Los vídeos no se transcriben | falta `ffmpeg` o `faster-whisper` | §5 |
| Falla al guardar en Drive | remoto `rclone` no se llama `gdrive` o sin permisos | `rclone lsd gdrive:` (§6) |
| Trae competidores que no pintan nada | no se está aplicando el FILTRO #1 | recuérdale: misma oferta **y** mismo comprador; si no, fuera |
