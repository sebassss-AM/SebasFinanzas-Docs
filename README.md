# SebasFinanzas 📊🤖

**SebasFinanzas** es una aplicación web progresiva (PWA) de gestión financiera personal con un asistente de Inteligencia Artificial integrado. Diseñada con **arquitectura modular profesional**, **seguridad de grado bancario** y **privacidad total** — tus datos financieros nunca salen de tu dispositivo a menos que tú lo decidas.

> 🔒 **Zero-Knowledge Architecture:** Ni siquiera nosotros podemos ver tus datos. Todo se cifra localmente antes de almacenarse.

---

## ✨ Funcionalidades

### 📊 Dashboard Financiero
- Resumen en tiempo real: saldo disponible, patrimonio total, ingresos y ahorro
- Gráficas interactivas de gastos por categoría y tendencias mensuales (Chart.js)
- Historial de movimientos con búsqueda y eliminación
- Exportación a CSV compatible con Excel (UTF-8 con BOM)
- Alertas automáticas de próximos pagos recurrentes

### 🤖 Agente Financiero IA (FinanzaIA)
- Chat en lenguaje natural: _"Gasté $15 en comida hoy"_ registra el movimiento al instante
- Análisis inteligente de tu situación financiera con datos reales
- Planificación de metas de ahorro con cuotas mensuales calculadas
- Alertas proactivas de ritmo de gasto (si vas a exceder tu presupuesto)
- Gestión de suscripciones y presupuestos por categoría
- Optimizado para consumo mínimo de tokens (tier gratuito de Groq)

### 🎯 Metas de Ahorro
- Creación de metas con progreso visual
- Abonos manuales o vía el asistente IA
- Seguimiento de compromiso mensual

### 📅 Suscripciones
- Registro de gastos recurrentes (Netflix, Spotify, etc.)
- Cálculo automático de pagos pendientes del mes
- Impacto en el saldo disponible en tiempo real

### ☁️ Sincronización con Google Drive
- Respaldo automático o manual de tu base de datos **cifrada**
- Restauración en cualquier dispositivo con tu cuenta de Google
- Carpeta `appDataFolder` invisible y privada (otras apps no pueden acceder)
- Polling inteligente con backoff exponencial (no consume cuota innecesaria)
- Multi-dispositivo: tus datos siempre al día

### 📱 PWA (Progressive Web App)
- Instalable en móvil y escritorio como app nativa
- Funciona offline gracias al Service Worker
- Diseño responsive (mobile-first)
- Tema claro y oscuro

---

## 🛡️ Seguridad

SebasFinanzas implementa múltiples capas de seguridad que superan el estándar de la mayoría de aplicaciones web:

### Cifrado de Datos
| Tecnología | Implementación | Propósito |
|---|---|---|
| **AES-256-GCM** | IV aleatorio de 12 bytes por operación | Cifrado de toda la base de datos del usuario |
| **PBKDF2** | 600,000 iteraciones + salt aleatorio | Derivación de clave maestra desde el PIN |
| **PBKDF2** | 100,000 iteraciones + salt | Hash del PIN (verificación de identidad) |
| **Crypto-Shredding** | Clave derivada del PIN | Si alguien extrae tus datos, solo verá texto ilegible |

### Protección Web
| Capa | Detalle |
|---|---|
| **CSP (Content Security Policy)** | Sin `unsafe-inline` en scripts — bloquea inyección de código XSS |
| **HSTS** | Fuerza HTTPS en toda comunicación |
| **X-Frame-Options: DENY** | Impide que tu app sea embebida en iframes maliciosos |
| **X-Content-Type-Options: nosniff** | Previene sniffing de MIME type |
| **Referrer-Policy** | Controla qué información se envía al navegar |
| **Sanitización HTML** | Todos los datos del usuario se sanitizan antes de renderizarse |
| **Protección CSV Injection** | Exportación segura que previene inyección de fórmulas en Excel |

### Protección de Sesión
| Mecanismo | Detalle |
|---|---|
| **Sesiones volátiles** | La clave AES vive en `sessionStorage` — al cerrar la pestaña, la DB queda bloqueada |
| **Brute-force protection** | Lockout exponencial: 5 intentos → 30s, 8 → 2min, 12+ → 10min |
| **PINs seguros** | Validación de longitud (4-12 dígitos), bloqueo de PINs comunes (1234, 0000, etc.) |
| **Migración automática** | Usuarios legacy con SHA-256 se migran automáticamente a PBKDF2 |

### Google Drive (Scope Mínimo)
- Scope utilizado: `drive.appdata` — el más restrictivo que existe
- Tu app **NO puede** ver archivos del usuario, su email, contactos ni nada personal
- Solo lee/escribe UN archivo en una carpeta oculta exclusiva de la app
- El token de acceso **nunca** se persiste en storage — solo vive en memoria
- Si el usuario desvincula la app desde su cuenta Google, la carpeta se elimina automáticamente

---

## 🏗️ Arquitectura

