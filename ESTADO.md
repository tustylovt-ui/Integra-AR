# IntegraAR — Estado del Proyecto
**Última actualización:** Mayo 2026
**Tipo:** Sitio web institucional estático (HTML + CSS vanilla)
**Propósito:** Presentación de la marca IntegraAR y sus aplicaciones

---

## 🏢 ¿Qué es IntegraAR?

**IntegraAR** es la marca desarrolladora de soluciones digitales para profesionales y pymes argentinas.
Cada producto resuelve un problema concreto del mercado local, con integración nativa a ARCA/AFIP,
Mercado Pago y los servicios más usados en Argentina.

Este repositorio contiene el **sitio web institucional** de IntegraAR — la cara pública de la marca,
desde donde los visitantes acceden a la presentación de cada aplicación, sus planes y sus instructivos.

---

## 🌐 Sitio web — estructura actual

### Deploy
- **Plataforma:** Vercel (proyecto separado del resto de las apps)
- **Tipo de build:** Estático — HTML puro, sin framework, sin build step
- **Framework Preset en Vercel:** Other
- **Root Directory:** *(vacío — los archivos están en la raíz del repo)*

### Archivos
```
Integra-AR/
├── index.html                → Landing principal de IntegraAR
├── agendar.html              → Página de detalle de AgendAR (planes, paneles, CTA)
├── facturar.html             → Página de detalle de FacturAR (features, planes, roles)
├── instructivo-agendar.html  → Guía completa de uso de AgendAR
├── instructivo-facturar.html → Guía completa de uso de FacturAR
├── vercel.json               → Rutas y serving de archivos estáticos + PNG
│
├── logo-completo-1.png       → Logo con fondo blanco
├── logo-completo-2.png       → Logo sin fondo (usado en nav, hero y footer)
├── logo-2.png                → Versión alternativa del logo
├── logo-isotipo.png          → Solo el isotipo/ícono
└── logo-tagline.png          → Logo con tagline "Soluciones digitales"
```

### Rutas (definidas en vercel.json)
| Ruta | Archivo | Descripción |
|------|---------|-------------|
| `/` | `index.html` | Landing principal |
| `/agendar` | `agendar.html` | Detalle de AgendAR |
| `/facturar` | `facturar.html` | Detalle de FacturAR |
| `/instructivos/agendar` | `instructivo-agendar.html` | Instructivo AgendAR |
| `/instructivos/facturar` | `instructivo-facturar.html` | Instructivo FacturAR |

---

## 🎨 Diseño del sitio

### Estilo
- **Tipografía:** DM Serif Display (títulos) + DM Sans (cuerpo) — Google Fonts
- **Paleta:** Blanco/minimalista/confiable — fondo blanco, acentos de color por app
- **Tono:** Mixto — profesional pero amigable, dirigido al mercado argentino
- **Logo:** `logo-completo-2.png` (sin fondo) en nav, hero y footer de todos los HTMLs

### Colores por app
| App | Color principal | Uso |
|-----|----------------|-----|
| **IntegraAR** | `#1a1a2e` (navy) | Nav, logo, marca |
| **AgendAR** | `#2563eb` (azul) | Hero, botones, cards |
| **FacturAR** | `#059669` (esmeralda) | Hero, botones, cards |

### Componentes visuales comunes
- Nav sticky con blur backdrop, logo + texto "IntegraAR" al lado
- Sección hero con gradiente suave de color de cada app
- Cards de features con checks y colores de la app
- Footer con logo, links a todas las páginas y copyright

---

## 📄 Contenido de cada página

### `index.html` — Landing principal
- **Eyebrow:** "🇦🇷 Software argentino para profesionales y pymes"
- **Hero:** Título principal + subtítulo + 2 CTAs (Ver apps / Contacto)
- **Sección Apps:** 2 cards — AgendAR y FacturAR con features, URL y botones
- **Sección Por qué IntegraAR:** 6 razones (ARCA, MP, PWA, seguridad, uptime, soporte local)
- **Sección Contacto:** Card con email, ubicación y horario de soporte
- **Footer:** Links a todas las páginas

### `agendar.html` — Detalle AgendAR
- Hero con badge "📅 Sistema de turnos", título y CTA "Empezar gratis"
- **Planes:** Free / Básico $4.900 / Pro $8.900 con features y badge "Popular"
- **Paneles:** 2 cards — Panel Profesional y Panel Clínica con lista de features
- CTA final "Crear cuenta gratis" + link a instructivo

### `facturar.html` — Detalle FacturAR
- Hero con badge "🧾 Facturación electrónica", título y CTA "Empezar gratis"
- **Planes:** Free (10 facturas/mes) / Pro (ilimitadas)
- **Roles:** 3 cards — Independiente, Con contador, Contador
- **Features:** 8 cards — ARCA real, PDF oficial, validaciones, consulta CUIT, control monotributo, PWA, modo oscuro, offline 8hs
- CTA final "Empezar a facturar hoy" + link a instructivo

