# Pruebas pendientes del ecosistema IntegraAR

> Creado el 15/11/2026 a pedido del dueño: **todo lo que falta probar vive acá** y no mezclado con otros temas. En `ECOSISTEMA.md` (§7, §10), `CAPACIDAD.md` y `LISTA_MEJORAS.md` quedó solo un enlace a este archivo.
> Cuando se hace una prueba: se tacha acá (con fecha y resultado), se anota en el MD de la app y se actualiza `ECOSISTEMA.md` (fila de la app en §10).
> Esto cubre los documentos de este repo. Los `PENDIENTES.md` y `ESTADO_*.md` de cada app pueden tener más detalle propio; si aparece una prueba nueva ahí, se suma a este archivo.

## Reglas que valen para todas las pruebas

- **Pruebas con CAE real (sección A):** necesitan **OK explícito del dueño** (factura e importe) antes de emitir. Orden acordado el 27/10/2026: quedan **para lo último**, después de probar en pantalla y de hacer las modificaciones pendientes. Se usa el punto de venta 7 y importes chicos (por ejemplo $10).
- **Pagos y cuentas:** las pruebas de pago real las hace el dueño; Claude no ejecuta pagos ni escribe contraseñas ni claves.
- **Cuentas de prueba:** no usar la cuenta del dueño en pruebas de vínculos. No tocar el vínculo de MARCELA (`6b7cf020`, cliente real).
- Las pruebas que cambian datos de producción piden OK del dueño.

## A. Con CAE real (necesitan OK del dueño; para lo último)

| # | App | Qué falta probar | Notas |
|---|---|---|---|
| A1 | AgendAR | NC y ND reales; **puente del contador** y **emisión de clínica**; protección de NC (saldo) | Profesional/núcleo ya validado con monotributista y RI; LR ya hecho el 25/10 |
| A2 | FinanciAR | Conversión X → fiscal y notas: FC y **Factura A/B a un RI**; **NC real** de la reversa del cobro (probada en producción solo con rollback); receptor Monotributista con CUIT (`resolverReceptor`, PR #90); protección de NC | Necesita un emisor RI |
| A3 | FacturAR Profesional | **Factura A** (la C y B ya probadas el 13/11); **emisor RI** desde Profesional (la cuenta de Marcela no tiene certificado en esa base); validaciones de CUIT con dígito incorrecto y ND sin comprobante asociado; protección de NC | La C, la B y las NC ya probadas el 13/11 |
| A4 | FacturAR Profesional | **Carga real en ARCA** del TXT de Libro IVA Digital Ventas (DNI real, 99 sin identificar; PR #45) | Que el contador la presente o revise |
| A5 | Facturador Local | **Factura A y B** con CAE real (necesita emisor RI); motor `nucleo` (A y B solo con ARCA simulado); **reposición con una NC real**; certificado propio de Marcela (PV 4, falta la clave que corresponda); cobro real; ARCA real en un cliente instalado | Con CAE real ya probadas: Facturas C, NC C y ND C |

Con CAE real ya probados (no repetir): LR (25/10), AgendAR núcleo (27/10), FacturAR común (12/11 y protección de NC el 14/11), Profesional (12 y 13/11), puente del contador de común (13/11).

## B. En pantalla o en producción, sin CAE

| # | App | Qué falta probar |
|---|---|---|
| B1 | FacturAR común | Lápiz de edición del Libro IVA en el núcleo (antes punto 5 de §7) |
| B2 | FacturAR común y Profesional | Importador de productos: imágenes, exports reales de Tiendanube/Shopify/Woo y «Cancelar importación». Profesional además con el **primer usuario Independiente/Contribuyente** (la cuenta Contador no ve el catálogo) |
| B3 | FinanciAR | **Alta de un dueño nuevo** con un registro real, después del cambio de privilegios de columna en `users` (02/11/2026) |
| B4 | Facturador Local | **Ventas del Libro IVA con facturas A/B reales** y que el contador revise el TXT |
| B5 | Tienda-AR | Cobro con `mercadopago` 3.x en producción; **costo de envío** en la preferencia de MP con un pedido real con envío; mirar la preview de Next 16 (rama `chore/next-16`) |

## C. En un dispositivo o entorno real

| # | App | Qué falta probar |
|---|---|---|
| C1 | Facturador Local | **Instalación en otra PC** (instalador `.exe` del 15/11/2026); lista nueva de condición de IVA en pantalla; **dos notas simultáneas** contra el servidor real; red entre dos PC físicas (puerto UDP 41234); caja en réplica sin reenvío; checklist de instalación; descuento de stock y plantilla `editorial` contra una **tienda real** (la base puente no tiene tiendas) |
| C2 | LR | **APK en el celular real** (compilado el 30/10/2026) junto con la búsqueda de calles de Rufino; probar en la calle; fotos reales de etiquetas para medir el lector |

## D. Cobros y correos (agregadas el 15/11/2026)

| # | App | Qué falta probar | Notas |
|---|---|---|---|
| D1 | FacturAR común | **Pago de punta a punta del adicional Cobros Mercado Pago** (FacturAR #82): checkout en Perfil → webhook → fila en `addons_contratados` y en `pagos` → alerta de vencimiento a los 5 días. También el **circuito de pago de los planes**, que tampoco se probó nunca de punta a punta | La tabla `pagos` de producción está vacía. Mercado Pago no acepta pagar con la misma cuenta que cobra: hace falta **otra cuenta** o el sandbox de Mercado Pago. Revisar también `MP_WEBHOOK_SECRET` y la URL de notificaciones |
| D2 | FinanciAR, LR y FacturAR Profesional | **Registro o recuperación de clave real** para confirmar que llegan los mails de Auth por el Gmail (SMTP propio cargado el 15/11/2026) | Una clave de aplicación mal copiada no se nota en la pantalla de Supabase |

## Ya probadas, con cuidados para repetirlas

- **Baja iniciada por el cliente desde las apps hijas:** probada en FinanciAR (08/10), AgendAR, LR y FacturAR común (09/10/2026). **Cuidado:** en el núcleo (`api/_lib/ecosistemaReceptor.js`) aceptar una invitación nueva revoca de su lado **todos** los vínculos activos previos de esa cuenta sin avisar a Profesional (queda ACTIVO allá: inconsistencia entre bases), y `desvincularme` da de baja todos los vínculos activos de la cuenta y avisa uno por uno. Con la cuenta del dueño (`fagottijorger@gmail.com`, vínculo operativo `d7512b6a`) la baja cortaría ese vínculo y el rol pasaría a `INDEPENDIENTE`. Usar una cuenta de FacturAR común de prueba nueva. Si se toca el núcleo, evaluar que `activar` avise a Profesional al revocar los vínculos previos.
- **Restauración de backups** (`restaurar-backup.mjs --ejecutar` sobre una base descartable): hecha el 09/10/2026 en LR y FinanciAR. El proyecto descartable ya está borrado. Queda borrar a mano los `restaurar-*-descartable.ps1` y `limpiar-buckets.mjs` de `C:\Users\Jorge` (tarea de limpieza, no es una prueba).
