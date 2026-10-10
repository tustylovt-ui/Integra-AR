# Tokens del ecosistema: dónde viven y cómo rotarlos (09/10/2026)

> Esta lista **no contiene valores**. Los secretos los carga el dueño; nunca se pegan en un chat ni en un archivo del repo.

## 1. Rotar YA: token personal de GitHub que se pegó una vez en un chat
Cualquier token que haya pasado por un chat se considera comprometido.
1. GitHub → Settings → Developer settings → Personal access tokens → buscar el que empieza con `ghp_` usado en esa conversación → **Revoke**.
2. Si ese token era el que se usa como `GH_PACKAGES_TOKEN` o `PACKAGES_READ_TOKEN`, hacer el punto 2 completo con uno nuevo.
3. Revisar en GitHub → Settings → Security log si hubo usos desconocidos.

## 2. Renovar antes del 02/01/2027: lectura de paquetes privados (`@tustylovt-ui/*`)
Un **PAT clásico con solo el permiso `read:packages`** (de la cuenta `tustylovt-ui`). Un mismo valor sirve para todo lo de abajo. Nombre sugerido: `ecosistema-read-packages-AAAAMM`.

### Dónde se carga (GitHub → repo → Settings → Secrets and variables → Actions)
| Repo | Nombre del secret |
|---|---|
| FacturAR (núcleo) | `GH_PACKAGES_TOKEN` |
| FacturAR-Profesional | `GH_PACKAGES_TOKEN` |
| AgendAR | `GH_PACKAGES_TOKEN` |
| FinanciAR | `GH_PACKAGES_TOKEN` |
| Logistica-y-Reparto | `GH_PACKAGES_TOKEN` |
| arca-core (workflow `publicar.yml`) | `PACKAGES_READ_TOKEN` |

`bcra-core` e `integraar-vinculos` publican con el `GITHUB_TOKEN` automático: no necesitan nada.

### Dónde se carga (Vercel → proyecto → Settings → Environment Variables, en Production, Preview y Development)
Variable `GH_PACKAGES_TOKEN` en: `factur-ar`, `facturar-profesional`, AgendAR, FinanciAR y Logística y Reparto (el dashboard). Tienda-AR y Facturador Local: verificar si usan paquetes privados antes de cargar.

### En tu máquina
Variable de entorno de usuario `GH_PACKAGES_TOKEN` (solo si instalás paquetes en local).

### Orden recomendado
1. Crear el token nuevo. 2. Cargarlo en los 6 repos y los 5 proyectos de Vercel. 3. Relanzar el último workflow de cada repo y un deploy de cada proyecto para comprobar que no da `401`. 4. Recién ahí revocar el viejo.

## 3. Tokens de las conexiones de Supabase del escritorio de Claude
Son 4 PAT (uno por cuenta/organización de Supabase) guardados en la configuración local de la app de escritorio. Si alguno da `Unauthorized`, venció: generar uno nuevo en Supabase → Account → Access Tokens y reemplazarlo en la configuración. Revocar los viejos.

## 4. Otras claves que no se tocaron pero conviene tener ubicadas
- `INTEGRAAR_INTERAPP_SECRET` (firma del vínculo entre apps): en Vercel de cada app; si se rota, hay que cambiarlo en **todas a la vez**.
- `service_role` de cada base de Supabase: solo en Vercel/entorno de cada app; no usarlas fuera de ahí. La prueba de restauración del 09/10/2026 usó la de un proyecto descartable que ya se borró.
- Certificados y claves de ARCA: se respaldan dentro del backup semanal (están en `arca_config`); el backup es sensible.
