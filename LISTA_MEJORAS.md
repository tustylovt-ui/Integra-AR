# Lista general de mejoras y pendientes del ecosistema

> 📌 Se lee después de `ECOSISTEMA.md` (§7 punto 28 apunta acá). Es una lista **viva**: se completa y se tilda desde el chat de trabajo del dueño. Al tildar un ítem se anota la fecha y el PR, y se actualiza `ECOSISTEMA.md` (cabecera, §2/§10 y el punto 28).
> Creada el **06/11/2026**. Criterio del dueño: **no se hacen pagos a ningún sistema o plataforma hasta que haya ingresos**; lo que los exige va a la sección C.

Leyenda: ✅ hecho · 🟡 parcial · ⬜ sin hacer · ❓ falta verificar

---

## A. Mejoras sin costo (revisadas contra el código el 06/11/2026)

Salieron de dos consultas externas (ChatGPT y Gemini) hechas con el organigrama publicado. Cada punto se **verificó contra el código** antes de anotarlo.

| # | Mejora | Estado | Qué hay hoy | Qué falta |
|---|---|---|---|---|
| A1 | Bandeja «qué requiere atención hoy» en FacturAR Profesional | 🟡 | Alertas inteligentes del contador (`AlertasInteligentes.jsx`: sin facturar, salto de facturación, CAE rechazado, CUIT observado, límite de monotributo, certificado por vencer o vencido), tareas con prioridad y vencimiento (`Tareas.jsx`) y widget «Salud del cliente». **Hecho el 07/11/2026 (FacturAR-Profesional #51):** pestaña «📌 Hoy», primera del panel del contador, con Urgente / Esta semana / Para tener en cuenta (alertas y tareas juntas, botones Ver y Resolver/Hecha); probada en pantalla con la cuenta del contador | Sumar avisos nuevos (comprobantes X sin convertir, documentos faltantes) |
| A2 | Diagnóstico de ARCA visible | 🟡 | «Probar conexión» en FacturAR y Profesional (Perfil → Integración ARCA), en AgendAR (`CertificadoARCA.tsx`) y en FinanciAR (`dashboard/facturacion/arca`) | Una vista del **estudio** con el estado de ARCA de todos sus clientes (certificado, punto de venta, último error) |
| A3 | Importación masiva | 🟡 En curso 09/11: **Profesional (clientes), AgendAR (pacientes) y FinanciAR (clientes, #88), FacturAR común (clientes, #58; **productos con talles/colores y conversor de imágenes, #59, con el módulo `tienda-core`**) y LR (paradas, #57) hechos y mergeados; productos también en Profesional, #64**, con registro en la base (fecha y hora de la base) y deshacer completo (Profesional #62 y #63, AgendAR #58); la cartera de FinanciAR (créditos) hecha el 14/11/2026 en el PR #94, mergeado y verificado en producción con la cuenta de prueba (y, si hace falta, productos en AgendAR/FinanciAR) (cada uno debe registrar la importación y poder deshacerla) | Profesional importa **facturas** desde Excel (`ImportacionFacturas.jsx`); Facturador Local tiene importación con IA | Importar **clientes o cartera** (estudio, FinanciAR, AgendAR); no se encontró en el código |
| A4 | Cierre de caja y conciliación | 🟡 **13/11/2026: FinanciAR hecho en el PR #92 (borrador, CI en verde); migración `20261113204834` aplicada en producción el 13/11; falta mergear y probar en pantalla** | FinanciAR: efectivo pendiente de rendición, confirmación del dueño, bloqueo por rendición vencida y detección de cobros de Mercado Pago. LR: cierre del día por chofer | ~~FinanciAR: diferencias de caja antes del cierre~~ (hecho: al cerrar se cuenta el efectivo, la base guarda la diferencia y se exige el motivo). LR: comparar lo cobrado contra entrega con lo declarado (y con Mercado Pago, más adelante) |
| A5 | Score por comportamiento de pago (FinanciAR) | ✅ | `src/lib/riesgo/scoring.ts`: 0 a 100, explicable, puntualidad 45 puntos (cuotas pagadas con atraso), exposición, antigüedad, documentación y garante, con política configurable | ❓ Verificar si cuenta las cuotas **vencidas todavía impagas** (hoy se ve solo el historial de las pagadas) |
| A6 | Ubicación del cobrador al registrar un cobro en efectivo (FinanciAR) | ⬜ | Nada: no hay uso de geolocalización | Guardar la ubicación sin bloquear el cobro y marcar los cobros lejos del domicilio. **Antes, decidir con los cobradores el tema de privacidad** |
| A7 | Reacomodar paradas a mano (LR, chofer o dueño) | ⬜ | El orden se calcula solo con Mapbox (panel y app); no se encontró reordenar a mano | Confirmar en el celular si el chofer puede hacerlo; si no, agregarlo |
| A8 | Cerrar los pendientes ya documentados | ⬜ | Anotados en `ECOSISTEMA.md` §7 | Facturador Local vinculado al panel del estudio; descontar stock al facturar un pedido de la tienda; Sueldos y Jornales como módulo del panel |
| A9 | AgendAR: lista de espera, seña y confirmación del turno | 🟡 | Estados del turno (pendiente, confirmado, atendido, cancelado, ausente), recordatorios automáticos por correo y botones de WhatsApp | Lista de espera y seña o anticipo (no están). ❓ Verificar si el paciente puede confirmar por link |
| A10 | Métricas de ahorro que el estudio pueda mostrar | ⬜ | — | Horas ahorradas, comprobantes sin reclamar, turnos cobrados, entregas efectivas y diferencias de caja detectadas |
| A11 | Ventana de WhatsApp Web dentro de cada app (pedido del dueño, 07/11/2026) | ⬜ | Los botones de WhatsApp de AgendAR, FinanciAR y LR abren la app o la web con el mensaje armado (`wa.me`) | Ver detalle abajo |
| A12 | Alertas del contador leyendo TODOS los puntos de venta desde ARCA (pedido del dueño, 07/11/2026) | ✅ **Hecho 07/11/2026** (Profesional #52 #53 #54 #55 #56 #57 y FacturAR común #48; probado en producción) | Las alertas (sin facturar, salto de facturación, tope de monotributo) usan solo lo que hay en la base de la app; el Libro IVA Ventas ya lee ARCA (`api/arca-scraping.js`) | Que `ultimaFecha`, `porMes` y `total12m` salgan del scraping de ARCA sumando todos los puntos de venta, porque el cliente puede facturar desde otro lado (el Facturador de ARCA u otro sistema) y hoy aparece «cayó al 0%» o «sin facturar» sin que sea cierto |

**A11 — detalle.** Sin costo, sin API de WhatsApp. Se aplica a **todas las apps**, no solo a una:
- **Apps web en la PC** (FacturAR, Profesional, AgendAR, FinanciAR, panel de LR): botón «WhatsApp» que abre `web.whatsapp.com` en una ventana aparte de tamaño fijo con `window.open` y un nombre fijo, para que un segundo clic reutilice la misma ventana. **Siempre con un clic del usuario**: abrirla sola al cargar la app la bloquea el navegador. Si no se puede abrir sola, mostrar el botón apenas se entra.
- **Sesión:** WhatsApp Web la guarda en el perfil del navegador, así que el QR se escanea una vez por PC. Hay que abrir siempre la misma dirección y no usar navegación privada. Vuelve a pedir el QR si el usuario desvincula el dispositivo, borra los datos del navegador, pasan unos 14 días sin abrir WhatsApp en el celular o WhatsApp cierra la sesión.
- **Facturador Local y Sueldos y Jornales Local (escritorio):** ventana propia de la app con perfil guardado en una carpeta de la app (partición persistente), puede abrirse sola al iniciar.
- **Celulares** (cobradores de FinanciAR, chofer de LR, PWA): no aplica una ventana de WhatsApp Web; siguen con los botones que abren la app de WhatsApp con el mensaje armado.
- **Límites:** la app no lee chats ni manda mensajes sola, y **no guarda ni copia la sesión en ninguna base o nube** (equivale a la llave de la cuenta y va contra los términos de WhatsApp). No reemplaza al bot de la sección C: solo evita salir de la app.
- Estas condiciones de sesión se conocen por cómo funciona WhatsApp Web; no se probaron. Probar en una app antes de replicar.

## B. Verificaciones pendientes

- ✅ **Facturador Local — organigrama contra documentación (resuelto el 07/11/2026, revisado el código en la PC del dueño).** El código sí tiene Factura A, B y C, notas de crédito y débito (`comprobante-letra.js`, `fiscal-nucleo.js`) y la consulta de padrón por DNI o CUIT (`padron-nube.js`, `POST /api/padron/consultar`). La frase «no tiene…» del punto 20 era el relevamiento histórico del 31/10, anterior al avance del 01/11 que el mismo punto ya describe. **Matices que siguen vigentes:** (1) el motor nuevo está **apagado por defecto** (hay que poner `config.arca.motor = "nucleo"`); (2) con CAE real solo se probaron Factura C y NC/ND C; **A y B solo con ARCA simulado**; (3) siguen sin vínculo con el contador y sin alertas de monotributo; **resueltos el 10/11/2026:** NC/ND en la ticketera con línea de comprobante asociado, NC que repone stock (opcional), Libro IVA Ventas y Compras (Excel y TXT del Libro IVA Digital) y búsqueda de cliente por DNI/CUIT en Facturación; falta probar Ventas con facturas A/B reales y la revisión del contador. **El organigrama se ajustó el 07/11/2026** (tarjeta de Facturador Local y tabla de ARCA) para aclarar que la A y la B esperan prueba con CAE real. Pendiente real: probar A y B con un emisor responsable inscripto.
- ❓ **FinanciAR — alta de un dueño nuevo** después del cambio de privilegios de columna en `users` (02/11/2026): falta probarlo con un registro real.
- ❓ **FinanciAR — «Consulta Cliente» y alta de cliente:** el organigrama dice que al dar de alta un cliente los datos se completan desde el padrón de ARCA (se vio el mensaje «Consultando padrón ARCA…»; no se revisó que efectivamente los complete).

## C. Descartado por ahora porque exige pagar (retomar cuando haya ingresos)

- Bot de WhatsApp con confirmación bidireccional (API de WhatsApp Business de Meta).
- Asistente de anomalías fiscales con IA (uso de un modelo pago).
- Validación de obra social en tiempo real (servicios de terceros).
- Herramientas externas de ruteo o telemetría de flotas (el orden ya se calcula con Mapbox).
- Financiación online en la tienda: está apagada por decisión de producto, no por costo.
- Historia clínica y receta electrónica en AgendAR (no existen; la receta requeriría convenios).

## D. Cosas a cargo del dueño (ya anotadas en `ECOSISTEMA.md`, repetidas para no perderlas)

- LR: marcar Gutiérrez (Venado Tuerto) como calle de un solo sentido en OpenStreetMap; mandar fotos reales de etiquetas; probar en la calle (Yrigoyen 300, opciones de calles, orden de visitas, Etiquetar con una y varias rutas).
- Dependabot: AgendAR #14 y #8 y FinanciAR #47 quedaron abiertos a propósito (esperan a Next 16 y a un `eslint-config-next` nuevo). Se puede borrar la rama remota `deps/eslint-10` de AgendAR.

## E. Referencia de mercado (relevada el 06/11/2026)

Búsqueda web hecha el 06/11/2026, más dos consultas del dueño a ChatGPT y a Gemini. **Los precios son de comparativas de terceros, no se verificaron en los sitios oficiales y cambian seguido.**

- No apareció un competidor que ofrezca **el conjunto**: panel del estudio conectado con apps por rubro, con la información volviendo al panel. Cada módulo tiene alternativas sueltas. No significa que no exista algo con precios privados.
- **Panel de estudios:** Xubio (clientes ilimitados desde $257.000 + IVA por mes en la consulta con ChatGPT; $95.900 + IVA por 30 clientes), Witmi ($131.500 con IVA), Colppy (desde $128.500 + IVA, panel multiempresa), Alegra (desde $27.999; usuario contador sin costo desde el plan Pyme), Contabilium ($122.000 + IVA), Dux ($36.000 + IVA). Fuente: Commercy (lista al 05/11/2026) y ChatGPT.
- **Turnos:** Turnera desde $14.900 (clínica $79.900 por 10 profesionales), Medicloud con plan gratuito y planes por médico, TurnoWeb $45.000 a $85.000 y Doctoralia. AgendAR cobra $8.900 por mes (Plan Pro); el relevamiento propio del 22/07/2026 (`agendar/INVESTIGACION_MERCADO.md`) está **desactualizado por la inflación**.
- **Créditos y cobranzas:** Prestami (gratis hasta 15 préstamos; premium desde US$45,99 al mes) y El Cobrador (por cotización).
- **Reparto:** RushMile cobra por envío ($145 + IVA hasta 20.000 envíos, mínimo mensual de $500.000 + IVA).
- **Tienda:** Tiendanube (gratis; Esencial $26.999; Impulso $78.999).
- **Advertencia sobre Gemini:** sus precios no traen fuente y afirma que AgendAR tiene historia clínica y receta electrónica; el código no las tiene. Los de ChatGPT sí traen fuente.
- **Posicionamiento que sugieren ambas consultas:** el estudio como canal de entrada y control de la cartera, y una app adecuada para cada actividad del cliente.
