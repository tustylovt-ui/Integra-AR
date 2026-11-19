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
1. **Aviso previo del estado de ARCA al emitir.** No encontré en el código una comprobación del estado del servicio (el método `FEDummy` de WSFE) antes de emitir. Un aviso «ARCA no responde ahora, probá más tarde o emití desde ARCA» evita el error crudo. Es lo más barato y de más valor; sale una sola vez en `facturador-core` y lo usan todas las apps. **Estado al 18/11/2026:** `facturador-core` **0.8.0 publicado** (`verificarEstadoArca`, puerto opcional `estadoServidores`; `emitir` devuelve `ARCA_NO_DISPONIBLE` sin pedir número ni CAE ni guardar). **Conectado y en producción** en FacturAR común (PR #87), FacturAR Profesional (PR #86), AgendAR (PR #70), Logística y Reparto (PR #66, 19/11/2026) FinanciAR (PR #121, 19/11/2026) y Facturador Local (19/11/2026, `master` local, solo motor `nucleo`; en venta rápida sale como Ticket X con aviso, en pasar a fiscal y NC/ND responde 503; el camino anterior sin `nucleo` no consulta FEDummy; instalador sin rearmar): el SDK propio no trae FEDummy, así que cada app hace la llamada SOAP sin autenticación a WSFEv1 (8 s de tope) y, si ARCA está caída o no responde, la ruta responde 503 con el aviso «ARCA informa problemas en… No se emitió nada ni se consumió ningún número. Probá de nuevo en unos minutos.» (sin marcar rechazo ni registrar error de emisión). **Falta:** rearmar el instalador de Facturador Local, y probarlo en producción con ARCA en línea (ver `PRUEBAS_PENDIENTES.md` sección I).
2. **Recuperar un CAE «perdido».** Si ARCA autoriza pero la respuesta no vuelve (corte de red), el comprobante queda autorizado en ARCA y sin registro local. `facturador-core` propaga el error y deja a la app decidir; no encontré en la muestra un camino que consulte el comprobante en ARCA (`FECompConsultar`) para completar el registro. **A verificar en el código de cada app antes de afirmarlo.** Profesional lo atenúa porque su Libro IVA se alimenta desde ARCA.
3. **Reintentos y cola para envíos** (correo, WhatsApp, sincronización con el contador): relevar caso por caso; hoy no hay una cola común.
4. **Aviso de mantenimiento / estado propio** en el sitio (una línea editable en `index.html`) para cuando falle Vercel o Supabase.
5. **Exportación fácil de datos** (planilla/Excel por app) para cumplir con lo que prometen los términos.

## Comercial y contractual (no es código)
- **Seguro de responsabilidad civil profesional / tecnológica:** consultarlo con un productor de seguros cuando haya ingresos.
- **Planes pagos con compromiso de disponibilidad (SLA):** los planes gratuitos de Supabase y Vercel no lo dan. Evaluar pasar a planes pagos con los primeros ingresos (ver `CAPACIDAD.md`).
- **⚠ Vercel Hobby no permite uso comercial (verificado el 16/11/2026).** Los Términos de Servicio de Vercel (sección «Hobby Plan») dicen que el plan Hobby se usa solo para «personal or non-commercial use» y que Vercel puede desactivar o eliminar proyectos y despliegues de ese plan en cualquier momento. El equipo `integraar` está en **Hobby** (confirmado en Billing con la sesión del dueño) y las apps ya cobran (Cobros Mercado Pago, $15.000/mes). Es el riesgo contractual con terceros más concreto: un corte de Vercel por este motivo frenaría todas las apps. **Recomendación:** pasar a **Pro** antes de depender del cobro. Pro cuesta **$20 por usuario/mes** (incluye $20 de crédito de uso; los asientos de solo lectura son gratis), sube los despliegues de 100 a 6.000 por día (el tope que nos frenó el 15/11), da 1 día de logs en vez de 1 hora y soporte por correo. **Decisión del dueño** (implica pagar; su criterio es no pagar hasta que haya ingresos: acá el ingreso ya existe).
- **Datos del responsable en los términos:** razón social, CUIT, correo, domicilio (Uruguay 55, Venado Tuerto, Santa Fe), horario de atención (lunes a viernes de 9:00 a 12:00) y tribunal (Venado Tuerto) ya cargados en las páginas. Falta la revisión de un abogado.
