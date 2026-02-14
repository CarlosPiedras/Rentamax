# 🚀 *Plantilla CMP · Pre-Proyecto React + Tailwind + Next.js (2025)*

---

## 🧭 *1. Definición del Proyecto*

•⁠  ⁠*Nombre del proyecto:* Rentamax
•⁠  ⁠*Descripción breve (1-2 líneas):* Plataforma PropTech de inteligencia de inversión inmobiliaria que identifica inmuebles rentables mediante scraping de portales, cálculos financieros avanzados (ROI neto, CAPEX, fiscalidad) y visualización geoespacial.
•⁠  ⁠*Objetivo principal:* Encontrar y analizar oportunidades de inversión inmobiliaria rentable, cruzando datos reales de mercado con carga fiscal y costes de reforma para obtener la Rentabilidad Neta Real.
•⁠  ⁠*Público objetivo:* Uso privado (2 usuarios: fundadores/inversores)
•⁠  ⁠*Tipo de proyecto:*
  - [ ] Web corporativa
  - [ ] Plataforma SaaS
  - [ ] Ecommerce
  - [x] Dashboard interno
  - [ ] Otro
•⁠  ⁠*KPIs o métricas de éxito:* Identificar inmuebles con Yield Neto >7%, reducir tiempo de análisis por propiedad, precisión en estimación de CAPEX y rentabilidad neta real.
•⁠  ⁠*Repositorio Github:* Rentamax

---

## 🏗️ *2. Arquitectura y Estructura*

### *2.1 Configuración Next.js*

•⁠  ⁠[x] *Versión de Next.js:* 15.x ⭐
•⁠  ⁠[x] *Routing:* App Router ⁠ /app ⁠ ⭐
•⁠  ⁠[x] *TypeScript:* activado (.tsx/.ts) ⭐
•⁠  ⁠[x] *Estrategia de renderizado:*
  - [ ] RSC (web estática)
  - [x] Híbrido (público + área privada)

### *2.2 Arquitectura del Código*

•⁠  ⁠[ ] *Por Capas* — ⁠ /components ⁠, ⁠ /lib ⁠, ⁠ /utils ⁠ (proyectos pequeños)
•⁠  ⁠[x] *Por Features* — ⁠ /features/auth ⁠, ⁠ /features/blog ⁠ (escalable)
•⁠  ⁠[ ] *Por Módulos* — ⁠ /modules/users ⁠, ⁠ /modules/products ⁠ (DDD)
•⁠  ⁠[ ] *Colocation* — componentes junto a ⁠ /app ⁠ (recomendado App Router)
•⁠  ⁠[ ] *Atomic Design* — ⁠ /atoms ⁠, ⁠ /molecules ⁠, ⁠ /organisms ⁠ (design systems)
•⁠  ⁠[ ] *Monorepo* — ⁠ /apps ⁠, ⁠ /packages ⁠ (múltiples proyectos)
•⁠  ⁠[ ] *Personalizada*

---

## 🎨 *3. Diseño y UX*

### *3.1 Diseño Responsive*

•⁠  ⁠[x] Responsive *mobile-first* ⭐
•⁠  ⁠[ ] Compatibilidad WebView

### *3.2 Tema Visual*

•⁠  ⁠[x] Modo claro/oscuro (next-themes)
•⁠  ⁠[x] Colores y fuentes personalizados

---

## 💻 *4. Frontend*

### *4.1 Estilos*

•⁠  ⁠[x] Tailwind 4.x ⭐
•⁠  ⁠[ ] Plugins: forms / typography / aspect-ratio

### *4.2 Componentes UI*

•⁠  ⁠[x] *UI Library:* shadcn/ui ⭐

### *4.3 Formularios*

•⁠  ⁠[x] *React Hook Form + Zod* ⭐ (validación)

### *4.5 Fetching de Datos*

•⁠  ⁠[x] Fetch nativo / React Query

---

## ⚙️ *5. Backend e Infraestructura*

### *5.1 Base de Datos*

•⁠  ⁠[x] *Supabase* ⭐
  - [x] DB (PostgreSQL)
  - [x] Storage
  - [x] Edge Functions

### *5.2 Autenticación y Seguridad*

•⁠  ⁠[ ] Supabase Auth ⭐
•⁠  ⁠[ ] Verificación de email
•⁠  ⁠[ ] *OAuth:*
  - [ ] Google
  - [ ] Microsoft
•⁠  ⁠[ ] *Roles:*
  - [ ] Admin Global
  - [ ] Admin
  - [ ] Cliente
  - [ ] Otros

---

## 💳 *6. Pagos y Transacciones*

•⁠  ⁠[ ] Stripe ⭐
•⁠  ⁠[ ] Webhooks (confirmación/reembolsos)
•⁠  ⁠[ ] Planes / Suscripciones / One-Time

---

## 📧 *7. Emails Transaccionales*

*Método de envío:*

•⁠  ⁠[ ] *API externa de correo*
•⁠  ⁠[ ] *Servidor SMTP propio* (Banahosting, Hostinger, VPS)
•⁠  ⁠[ ] *Autenticación integrada*
•⁠  ⁠[x] *Sin envío de emails*

*Tipos de emails:*

•⁠  ⁠[ ] Verificación de cuenta
•⁠  ⁠[ ] Recuperación de contraseña
•⁠  ⁠[ ] Notificaciones / alertas
•⁠  ⁠[ ] Confirmaciones de pago
•⁠  ⁠[ ] Formularios de contacto

