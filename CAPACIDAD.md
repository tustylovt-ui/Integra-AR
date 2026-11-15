# Capacidad del ecosistema con planes gratis — evaluación y plan de mejoras

> Evaluación hecha el **15/11/2026** (pedido del dueño). Mide el uso real de cada base de Supabase, de GitHub Actions y los límites vigentes de los planes gratis de Supabase, Vercel y GitHub, y lista los cuellos de botella de infraestructura y de código.
> Este archivo es el punto de partida para trabajar las mejoras: cada una tiene qué hacer, dónde y cómo comprobarla. Al cerrar una, se tilda acá y en `ECOSISTEMA.md` §7 punto 29.
> Los números de «cuánto entra» son órdenes de magnitud (muestras chicas). Lo que no se midió está dicho en §6.

## 1. Límites de los planes gratis (leídos el 15/11/2026 en las páginas oficiales)

| Servicio | Límite |
|---|---|
| **Supabase Free** (por proyecto) | 500 MB de base · 1 GB de archivos · 5 GB de salida de datos por mes (+5 GB en caché) · 50.000 usuarios activos por mes · archivo máximo 50 MB · 200 conexiones Realtime simultáneas · **2 proyectos activos por cuenta** · **pausa tras 1 semana sin actividad** · **sin copias de seguridad** |
| **Vercel Hobby** (por equipo, por mes) | 100 GB de transferencia · 10 GB de transferencia de origen · 1.000.000 de invocaciones · 4 horas de CPU activa · 360 GB-h de memoria · 5.000 transformaciones de imagen · 100 despliegues por día · función de hasta 300 s · registros de ejecución de 1 hora · al superar un límite, el recurso queda cortado hasta 30 días |
| **GitHub Free** | 2.000 minutos de Actions por mes en repos privados · 500 MB de Packages |

Fuentes: <https://supabase.com/docs/pricing> · <https://vercel.com/docs/plans/hobby> · <https://vercel.com/docs/limits/fair-use-guidelines> · <https://docs.github.com/en/get-started/learning-about-github/githubs-plans>

## 2. Uso actual (medido el 15/11/2026 con SQL en cada base)

| Base | Proyecto | Tamaño (de 500 MB) | Archivos (de 1 GB) | Usuarios | Tablas `public` |
|---|---|---|---|---|---|
| FacturAR común | `tehgzhnpcicuucwdefjm` | 17,9 MB | 2,3 MB (23 archivos) | 2 | 50 |
| FacturAR Profesional | `pvukzwxwbzkmspdgjoqw` | 17,1 MB | 0,3 MB (3) | 5 | 59 |
| AgendAR | `fkbspjyoawjwuthfhgvs` | 14,9 MB | 0,8 MB (12) | 5 | 25 |
| FinanciAR | `izprfefxvwsjtnmpempi` | 15,1 MB | 0 MB (6) | 6 | 32 |
| Logística y Reparto | `mnuilbwxanbhnlrblitu` | 22,3 MB | 2,0 MB (4) | 2 | 19 |
| Tienda-AR (base puente) | `afbuaxrccittcfhbcwoi` | 11,3 MB | 0 MB | 5 | 4 |

Unos 10 MB de cada base son el piso de Postgres vacío (en LR, 7 MB son de PostGIS). Uso real: 2 % a 4 % del límite. `max_connections` = 60 en todas.

**Tamaño por fila medido (tabla + índices; muestras chicas, tienden a exagerar):** `comprobantes_emitidos_arca` ≈ 1–1,4 KB · `comprobantes_recibidos` ≈ 1,5 KB · `comprobantes` (FacturAR común, 36 filas) ≈ 5 KB · `asientos` ≈ 5 KB y `asientos_partidas` ≈ 2,8 KB (Profesional) · `posiciones_historial` (LR, 651 filas) ≈ 450 bytes · `puntos_entrega` ≈ 1 KB.

**Cuánto entra antes de pagar (orden de magnitud):** FacturAR común ~100.000 comprobantes en total; Profesional decenas de miles de ventas contabilizadas (comprobante de ARCA + asiento + partidas: es la que más crece por cliente); AgendAR y FinanciAR cientos de miles de turnos/cuotas/cobros; LR depende del GPS (§3.3).

**GitHub Actions, últimos 30 días (estimado por duración de corridas; el cobro real puede ser mayor porque los trabajos en paralelo se cuentan por separado):** ~1.360 minutos en repos privados de 2.000. FinanciAR ~633 (CI 355, sondeo de Mercado Pago 155, keep-alive 32, Dependabot), FacturAR ~218, AgendAR ~168, Profesional ~132, LR ~110, Tienda-AR ~32, paquetes ~65. Integra-AR es público (no descuenta). Fue un mes de mucho desarrollo.

## 3. Cuellos de botella de infraestructura (por gravedad)