### `instructivo-agendar.html` — Guía completa AgendAR
Sidebar fijo con índice + contenido con scroll activo. Secciones:
1. Primeros pasos (acceso web, instalación PWA)
2. Registro y onboarding (profesional, por invitación de clínica)
3. Panel Profesional (tabla de módulos)
4. Calendario (crear turnos, estados, reservas online)
5. Pacientes (ficha, historial, notas de sesión)
6. Horarios disponibles y ausencias
7. Obras sociales
8. Recordatorios automáticos
9. Facturación — configurar ARCA + emitir facturas + recibos
10. Panel Clínica (crear clínica, invitar profesionales)
11. Salas y consultorios
12. Reportes de la clínica
13. Google Calendar
14. Planes y pagos

### `instructivo-facturar.html` — Guía completa FacturAR
Sidebar fijo con índice + contenido con scroll activo. Secciones:
1. Primeros pasos
2. Registro (independiente, por invitación de contador)
3. Datos fiscales + autocompletar desde ARCA
4. ¿Qué es ARCA? — por qué se necesita el certificado
5. Obtener certificado — guía de 6 pasos
6. Configurar ARCA en FacturAR
7. Probar la conexión
8. Emitir una factura — guía de 6 pasos
9. Tipos de comprobante (A, B, C, X) con tabla de condiciones
10. PDF formato oficial (3 copias + QR + CAE)
11. Lista de comprobantes + exportar Excel
12. Gestión de clientes
13. Catálogo de productos/servicios
14. Presupuestos
15. Control de monotributo
16. Panel Contador
17. Planes y pagos
18. Modo offline (8hs)

---

## 📱 Aplicaciones presentadas

### AgendAR
- **URL:** https://agend-ar-pi.vercel.app
- **Repo:** https://github.com/tustylovt-ui/AgendAR
- **Descripción:** Sistema de turnos online para profesionales de la salud, coaches y clínicas
- **Stack:** Next.js 16 + Supabase + Tailwind CSS
- **Planes:** Free / Básico $4.900 / Pro $8.900 (mensual, -20% anual)
- **Estado:** ✅ Producción

### FacturAR
- **URL:** https://factu-ar.vercel.app
- **Repo:** privado
- **Descripción:** Facturación electrónica con CAE oficial desde cualquier dispositivo
- **Stack:** Vite + React + Supabase
- **Planes:** Free (10 facturas/mes) / Pro (ilimitadas)
- **Estado:** ✅ Producción

---

## 📞 Contacto IntegraAR

- **Email:** integraar.soluciones@gmail.com
- **Horario:** Lunes a viernes, 9 a 18 hs
- **Ubicación:** Argentina

---

## 🗺️ Roadmap del sitio web

### Corto plazo
- [ ] Comprar dominio — opciones:
  - `.com.ar` en **NIC.ar** (gratis para argentinos con DNI)
  - `.com` en **Donweb** o **Namecheap**
- [ ] Conectar dominio propio en Vercel (Settings → Domains)
- [ ] Optimizar logos (reducir tamaño sin perder calidad)
- [ ] Agregar Open Graph tags para compartir en redes sociales
- [ ] Meta descriptions únicas por página

### Mediano plazo (cuando estén listas las apps)
- [ ] Agregar **GestionAR** — cuando esté en producción
- [ ] Agregar otras apps de IntegraAR según se desarrollen
- [ ] Blog o sección de novedades
- [ ] Formulario de contacto con backend (Resend o Formspree)
- [ ] Sección de testimonios/casos de uso

### Redes sociales (en proceso de creación)
- [ ] Instagram — @integraar (o similar)
- [ ] LinkedIn — IntegraAR
- [ ] Links a redes en nav y footer una vez activas

---

## 🔧 Cómo hacer cambios

El sitio es HTML puro — cualquier cambio se hace editando los `.html` directamente.

```bash
# 1. Editar el archivo HTML correspondiente
# 2. Commit y push
git add .
git commit -m "descripción del cambio"
git push
# Vercel auto-deploya desde main en segundos
```

### Agregar una nueva app
1. Crear `nuevaapp.html` siguiendo la estructura de `agendar.html`
2. Crear `instructivo-nuevaapp.html` siguiendo la estructura de `instructivo-agendar.html`
3. Agregar las rutas en `vercel.json`
4. Agregar la card en `index.html` en la sección "Nuestras aplicaciones"
5. Actualizar los footers de todas las páginas con el nuevo link

---

## 📝 Notas de diseño

- Los **instructivos** tienen sidebar fijo con JS de scroll activo — el ítem del índice se marca en el color de la app cuando la sección entra en el viewport
- Las **cards de planes** tienen un badge "Popular" o "Activo" con posicionamiento absoluto
- Los **logos** van en la raíz del repo (no en subcarpeta) porque Vercel los sirve desde `/logo-*.png`
- El **nav** es el mismo en todas las páginas — logo + "IntegraAR" a la derecha + link volver (en páginas internas) + CTA verde/azul
- Todo el CSS está **inline en cada HTML** — sin archivos externos, zero dependencias
