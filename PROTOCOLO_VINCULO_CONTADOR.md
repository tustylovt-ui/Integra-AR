# Protocolo de Vínculo Contador ↔ Cliente (cross-app) — v1.0.0

> Documento de especificación técnica para el ecosistema IntegraAR. Define cómo un
> **contador** (profesional que gestiona cuentas de terceros, hoy modelado en FacturAR)
> puede vincularse a un **cliente** que usa cualquier app del ecosistema — FacturAR,
> AgendAR, FinanciAR, y las que se sumen — sin importar si el cliente ya tiene cuenta en
> esa app o la crea en el momento, y sin importar que cada app viva en un proyecto de
> Supabase distinto (auth y base de datos 100% separados entre apps).
>
> Ver también: [`GUIA_CLAUDE_ECOSISTEMA.md`](./GUIA_CLAUDE_ECOSISTEMA.md) (patrones
> generales del ecosistema) y el `ESTADO_[APP].md` de cada repo (estado real,
> actualizado). Este documento es el contrato — los detalles de implementación de cada
> app viven en su propio repo.

**Estado:** diseño aprobado, implementación pendiente (ver `Rollout` al final).
**Repos involucrados:** `FACTURAR/FacturAR`, `AGENDAR/AgendAR`, `FINANCIAR/FinanciAR`, y
cualquier app nueva del ecosistema.

---

## 1. Por qué no se puede hacer "fácil"

Verificado en código (no supuesto): cada app tiene su propio proyecto de Supabase,
con su propio `auth.users` y su propia base:

| App | Supabase project ref | Auth de usuario en sus API |
|-----|----------------------|------------------------------|
| FacturAR | `tehgzhnpcicuucwdefjm` | JWT Bearer (`Authorization: Bearer <access_token>`) |
| AgendAR | `fkbspjyoawjwuthfhgvs` | Supabase Auth vía `@supabase/ssr` (cookies + Bearer en algunos casos) |
| FinanciAR | `izprfefxvwsjtnmpempi` | 100% cookies de sesión SSR (`@supabase/ssr`), sin Bearer para usuarios |

No hay ningún FK posible entre proyectos, no hay sesión compartida, y FinanciAR ni
siquiera tiene el mismo mecanismo de auth que FacturAR. **Cualquier diseño que asuma
"identidad compartida" o "SSO real" no es viable sin una reescritura mayor de auth en
las 3 apps** — fuera de alcance y de presupuesto de este protocolo.

**Principio de diseño:** el vínculo viaja entre apps como un **código opaco**, nunca
como una identidad autenticada. Cada app canjea ese código **en su propio dominio, con
su propia sesión** — nunca se comparte una cookie ni un JWT entre apps. Esto es lo que
hace que el protocolo funcione incluso con FinanciAR (cookies same-site) sin tocar su
auth.

---

## 2. Roles del protocolo

- **Emisor**: la app donde vive el "módulo contador" que origina la invitación. Hoy es
  FacturAR (`PanelContador.jsx`). En el futuro, cualquier app podría ser emisora si
  desarrolla su propio concepto de "gestor de cuentas de terceros" — el contrato de
  abajo es simétrico, no está atado a FacturAR.
- **Receptor**: cualquier app cuyo usuario puede ser invitado a vincularse. Todas las
  apps del ecosistema deben implementar el lado receptor (tabla + 2 endpoints + página),
  incluida la propia app emisora (FacturAR ya lo tiene, adaptado a este contrato).

Una misma app puede ser emisora y receptora a la vez.

---

## 3. Flujo de punta a punta