```
SebasFinanzas/
├── index.html                  # SPA principal
├── css/
│   ├── style.css               # Importador central
│   ├── base.css                # Variables, reset, tipografía
│   ├── layout.css              # Grid, sidebar, responsive
│   ├── components.css          # Botones, cards, inputs
│   ├── chat.css                # Interfaz del agente IA
│   ├── auth.css                # Pantalla de login
│   └── feedback.css            # Toasts, modales
├── js/
│   ├── config.js               # Configuración (gitignored)
│   ├── main.js                 # Entry point, routing, inicialización
│   ├── services/               # Lógica de negocio (sin UI)
│   │   ├── database.js         # IndexedDB, crypto, sesiones, cache
│   │   ├── authService.js      # Registro, login, brute-force protection
│   │   ├── aiService.js        # Integración Groq (prompt optimizado)
│   │   ├── statsService.js     # Cálculos financieros
│   │   ├── movementService.js  # CRUD de movimientos
│   │   ├── goalService.js      # CRUD de metas
│   │   ├── subsService.js      # CRUD de suscripciones
│   │   ├── settingsService.js  # API keys, presupuestos, backup
│   │   ├── driveService.js     # Google Drive API (OAuth, upload, download)
│   │   └── syncService.js      # Auto-sync bidireccional con backoff
│   ├── ui/                     # Controladores de vista (sin lógica de negocio)
│   │   ├── authUI.js           # Login, registro, Drive restore
│   │   ├── dashboardUI.js      # Stats, historial, exportación
│   │   ├── agentUI.js          # Chat IA, acciones, configuración
│   │   ├── goalsUI.js          # Vista de metas
│   │   ├── subsUI.js           # Vista de suscripciones
│   │   └── chartsUI.js         # Gráficas Chart.js
│   └── utils/                  # Utilidades puras
│       ├── helpers.js          # Formateo, sanitización, validación PIN
│       └── validators.js       # Validación de acciones de la IA
├── sw.js                       # Service Worker (PWA offline)
├── manifest.json               # Manifest PWA
├── vercel.json                 # Headers de seguridad para producción
└── build-config.js             # Genera config.js en deploy
```

### Patrones de Diseño
- **Separación Services / UI:** La lógica de negocio está completamente desacoplada de la interfaz
- **withUserDB():** Patrón transparente para decrypt → modificar → re-encrypt automáticamente
- **Cache con TTL:** `getCurrentUser()` cachea en memoria por 5 segundos para evitar descifrados redundantes
- **ES Modules nativos:** Sin bundler, sin dependencias de build, sin node_modules

---

## 📦 Tecnologías

| Categoría | Tecnología |
|---|---|
| **Frontend** | HTML5, CSS3 (Vanilla), JavaScript ES6+ Modules |
| **Gráficos** | Chart.js |
| **Criptografía** | Web Crypto API (AES-256-GCM, PBKDF2, SHA-256) |
| **Base de datos** | IndexedDB (principal) + LocalStorage (fallback) |
| **IA** | Groq API (Llama 3.1) |
| **Nube** | Google Drive API (scope: appdata) |
| **PWA** | Service Workers, Web App Manifest |
| **Deploy** | Vercel (headers de seguridad) |
| **Iconos** | Phosphor Icons |
| **Tipografía** | Inter + Outfit (Google Fonts) |

---

## 🚀 Inicio Rápido

### 1. Crear cuenta
1. Abre la aplicación y haz clic en **Regístrate**
2. Ingresa un nombre de usuario y un PIN seguro (4-12 dígitos)
3. ¡Listo! Ya puedes registrar movimientos desde el Dashboard

### 2. Configurar la IA
1. Ve al icono de **Engranaje** ⚙️ en el sidebar
2. Ingresa tu [Groq API Key](https://console.groq.com/keys) (gratuita)
3. Ve a la sección **Agente IA** y empieza a chatear

### 3. Sincronizar con Google Drive
1. En **Ajustes** ⚙️, haz clic en **"Crear copia de seguridad en Drive"**
2. Inicia sesión con tu cuenta de Google
3. Activa la **sincronización automática** para mantener tus datos al día
4. Para restaurar en otro dispositivo, usa **"Restaurar desde Drive"**

### 4. Desarrollo local
```bash
python3 -m http.server 5500
# Abrir http://localhost:5500
```

---

## 📝 Privacidad

- **Client-Side First:** Tus datos financieros nunca pasan por nuestros servidores
- **Cada usuario gestiona su propia API Key** de Groq — no compartimos ni almacenamos claves
- **Google Drive** usa `appDataFolder`: el archivo de respaldo es invisible para otras apps y para el usuario en la interfaz normal de Drive
- **Sin analytics, sin tracking, sin cookies de terceros**
- ⚠️ Realiza sincronizaciones periódicas con Drive — si borras la caché del navegador, perderás los datos locales

---

*Proyecto desarrollado por Sebas — Finanzas personales con IA, privacidad y cifrado de grado bancario.*
