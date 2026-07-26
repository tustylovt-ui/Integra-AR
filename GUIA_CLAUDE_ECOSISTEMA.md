# Guía de continuidad para Claude — Ecosistema IntegraAR

> Este archivo NO es parte del sitio institucional (eso es `ESTADO.md` en este mismo repo).
> Es una nota de continuidad para mí mismo (Claude), escrita al final de una sesión muy larga
> de trabajo (Junio 2026) antes de que el usuario borre esa conversación y arranque una nueva.
> Si estás leyendo esto al empezar una sesión nueva: léelo entero antes de tocar código.

---

## Qué es esto

**IntegraAR** es un ecosistema de 3+ aplicaciones SaaS para pymes/profesionales argentinos,
todas con integración real a ARCA/AFIP y Mercado Pago:

| Repo | Carpeta | Stack | Qué hace |
|---|---|---|---|
| **FacturAR** | `FACTURAR/FacturAR` | Vite + React + Supabase | Facturación electrónica con CAE real |
| **FinanciAR** | `FINANCIAR/FinanciAR` | Next.js + TS + Supabase | Gestión de créditos particulares y cobranzas |
| **Tienda-AR** | `TIENDAAR/Tienda-AR` | Next.js + TS, sin DB propia | Tienda online que vende productos de FacturAR/FinanciAR |
| Integra-AR (acá) | `INTEGRAAR/Integra-AR` | HTML estático | Sitio institucional/landing |

Cada uno tiene su propio **`ESTADO [NOMBRE].md`** en la raíz del repo — son la fuente de
verdad técnica, mantenidos al día con cada feature. **Léelos antes de tocar nada.** Si el
usuario pide algo y no tenés el doc en contexto, abrilo primero.

---

## Lecciones operativas (caras, aprendidas a los golpes — no las repitas)

### 1. El límite de 12 funciones serverless de Vercel Hobby es REAL y duro
No es solo documentación — **se reprodujo un deploy fallido real** al agregar una función
13ª a FacturAR. El build de Vite/Next sale bien; el error pasa después, en el empaquetado de
funciones ("Deploying outputs..."), y no aparece como error de build.

**Antes de crear un archivo nuevo en `/api`**, contá cuántos hay. Si está cerca del límite
(o ya en 12), **extendé un endpoint existente** en vez de sumar uno:
- FacturAR: `arca-herramientas.js` es el endpoint multipropósito — agregale un `servicio`
  nuevo (`{ servicio: 'nombre', ...params }`, dispatch con `if (servicio === ...)`).
- FinanciAR: agregale un campo discriminador (`producto`, `addon`, `accion`) al body de una
  ruta existente del mismo dominio (ej. todo lo de MP vive en `mp/*`).
- TiendaAR es una app nueva con presupuesto propio sin usar — hoy solo ocupa 3 de 12.

### 2. `Filesystem:edit_file` falla en silencio con comentarios de guiones largos
Anchors tipo `// ── Título ─────────────────────` fallan por conteo exacto de caracteres
Unicode (los guiones `─` son fáciles de copiar mal). Cuando un edit con varios `edits[]` en
la misma llamada falla, **toda la llamada se revierte, no solo el que falló** — no asumas que
los demás se aplicaron. Si un edit falla:
- Volvé a leer el archivo (no confíes en lo que tenías en contexto)
- Usá como ancla una línea de código plano sin caracteres raros, no el comentario decorativo
- Hacé un edit a la vez si hay dudas, en vez de un batch grande

### 3. `create_file` / `str_replace` / `view` (las del bloque "computer use") escriben en MI computadora, no en la del usuario
Esto pasó dos veces esta sesión. Para cualquier archivo que tenga que vivir en el proyecto
del usuario, usar **siempre** las herramientas `Filesystem:*` (`write_file`, `edit_file`,
`create_directory`, `move_file`, `read_text_file`). Las herramientas sin el prefijo
`Filesystem:` son el sandbox Linux propio de Claude — sirven para scratch work, generar
PDFs/Excel para descargar, etc., pero nunca para tocar el código real de estos repos.

### 4. `$$` (dollar-quoting de Postgres) se puede comer un `$` al pasar por `edit_file`
Si necesitás documentar una función SQL con `$$ ... $$` dentro de un `.md`, mejor referenciá
el archivo `.sql` real en vez de inlinear el código — y si lo inlineás, releé el resultado
para confirmar que los `$$` sobrevivieron.