```
Contador (en Panel Contador de FacturAR)
  │
  │ 1. Tilda apps destino: [x] FacturAR  [x] AgendAR  [ ] FinanciAR
  │ 2. Genera UN código (codigo_invitacion), igual al que ya usa FacturAR hoy
  ▼
FacturAR (emisor)
  │ 3. INSERT propio en vinculos_contador (estado=PENDIENTE) — sin cambios, ya existe
  │ 4. Por cada app tildada además de sí misma:
  │      POST https://{app}/api/.../vinculo_externo_crear
  │      header: X-IntegraAR-Secret: <secreto compartido>
  │      body:  { codigo, contador_nombre, contador_email, contador_app_origen:'facturar' }
  ▼
AgendAR (receptor)
  │ 5. INSERT en su propia tabla vinculos_contador_externo (estado=PENDIENTE)
  │    — SIN FK a FacturAR, todo denormalizado (contador_nombre/email como texto)
  ▼
Contador comparte: https://factur-ar.vercel.app/acceso/{codigo}
  │
  ▼
Cliente abre el link (en FacturAR, la app emisora que hospeda el link único)
  │ 6. FacturAR valida el código contra SU tabla (ya lo hace hoy)
  │ 7. Además pregunta a cada app tildada (llamada pública, sin PII):
  │      GET https://{app}/api/.../vinculo_externo_verificar?codigo=...
  │      → { valido: true, appNombre: 'AgendAR' }
  │ 8. Si hay más de una app válida, muestra selector:
  │      "Activá tu cuenta en:"  [FacturAR]  [AgendAR]
  │
  ├─ Click "FacturAR" → sigue el flujo YA EXISTENTE de AccesoContribuyente.jsx
  │  (cuenta nueva vía signUp+emailRedirectTo, o cuenta logueada → vincular)
  │
  └─ Click "AgendAR" → window.location = https://agend-ar.vercel.app/acceso/{codigo}
       │ Corre en el dominio y la sesión de AgendAR — sin compartir cookies ni JWT
       │ AgendAR implementa su PROPIA página /acceso/:codigo (mismo patrón que
       │ FacturAR: cuenta nueva vs cuenta ya logueada) contra SU auth.users
       ▼
     AgendAR activa su propio vinculos_contador_externo (estado=ACTIVO, usuario_id=...)
```

Nota clave: el paso 8 (elegir app) no requiere que el cliente esté logueado en ninguna
de las dos — es solo un router de links, cada click navega al dominio real de esa app,
que resuelve login/signup con su propia sesión de siempre. **Implementado y verificado
en producción** (25-26/07/2026) tras una prueba real del usuario que reveló que faltaba
justo este paso — antes `/acceso/:codigo` de FacturAR no ofrecía ninguna opción, solo
el flujo nativo de FacturAR.

En cada app, "cuenta nueva vs. cuenta ya logueada" en realidad son 3 casos, no 2: sin
sesión, la página todavía tiene que decidir **login vs. registro** — y esa decisión la
informa `cuentaExistente` (§5.3), calculado sobre el `email_invitado` que viajó en el
fan-out. Sin este dato, un cliente con cuenta previa se topaba con un error recién al
intentar registrarse de nuevo.

---

## 4. Contrato de datos (por app receptora)

Tabla nueva, misma forma en todas las apps (nombres de columna literales, no solo el
concepto — para que el código sea copiable entre repos):

```sql
create table vinculos_contador_externo (
  id                    uuid primary key default gen_random_uuid(),
  codigo_invitacion     text not null,              -- mismo código que generó el emisor
  contador_app_origen   text not null,               -- 'facturar' | 'agendar' | 'financiar' | ...
  contador_nombre       text,                        -- denormalizado, NO es FK — no existe FK cruzada posible
  contador_email        text,                        -- idem
  email_invitado        text,                        -- email del CLIENTE cargado por el contador al invitar
  nombre_invitado       text,                        -- idem, nombre. Sirven para: (a) prellenar el form de
                                                       -- /acceso/:codigo, (b) decidir login vs. registro por
                                                       -- defecto (ver §5.3 — "cuentaExistente")
  estado                text not null default 'PENDIENTE'
                          check (estado in ('PENDIENTE','ACTIVO','REVOCADO','EXPIRADO')),
  usuario_id            uuid references usuarios(id), -- o la tabla de usuario interna de cada app; null hasta ACTIVO
  permisos              jsonb default '{}'::jsonb,    -- flags finos, específicos de cada app (ej. puede_ver_turnos)
  creado_at             timestamptz not null default now(),
  vinculado_at          timestamptz,
  expira_at             timestamptz not null,         -- creado_at + 7 días, mismo criterio que clinic_invitations de AgendAR

  unique (codigo_invitacion, contador_app_origen)
);
```

