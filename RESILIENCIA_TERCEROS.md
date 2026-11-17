# Resiliencia ante fallas de terceros (relevamiento del 16/11/2026)

Pedido del dueño: «¿cómo cubrirse ante fallas de servicios de terceros?» → además de la cláusula en los términos (`terminos.html`, punto 6), lo que ayuda más es lo técnico y lo comercial. Este documento dice **qué hay hoy** (verificado leyendo el código; es un relevamiento por muestreo, no una auditoría completa) y **qué falta**.

## Ya existe
| Qué | Dónde |
| --- | --- |
| Backups diarios con restauración probada (09/10/2026) y regla de actualizarlos en cada cambio de base | cada app (`BACKUPS.md`), ECOSISTEMA §8 |
| Keep-alive de las 6 bases de Supabase (lunes y jueves) | workflow `supabase-keepalive.yml` |
| Mercado Pago: el webhook se reconcilia a diario contra la API (cura webhooks perdidos, 3 días de gracia antes de bajar un plan) | AgendAR `subscriptions/reconcile-cron`; cobros automáticos de FacturAR con `pg_cron` |
| Libro IVA Ventas desde ARCA: si ARCA o internet fallan, aviso con la fecha de la última actualización y botón para reintentar | Profesional |
| Alertas del contador: si el dato de ARCA tiene más de 7 días lo avisa en vez de afirmar «0%» | Profesional |
| Protección de notas de crédito (saldo y simultaneidad) con `facturador-core` 0.7.0 | las apps con emisión |
| El número de comprobante se pide a ARCA en cada intento (`ultimoAutorizado + 1`): un reintento no repite número | `facturador-core/src/emitir.ts` |

## Falta (propuesto, sin empezar)
1. **Aviso previo del estado de ARCA al emitir.** No encontré en el código una comprobación del estado del servicio (el método `FEDummy` de WSFE) antes de emitir. Un aviso «ARCA no responde ahora, probá más tarde o emití desde ARCA» evita el error crudo. Es lo más barato y de más valor; sale una sola vez en `facturador-core` y lo usan todas las apps.
2. **Recuperar un CAE «perdido».** Si ARCA autoriza pero la respuesta no vuelve (corte de red), el comprobante queda autorizado en ARCA y sin registro local. `facturador-core` propaga el error y deja a la app decidir; no encontré en la muestra un camino que consulte el comprobante en ARCA (`FECompConsultar`) para completar el registro. **A verificar en el código de cada app antes de afirmarlo.** Profesional lo atenúa porque su Libro IVA se alimenta desde ARCA.
3. **Reintentos y cola para envíos** (correo, WhatsApp, sincronización con el contador): relevar caso por caso; hoy no hay una cola común.
4. **Aviso de mantenimiento / estado propio** en el sitio (una línea editable en `index.html`) para cuando falle Vercel o Supabase.
5. **Exportación fácil de datos** (planilla/Excel por app) para cumplir con lo que prometen los términos.

## Comercial y contractual (no es código)
- **Seguro de responsabilidad civil profesional / tecnológica:** consultarlo con un productor de seguros cuando haya ingresos.
- **Planes pagos con compromiso de disponibilidad (SLA):** los planes gratuitos de Supabase y Vercel no lo dan. Evaluar pasar a planes pagos con los primeros ingresos (ver `CAPACIDAD.md`).
- **⚠ Verificar los términos de Vercel Hobby:** el plan Hobby de Vercel está pensado para uso **personal y no comercial**; las apps ya cobran (Cobros Mercado Pago, $15.000/mes). Leer la política vigente de Vercel (Fair Use / Terms) y, si corresponde, pasar a Pro antes de depender del cobro. Es el riesgo contractual con terceros más concreto que encontré.
- **Datos del responsable en los términos:** razón social, CUIT, correo, domicilio (Uruguay 55, Venado Tuerto, Santa Fe), horario de atención (lunes a viernes de 9:00 a 12:00) y tribunal (Venado Tuerto) ya cargados en las páginas. Falta la revisión de un abogado.
