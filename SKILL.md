---
name: radar-competencia
description: Radar de competencia de Flowboost vía la Biblioteca de Anuncios de Meta (ads_library_search). Se dispara AL TERMINAR EL BRIEF de un cliente: toma los competidores directos/indirectos del brief, busca sus anuncios ACTIVOS, y arma un informe de ángulos, hooks y anuncios "ganadores" (los que llevan más tiempo corriendo). Guarda el informe en la carpeta interna del cliente en Drive. Usar cuando Diego pida "radar de competencia", "qué anuncios corre la competencia", "espiar la competencia de <cliente>", o automáticamente después del brief.
---

# Radar de competencia (Meta Ad Library)

Escaneás qué anuncios están corriendo los competidores de un cliente usando la **Biblioteca de Anuncios de Meta** (pública). Sale como paso automático **después del brief** dentro de `/funnel`, y también a pedido.

## De dónde salen los competidores
Dos caminos — el paso de **descubrimiento es obligatorio** aunque el brief los tenga (el dueño casi siempre se olvida de la mitad):

**A) Los que el brief nombra** (competencia directa/indirecta) → punto de partida.

**B) Descubrir los que faltan (SIEMPRE hacerlo, no es opcional):**
1. **Por keyword en la propia Ad Library:** derivá 4-8 términos del rubro/oferta/avatar del brief (en el idioma del mercado) y buscalos con `ads_library_search`. **Los anunciantes que aparecen corriendo anuncios en esos términos SON los competidores activos** — esa lista es el hallazgo, no un dato que haga falta pedir.
2. **`WebSearch`** "mejores/top <rubro> en <ciudad/país>", "<rubro> <ciudad>" → sacar nombres de marcas/páginas y volver a pasarlos por `ads_library_search` para ver cuáles pautan.
3. Cruzar A + B, sacar duplicados, y quedarte con los que **efectivamente tienen anuncios activos** (los que no pautan no son competencia publicitaria, aunque existan).

Solo pedile nombres a Diego si el rubro es tan de nicho que ni keywords ni WebSearch devuelven nada — no como primer recurso.

- País/mercado: **por defecto España** (la mayoría de los clientes). Usar otro solo si el brief lo dice (ej. Dipa = Dubái/Emiratos). `countries:["ES"]` salvo excepción.

## Herramienta
`ads_library_search` (MCP de Meta). Parámetros: `search_terms` (nombre de página del competidor o keyword), `countries` (ej. `["ES"]`), `limit`. Devuelve: page_name, ad_creative_link_title, ad_delivery_start_time, ad_snapshot_url. **NO devuelve el destino ni "activo desde" real** (el listado sale ordenado por lo más reciente).

### Para antigüedad real + destino/funnel (probado 04-09): abrir la ficha en el navegador
El listado no alcanza para el "ganador por antigüedad" ni para el funnel. Método real:
1. Para cada competidor top, abrir en el navegador integrado la vista de TODOS sus anuncios: `https://www.facebook.com/ads/library/?active_status=active&ad_type=all&country=ES&view_all_page_id=<page_id>` y leer con `get_page_text`.
2. Ahí cada anuncio muestra **"En circulación desde el <fecha> · Tiempo de actividad total: <X>"** (antigüedad real → winner) + el **texto completo** + el **botón CTA y el destino** (FB.ME/Messenger, formulario nativo, o dominio propio).
3. Con eso sí se arma el ranking de ganadores y el tipo de funnel. No inventar: si un anunciante rota creativos rápido (todos "6 h"), reportarlo como "rota mucho", no como que no tiene ganadores.
> Corre por MCP → dentro de una sesión de chat (como `/funnel`). El escaneo inicial post-brief sale perfecto. El **resumen semanal recurrente** NO se puede hacer solo sin sesión: queda pendiente del token de Meta del VPS (igual que los reportes). No prometer semanal automático hasta que exista ese token.