*Plantillas:*

•⁠  ⁠[ ] *React Email / JSX*
•⁠  ⁠[ ] HTML estático
•⁠  ⁠[ ] Texto plano

---

## 🔍 *8. SEO y Metadatos*

•⁠  ⁠[ ] Metadatos + OpenGraph
•⁠  ⁠[ ] Schema.org (JSON-LD)
•⁠  ⁠[ ] Sitemap.xml + robots.txt
•⁠  ⁠[ ] Canonical URLs
•⁠  ⁠[x] Favicons y manifest.json
•⁠  ⁠[ ] *Previsualización social* (Twitter Cards, OG Images dinámicas)

---

## 📊 *9. Analytics y Conversión*

•⁠  ⁠[ ] *Google Analytics 4* ⭐
•⁠  ⁠[ ] Google Search Console
•⁠  ⁠[ ] Pixel de conversión (Facebook, Google Ads)
•⁠  ⁠[ ] Heatmaps / Session Recording

---

## 🌍 *10. Internacionalización (i18n)*

•⁠  ⁠[x] *Sin traducción* (solo un idioma)
•⁠  ⁠[ ] *Manual* (next-intl + JSON)
•⁠  ⁠[ ] *Automática*
•⁠  ⁠[ ] *Mixta* (manual + automática)
•⁠  ⁠[ ] *Optimización SEO por idioma* (si multi-idioma)

---

## ⚡ *11. Rendimiento y Optimización*

•⁠  ⁠[x] Imágenes optimizadas (⁠ next/image ⁠)
•⁠  ⁠[x] Lazy loading
•⁠  ⁠[ ] PWA (next-pwa o Serwist)
•⁠  ⁠[x] Code splitting
•⁠  ⁠[ ] Compresión (Gzip/Brotli)

---

## 🧪 *12. Testing y Calidad de Código*

### *12.1 Testing*

•⁠  ⁠[x] Vitest / Jest
•⁠  ⁠[x] *Playwright (E2E)* ⭐

### *12.2 Code Quality*

•⁠  ⁠[x] ESLint + Prettier
•⁠  ⁠[x] Husky + lint-staged ⭐

---

## 🚀 *13. Control de Versiones y Deployment*

### *13.1 Git*

•⁠  ⁠[x] Branches: ⁠ main ⁠, ⁠ dev ⁠
•⁠  ⁠[x] Commits convencionales
•⁠  ⁠[ ] Pull Requests / Code Review

### *13.2 CI/CD*

•⁠  ⁠[ ] GitHub Actions
•⁠  ⁠[ ] Tests automáticos
•⁠  ⁠[ ] Deploy automático

### *13.3 Hosting*

•⁠  ⁠[ ] *Serverless (Vercel)* ⭐
•⁠  ⁠[ ] *Hosting estático*
•⁠  ⁠[ ] *Hosting VPS*

---

## ⚖️ *14. Legal y Privacidad*

•⁠  ⁠[ ] Banner cookies (RGPD)
•⁠  ⁠[ ] Páginas legales: Aviso / Privacidad / Términos
•⁠  ⁠[ ] Política de cookies

---

## 📄 *15. Páginas y Navegación*

### *15.1 Páginas Públicas*

•⁠  ⁠[ ] Inicio (Home)
•⁠  ⁠[ ] Sobre nosotros / Quiénes somos
•⁠  ⁠[ ] Servicios / Qué ofrecemos
•⁠  ⁠[ ] Precios / Planes
•⁠  ⁠[ ] Preguntas frecuentes (FAQ)
•⁠  ⁠[ ] Contacto / Formulario de contacto
•⁠  ⁠[ ] Blog / Noticias
•⁠  ⁠[ ] Política de privacidad / Aviso legal / Cookies

### *15.2 Páginas Privadas*

•⁠  ⁠[ ] Registro / Login / Recuperación de contraseña
•⁠  ⁠[ ] Perfil de usuario / Configuración
•⁠  ⁠[x] Dashboard principal
•⁠  ⁠[ ] Panel de administración
•⁠  ⁠[ ] Gestión de usuarios
•⁠  ⁠[ ] Gestión de proyectos / pedidos / contenidos

---

## ⚙️ *16. Funcionalidades Avanzadas*

•⁠  ⁠[x] Sistema de notificaciones
•⁠  ⁠[ ] Mensajería interna / soporte
•⁠  ⁠[x] Buscador interno / Filtros
•⁠  ⁠[ ] Comentarios o valoraciones
•⁠  ⁠[x] Sistema de favoritos / Guardados
•⁠  ⁠[x] Exportación de datos (PDF, Excel, CSV)
•⁠  ⁠[ ] Carga masiva de archivos
•⁠  ⁠[ ] Sistema de permisos / Roles avanzados

---

## 🔌 *17. Integraciones con APIs Externas*

•⁠  ⁠[ ] CRM (Salesforce, HubSpot, etc.)
•⁠  ⁠[ ] ERP (SAP, Odoo, etc.)
•⁠  ⁠[ ] Herramientas de marketing (Mailchimp, SendGrid)
•⁠  ⁠[ ] Redes sociales (Facebook, Twitter, LinkedIn)
•⁠  ⁠[ ] Almacenamiento en la nube (AWS S3, Google Drive, Dropbox)
•⁠  ⁠[x] Servicios de mapas (Google Maps, Mapbox)
•⁠  ⁠[x] APIs de datos (clima, finanzas, etc.)
•⁠  ⁠[x] Otras