**Por qué denormalizado y no FK:** no existe (ni puede existir) una FK entre proyectos
de Supabase distintos. `contador_nombre`/`contador_email` son solo para mostrar en la UI
del receptor ("vinculado por: Juan Pérez, vía FacturAR") — la fuente de verdad de quién
es el contador sigue viviendo en la app emisora.

RLS de la tabla: mismo criterio que `vinculos_contador` en FacturAR hoy — el dueño
(`usuario_id = auth.uid()` resuelto vía la tabla de usuario interna de esa app) puede
leer su propio vínculo; el service role (usado por los 2 endpoints de abajo) bypassa
RLS a propósito.

---

## 5. Contrato de API (por app receptora)

Dos operaciones, sin excepción se implementan como parte de un endpoint existente
multipropósito si la app tiene límite de funciones serverless (ver §7) — nunca archivos
nuevos en `/api` si ya se está cerca del límite.

### 5.1 `vinculo_externo_crear` (protegido, server-to-server)

- **Quién la llama:** la app emisora, inmediatamente después de crear su propia
  invitación, una vez por cada app receptora tildada.
- **Auth:** header `X-IntegraAR-Secret` con el valor de `INTEGRAAR_INTERAPP_SECRET`
  (ver §6) — **no** es un JWT de usuario, es un secreto de servicio a servicio.
- **Body:**
  ```json
  {
    "codigo": "AB12CD34",
    "contador_nombre": "Juan Pérez",
    "contador_email": "juan@estudio.com",
    "email_invitado": "cliente@email.com",
    "nombre_invitado": "María Gómez",
    "contador_app_origen": "facturar",
    "expira_at": "2026-08-01T00:00:00Z"
  }
  ```
  `email_invitado`/`nombre_invitado` son del **cliente**, no del contador — los datos que
  el contador cargó al crear la invitación en la app emisora. Sin esto, la app receptora
  no puede prellenar el formulario de `/acceso/:codigo` ni resolver `cuentaExistente`
  (§5.3) — versión inicial del protocolo solo mandaba los datos del contador, corregido
  tras la primera prueba real end-to-end.
- **Efecto:** `INSERT` en `vinculos_contador_externo` con `estado='PENDIENTE'`. Si ya
  existe una fila con el mismo `(codigo_invitacion, contador_app_origen)`, es
  idempotente (upsert, no duplica).
- **Rate limit:** sí, por IP + por `contador_app_origen` (reusar
  `requestGuard.js` de FacturAR como referencia de implementación).

### 5.2 `vinculo_externo_revocar` (protegido, server-to-server)

- **Quién la llama:** la app emisora, cuando el contador desvincula la cuenta (a pedido
  del cliente o por decisión propia) — debe propagar la baja a todas las apps
  compañeras donde ese `codigo` haya quedado activo/pendiente, no solo borrarlo en la
  app emisora. Sin esto, el cliente quedaría "vinculado" en AgendAR/FinanciAR aunque el
  contador ya lo haya dado de baja en FacturAR — inconsistencia real, no cosmética.
- **Auth:** mismo header `X-IntegraAR-Secret` que §5.1.
- **Body:** `{ "codigo": "AB12CD34" }`
- **Efecto:** `UPDATE vinculos_contador_externo SET estado='REVOCADO' WHERE
  codigo_invitacion = :codigo` en la app receptora. Idempotente — si ya estaba
  `REVOCADO`, no hace nada.
- **Quién puede pedir la desvinculación:** el cliente la solicita desde la solapa de
  configuración de su cuenta (ver §9), pero quien la **ejecuta** es siempre el contador
  desde su propio panel — mismo criterio que ya usa FacturAR hoy (el botón "Desvincular"
  vive en `PanelContador.jsx`, no en el perfil del contribuyente). La solapa del cliente
  solo marca un flag de "pidió desvincularse" que el panel del contador muestra como
  aviso; no hay auto-desvinculación unilateral del lado del cliente.

### 5.3 `vinculo_externo_verificar` (público, solo lectura)

- **Quién la llama:** la página `/acceso/:codigo` de la app emisora (o de cualquier app
  que esté armando el selector), desde el navegador del cliente — sin autenticar.
