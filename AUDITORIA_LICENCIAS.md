# Auditoría de licencias de dependencias (20/11/2026)

> Punto 9 del blindaje legal (`ECOSISTEMA.md` §7, punto 31). Hecha por Claude **leyendo los `package-lock.json`** de cada repo (campo `license` de cada paquete) y verificando a mano los casos dudosos contra el paquete publicado. **No es dictamen legal**: es el insumo para la revisión del abogado.

## Alcance
Repos revisados: AgendAR, FacturAR común, FacturAR Profesional, FinanciAR, Tienda-AR, LR (`dashboard-web`, `app-repartidor`, `updates-server`) y `arca-core`.
**No revisados:** Facturador Local (el repo vive en la PC del dueño, sin `package-lock.json` en el repo que se leyó), `facturador-core`, `tienda-core` (sin dependencias de ejecución propias en el lockfile) y los repos de AgroDron y agentes.
**Límite del método:** el campo `license` puede faltar o estar mal en un paquete; los textos de licencia de cada paquete no se leyeron uno por uno (sí los de los marcados abajo).

## Resultado general
Entre 560 y 750 paquetes por app. Más del 95 % es MIT, Apache-2.0, ISC o BSD. **No hay ninguna dependencia GPL, AGPL ni SSPL que se use sin alternativa permisiva.**

## Para revisar con el abogado (en orden de importancia)
1. **`mapbox-gl` 3.x (LR `dashboard-web`, producción).** Licencia propietaria de Mapbox («SEE LICENSE IN LICENSE.txt»), no de código abierto: su uso queda sujeto a los términos de Mapbox (cuenta, token y condiciones comerciales). No es un impedimento, pero hay que leer esos términos y que los acepte la cuenta de Mapbox que se usa. Alternativa libre si hiciera falta: MapLibre GL.
2. **`sharp-libvips` (LGPL-3.0, en AgendAR, FinanciAR, Tienda-AR y LR `dashboard-web`).** Es la biblioteca de imágenes que usa Next.js. Entra como dependencia **opcional** (binarios por plataforma) y se usa enlazada dinámicamente, que es el caso que la LGPL permite sin obligar a abrir el código propio. Lo que sí pide es conservar el aviso de licencia. Riesgo bajo.
3. **Paquetes propios `@tustylovt-ui/*` (`arca-core`, `facturador-core`, `tienda-core`, `bcra-core`) figuran `UNLICENSED`.** Es lo esperado para código privado: significa «todos los derechos reservados». Conviene dejarlo así.
4. **`mercadopago` (SDK oficial).** El campo `license` viene vacío en el paquete, pero el archivo `LICENSE` del paquete publicado es **MIT** (verificado en la versión 3.6.2).

## Revisados y sin problema
- **`png-js` 2.0.0** (FinanciAR, vía pdfkit): sin campo en el lockfile, **MIT** según el registro de npm.
- **`jsonify` 0.0.1:** «Public Domain».
- **`node-forge`:** «BSD-3-Clause OR GPL-2.0»: se usa bajo BSD (opción permisiva).
- **`jszip`** (FacturAR y Profesional): «MIT OR GPL-3.0-or-later»: se usa bajo MIT.
- **`caniuse-lite`:** CC-BY-4.0 (datos de compatibilidad de navegadores; pide atribución, que ya cumple el paquete).
- **MPL-2.0 (`lightningcss`, `axe-core`):** copyleft débil, solo de **desarrollo** (compilación de estilos y pruebas de accesibilidad), no viaja en lo que se publica. Excepción: `lightningcss` aparece también en `app-repartidor`, que es app móvil; se usa en el build, no se incluye en la app.

## Qué falta
- Revisar Facturador Local desde su PC (`npm ls --all` o su lockfile) y el empaquetado del instalador.
- Repetir esta lectura cuando cambien dependencias grandes (Next, Tailwind, Expo). Se puede automatizar con `license-checker` en el CI si el abogado lo pide.