## Qué da y qué NO
- **Da:** página/anunciante, **hook/titular** de cada anuncio, **fecha de inicio**, formato, link al creativo, cuántos anuncios activos tiene, y el **CTA + destino** del anuncio.
- **DESTINO / TIPO DE FUNNEL (clave — capturarlo siempre):** a dónde manda el anuncio del referente. Clasificar cada uno en:
  - **Landing propia** (link a su web / VSL) — mirar el dominio y, si aporta, abrir la landing para ver si usa video/VSL + formulario.
  - **Formulario nativo de Meta** (Instant Form / "Más información" que abre form dentro de Meta).
  - **WhatsApp / Messenger** (CTA "Enviar mensaje").
  - **Perfil IG / tienda / llamada / otro.**
  Esto dice cómo capta la competencia (mismo enfoque que Flowboost: landing+VSL+form vs. form nativo). Es una de las señales más valiosas del radar.
- **NO da:** gasto, resultados, métricas de rendimiento (no son públicos). No inventarlos. (El destino a veces no está expuesto en la Ad Library; si no está, marcar "destino no visible", no adivinar.)

## Filtro: qué NO cuenta como competidor
Antes de analizar, descartar (no son competencia de captación de leads):
- Anuncios cuyo destino es **perfil de IG/Facebook** (buscan seguidores/branding, no leads).
- Anuncios de **solo interacción/alcance** sin CTA de captación (me gusta, ver más del perfil).
- Marcas del rubro que **no pautan** (no tienen anuncios activos) — existen pero no compiten en ads.
- Reventa/marketplace/afiliados que no son el servicio real.
Si tras filtrar un "competidor" se queda sin anuncios de captación, marcarlo como *"presente pero no compite en ads"* y no ocupa el análisis.

## REGLA #0 — JUZGAR POR EL CREATIVO, NO POR EL COPY DE META (lo más importante)
El texto/titular que devuelve la Ad Library **NO es el anuncio** — es solo el pie. La oferta real, el ángulo y el hook están en el **VIDEO (lo que se dice y se muestra)** o en el **ESTÁTICO (la imagen)**. Clasificar relevancia o sacar ángulos **leyendo solo el copy es un error** (un copy genérico puede tapar un anuncio que sí es competencia, y al revés).

**Obligatorio antes de clasificar o analizar un anuncio: ABRIR el creativo y verlo.**
- Abrir el `ad_snapshot_url` en el navegador integrado (`navigate` + `computer screenshot`).
- **Estático:** mirar la imagen (screenshot) → leer texto en imagen, qué producto/servicio muestra, oferta, prueba social.
- **Video:** el sistema **descarga el video solo y lo transcribe** — Diego NO hace nada, no aprueba ni responde. Pipeline automático (ver abajo). Con la transcripción + los frames se saca el ángulo/oferta real.

### Descarga + transcripción de video AUTOMÁTICA (sin intervención humana)
Para cada video de anuncio que haya que analizar, la skill lo baja y transcribe sola:
1. **Conseguir el MP4 del anuncio (por su ID), en este orden:**
   a. **Directo de la página del anuncio (preferido):** abrir `https://www.facebook.com/ads/library/?id=<AD_ID>` en el navegador integrado y extraer la URL del `.mp4` (fbcdn) — con `read_network_requests` filtrando `.mp4`, o `javascript_tool` leyendo el `src` del `<video>`. Descargar con `curl -L -o <AD_ID>.mp4 "<url>"`.
   b. **Fallback — EachSpy downloader:** navegar a `https://www.eachspy.com/es/tools/facebook-ad-library-downloader/`, pegar el `<AD_ID>` (o el ad_snapshot_url) en su campo, y descargar el MP4 que genera. Usar esto solo si (a) no devuelve el mp4.