- **Método:** `GET ?codigo=AB12CD34`
- **Respuesta (siempre, exista o no):**
  ```json
  {
    "valido": true,
    "appNombre": "AgendAR",
    "contadorNombre": "Juan Pérez",
    "emailInvitado": "cliente@email.com",
    "nombreInvitado": "María Gómez",
    "cuentaExistente": false
  }
  ```
  o `{ "valido": false }` si no existe, está `REVOCADO`/`EXPIRADO`, o ya está `ACTIVO`
  (un código ya usado no debe seguir apareciendo como opción).
- **`cuentaExistente`**: `true` si ya hay una cuenta en esta app con el email del
  `email_invitado` de esta invitación puntual. Permite que `/acceso/:codigo` ofrezca
  "ingresá" en vez de "registrate" de entrada, en vez de que el cliente se entere recién
  al chocar con un error de "email ya registrado". **No es un endpoint de enumeración**:
  solo resuelve la existencia para el email atado a un código PENDIENTE que el llamador
  ya tiene en su poder — nunca acepta un email arbitrario como parámetro.
- **Nunca devuelve:** contraseña, ID interno del cliente, ni ningún otro dato de la
  cuenta más allá de "existe sí/no". `contadorNombre`/`emailInvitado`/`nombreInvitado` no
  son sensibles — son datos que el propio cliente ya conoce (su contador, su propio
  email/nombre) y que la página de canje necesita para mostrarse y prellenarse.
- **CORS:** esta ruta se llama desde el navegador del cliente, potencialmente desde el
  origen de la app **emisora** (ej. `AccesoContribuyente.jsx` de FacturAR consultando el
  `verificar` de AgendAR para armar el selector de apps del paso 8, §3) — necesita
  `Access-Control-Allow-Origin: *` (respuesta sin PII, seguro de exponer sin restringir
  origen) + un handler `OPTIONS` que devuelva el mismo header.
- **Rate limit:** sí, agresivo (es público) — mismo criterio que
  `arca-consulta-cuit.js` en FacturAR (20 req/min/IP).

### 5.4 `notificar_activacion_externa` (protegido, server-to-server, dirección inversa)

Agregado el 26/07/2026 tras la primera prueba real end-to-end del usuario: sin esto, la
app **emisora** no se enteraba de que el cliente había activado su cuenta en una app
**receptora** — el vínculo quedaba mostrando "PENDIENTE" para siempre en el panel del
contador, aunque el cliente ya estuviera vinculado en la otra app.

- **Quién la llama:** la app receptora (AgendAR/FinanciAR), inmediatamente después de que
  `vinculo_externo_activar` (el paso del cliente, no cubierto por §5.1-5.3 porque no es
  server-to-server) confirma la activación local.
- **Quién la implementa:** la app **emisora** (hoy solo FacturAR) — es la única dirección
  del protocolo donde el receptor llama de vuelta al emisor.
- **Auth:** mismo header `X-IntegraAR-Secret` que el resto de las llamadas server-to-server.
- **Body:** `{ "codigo": "AB12CD34", "app": "agendar" }`
- **Efecto:** actualiza el estado por app (ej. `vinculos_contador_apps.estado='ACTIVO'`
  en FacturAR) — **no** cambia el estado del vínculo maestro de la app emisora (eso sigue
  exigiendo una cuenta real en esa app). El panel del contador debe mostrar este estado
  por separado (un chip por app), visible incluso si el vínculo maestro sigue pendiente.
- **Best-effort:** la app receptora no debe bloquear la activación local si esta llamada
  falla (app emisora caída, secreto no configurado) — solo hace un mejor esfuerzo, con
  `try/catch` silencioso.

---

## 6. Secreto compartido

`INTEGRAAR_INTERAPP_SECRET` — mismo valor exacto en las env vars de Vercel de **todas**
las apps del ecosistema (mismo patrón operativo que ya usan con
`TIENDA_MP_TOKEN_ENCRYPTION_KEY`: si se rota, hay que cambiarlo en todas a la vez, y
documentar el valor real en el `SECRETOS_*.md` de cada repo, nunca en este documento
público del protocolo).

No es una clave de cifrado (no cifra nada) — es un secreto compartido de autenticación
servicio-a-servicio, tipo `CRON_SECRET`. Longitud recomendada: 32+ bytes en hex, igual
criterio que `MP_POLLING_SECRET`.

---

## 7. Restricciones de infraestructura a respetar

