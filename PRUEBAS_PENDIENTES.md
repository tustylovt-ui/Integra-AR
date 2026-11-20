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
| A1 | AgendAR | **Puente del contador** (`api/ecosistema/emitir-externo`, necesita el secreto entre apps) y **emisión de clínica** (`clinic/billing/emitir`) con CAE real; protección de NC (saldo y NC simultáneas) con CAE real | **NC C y ND C reales ya hechas el 27/10/2026** por el camino nativo (FC 0007-00000039/40/41, NC C 4, ND C 5). Falta lo de arriba y NC/ND A/B (necesitan emisor RI) |
| A2 | FinanciAR | Conversión X → fiscal y notas: FC y **Factura A/B a un RI**; **NC real** de la reversa del cobro (probada en producción solo con rollback); receptor Monotributista con CUIT (`resolverReceptor`, PR #90); protección de NC | Necesita un emisor RI |
| A3 | FacturAR Profesional | **Factura A** (la C y B ya probadas el 13/11); **emisor RI** desde Profesional (la cuenta de Marcela no tiene certificado en esa base); validaciones de CUIT con dígito incorrecto y ND sin comprobante asociado; protección de NC | La C, la B y las NC ya probadas el 13/11 |
| A4 | FacturAR Profesional | **Carga real en ARCA** del TXT de Libro IVA Digital Ventas (DNI real, 99 sin identificar; PR #45) | Que el contador la presente o revise |
| A5 | Facturador Local | **Factura A y B** con CAE real (necesita emisor RI); motor `nucleo` (A y B solo con ARCA simulado); **reposición con una NC real**; certificado propio de Marcela (PV 4, falta la clave que corresponda); cobro real; ARCA real en un cliente instalado | Con CAE real ya probadas: Facturas C, NC C y ND C |

Con CAE real ya probados (no repetir): LR (25/10), AgendAR camino nativo con FC, NC y ND (27/10), FacturAR común (12/11 y protección de NC el 14/11), Profesional (12 y 13/11), puente del contador de común (13/11).

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

## E. Relevadas de los MD de cada app (15/11/2026)

Salen de los `PENDIENTES.md`, `ESTADO_*.md` y `README.md` de cada repo, leídos el 15/11/2026. Cada fila dice **dónde estaba anotada** (para encontrar el detalle original) y su **vigencia**: ✅ vigente (confirmada con el resto de la documentación) o ❓ sin confirmar (la nota es vieja y no encontré en ningún MD que se haya hecho; se confirma al probar o se tacha).

### FacturAR común
| # | Qué falta probar | Dónde estaba | Vigencia |
|---|---|---|---|
| E1 | **Prueba de humo de Sentry:** forzar un error controlado en el frontend (`facturar-frontend`) y en el serverless (`facturar-serverless`) y ver que aparece en Sentry. La última medición (29/07) tenía 0 eventos en 14 días | `README.md` de FacturAR | ❓ |
| E2 | **Escanear el QR de ARCA** de una factura real con CAE (comprobar que abre la verificación de ARCA con los datos correctos). Los tests de `urlQrArca` pasan en las cinco apps, pero nunca se escaneó uno real | `PENDIENTES.md` de FacturAR y Profesional (bitácora de septiembre) | ✅ (aplica a las cinco apps que emiten) |

### FacturAR Profesional
| # | Qué falta probar | Dónde estaba | Vigencia |
|---|---|---|---|
| E3 | **Aviso de diferencia de condición frente al IVA:** con un cliente cuya condición en ARCA y en su app difieran (hoy solo hay test unitario) | `PENDIENTES.md` (bitácora de septiembre) | ❓ |
| E4 | **Botón «🏛 ARCA» de la tarjeta del cliente:** el clic en pantalla (confirmación, copia de CUIT y clave al portapapeles, apertura del portal de ARCA); solo se probó el servicio | `PENDIENTES.md` (bitácora de octubre) | ❓ |
| E5 | **Sentry:** misma prueba de humo que E1 (proyectos `facturar-frontend` y `facturar-serverless` son los mismos de FacturAR común) | `README.md` | ❓ |

### FinanciAR
| # | Qué falta probar | Dónde estaba | Vigencia |
|---|---|---|---|
| E6 | **Cobros MP con una cuenta real:** conectar una cuenta de Mercado Pago y confirmar que el sondeo detecta un pago real y lo factura. Hoy 0 organizaciones lo usan, y el sondeo de FinanciAR no corre en ningún lado (ver `CAPACIDAD.md` M3) | `ESTADO_FINANCIAR.md` (lista de pendientes) | ✅ |
| E7 | **Importar clientes reales:** importar 2 o 3 clientes ficticios desde Excel/CSV/vCard y deshacer la importación desde «Importaciones anteriores» (PR #88) | `PENDIENTES.md` | ✅ |
| E8 | **Buscar cliente por DNI/CUIT en pantalla** (Factura X y modal de conversión a fiscal) con un DNI real; el modal de conversión sigue buscando solo por nombre. No se pudo correr el build local por los paquetes privados | `PENDIENTES.md` (31/10) y `ESTADO_FINANCIAR.md` | ✅ |
| E9 | **Datos del emisor en el PDF con el certificado real:** abrir ARCA, revisar condición e inicio de actividades, generar un PDF fiscal (29/10) | `PENDIENTES.md` | ✅ |
| E10 | **Datos fiscales en la activación del vínculo:** probar en producción con un cliente sin ficha previa que la tarjeta del contador no quede con CUIT `PENDIENTE` (PR #27) | `ESTADO_FINANCIAR.md` (09/10) | ❓ |
| E11 | **Aviso real de vínculo** hacia FacturAR Profesional con `INTEGRAAR_EMISOR_URL` | `ESTADO_FINANCIAR.md` y `PENDIENTES.md` | ❓ (el 08/10 se probó alta y baja reales con FinanciAR, ver abajo) |
| E12 | **Sentry:** prueba de humo (proyecto `financiar`) | `README.md` | ❓ |

### AgendAR
| # | Qué falta probar | Dónde estaba | Vigencia |
|---|---|---|---|
| E13 | **Revinculación con datos fiscales** (CUIT, razón social, condición) con AgendAR; con LR sí se probó | `PENDIENTES.md` («Pendientes reales al 22/10») | ❓ (el 09/10 hubo un alta nueva con AgendAR en el panel del contador) |
| E14 | **Sentry:** prueba de humo (proyecto `agendar`) | `README.md` | ❓ |

AgendAR: NC/ND con CAE real, puente del contador y clínica están en A1.

### Logística y Reparto (LR)
| # | Qué falta probar | Dónde estaba | Vigencia |
|---|---|---|---|
| E15 | **Etiquetas de 3 dígitos (100+ paradas) en el pin de la app** (el del panel web ya se confirmó con una ruta real de 110 paradas): puede hacer falta retocar `textOffset` con el «Calibrador de Pines» | `README.md` | ✅ |
| E16 | **Etiquetar con la cámara:** medir la lectura con **fotos reales** de etiquetas (ya anda con etiquetas generadas) | `README.md` punto 20 (05/11) | ✅ (también C2) |
| E17 | **Paradas numeradas por ruta** (PR #53): probar en el celular con **más de una ruta** el mismo día; un paquete agregado desde otra ruta debe aparecer sin número | `README.md` punto 21 (05/11) | ✅ |
| E18 | **Compilaciones del 21/08, 23/08, 30/08 y 08/09 en el celular real:** seguimiento por WhatsApp con link, cobro contra entrega, modo sin conexión (confirmar o marcar fallida sin señal), nuevas pantallas del repartidor y los tres cambios del 08/09 | `README.md` | ❓ (se probó el mapa y el pin; el resto no figura como confirmado) |
| E19 | **Datos del emisor en el PDF con el certificado real:** guardar el certificado, revisar condición e inicio, cargar Ingresos Brutos y generar un PDF | `README.md` (29/10) | ✅ |
| E20 | **Prioridad alta del 21/08** (puntos 5 a 6 y siguientes) contra un caso real de punta a punta | `README.md` | ❓ |

### Paquetes compartidos
| # | Paquete | Qué falta probar | Vigencia |
|---|---|---|---|
| E21 | `tienda-core` | **Imágenes** (descarga y conversión) con una importación real, y **exportaciones reales** de Tiendanube, Shopify y WooCommerce; después ajustar `origenes.ts`. Falta también el importador en AgendAR y FinanciAR si venden productos | ✅ (misma prueba que B2) |
| E22 | `facturador-core` | **Padrón por DNI contra ARCA real** (`cuitsPosiblesDesdeDni`, prefijos 20/27/23; el 24 queda sin cubrir) | ❓ (FacturAR #33 y Facturador Local lo probaron con un DNI real, ver `ECOSISTEMA.md`) |

### Tienda-AR
No tiene pruebas pendientes anotadas en su propio MD; las que faltan (cobro con `mercadopago` 3.x, envío en la preferencia y preview de Next 16) están en B5.

## G. ✅ Verificación de LR tras el tope de despliegues de Vercel (hecha el 16/11/2026)

**Resultado (16/11/2026, 21:30):** relanzado el despliegue de producción de LR con el OK del dueño (commit `ead4e6d`, Ready; los rechazos por tope no quedan registrados como despliegues). Verificado en el navegador con la sesión del dueño: Inter cargada (variable 100–900), 0 pedidos a Google, sin desborde horizontal ni texto cortado en Historial, Cierre, Facturación y Equipo. JetBrains Mono carga solo donde se usa. `lr-updates` (OTA de la app del chofer) no necesita relanzarse: no cambió.

- [x] **G1 · Relanzar el despliegue de producción de LR** (proyectos `logistica-reparto` y `lr-updates-integraar`, commit `ead4e6d` de `master`, que trae las fuentes Inter y JetBrains Mono propias; PR #63). El 15/11/2026 falló por «Deployment rate limited» (100 por día, ventana móvil de 24 h). Reintentar después de las 21:00 del 16/11; si sigue rechazando, esperar y repetir.
- [x] **G2 · Revisar los despliegues rechazados del 15/11** (LR #63 y la vista previa de Integra-AR #157): confirmar que el último de cada proyecto quedó en «Ready» y que producción sirve el CSS nuevo.
- [x] **G3 · Verificar LR en pantalla con la sesión del dueño** (Historial, Cierre, tablas): tipografía Inter, sin texto desbordado, sin pedidos a `fonts.googleapis.com`. La vista previa no sirve: falta `NEXT_PUBLIC_MAPBOX_TOKEN` en el entorno Preview (el dueño puede cargarlo en Vercel).

## H. ⚖️ Términos v2.0 (publicados el 16/11/2026; verificados en pantalla el 18/11)

- [x] **H1 · Despliegues en producción** de Integra-AR, FacturAR, Profesional, AgendAR, FinanciAR y LR: los 6 en Ready.
- [x] **H2 · Casilla y enlaces en pantalla** en las 5 apps y `/terminos` v2.0 en el sitio.
- [ ] **H3 · Registro real de punta a punta** (crear una cuenta de prueba en cada app y ver `terminos_version` = '2.0' y `terminos_fecha` en la metadata del usuario). No se hizo para no crear cuentas de prueba.
- [ ] **H4 · Revisión del abogado** de `terminos`, `privacidad` y `arrepentimiento` (TXT entregado el 16/11). Si cambia el texto: subir la versión en las 5 apps.
- [ ] **H5 · Re-aceptación** de las cuentas registradas antes de la v2.0 (sin `terminos_version`).
- [ ] **H6 · Probar el botón de arrepentimiento** (hoy abre el correo con el código de constancia; sin envío automático).

## I. 🛰️ Aviso «ARCA no disponible» antes de emitir (en producción: FacturAR, Profesional y AgendAR desde el 18/11/2026; LR y FinanciAR desde el 19/11; Facturador Local en `master` local desde el 19/11, instalador sin rearmar)

- [ ] **I1 · Emitir una factura real chica** en FacturAR común, en Profesional, en AgendAR, en LR, en FinanciAR y en Facturador Local (venta de prueba chica) con ARCA en línea: tiene que salir igual que siempre (CAE y número correlativo), sin demora notable. Emite un CAE real: solo con OK explícito del dueño.
- [ ] **I2 · Simular una caída** (por ejemplo, en un entorno de prueba, cambiando la dirección de WSFEv1 por una que no responda): tiene que mostrarse el aviso y **no** consumirse ningún número. No se puede probar contra ARCA real.
- [ ] **I3 · AgendAR:** revisar que las pantallas de facturación y de clínica muestren el texto del aviso (la ruta responde 503 con `arcaNoDisponible: true`, sin `resultado: 'R'`).
- [ ] **I4 · Facturador Local:** conectado (motor `nucleo`). Probar con ARCA caída simulada: venta rápida fiscal tiene que salir como Ticket X con el aviso «ARCA informa problemas…», y «Pasar a fiscal» tiene que mostrar el aviso sin emitir. Falta rearmar el instalador para que llegue a los comercios.
- [ ] **I5 · CAE «perdido» (`facturador-core` 0.9.0, 19/11/2026):** no se puede simular contra ARCA real (está cubierto por 5 tests del núcleo). Lo que sí conviene: en cada app, una emisión real chica con ARCA en línea (se junta con I1) para confirmar que el camino normal no cambió. Si algún día un corte real deja un comprobante autorizado sin respuesta, tiene que guardarse solo y mostrar la advertencia «No lo emitas de nuevo». Facturador Local sigue en 0.8.0 hasta subirlo.

## F. Anotadas como pendientes en algún MD pero ya resueltas (no repetir)

Para que nadie las vuelva a cargar como pendientes:

- **Emisión real de NC y ND en LR:** hecha el 25/10/2026 (README de LR, punto 10; el punto 398 viejo sigue diciendo «falta»).
- **NC y ND con CAE real en AgendAR por el camino nativo:** hechas el 27/10/2026. La lista «Pendientes reales al 22/10» de AgendAR todavía dice que no.
- **Emisión real desde «Atender» en una tarjeta de app compañera** (Profesional): hecha con el puente el 13/11/2026.
- **Emisión real con un RI desde Profesional (Factura B, NC B, ND B):** hecha el 05/10/2026 con el certificado de Yungblut y el 13/11 con otro RI.
- **Libro IVA Ventas por el puente y contabilización con un RI:** hechos y probados el 31/10/2026 con MARCELA.
- **Baja y alta de vínculos** (cliente y panel) en FinanciAR, AgendAR, LR y FacturAR común: probadas el 08 y 09/10/2026.
- **Restauración de backups en una base descartable:** hecha el 09/10/2026. El `RECUPERACION_DESASTRES.md` de cada app todavía dice que nunca se ensayó; está desactualizado.

## Ya probadas, con cuidados para repetirlas

- **Baja iniciada por el cliente desde las apps hijas:** probada en FinanciAR (08/10), AgendAR, LR y FacturAR común (09/10/2026). **Cuidado:** en el núcleo (`api/_lib/ecosistemaReceptor.js`) aceptar una invitación nueva revoca de su lado **todos** los vínculos activos previos de esa cuenta sin avisar a Profesional (queda ACTIVO allá: inconsistencia entre bases), y `desvincularme` da de baja todos los vínculos activos de la cuenta y avisa uno por uno. Con la cuenta del dueño (`fagottijorger@gmail.com`, vínculo operativo `d7512b6a`) la baja cortaría ese vínculo y el rol pasaría a `INDEPENDIENTE`. Usar una cuenta de FacturAR común de prueba nueva. Si se toca el núcleo, evaluar que `activar` avise a Profesional al revocar los vínculos previos.
- **Restauración de backups** (`restaurar-backup.mjs --ejecutar` sobre una base descartable): hecha el 09/10/2026 en LR y FinanciAR. El proyecto descartable ya está borrado. Queda borrar a mano los `restaurar-*-descartable.ps1` y `limpiar-buckets.mjs` de `C:\Users\Jorge` (tarea de limpieza, no es una prueba).