1. **Vercel Hobby.** Los límites técnicos sobran para el tráfico actual. Los problemas: tareas programadas una vez por día como máximo (lo anota `FinanciAR/.github/workflows/mp-polling.yml`), registros de 1 hora y, **a verificar**, que el plan Hobby es solo para uso no comercial (no se pudo leer la cláusula el 15/11 ni confirmar por API en qué plan está el equipo `integraar`). Revisarlo antes de cobrar a clientes. Pro: USD 20 por mes por usuario.
2. **Detección de cobros de Mercado Pago en FinanciAR.** `mp-polling.yml` está programado cada 10 minutos, pero GitHub lo corre **cada 4 a 7 horas** (medido: 153 corridas en 30 días; 14 últimas corridas entre el 11 y el 15/11). Si corriera cada 10 minutos gastaría ~4.300 minutos por mes (más del doble del cupo).
3. **Historial de GPS de Logística y Reparto.** `app-repartidor/screens/MapScreen.tsx` guarda en `posiciones_historial` cada 5 segundos o 20 metros (`timeInterval: 5000, distanceInterval: 20`) y no hay limpieza (la migración `0011_historial_de_posiciones.sql` solo borra en cascada con la ruta o el vehículo). ~2,6 MB por vehículo por jornada de 8 horas: con 5 vehículos, 500 MB en unas 40 jornadas.
4. **Minutos de GitHub Actions.** ~1.360 de 2.000 estimados; ver el número real en la facturación de GitHub.
5. **Supabase Free.** 2 proyectos por cuenta (una app nueva como Sueldos puede necesitar otra cuenta); pausa a los 7 días (lo cubre `supabase-keepalive.yml`, lunes y jueves); sin copias (las cubren los scripts de backup); 5 GB de salida por mes (sin medir).

## 4. Cuellos de botella de código

1. **Consultas sin paginar.** PostgREST devuelve como máximo 1.000 filas por consulta. `FinanciAR/src/app/dashboard/exportar/page.tsx` filtra `cobros` por fecha pero no pagina: con más de 1.000 filas en el rango, el Excel y sus totales salen incompletos sin aviso. Por búsqueda automática (`.select(` contra `.range/.limit/.single/maybeSingle/head`), cerca de un tercio de las consultas de cada app no tiene tope explícito (FacturAR 334/210, Profesional 410/237, AgendAR 269/199, FinanciAR 270/188, Tienda-AR 25/19, LR 68/43): **no están revisadas una por una**; muchas son por id o por organización y no tienen riesgo.
2. **Sumas en el navegador.** Reportes y exportaciones traen filas y suman del lado del usuario; con volumen conviene una función o vista en la base.
3. **Lectura de ARCA (scraping).** `maxDuration: 60` en las funciones de FacturAR común y Profesional (`vercel.json`); un cliente con muchos comprobantes o puntos de venta puede no terminar, y el estudio actualiza cliente por cliente al abrirlo.
4. **Correos de alta de usuarios.** El correo incluido de Supabase Auth tiene un tope bajo por hora: **sin verificar** si las apps usan SMTP propio para registro y recuperación.

## 5. Plan de mejoras (en este orden)

- [ ] **M1 · Retención del GPS en LR.** Decidir con el dueño el criterio (propuesta: conservar 30 días y borrar lo anterior; alternativa: resumir a un punto por minuto). Migración versionada con función de limpieza + tarea programada (pg_cron si está disponible en el proyecto, o el cron diario de Vercel). Revisar `scripts/backup-supabase.mjs` y `restaurar-backup.mjs` (regla del repo). **OK explícito antes de aplicar en producción.** Comprobar: filas y tamaño antes/después; el historial reciente sigue viéndose en `dashboard/historial`.
- [ ] **M2 · Paginar exportaciones y reportes de FinanciAR.** Helper que recorra con `.range()` hasta agotar; aplicarlo en `exportar/page.tsx` (cobros, clientes, créditos, cobradores) y en `reportes/page.tsx`; test con más de 1.000 filas simuladas. Después, revisar la lista de consultas sin tope en las demás apps (empezar por listados, reportes, libros de IVA y exportaciones).
- [ ] **M3 · Cobros de Mercado Pago en FinanciAR.** Ya existe `src/app/api/mp/webhook/route.ts`: verificar que el aviso directo de Mercado Pago cubra la detección de cobros y dejar el sondeo como respaldo (una o dos veces por día), o aceptar el retraso actual. Decisión del dueño. Baja además el gasto de minutos.
- [ ] **M4 · Vercel.** Confirmar el plan del equipo y la cláusula de uso comercial; decidir cuándo pasar a Pro (primer gasto fijo al empezar a cobrar).
- [ ] **M5 · Minutos de Actions.** Mirar el consumo real en GitHub; si se acerca al cupo: agrupar Dependabot semanal, cancelar corridas reemplazadas (`concurrency`), y no correr CI completo en cambios solo de `.md`.
- [ ] **M6 · Lectura de ARCA.** Medir duración real por cliente; si se acerca a 60 s, partir por punto de venta o por mes, o subir `maxDuration`.
- [ ] **M7 · Correos de Auth.** Verificar el SMTP de cada proyecto de Supabase y, si es el incluido, configurar el propio.
- [ ] **M8 · Sumas en la base.** Pasar los totales de reportes a vistas o funciones (después de M2).

## 6. Lo que no se midió

Uso real de Vercel (transferencia, invocaciones, CPU), salida de datos de Supabase, minutos de Actions facturados, almacenamiento de GitHub Packages, y el plan exacto del equipo de Vercel. Todo eso se ve en el panel de cada servicio.