- **FacturAR está en 12/12 funciones serverless (Vercel Hobby).** Los dos endpoints de
  §5, del lado de FacturAR (cuando actúa como receptora, ej. si a futuro otra app quiere
  invitar hacia FacturAR), se agregan como `servicio` nuevo dentro de
  `arca-herramientas.js` — nunca como archivo nuevo. Ver `GUIA_CLAUDE_ECOSISTEMA.md`.
- **AgendAR y FinanciAR son Next.js** — confirmar antes de sumar archivos en `app/api/`
  si el límite de 12 les aplica igual (el límite de Vercel Hobby es sobre "Serverless
  Functions" totales del proyecto; el empaquetado de Next.js puede agrupar rutas
  distinto a como lo hace un `/api/*.js` de Vite — no asumir, medirlo en cada repo antes
  de decidir cuántos archivos nuevos entran).
- **Página `/acceso/:codigo` de cada app**: en FacturAR ya existe
  (`AccesoContribuyente.jsx`); en AgendAR se puede partir de la lógica ya probada de
  `clinic_invitations`/`onboarding/profesional` (mismo patrón: token, expiración,
  `signUp` con `emailRedirectTo` de vuelta a la misma URL); en FinanciAR se construye
  desde cero siguiendo el mismo recetario, sin tocar su modelo de auth por cookies (la
  página vive en su dominio, la sesión SSR funciona igual que en cualquier otra ruta).

---

## 8. Qué NO resuelve este protocolo (fuera de alcance, a propósito)

- **No construye un panel de contador dentro de AgendAR o FinanciAR.** Este protocolo
  solo establece el vínculo (quién es cliente de qué contador, en qué apps) y deja
  disponible `permisos jsonb` para que cada app modele sus propios flags finos el día
  que construya ese panel. Hoy ni AgendAR ni FinanciAR tienen el concepto de "un
  tercero administra mi cuenta" más allá de este vínculo.
- **No es SSO.** El cliente sigue logueándose por separado en cada app — el protocolo
  evita justamente tener que resolver sesión compartida.
- **No sincroniza datos entre apps.** Si a futuro un contador necesita ver, desde
  FacturAR, datos reales de AgendAR/FinanciAR de un cliente vinculado, eso requiere una
  llamada API adicional server-to-server (autenticada con el mismo
  `INTEGRAAR_INTERAPP_SECRET` u otro mecanismo), no está cubierto acá.

---

## 9. Checklist para una app nueva del ecosistema (preparar desde el día 1)

Al crear una app nueva, sumar de entrada (aunque el módulo contador no sea prioridad
todavía):

- [ ] Tabla `vinculos_contador_externo` (§4), con su RLS.
- [ ] Endpoint `vinculo_externo_crear` (§5.1), protegido con
      `INTEGRAAR_INTERAPP_SECRET`.
- [ ] Endpoint `vinculo_externo_revocar` (§5.2), protegido, idempotente.
- [ ] Endpoint `vinculo_externo_verificar` (§5.3), público, rate-limited, con CORS
      abierto (`Access-Control-Allow-Origin: *` + handler `OPTIONS`) — lo llaman
      navegadores de otros orígenes, no solo server-to-server.
- [ ] Página `/acceso/:codigo` con el flujo cuenta-nueva-vs-cuenta-existente (copiar el
      recetario de `AccesoContribuyente.jsx` de FacturAR) — **con opción de login, no
      solo registro**: si `cuentaExistente` (§5.3) da `true`, ofrecer login de entrada.
- [ ] Al activar el vínculo (el paso del cliente, con su propia sesión), llamar de vuelta
      a `notificar_activacion_externa` (§5.4) en la app emisora — best-effort, sin
      bloquear la activación local si falla. Sin esto, el panel de la app emisora nunca
      se entera de que el cliente ya se vinculó acá.
- [ ] **Solapa "Vínculo Contador" en la configuración/perfil de la app** (no una
      pantalla nueva aparte) — visible para el cliente vinculado, mostrando: nombre del
      contador, en qué apps del ecosistema está vinculado y su estado (✅ activo / ⏳
      pendiente / — no vinculado), y un botón "Solicitar desvinculación" que solo
      marca el pedido (el contador es quien ejecuta la baja real, ver §5.2). Mismo
      patrón de solapas que ya usa `ProfileEdit.jsx` de FacturAR (Datos fiscales /
      Integración ARCA / Cobros Mercado Pago) — se agrega como una solapa más ahí, no
      como pantalla separada.