2. **Transcribir** el audio con **faster-whisper** (mismo motor que las skills de edición; `ffmpeg -i <mp4> -vn -ac 1 -ar 16000 a.wav` → whisper `es`). Guardar el texto.
3. **Analizar** con la transcripción + frames: ahí SÍ está la oferta, el hook (primeros segundos), el mecanismo y el CTA reales.
4. Todo esto corre solo dentro de la sesión de `/funnel`; **Diego no baja, no aprueba, no responde nada.**
- Honestidad: no "escucho" el audio directo — por eso el pipeline descarga+transcribe. Si un video puntual no se puede bajar por ningún método, marcarlo "no se pudo descargar" y seguir; nunca inventar el pitch desde el copy.
- Recién CON el creativo visto, decidir si es competencia (Filtro #1) y extraer el ángulo/hook real.
- En el informe, el ángulo descrito debe salir de lo que se VE/DICE en el creativo, citando el `ad_snapshot_url`. Nunca describir un anuncio que no abriste.

## FILTRO #1 — RELEVANCIA DE OFERTA (manda sobre todo lo demás)
Antes que la longevidad y el volumen, un anunciante entra **solo si su OFERTA es la misma o casi idéntica a la del cliente** (mismo servicio + mismo tipo de comprador). Un anunciante inmenso y viejísimo con OTRA oferta **NO es referencia** — sus creativos no sirven para copiar. Preguntarse siempre: *"¿este anuncio podría ser de mi cliente cambiando la marca?"* Si no, fuera.

**Excluir SIEMPRE (salvo que sea EXACTAMENTE el negocio del cliente):**
- **Infoproductos / formación / mentorías / cursos / eventos-webinar / "masterclass gratis"** — venden conocimiento, no el servicio. Se reconocen por el funnel "evento/clase/reto gratis". (Ej. Solventia: "Sam/Club Rentable" queda fuera.)
- **Otro modelo de negocio** aunque sea del mismo rubro: venta de propiedad, marketplaces, agencias inmobiliarias, sourcing puro, etc. cuando el cliente hace gestión/inversión llave en mano. (Ej. Solventia: "The Last Property" queda fuera — otra oferta.)
- Si de un anunciante **solo 1 anuncio de muchos** se parece de casualidad → **no es parámetro**, no lo metas.

El objetivo es traer **anunciantes cuya CARTERA de anuncios sea comparable** a la del cliente, para robar ángulos/hooks/funnels que aplican. Pocos competidores MUY relevantes > muchos poco relevantes.

## BARRA DE CALIDAD — qué merece entrar al informe (feedback duro de Diego)
El informe es de **benchmark de ganadores**, NO una lista de todo el que pauta. **NO incluir** como competidor a analizar:
- Anunciantes con **1-2 anuncios** y **pocos días** activos (test recién lanzado ≠ competidor).
- Nadie **sin al menos un anuncio corriendo hace ≥ 2-3 semanas** (si no hay longevidad, no hay ganador que copiar).
- Baja señal de inversión (sin volumen ni variantes).
Un competidor entra al informe **solo si** (además de pasar el FILTRO #1 de relevancia) tiene: **longevidad del MISMO creativo** (un anuncio concreto activo hace semanas/meses — NO que la página exista hace tiempo) **y/o volumen serio** de variantes. **OJO:** si TODOS sus anuncios son nuevos (todos "hace pocos días") NO es un ganador probado, aunque la marca sea vieja — es que están testeando o relanzaron; no sirve como referencia de "esto ya ganó". Si un anunciante no llega a la barra, va a "descartados", no al análisis.

**"Mismo modelo" es ESTRICTO = misma oferta AL MISMO comprador.** No alcanza con "mismo rubro". Ej. Solventia = *inversión inmobiliaria gestionada / renta por habitaciones para INVERSORES que ponen capital*. NO son su competidor: rent-to-rent/garantía a propietarios (otro comprador), venta de propiedad, formación/infoproducto, sourcing puro. Si dudás si "se parece", asumí que NO hasta probar que la oferta y el comprador son los mismos.

**Si NO hay ningún anunciante que pase relevancia + barra (caso real y frecuente):**
1. **Decirlo claro y sin vueltas:** "No hay competidor directo con la misma oferta y recorrido probado en Meta ES ahora." Eso ES un resultado válido y valioso (nicho vacío = oportunidad), no un fracaso.
2. **Ampliar SOLO al MISMO modelo/comprador en otros mercados hispanos** (México, Argentina, Colombia, etc.) para robar ángulos de alguien que lleve meses ganando con la MISMA oferta. Si aparece, es referencia creativa (no competidor local).
3. Si tampoco ahí hay nada del mismo modelo con recorrido → cerrar con "sin benchmark directo" + listar los descartados y por qué. **NUNCA meter un anunciante de otro modelo, de volumen basura, o todo-nuevo para llenar el informe.** Un informe honesto de "no hay competidor válido" es mejor que uno con referencias que no sirven.

## Cómo encontrar GANADORES de verdad (no lo más reciente)
`ads_library_search` devuelve **lo más nuevo primero** → si te quedás con eso, solo ves tests recientes (basura). Para los ganadores:
1. Corré **varias keywords** del nicho y juntá TODOS los `page_name` que aparecen. **Los que se repiten mucho = volumen = candidatos a ganador.**
2. Para cada candidato, abrí su biblioteca completa en el navegador (`view_all_page_id`) y ordená/buscá el anuncio con **mayor "Tiempo de actividad total"**. Ese es su ganador; anotá desde cuándo corre.
3. Priorizá el informe por **antigüedad del anuncio más viejo activo** de cada anunciante (semanas/meses), no por lo que salió arriba en la búsqueda.
4. Si hace falta, sumá keywords en otras ciudades/países hispanos del mismo modelo para encontrar ganadores de larga duración a copiar.

## Criterio de "ganador" (winner) — concreto, no a ojo
Un anuncio se marca ganador combinando señales objetivas de la Ad Library:
- **Antigüedad (principal):** activo **≥ 21 días** = ganador fuerte · **7-20 días** = candidato · **< 7 días** = test reciente (no concluir aún). Nadie sostiene semanas un anuncio que no rinde.
- **El anunciante duplica la apuesta:** el mismo ángulo/creativo corriendo en **varias variantes o placements** = le están metiendo presupuesto.
- **Volumen de anuncios activos** del anunciante (muchos a la vez = invierte en serio).
- **Reincidencia del ángulo** entre varios competidores (si 3 usan el mismo gancho, ese ángulo funciona en el nicho).
Marcar el nivel (fuerte/candidato/test) en el informe, con la fecha de inicio como evidencia. No inventar rendimiento: la antigüedad es el proxy, no una métrica de resultados.

## Proceso
1. Reunir competidores (brief) + descubrir los que faltan (paso B) + país (ES por defecto).
2. Por cada competidor/keyword: `ads_library_search`. Si un nombre no devuelve nada, probar variantes (con/sin S.L., marca vs dominio) antes de darlo por vacío.
3. **Filtrar** los que no compiten (ver arriba).
4. Aplicar el **criterio de ganador** y ordenar.
5. Capturar por anuncio: hook, fecha inicio, formato, **destino/funnel**, oferta que asoma en el copy, prueba social usada.
6. Detectar **patrones** del nicho: ángulos repetidos, hooks recurrentes, formato dominante, **tipo de funnel dominante**, cada cuánto renuevan creativos, ofertas/garantías típicas.
7. Armar el informe (abajo).

## Informe (formato) — TODO LINKEADO (obligatorio)
**Regla dura (feedback de Diego):** ningún anuncio ni competidor va sin sus links. CADA anuncio lleva su **ID** + su **URL de Ad Library** (`https://www.facebook.com/ads/library/?id=<AD_ID>`). CADA competidor lleva su **URL de página en Ad Library** (`https://www.facebook.com/ads/library/?active_status=active&ad_type=all&country=ES&view_all_page_id=<PAGE_ID>`) + su **web/landing de destino** + su **web pública** si se conoce. Un informe sin IDs y URLs está mal hecho — no entregarlo así.

```
# Radar de competencia — <Cliente> · <fecha> · <país>

## Resumen (3-5 bullets)
- Ángulos que más repiten · formatos dominantes · qué ofrecen.
- **Tipo de funnel dominante** (landing propia vs form nativo Meta vs WhatsApp/Messenger vs perfil IG).
- Ofertas/garantías típicas · ritmo de renovación · quién invierte más (volumen).

## Anuncios ganadores
| Competidor | Hook/titular | Desde (días) | Nivel | Formato | Destino/funnel (URL) | Oferta | Ad ID | Ad Library URL |
(el destino: si es landing/web, poner la URL real; si es Messenger/form nativo/IG, decirlo)

## Por competidor
### <Competidor> — N anuncios activos · funnel: <...>
- **Ad Library (todos):** https://www.facebook.com/ads/library/?active_status=active&ad_type=all&country=ES&view_all_page_id=<PAGE_ID>
- **Web/landing:** <URL>  ·  **Página FB/IG:** <handle/URL si aplica>
- Ángulos / hooks / oferta-garantía / prueba social / ritmo de renovación
- Anuncios clave: ID `<AD_ID>` → https://www.facebook.com/ads/library/?id=<AD_ID> (uno por línea)

## Descartados (con motivo)
- <marca> — <por qué no compite: perfil IG / venta propiedad / no pauta> + link Ad Library

## Situación actual del cliente (si ya pauta)
- Ad Library del cliente + nº anuncios + desde + destino, para contraste.

## Oportunidades para <Cliente>
- Ángulos libres · dónde diferenciarse · qué copiar del ganador · lectura de funnel.
```

## Entrega
- Guardar en Drive (vía rclone `gdrive:`) en la subcarpeta **Competencia** de Ads del cliente: **`i_<Cliente>/c_<Cliente>/2. Ads/Competencia/Radar-competencia-<fecha>.md`** (crear la subcarpeta `Competencia` si no existe). Estructura real: el trabajo vive en `i_<Cliente>/c_<Cliente>/<Nº. Carpeta>/`.
- Actualizar ESTADO.md: `python3 ~/Desktop/FLOWBOOST-BACKUP-MAC/Documentos-Flowboost/Estandar-carpetas/estado_cliente.py "<Cliente>" set "Radar de competencia" hecho "<fecha>"` — **nombre de etapa LITERAL** (lista canónica en `estado_cliente.py`).
- Avisar a Diego: si hay sesión de chat abierta, en el resumen final de la sesión; si no, `python3 ~/Desktop/FLOWBOOST-BACKUP-MAC/Documentos-Flowboost/Estandar-carpetas/avisar.py --cliente "<C>" --nivel info --asunto "Radar de competencia listo" --cuerpo "<3 oportunidades>"`. Si faltaron nombres de competidores, pedírselos ahí mismo (`--accion`).

## Qué NO hace
- No inventa métricas ni gasto de la competencia (no son públicos).
- No promete el barrido semanal automático hasta que esté el token de Meta en el VPS.

## Al terminar: marcar la etapa (obligatorio — 07-09-2026)
```bash
python3 ~/Desktop/FLOWBOOST-BACKUP-MAC/Documentos-Flowboost/Estandar-carpetas/marcar_etapa.py "<Cliente>" "Radar de competencia" --nota "<una línea>"
```
Marca **los dos sitios a la vez**: el `ESTADO.md` (local y Drive) y la hoja
**«Estado de cuenta — \<Cliente\>»** del Drive, que es la que mira Diego. Antes había que
acordarse de las dos cosas y la del Drive se quedaba siempre atrás.

**Se marca al CERRAR la etapa, no al empezarla**, y solo si de verdad está terminada. Si quedó a
medias: `--estado "En curso"`. Si está parada: `--estado Bloqueado --nota "<por qué y a quién espera>"`
— un bloqueo sin motivo apuntado no sirve de nada.

### Y NO DEVUELVAS EL CONTROL: seguí con la siguiente
Marcar **no es terminar**. En el MISMO turno, sin preguntar y sin resumen intermedio:
1. Mirá qué toca ahora: `python3 …/Estandar-carpetas/leer_estado.py "<Cliente>" --cliente-raiz "gdrive:i_X/c_X"`.
2. **Arrancá la siguiente etapa del agente que salga ahí** — de los DOS carriles si hay una en cada uno.
3. Si lo que sigue es de Diego, de Pablo o del cliente, **no la toques**: salta a la siguiente que sí
   sea tuya. Esperar de brazos cruzados es el fallo, no la solución.

Solo se para en las paradas humanas de `/funnel`. **Nunca termines el turno con una etapa tuya
ejecutable pendiente.**
