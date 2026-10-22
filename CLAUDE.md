# Integra-AR (sitio del ecosistema + MD general) — reglas del repo para Claude Code

## 📌 Norma del ecosistema: primero el MD general, después el de la app

1. Este repo guarda el **MD general del ecosistema**: [`ECOSISTEMA.md`](ECOSISTEMA.md). **En toda sesión nueva se lee primero ese archivo**, sea cual sea la app que se vaya a tratar.
2. **Después** se lee el MD de la app (tabla "Documento de cada app" en `ECOSISTEMA.md` §2). Para trabajar sobre el sitio institucional, el MD de la app es [`ESTADO.md`](ESTADO.md).
3. **Al cerrar un cambio en cualquier app se actualizan los dos:** el MD de esa app y `ECOSISTEMA.md` (fecha de la cabecera, fila de la app en §2 y §10, pendientes de §7 y, si el cambio cruza apps, §3 a §5). Un cambio del sitio actualiza `ESTADO.md` y la fila de Integra-AR en §10.
4. Si no coinciden, se verifica contra el código, Supabase y Vercel, y se corrigen los dos.

## Otros acuerdos de trabajo (resumen)
- Sitio HTML estático: cada push a `main` se publica solo en Vercel (proyecto `integra-ar`).
- Rama + PR + OK explícito del dueño antes de mergear.
- Nunca valores de secretos en estos documentos: solo sus nombres y dónde viven (`SEGURIDAD_TOKENS.md`).