- [ ] `INTEGRAAR_INTERAPP_SECRET` cargado en Vercel (mismo valor que las demás apps) y
      documentado en su `SECRETOS_*.md` interno (gitignored).
- [ ] Si la app también va a **emitir** invitaciones (tiene su propio módulo de gestión
      de terceros), implementar el paso de fan-out del §3 (llamar
      `vinculo_externo_crear` de las apps destino tildadas) y el selector de apps del
      §3 paso 8.

---

## 10. Rollout propuesto

1. ✅ **FacturAR (emisor + receptor propio) — implementado 25/07/2026.** Migración
   `vinculo_contador_multiapp`, 4 servicios en `arca-herramientas.js`
   (`crear_vinculo_apps_externas`, `revocar_vinculo_contador`, `solicitar_desvinculacion`,
   `obtener_mi_vinculo`), `api/_lib/vinculoEcosistemaHelpers.js` (fan-out, no-op si no
   hay apps configuradas), solapa "Vínculo Contador" en `ProfileEdit.jsx`. Selector de
   apps destino real (checkboxes) en `PanelContador.jsx` + selector de "dónde activar tu
   cuenta" (paso 8, §3) en `AccesoContribuyente.jsx` — implementados y verificados en
   producción 25-26/07/2026, tras una prueba real end-to-end del usuario.
2. ✅ **AgendAR (receptor) — implementado 25/07/2026.** Tabla `vinculos_contador_externo`,
   `src/app/api/ecosistema/{vinculo-externo,activar}/route.ts`, página
   `/acceso/[codigo]` (cubre cuenta nueva y cuenta ya logueada — a diferencia del patrón
   `clinic_invitations` existente, que solo cubría signup nuevo), card en
   `dashboard/settings/page.tsx`. Sin autoservicio de desvinculación desde AgendAR — el
   camino inverso (avisarle a la app emisora que el cliente pidió desvincularse estando
   logueado en la app receptora) queda para una próxima vuelta si hace falta.
3. ✅ **FinanciAR (receptor) — implementado 25/07/2026.** El proyecto de Supabase de
   FinanciAR (`izprfefxvwsjtnmpempi`) vive en una cuenta distinta a la de FacturAR/
   AgendAR — la migración (`vinculo_contador_externo`) quedó aplicada vía MCP una vez
   que el usuario conectó esa cuenta a la sesión (verificado: tabla/columnas/RLS
   correctas, `get_advisors` sin issues nuevos). El resto (endpoints, página
   `/acceso/[codigo]`, card en Configuración) sigue el mismo contrato que AgendAR, sin
   tocar el auth por cookies SSR de FinanciAR.
4. **Apps futuras:** checklist del §9 desde el primer commit.
5. ✅ **Secretos configurados en producción — 25/07/2026.** `INTEGRAAR_INTERAPP_SECRET`
   (mismo valor en Vercel de FacturAR, AgendAR y FinanciAR) e `INTEGRAAR_APPS_ECOSISTEMA`
   en FacturAR (JSON con las entradas de AgendAR y FinanciAR).
6. ✅ **Primera prueba real end-to-end del usuario — 26/07/2026.** Reveló y cerró 3
   huecos (ver §5.4 y la nota en §3): FacturAR ahora es opcional al invitar
   (`incluye_facturar`), el selector respeta esa elección, y `notificar_activacion_externa`
   cierra el circuito para que el panel de la app emisora refleje activaciones en apps
   compañeras. **El protocolo está validado de punta a punta en producción real**, no
   solo en teoría.

---

*v1.0.0 — escrito 25/07/2026, a partir de una auditoría de código real de los 3 repos
(no supuestos): project refs de Supabase confirmados por `.env`, mecanismo de
`vinculos_contador` de FacturAR leído línea por línea, patrón `clinic_invitations` de
AgendAR y auth por cookies de FinanciAR confirmados por exploración de código. Si algo
de esto queda desactualizado, corregir acá antes de implementar — este documento es el
contrato que las 3 apps deben cumplir, no una nota de una sesión.*