### 5. Vistas Supabase pensadas para el rol `anon` necesitan `security_invoker = false`
Es lo opuesto a la convención que usan el resto de las vistas en estos proyectos (pensadas
para que un usuario autenticado solo vea lo suyo, con `security_invoker = on`). Con `= true`
en una vista que debe ser pública, hereda el RLS de la tabla base y el rol `anon` queda con
cero filas **sin ningún error visible** — el bug más insidioso de toda la sesión
(`v_tienda_publica`, ya corregido en FacturAR y FinanciAR).

### 6. `upsert(..., {onConflict: 'col1,col2'})` necesita una constraint UNIQUE real en esas columnas
Si no existe, Postgres tira error en cuanto se usa ("no unique or exclusion constraint
matching..."), no antes. Al diseñar una tabla nueva tipo "una fila por X", agregar el
`UNIQUE` en la misma migración, no después.

---

## Patrones de arquitectura ya establecidos (seguirlos, no reinventar)

- **Conectores de origen** (`Tienda-AR/src/lib/origenes/`): cada app de origen (FacturAR,
  FinanciAR, y a futuro AgendAR/GestionAR) implementa la interfaz `OrigenConector` que
  traduce sus tablas reales a tipos normalizados (`ProductoNormalizado`,
  `TiendaConfigNormalizada`). El registro (`registry.ts`) las junta por prefijo de URL
  (`f`, `c`, ...). Sumar una app nueva a la tienda = escribir su conector, nada más.
- **Plantillas de TiendaAR** (`components/tienda/plantillas/`): mismo patrón de registro,
  una carpeta por plantilla con `Header.tsx`/`ProductGrid.tsx`/`ProductCard.tsx`.
- **Secreto compartido entre los 3 repos:** `TIENDA_MP_TOKEN_ENCRYPTION_KEY` (AES-256-GCM)
  tiene que tener el **mismo valor exacto** en FacturAR, FinanciAR y TiendaAR — uno cifra
  (FacturAR/FinanciAR al guardar la cuenta de MP del comerciante), el otro descifra
  (TiendaAR al cobrar). Si alguna vez se rota, hay que cambiarla en los tres a la vez.
- **Auth de rutas API en FacturAR (Vite, sin sesión server-side nativa):** el cliente manda
  `Authorization: Bearer {access_token}`; el endpoint arma un cliente Supabase con anon key
  + ese header para validar `auth.getUser()`, y de ahí resuelve `usuarios.id`.

---

## Estado de alto nivel al cierre de esta sesión (Junio 2026)

Para el detalle real, completo y actualizado: **siempre los `ESTADO *.md` de cada repo**,
esto es solo un resumen de qué tan parejos están entre sí.

- **FinanciAR** es el más completo: Tienda Virtual con addon pago ($30.000/mes, descuentos
  por período), paywall real, control manual desde Super Admin (activar/bloquear), las 3
  plantillas habilitadas, compresión de imágenes, cobro con MP propio del comerciante.
- **FacturAR** tiene la vidriera + cobros (MP propio) + las 3 plantillas + compresión de
  imágenes, pero **todavía NO tiene el paywall del addon** — el switch "Tienda activa" sigue
  siendo gratis y manual. Es la asimetría más grande pendiente entre los dos.
- **TiendaAR** tiene el checkout funcionando de punta a punta para los dos orígenes. Falta:
  carrito multi-producto (hoy es "comprar ahora" de a un item), una pantalla para que el
  comerciante vea sus pedidos (hoy `tienda_pedidos` se llena pero nadie la lista), y avisos
  de pedido nuevo.

---

## Identificadores útiles (Vercel)

```
Team ID: team_i0GhK9dzwQtB6TJ8GBNdvDEV
FacturAR project ID: prj_3Zqeg4DMmymckjQSVdViSA8xM2W7  (factur-ar)
TiendaAR project ID: prj_wvv3jdBUmx5PFKoFHkszP199WDQU  (tienda-ar)
```
(El de FinanciAR no quedó anotado — buscarlo con `Vercel:list_projects` si hace falta.)

---

*Escrito por Claude al cierre de la sesión del 21/06/2026, a pedido del usuario, antes de
borrar esa conversación. Si encontrás algo desactualizado acá, corregilo — esto también es
un documento vivo, no un mensaje congelado en el tiempo.*
