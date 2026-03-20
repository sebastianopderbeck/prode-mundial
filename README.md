# Prode Mundial ⚽

Sistema de prode (predicción de resultados) del mundial de fútbol para competir con amigos del trabajo.

## 🚀 Stack Tecnológico

- **Frontend**: React 18 + Vite 5 + TypeScript
- **UI**: Design System inspirado en Apple (iOS/macOS)
- **Backend**: Firebase Functions (serverless)
- **Base de Datos**: Cloud Firestore (NoSQL)
- **Autenticación**: Firebase Authentication (Google OAuth + Email/Password)
- **Hosting**: Vercel
- **PWA**: Vite PWA Plugin con Workbox
- **Package Manager**: Bun
- **Testing**: Vitest + fast-check

## ✨ Características

- 🔐 Autenticación con Google OAuth o email/password
- ⚽ Predicción de resultados de partidos del mundial
- 🏆 Sistema de puntaje: 3 puntos por resultado exacto, 1 punto por acertar ganador
- 📊 Ranking en tiempo real de participantes
- 🔄 Sincronización automática de fixtures y resultados desde API externa
- 📱 Progressive Web App (instalable, funciona offline)
- 🎨 Interfaz con estética Apple (iOS/macOS)
- 👨‍💼 Panel de administración para gestionar partidos

## 📋 Requisitos Previos

- [Bun](https://bun.sh/) >= 1.0
- [Node.js](https://nodejs.org/) >= 20 (para Firebase Functions)
- Cuenta de [Firebase](https://firebase.google.com/)
- Cuenta de [Vercel](https://vercel.com/)
- API Key de [API-Football](https://www.api-football.com/)

## 🛠️ Instalación

1. Clonar el repositorio:
```bash
git clone https://github.com/tu-usuario/prode-mundial.git
cd prode-mundial
```

2. Instalar dependencias con Bun:
```bash
bun install
```

3. Configurar Firebase:
```bash
# Instalar Firebase CLI
bun add -g firebase-tools

# Login en Firebase
firebase login

# Inicializar proyecto (si no está inicializado)
firebase init
```

4. Configurar variables de entorno:

Crear archivo `.env` en la raíz del proyecto:
```env
VITE_FIREBASE_API_KEY=tu_api_key
VITE_FIREBASE_AUTH_DOMAIN=tu_proyecto.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=tu_proyecto_id
VITE_FIREBASE_STORAGE_BUCKET=tu_proyecto.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=tu_sender_id
VITE_FIREBASE_APP_ID=tu_app_id
VITE_FOOTBALL_API_KEY=tu_football_api_key
```

5. Configurar Firebase Functions:
```bash
cd functions
bun install
cd ..
```

## 🚀 Desarrollo

Iniciar servidor de desarrollo:
```bash
bun run dev
```

Iniciar Firebase Emulators (para testing local):
```bash
firebase emulators:start
```

## 🧪 Testing

Ejecutar tests:
```bash
bun test
```

Ejecutar tests con UI:
```bash
bun test:ui
```

Ejecutar tests de cobertura:
```bash
bun test:coverage
```

## 🏗️ Build

Build para producción:
```bash
bun run build
```

Preview del build:
```bash
bun run preview
```

## 📦 Deploy

### Deploy a Vercel

1. Conectar repositorio de GitHub con Vercel
2. Configurar variables de entorno en Vercel Dashboard
3. Deploy automático en cada push a `main`

O usar Vercel CLI:
```bash
vercel
```

### Deploy Firebase Functions

```bash
firebase deploy --only functions
```

### Deploy Firestore Rules

```bash
firebase deploy --only firestore:rules
```

## 📁 Estructura del Proyecto

```
prode-mundial/
├── src/                      # Código fuente del frontend
│   ├── components/          # Componentes React
│   ├── design-system/       # Design system Apple-inspired
│   ├── firebase/            # Configuración Firebase
│   ├── hooks/               # Custom hooks
│   ├── pages/               # Páginas de la app
│   ├── pwa/                 # Componentes PWA
│   └── utils/               # Utilidades
├── functions/               # Firebase Functions
│   └── src/
│       ├── index.ts
│       ├── sync.ts
│       ├── scoring.ts
│       └── admin.ts
├── public/                  # Assets estáticos
│   ├── icons/              # Iconos PWA
│   └── manifest.json       # PWA manifest
├── firestore.rules         # Firestore Security Rules
├── firestore.indexes.json  # Índices de Firestore
├── firebase.json           # Configuración Firebase
├── vite.config.ts          # Configuración Vite
└── package.json

```

## 🔒 Firestore Security Rules

Las Security Rules validan:
- Autenticación de usuarios
- Permisos de administrador
- Validación de deadlines en predicciones
- Protección de campos calculados (scores, puntos)

## 🎯 Roadmap

- [x] Autenticación con Google OAuth y email/password
- [x] CRUD de predicciones con validación de deadline
- [x] Sistema de puntaje automático
- [x] Ranking en tiempo real
- [x] Sincronización automática de fixtures y resultados
- [x] PWA con soporte offline
- [x] Design system Apple-inspired
- [ ] Notificaciones push
- [ ] Compartir resultados en redes sociales
- [ ] Estadísticas avanzadas por usuario
- [ ] Modo oscuro

## 🤝 Contribuir

Las contribuciones son bienvenidas! Por favor:

1. Fork el proyecto
2. Crea una rama para tu feature (`git checkout -b feature/AmazingFeature`)
3. Commit tus cambios (`git commit -m 'Add some AmazingFeature'`)
4. Push a la rama (`git push origin feature/AmazingFeature`)
5. Abre un Pull Request

## 📄 Licencia

Este proyecto está bajo la Licencia MIT. Ver archivo `LICENSE` para más detalles.

## 👥 Autores

- Tu Nombre - [@tu-usuario](https://github.com/tu-usuario)

## 🙏 Agradecimientos

- [API-Football](https://www.api-football.com/) por los datos de partidos
- [Firebase](https://firebase.google.com/) por la infraestructura backend
- [Vercel](https://vercel.com/) por el hosting
- Diseño inspirado en [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)
