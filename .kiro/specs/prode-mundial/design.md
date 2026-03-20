# Diseño Técnico: Prode Mundial

## Overview

El sistema Prode Mundial es una aplicación web full-stack que permite a usuarios de un mismo grupo de trabajo competir prediciendo resultados de partidos del mundial de fútbol. El sistema otorga puntos según la precisión de las predicciones (3 puntos por resultado exacto, 1 punto por acertar el ganador) y mantiene un ranking en tiempo real.

La arquitectura se basa en una aplicación React con Vite como bundler, un design system personalizado que replica el estilo de Apple (iOS/macOS), y Node.js como backend. La aplicación está configurada como Progressive Web App (PWA) para permitir instalación y funcionamiento offline. El sistema se integra con APIs externas de fútbol para obtener fixtures y resultados automáticamente, reduciendo la carga administrativa manual.

**Referencia de Diseño**: El diseño visual sigue los lineamientos definidos en Figma: https://www.figma.com/design/V5RipINd9Q78cB2X4C9ORz/Untitled

### Stack Tecnológico

- **Frontend**: React 18+ con Vite 5+
- **UI Framework**: Custom Apple-inspired design system (replicando iOS/macOS)
- **Tipografía**: SF Pro Display / SF Pro Text (con fallback a system-ui)
- **PWA**: Vite PWA Plugin (workbox) para service workers y manifest
- **Backend**: Firebase Functions (serverless)
- **Autenticación**: Firebase Authentication (Google OAuth + Email/Password)
- **Base de Datos**: Cloud Firestore (NoSQL)
- **Hosting**: Firebase Hosting
- **API Externa**: API-Football (api-football.com) para fixtures y resultados
- **Scheduler**: Firebase Cloud Scheduler para sincronización automática
- **Package Manager**: Bun (en lugar de npm)
- **Testing**: Vitest + React Testing Library + fast-check (property-based testing)

## Architecture

### Arquitectura General

El sistema sigue una arquitectura serverless con Firebase como backend:

```
┌─────────────────────────────────────────────────────────────┐
│                Frontend (React + Vite + PWA)                 │
│                  Hosted on Firebase Hosting                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Auth UI    │  │  Predictions │  │   Ranking    │      │
│  │   (Apple)    │  │      UI      │  │      UI      │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│           │                │                  │              │
│           └────────────────┴──────────────────┘              │
│                            │                                 │
│                    ┌───────▼────────┐                        │
│                    │ Firebase SDK   │                        │
│                    │ (Auth + DB)    │                        │
│                    └───────┬────────┘                        │
└────────────────────────────┼──────────────────────────────────┘
                             │ HTTPS
┌────────────────────────────▼──────────────────────────────────┐
│                    Firebase Services                          │
│  ┌──────────────────────────────────────────────────────┐    │
│  │           Firebase Authentication                    │    │
│  │  (Google OAuth + Email/Password)                     │    │
│  └──────────────────────────────────────────────────────┘    │
│  ┌──────────────────────────────────────────────────────┐    │
│  │           Cloud Firestore (NoSQL)                    │    │
│  │  Collections: users, matches, predictions, config    │    │
│  └──────────────────────────────────────────────────────┘    │
│  ┌──────────────────────────────────────────────────────┐    │
│  │           Firebase Functions (Serverless)            │    │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐     │    │
│  │  │  Sync      │  │  Scoring   │  │  Admin     │     │    │
│  │  │  Results   │  │  Engine    │  │  Functions │     │    │
│  │  └────────────┘  └────────────┘  └────────────┘     │    │
│  └──────────────────────────────────────────────────────┘    │
└───────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────┐
│              External Services                                 │
│  ┌──────────────────┐         ┌──────────────────┐            │
│  │  API-Football    │         │ Cloud Scheduler  │            │
│  │  (Fixtures/      │         │ (Daily Sync)     │            │
│  │   Results)       │         │                  │            │
│  └──────────────────┘         └──────────────────┘            │
└────────────────────────────────────────────────────────────────┘
```

### Flujo de Datos Principal

1. **Autenticación**: Usuario → Firebase Auth (Google OAuth o Email/Password) → Frontend recibe token → Firestore actualiza user document
2. **Carga de Fixture**: Cloud Scheduler trigger → Firebase Function → API-Football → Function procesa → Firestore almacena matches
3. **Predicción**: Usuario ingresa predicción → Frontend valida deadline → Firestore almacena directamente → Security Rules validan
4. **Sincronización**: Cloud Scheduler trigger → Firebase Function → API-Football → Function actualiza matches → Firestore Trigger → Scoring Function calcula puntos → Firestore actualiza scores
5. **Ranking**: Frontend consulta Firestore → Query ordenada por totalScore → Frontend muestra ranking en tiempo real

### Decisiones Arquitectónicas

1. **Firebase Serverless**: Elimina gestión de servidores, escalabilidad automática, pay-per-use
2. **Firestore NoSQL**: Queries en tiempo real, escalabilidad horizontal, integración nativa con Firebase
3. **Firebase Authentication**: Gestión completa de auth (OAuth, email/password), tokens seguros, integración directa
4. **Security Rules**: Validación de permisos a nivel de base de datos, sin necesidad de middleware custom
5. **Cloud Scheduler**: Ejecución programada confiable, integración nativa con Firebase Functions
6. **Firebase Hosting**: CDN global, HTTPS automático, integración con PWA
7. **Apple Design System**: Interfaz familiar y consistente para usuarios de iOS/macOS, estética minimalista y moderna
8. **PWA**: Permite instalación en dispositivos, funcionamiento offline, y experiencia similar a app nativa
9. **Bun**: Package manager más rápido que npm, mejor performance en instalación y ejecución

## Components and Interfaces

### Design System: Apple-Inspired UI

El sistema utiliza un design system personalizado que replica la estética de Apple (iOS/macOS):

**Tipografía**:
- SF Pro Display para títulos y headings
- SF Pro Text para body text
- Fallback: system-ui, -apple-system, BlinkMacSystemFont

**Colores** (siguiendo paleta de Apple):
- Primary: #007AFF (iOS Blue)
- Secondary: #5856D6 (iOS Purple)
- Success: #34C759 (iOS Green)
- Warning: #FF9500 (iOS Orange)
- Error: #FF3B30 (iOS Red)
- Background Light: #F2F2F7
- Background Dark: #000000
- Surface Light: #FFFFFF
- Surface Dark: #1C1C1E
- Text Primary Light: #000000
- Text Primary Dark: #FFFFFF
- Text Secondary Light: #3C3C43 (60% opacity)
- Text Secondary Dark: #EBEBF5 (60% opacity)

**Espaciado** (sistema de 4px):
- xs: 4px
- sm: 8px
- md: 16px
- lg: 24px
- xl: 32px
- xxl: 48px

**Bordes y Sombras**:
- Border radius: 10px (iOS standard)
- Card radius: 12px
- Button radius: 10px
- Shadows: sutiles, siguiendo elevación de iOS

**Componentes Base**:
- Button (filled, outlined, text variants)
- Card (con glassmorphism opcional)
- Input (estilo iOS con label flotante)
- Modal (sheet-style de iOS)
- List (grouped style de iOS)
- Navigation (iOS tab bar style)

### PWA Configuration

**Manifest** (`public/manifest.json`):
```json
{
  "name": "Prode Mundial",
  "short_name": "Prode",
  "description": "Predicción de resultados del mundial de fútbol",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#F2F2F7",
  "theme_color": "#007AFF",
  "orientation": "portrait",
  "icons": [
    {
      "src": "/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

**Workbox Strategy** (usando Workbox via Vite PWA Plugin):
- **NetworkFirst**: Para Firestore queries (matches, predictions, ranking)
- **CacheFirst**: Para assets estáticos (CSS, JS, fonts, images)
- **StaleWhileRevalidate**: Para datos que pueden estar desactualizados (fixture)

**Offline Support con Firestore**:
- Firestore tiene offline persistence automática (enableIndexedDbPersistence)
- Cache de matches y predicciones del usuario
- Modo offline muestra datos cacheados con indicador visual
- Writes offline se guardan en queue local de Firestore
- Sincronización automática al recuperar conexión (manejado por Firestore SDK)

**Vite PWA Plugin Configuration** (`vite.config.ts`):
```typescript
import { VitePWA } from 'vite-plugin-pwa';

export default {
  plugins: [
    VitePWA({
      registerType: 'autoUpdate',
      includeAssets: ['favicon.ico', 'robots.txt', 'icons/*.png'],
      manifest: {
        // contenido del manifest.json
      },
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg,woff2}'],
        runtimeCaching: [
          {
            // Firestore ya maneja su propio cache offline
            // Solo necesitamos cachear assets estáticos
            urlPattern: /\.(woff2|woff|ttf)$/,
            handler: 'CacheFirst',
            options: {
              cacheName: 'fonts-cache',
              expiration: {
                maxEntries: 10,
                maxAgeSeconds: 31536000 // 1 año
              }
            }
          },
          {
            urlPattern: /\.(png|jpg|jpeg|svg|gif)$/,
            handler: 'CacheFirst',
            options: {
              cacheName: 'images-cache',
              expiration: {
                maxEntries: 50,
                maxAgeSeconds: 2592000 // 30 días
              }
            }
          }
        ]
      }
    })
  ]
};
```

**Firebase Offline Persistence** (`src/firebase.ts`):
```typescript
import { initializeApp } from 'firebase/app';
import { getFirestore, enableIndexedDbPersistence } from 'firebase/firestore';

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

// Habilitar persistencia offline
enableIndexedDbPersistence(db)
  .catch((err) => {
    if (err.code === 'failed-precondition') {
      // Multiple tabs open, persistence can only be enabled in one tab at a time
      console.warn('Offline persistence failed: multiple tabs open');
    } else if (err.code === 'unimplemented') {
      // Browser doesn't support persistence
      console.warn('Offline persistence not supported by browser');
    }
  });
```

### Frontend Components

#### 1. Authentication Module

**AuthProvider** (Context con Firebase Auth)
```typescript
interface AuthContextType {
  user: User | null;
  isAdmin: boolean;
  loginWithGoogle: () => Promise<void>;
  loginWithEmail: (email: string, password: string) => Promise<void>;
  register: (email: string, password: string) => Promise<void>;
  logout: () => Promise<void>;
  isAuthenticated: boolean;
}

// Implementación con Firebase Auth
import { 
  getAuth, 
  signInWithPopup, 
  GoogleAuthProvider,
  signInWithEmailAndPassword,
  createUserWithEmailAndPassword,
  signOut
} from 'firebase/auth';

const auth = getAuth();
const googleProvider = new GoogleAuthProvider();
```

**LoginPage** (Component)
- Google OAuth button con estilo iOS (signInWithPopup)
- Email/password form con inputs estilo Apple
- Manejo de errores de autenticación
- Animaciones suaves (iOS-style transitions)

#### 2. Matches Module

**MatchList** (Component con Firestore realtime listener)
```typescript
interface MatchListProps {
  matches: Match[];
  onSelectMatch: (matchId: string) => void;
}

// Implementación con Firestore
import { collection, query, orderBy, onSnapshot } from 'firebase/firestore';

const matchesRef = collection(db, 'matches');
const q = query(matchesRef, orderBy('matchDate', 'asc'));

// Realtime listener
onSnapshot(q, (snapshot) => {
  const matches = snapshot.docs.map(doc => ({
    id: doc.id,
    ...doc.data()
  }));
});
```

**MatchCard** (Component)
```typescript
interface MatchCardProps {
  match: Match;
  userPrediction?: Prediction;
  onClick: () => void;
}
```

Muestra:
- Equipos, fecha, hora
- Estado (pendiente/en curso/finalizado)
- Indicador si usuario ya predijo
- Resultado final si está disponible
- Estilo: Card con border radius 12px, sombra sutil, glassmorphism opcional

#### 3. Predictions Module

**PredictionForm** (Component con Firestore)
```typescript
interface PredictionFormProps {
  match: Match;
  existingPrediction?: Prediction;
  onSubmit: (prediction: PredictionInput) => Promise<void>;
  onCancel: () => void;
}

// Implementación con Firestore
import { doc, setDoc, updateDoc, serverTimestamp } from 'firebase/firestore';

// Crear predicción
const predictionRef = doc(collection(db, 'predictions'));
await setDoc(predictionRef, {
  userId: currentUser.uid,
  matchId: match.id,
  homeScore: homeScore,
  awayScore: awayScore,
  pointsEarned: null,
  createdAt: serverTimestamp(),
  updatedAt: serverTimestamp()
});

// Actualizar predicción
await updateDoc(predictionRef, {
  homeScore: newHomeScore,
  awayScore: newAwayScore,
  updatedAt: serverTimestamp()
});
```

Validaciones:
- Marcadores >= 0
- Deadline no alcanzado (validado en Security Rules también)
- Números enteros
- Estilo: Modal sheet-style de iOS, inputs con estilo Apple, botones con haptic feedback visual

**PredictionList** (Component con Firestore realtime)
```typescript
// Query de predicciones del usuario
import { collection, query, where, onSnapshot } from 'firebase/firestore';

const predictionsRef = collection(db, 'predictions');
const q = query(predictionsRef, where('userId', '==', currentUser.uid));

onSnapshot(q, (snapshot) => {
  const predictions = snapshot.docs.map(doc => ({
    id: doc.id,
    ...doc.data()
  }));
});
```
- Muestra predicciones del usuario en tiempo real
- Resultado real vs predicción
- Puntos obtenidos por cada match

#### 4. Ranking Module

**RankingTable** (Component con Firestore realtime)
```typescript
interface RankingTableProps {
  rankings: RankingEntry[];
  currentUserId: string;
}

interface RankingEntry {
  position: number;
  userId: string;
  userName: string;
  totalScore: number;
  isCurrentUser: boolean;
}

// Implementación con Firestore
import { collection, query, orderBy, limit, onSnapshot } from 'firebase/firestore';

const usersRef = collection(db, 'users');
const q = query(
  usersRef, 
  orderBy('totalScore', 'desc'),
  orderBy('name', 'asc'),
  limit(100) // Top 100 usuarios
);

onSnapshot(q, (snapshot) => {
  const rankings = snapshot.docs.map((doc, index) => ({
    position: index + 1,
    userId: doc.id,
    userName: doc.data().name,
    totalScore: doc.data().totalScore,
    isCurrentUser: doc.id === currentUser.uid
  }));
});
```

Estilo: Lista agrupada estilo iOS, highlight del usuario actual con color accent, animaciones de transición suaves, actualización en tiempo real

#### 5. Admin Module

**AdminPanel** (Component con Firebase Functions)
```typescript
// Implementación con Firebase Callable Functions
import { getFunctions, httpsCallable } from 'firebase/functions';

const functions = getFunctions();

// Sincronización manual
const syncMatches = httpsCallable(functions, 'syncMatches');
const handleSync = async () => {
  try {
    const result = await syncMatches();
    // Mostrar success message
  } catch (error) {
    // Mostrar error message
  }
};

// Actualización manual de match
const updateMatch = httpsCallable(functions, 'updateMatch');
const handleUpdateMatch = async (matchId: string, homeScore: number, awayScore: number) => {
  try {
    const result = await updateMatch({ matchId, homeScore, awayScore });
    // Mostrar success message
  } catch (error) {
    // Mostrar error message
  }
};
```

- Botón de sincronización manual con estilo iOS
- Formulario de carga manual de resultados
- Configuración de scheduled job (actualiza documento en config/scheduler)
- Solo visible para administradores (verificado con custom claims)
- Estilo: Cards con glassmorphism, botones con estados claros (loading, success, error)

#### 6. PWA Components

**InstallPrompt** (Component)
- Detecta si la app es instalable
- Muestra banner de instalación estilo iOS
- Maneja evento beforeinstallprompt
- Se oculta después de instalación

**OfflineIndicator** (Component)
- Muestra estado de conexión
- Indicador visual cuando está offline
- Toast notification al recuperar conexión

**UpdatePrompt** (Component)
- Detecta cuando hay nueva versión disponible
- Prompt para recargar y actualizar
- Estilo: Alert sheet de iOS

### Backend Components

#### 1. Firebase Functions (Cloud Functions)

```typescript
// Auth functions (HTTP)
// Firebase Auth maneja automáticamente Google OAuth y Email/Password
// Solo necesitamos functions para lógica custom

// Matches functions
exports.syncMatches = functions.https.onCall(async (data, context) => {
  // Admin only - sync matches from API
  // Verifica context.auth.token.admin
});

exports.updateMatch = functions.https.onCall(async (data, context) => {
  // Admin only - update match manually
});

// Scheduled functions
exports.scheduledSync = functions.pubsub
  .schedule('0 0 * * *') // Daily at midnight
  .timeZone('America/Argentina/Buenos_Aires')
  .onRun(async (context) => {
    // Sync results from API-Football
    // Calculate scores for finished matches
  });

// Firestore triggers
exports.onMatchFinished = functions.firestore
  .document('matches/{matchId}')
  .onUpdate(async (change, context) => {
    // When match status changes to FINISHED
    // Calculate points for all predictions
  });

exports.onPredictionScored = functions.firestore
  .document('predictions/{predictionId}')
  .onUpdate(async (change, context) => {
    // When prediction.pointsEarned is updated
    // Update user.totalScore
  });
```

#### 2. Firestore Security Rules

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Helper functions
    function isAuthenticated() {
      return request.auth != null;
    }
    
    function isAdmin() {
      return isAuthenticated() && 
             get(/databases/$(database)/documents/users/$(request.auth.uid)).data.isAdmin == true;
    }
    
    function isOwner(userId) {
      return isAuthenticated() && request.auth.uid == userId;
    }
    
    function matchDeadlineNotPassed(matchId) {
      let match = get(/databases/$(database)/documents/matches/$(matchId));
      return request.time < match.data.deadline;
    }
    
    // Users collection
    match /users/{userId} {
      allow read: if isAuthenticated();
      allow create: if isAuthenticated() && isOwner(userId);
      allow update: if isOwner(userId) && 
                      (!request.resource.data.diff(resource.data).affectedKeys().hasAny(['isAdmin', 'totalScore']));
      // isAdmin y totalScore solo pueden ser modificados por functions
    }
    
    // Matches collection
    match /matches/{matchId} {
      allow read: if isAuthenticated();
      allow write: if isAdmin();
    }
    
    // Predictions collection
    match /predictions/{predictionId} {
      allow read: if isAuthenticated() && 
                    (isOwner(resource.data.userId) || isAdmin());
      allow create: if isAuthenticated() && 
                      isOwner(request.resource.data.userId) &&
                      matchDeadlineNotPassed(request.resource.data.matchId) &&
                      request.resource.data.homeScore >= 0 &&
                      request.resource.data.awayScore >= 0;
      allow update: if isAuthenticated() && 
                      isOwner(resource.data.userId) &&
                      matchDeadlineNotPassed(resource.data.matchId) &&
                      request.resource.data.homeScore >= 0 &&
                      request.resource.data.awayScore >= 0 &&
                      !request.resource.data.diff(resource.data).affectedKeys().hasAny(['pointsEarned']);
      // pointsEarned solo puede ser modificado por functions
      allow delete: if isOwner(resource.data.userId) && 
                      matchDeadlineNotPassed(resource.data.matchId);
    }
    
    // Config collection
    match /config/{document} {
      allow read: if isAuthenticated();
      allow write: if isAdmin();
    }
  }
}
```

#### 3. Service Layer (Firebase Functions)

**AuthService** (manejado por Firebase Auth)
```typescript
// Firebase Auth maneja automáticamente:
// - Google OAuth
// - Email/Password registration y login
// - Token generation y validation
// - Session management

// Custom claims para admin
interface AdminService {
  setAdminRole(uid: string): Promise<void>;
  verifyAdmin(uid: string): Promise<boolean>;
}
```

**MatchService**
```typescript
interface MatchService {
  getAllMatches(): Promise<Match[]>;
  getMatchById(id: string): Promise<Match | null>;
  syncMatchesFromAPI(): Promise<Match[]>;
  updateMatchResult(id: string, homeScore: number, awayScore: number): Promise<Match>;
  isDeadlinePassed(match: Match): boolean;
}
```

**PredictionService**
```typescript
interface PredictionService {
  // Las operaciones CRUD son manejadas directamente por Firestore desde el frontend
  // con Security Rules validando permisos
  
  // Solo necesitamos lógica de validación en functions
  validatePrediction(matchId: string, homeScore: number, awayScore: number): Promise<void>;
}
```

**ScoringService**
```typescript
interface ScoringService {
  calculatePointsForMatch(matchId: string): Promise<void>;
  calculatePoints(prediction: Prediction, actualResult: MatchResult): number;
  updateUserScore(userId: string, pointsDelta: number): Promise<void>;
  recalculateAllScores(): Promise<void>;
}
```

**RankingService**
```typescript
interface RankingService {
  // Ranking se obtiene directamente con query de Firestore
  // ordenado por totalScore descendente
  // No necesita service layer, se hace desde frontend
}
```

**FootballAPIService**
```typescript
interface FootballAPIService {
  fetchWorldCupFixtures(): Promise<APIMatch[]>;
  fetchMatchResults(): Promise<APIMatchResult[]>;
  validateAPIResponse(data: any): boolean;
}
```

**SchedulerService**
```typescript
interface SchedulerService {
  // Cloud Scheduler se configura via Firebase Console o CLI
  // No necesita código, solo configuración
  
  isWorldCupPeriod(): boolean;
  updateScheduleConfig(config: SchedulerConfig): Promise<void>;
}
```

## Data Models

### Firestore Collections Structure

```
firestore/
├── users/
│   └── {userId}/
│       ├── email: string
│       ├── name: string
│       ├── photoURL: string | null
│       ├── isAdmin: boolean
│       ├── totalScore: number
│       ├── createdAt: timestamp
│       └── updatedAt: timestamp
│
├── matches/
│   └── {matchId}/
│       ├── homeTeam: string
│       ├── awayTeam: string
│       ├── homeScore: number | null
│       ├── awayScore: number | null
│       ├── matchDate: timestamp
│       ├── deadline: timestamp
│       ├── status: 'PENDING' | 'IN_PROGRESS' | 'FINISHED'
│       ├── externalId: string | null
│       ├── createdAt: timestamp
│       └── updatedAt: timestamp
│
├── predictions/
│   └── {predictionId}/
│       ├── userId: string (reference to users/{userId})
│       ├── matchId: string (reference to matches/{matchId})
│       ├── homeScore: number
│       ├── awayScore: number
│       ├── pointsEarned: number | null
│       ├── createdAt: timestamp
│       └── updatedAt: timestamp
│
└── config/
    └── scheduler/
        ├── cronExpression: string
        ├── isActive: boolean
        ├── worldCupStart: timestamp
        ├── worldCupEnd: timestamp
        └── updatedAt: timestamp
```

### Firestore Indexes

```javascript
// Composite indexes necesarios para queries eficientes

// Index 1: Predictions por usuario
// Collection: predictions
// Fields: userId (Ascending), createdAt (Descending)

// Index 2: Predictions por match
// Collection: predictions
// Fields: matchId (Ascending), createdAt (Descending)

// Index 3: Ranking (usuarios ordenados por score)
// Collection: users
// Fields: totalScore (Descending), name (Ascending)

// Index 4: Matches por fecha
// Collection: matches
// Fields: matchDate (Ascending), status (Ascending)

// Index 5: Predictions por usuario y match (para unique constraint)
// Collection: predictions
// Fields: userId (Ascending), matchId (Ascending)
```

### TypeScript Interfaces

```typescript
interface User {
  id: string;
  email: string;
  name: string;
  photoURL: string | null;
  isAdmin: boolean;
  totalScore: number;
  createdAt: Date;
  updatedAt: Date;
}

interface Match {
  id: string;
  homeTeam: string;
  awayTeam: string;
  homeScore: number | null;
  awayScore: number | null;
  matchDate: Date;
  deadline: Date;
  status: 'PENDING' | 'IN_PROGRESS' | 'FINISHED';
  externalId: string | null;
  createdAt: Date;
  updatedAt: Date;
}

interface Prediction {
  id: string;
  userId: string;
  matchId: string;
  homeScore: number;
  awayScore: number;
  pointsEarned: number | null;
  createdAt: Date;
  updatedAt: Date;
}

interface SchedulerConfig {
  cronExpression: string;
  isActive: boolean;
  worldCupStart: Date;
  worldCupEnd: Date;
  updatedAt: Date;
}
```

### Data Relationships

- User 1:N Prediction (un usuario tiene muchas predicciones)
- Match 1:N Prediction (un match tiene muchas predicciones)
- User.totalScore se calcula sumando Prediction.pointsEarned
- Unique constraint en (userId, matchId) se implementa via Security Rules y validación en frontend

### Data Validation Rules

1. **User**:
   - Email debe ser válido y único (manejado por Firebase Auth)
   - isAdmin default false
   - totalScore solo modificable por Cloud Functions

2. **Match**:
   - homeScore y awayScore deben ser >= 0 cuando no son null
   - deadline debe ser igual a matchDate
   - status debe transicionar: PENDING → IN_PROGRESS → FINISHED

3. **Prediction**:
   - homeScore y awayScore deben ser >= 0
   - No se puede crear/modificar si deadline pasó (validado en Security Rules)
   - Unique constraint en (userId, matchId) validado en frontend antes de crear
   - pointsEarned solo modificable por Cloud Functions

4. **SchedulerConfig**:
   - cronExpression debe ser válido
   - worldCupStart < worldCupEnd

### Firestore vs PostgreSQL: Diferencias Clave

**Ventajas de Firestore**:
- Queries en tiempo real (listeners)
- Escalabilidad automática
- Integración nativa con Firebase Auth
- Security Rules a nivel de base de datos
- Offline persistence automática
- No necesita ORM

**Consideraciones**:
- NoSQL: No hay joins, se desnormalizan datos cuando es necesario
- Transacciones limitadas a 500 documentos
- Queries más limitadas que SQL (no hay OR, IN limitado)
- Costo basado en lecturas/escrituras (optimizar queries)

## Frontend Architecture

### Dependencias Frontend

**Core**:
- `react` ^18.0.0
- `react-dom` ^18.0.0
- `vite` ^5.0.0
- `typescript` ^5.0.0

**Firebase**:
- `firebase` ^10.0.0 (SDK completo: Auth, Firestore, Functions)

**PWA**:
- `vite-plugin-pwa` ^0.17.0
- `workbox-window` ^7.0.0

**Routing & State**:
- `react-router-dom` ^6.0.0
- `zustand` ^4.0.0 (state management ligero)

**Styling**:
- `styled-components` ^6.0.0 (CSS-in-JS para design system)

**Utils**:
- `date-fns` ^3.0.0 (manejo de fechas)

**Testing**:
- `vitest` ^1.0.0
- `@testing-library/react` ^14.0.0
- `@testing-library/jest-dom` ^6.0.0
- `fast-check` ^3.0.0

**Package Manager**:
- `bun` (en lugar de npm)

### Configuración de Bun

**Instalación de dependencias**:
```bash
bun install
```

**Scripts en package.json**:
```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "lint": "eslint . --ext ts,tsx",
    "format": "prettier --write \"src/**/*.{ts,tsx}\""
  }
}
```

**Ventajas de Bun**:
- Instalación de dependencias 10-20x más rápida que npm
- Ejecución de scripts más rápida
- Compatible con npm packages
- Built-in test runner (alternativa a Vitest)
- Menor uso de disco (lockfile más pequeño)

### Design System Structure

```
src/
├── design-system/
│   ├── tokens/
│   │   ├── colors.ts          # Paleta de colores Apple
│   │   ├── typography.ts      # SF Pro fonts y escalas
│   │   ├── spacing.ts         # Sistema de espaciado 4px
│   │   ├── shadows.ts         # Sombras iOS-style
│   │   └── animations.ts      # Transiciones suaves
│   ├── components/
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.styles.ts
│   │   │   └── Button.test.tsx
│   │   ├── Card/
│   │   │   ├── Card.tsx
│   │   │   ├── Card.styles.ts
│   │   │   └── Card.test.tsx
│   │   ├── Input/
│   │   │   ├── Input.tsx
│   │   │   ├── Input.styles.ts
│   │   │   └── Input.test.tsx
│   │   ├── Modal/
│   │   │   ├── Modal.tsx
│   │   │   ├── Modal.styles.ts
│   │   │   └── Modal.test.tsx
│   │   └── List/
│   │       ├── List.tsx
│   │       ├── List.styles.ts
│   │       └── List.test.tsx
│   ├── hooks/
│   │   ├── useTheme.ts        # Light/Dark mode
│   │   ├── useMediaQuery.ts   # Responsive breakpoints
│   │   └── useHaptic.ts       # Visual haptic feedback
│   └── utils/
│       ├── glassmorphism.ts   # Glassmorphism effects
│       └── transitions.ts     # iOS-style transitions
```

### PWA Structure

```
src/
├── firebase/
│   ├── config.ts                   # Firebase configuration
│   ├── auth.ts                     # Auth helpers
│   ├── firestore.ts                # Firestore helpers
│   └── functions.ts                # Callable functions
├── pwa/
│   ├── hooks/
│   │   ├── useOnlineStatus.ts      # Detecta conexión
│   │   ├── useInstallPrompt.ts     # Maneja instalación
│   │   └── useServiceWorker.ts     # SW lifecycle
│   ├── components/
│   │   ├── InstallPrompt.tsx       # Banner de instalación
│   │   ├── OfflineIndicator.tsx    # Indicador offline
│   │   └── UpdatePrompt.tsx        # Prompt de actualización
│   └── utils/
│       └── cache.ts                # Cache helpers
└── registerSW.ts                   # SW registration
```

### PWA Assets

```
public/
├── manifest.json
├── robots.txt
├── icons/
│   ├── icon-72x72.png
│   ├── icon-96x96.png
│   ├── icon-128x128.png
│   ├── icon-144x144.png
│   ├── icon-152x152.png
│   ├── icon-192x192.png
│   ├── icon-384x384.png
│   └── icon-512x512.png
└── splash/
    ├── splash-640x1136.png
    ├── splash-750x1334.png
    ├── splash-1242x2208.png
    └── splash-1125x2436.png
```

### Responsive Breakpoints (Apple-style)

```typescript
const breakpoints = {
  mobile: '0px',        // iPhone SE
  tablet: '744px',      // iPad Mini
  desktop: '1024px',    // iPad Pro / Desktop
  wide: '1440px'        // Large Desktop
};
```

### Theme Configuration

```typescript
interface Theme {
  mode: 'light' | 'dark';
  colors: {
    primary: string;
    secondary: string;
    success: string;
    warning: string;
    error: string;
    background: string;
    surface: string;
    textPrimary: string;
    textSecondary: string;
  };
  typography: {
    fontFamily: string;
    fontSize: {
      xs: string;
      sm: string;
      md: string;
      lg: string;
      xl: string;
      xxl: string;
    };
    fontWeight: {
      regular: number;
      medium: number;
      semibold: number;
      bold: number;
    };
  };
  spacing: {
    xs: string;
    sm: string;
    md: string;
    lg: string;
    xl: string;
    xxl: string;
  };
  borderRadius: {
    sm: string;
    md: string;
    lg: string;
  };
  shadows: {
    sm: string;
    md: string;
    lg: string;
  };
}
```

### Accesibilidad y Performance

**Accesibilidad (WCAG 2.1 AA)**:
- Contraste de colores mínimo 4.5:1 para texto normal
- Contraste de colores mínimo 3:1 para texto grande y elementos UI
- Soporte completo de navegación por teclado
- ARIA labels en todos los componentes interactivos
- Focus indicators visibles (outline estilo iOS)
- Soporte de screen readers
- Respeto de preferencias de sistema (prefers-reduced-motion, prefers-color-scheme)

**Performance**:
- Lazy loading de componentes con React.lazy()
- Code splitting por rutas
- Optimización de imágenes (WebP con fallback)
- Fonts con font-display: swap
- Service Worker con precaching de assets críticos
- Debouncing en inputs de búsqueda/filtrado
- Virtual scrolling para listas largas (ranking con muchos usuarios)
- Memoización de componentes pesados con React.memo()

**Métricas Target**:
- First Contentful Paint (FCP): < 1.8s
- Largest Contentful Paint (LCP): < 2.5s
- Time to Interactive (TTI): < 3.8s
- Cumulative Layout Shift (CLS): < 0.1
- First Input Delay (FID): < 100ms
- Lighthouse Score: > 90 (Performance, Accessibility, Best Practices, SEO)


## Correctness Properties

*Una propiedad es una característica o comportamiento que debe mantenerse verdadero en todas las ejecuciones válidas de un sistema - esencialmente, una declaración formal sobre lo que el sistema debe hacer. Las propiedades sirven como puente entre especificaciones legibles por humanos y garantías de corrección verificables por máquinas.*

### Property 1: Email capturado en autenticación OAuth

*Para cualquier* token de Google OAuth válido, cuando un usuario se autentica, el sistema debe extraer y almacenar el email del usuario en la base de datos.

**Validates: Requirements 1.2**

### Property 2: Round-trip de autenticación con credenciales

*Para cualquier* email y contraseña válidos, si un usuario se registra con esas credenciales, entonces debe poder autenticarse exitosamente usando las mismas credenciales.

**Validates: Requirements 1.4**

### Property 3: Rechazo de emails duplicados

*Para cualquier* email ya registrado en el sistema, intentar registrar un nuevo usuario con ese mismo email debe resultar en un error y no crear un usuario duplicado.

**Validates: Requirements 1.5**

### Property 4: Persistencia de sesión con JWT

*Para cualquier* token JWT válido generado por el sistema, mientras el token no haya expirado, debe mantener la sesión del usuario activa y permitir acceso a recursos protegidos.

**Validates: Requirements 1.6**

### Property 5: Identificación correcta de rol admin

*Para cualquier* usuario autenticado, el sistema debe identificar correctamente si tiene rol de administrador basándose en el campo isAdmin de la base de datos.

**Validates: Requirements 1.1.2**

### Property 6: Restricción de endpoints administrativos

*Para cualquier* usuario sin rol de administrador, los intentos de acceder a endpoints administrativos deben ser rechazados con código de estado 403.

**Validates: Requirements 1.1.3**

### Property 7: Completitud de información de matches

*Para cualquier* conjunto de matches en la base de datos, al solicitar la lista de matches, todos deben incluir homeTeam, awayTeam, matchDate, deadline y status.

**Validates: Requirements 2.1**

### Property 8: Indicador de predicción existente

*Para cualquier* match y usuario, el sistema debe indicar correctamente si existe una predicción del usuario para ese match (true si existe, false si no).

**Validates: Requirements 2.2**

### Property 9: Visibilidad de estado de match

*Para cualquier* match, su estado (PENDING, IN_PROGRESS, FINISHED) debe ser visible y corresponder al estado almacenado en la base de datos.

**Validates: Requirements 2.3**

### Property 10: Visibilidad de resultado final

*Para cualquier* match con status FINISHED, los campos homeScore y awayScore deben ser no-null y visibles en la respuesta.

**Validates: Requirements 2.4**

### Property 11: Creación de predicción antes de deadline

*Para cualquier* match cuyo deadline no ha sido alcanzado, debe ser posible crear una predicción válida con marcadores enteros no negativos.

**Validates: Requirements 3.1**

### Property 12: Modificación de predicción antes de deadline

*Para cualquier* predicción existente cuyo match no ha alcanzado el deadline, debe ser posible modificar los marcadores de la predicción.

**Validates: Requirements 3.2**

### Property 13: Bloqueo de predicciones después de deadline

*Para cualquier* match cuyo deadline ha sido alcanzado o superado, los intentos de crear o modificar predicciones deben ser rechazados con un error.

**Validates: Requirements 3.3, 10.1, 10.2, 10.3, 10.4**

### Property 14: Validación de marcadores no negativos

*Para cualquier* intento de crear o modificar una predicción, si los marcadores no son números enteros no negativos, la operación debe ser rechazada con un error de validación.

**Validates: Requirements 3.4**

### Property 15: Puntos por resultado exacto

*Para cualquier* predicción donde homeScore y awayScore coinciden exactamente con el resultado final del match, el sistema debe otorgar exactamente 3 puntos.

**Validates: Requirements 4.1, 4.2**

### Property 16: Actualización inmediata de score total

*Para cualquier* cálculo de puntos de una predicción, el campo totalScore del usuario debe actualizarse inmediatamente sumando los puntos obtenidos.

**Validates: Requirements 4.3, 8.6**

### Property 17: Puntos por acertar ganador sin resultado exacto

*Para cualquier* predicción donde el ganador predicho coincide con el ganador real pero el resultado no es exacto, el sistema debe otorgar exactamente 1 punto. Esto incluye empates: si ambos resultados son empate pero los marcadores difieren, se otorga 1 punto.

**Validates: Requirements 5.1, 5.2, 5.3**

### Property 18: No acumulación de puntos exacto + ganador

*Para cualquier* predicción que coincide exactamente con el resultado final, el sistema debe otorgar únicamente 3 puntos, no 4 puntos (no debe sumar 3 por exacto + 1 por ganador).

**Validates: Requirements 5.4**

### Property 19: Ordenamiento descendente de ranking

*Para cualquier* conjunto de usuarios con scores, el ranking debe estar ordenado de mayor a menor score, donde usuarios con mayor puntaje aparecen primero.

**Validates: Requirements 6.1**

### Property 20: Completitud de información en ranking

*Para cualquier* usuario en el ranking, debe mostrarse su nombre (name) y score total (totalScore).

**Validates: Requirements 6.2**

### Property 21: Actualización de ranking tras cálculo de puntos

*Para cualquier* cálculo de puntos que modifica el totalScore de usuarios, al solicitar el ranking inmediatamente después, debe reflejar los scores actualizados.

**Validates: Requirements 6.3**

### Property 22: Cálculo correcto de posición en ranking

*Para cualquier* usuario, su posición en el ranking debe ser igual a 1 + (cantidad de usuarios con score estrictamente mayor que el suyo).

**Validates: Requirements 6.4**

### Property 23: Recuperación completa de predicciones propias

*Para cualquier* usuario, al solicitar sus predicciones, debe recibir todas las predicciones que ha creado, sin omisiones.

**Validates: Requirements 7.1**

### Property 24: Visibilidad de puntos en predicciones de matches finalizados

*Para cualquier* predicción cuyo match tiene status FINISHED, el campo pointsEarned debe ser no-null y visible.

**Validates: Requirements 7.2**

### Property 25: Visibilidad de resultado real junto a predicción

*Para cualquier* predicción cuyo match tiene status FINISHED, el resultado real (homeScore y awayScore del match) debe estar disponible junto a la predicción.

**Validates: Requirements 7.3**

### Property 26: Score total como suma de puntos

*Para cualquier* usuario, su totalScore debe ser igual a la suma de todos los valores pointsEarned de sus predicciones (considerando null como 0).

**Validates: Requirements 7.4**

### Property 27: Actualización de matches desde API

*Para cualquier* resultado válido obtenido de la API externa (con marcadores enteros no negativos), el match correspondiente en la base de datos debe actualizarse con el nuevo estado y marcadores.

**Validates: Requirements 8.3**

### Property 28: Cálculo de puntos tras actualización de match

*Para cualquier* match que se actualiza a status FINISHED, todas las predicciones asociadas a ese match deben ser evaluadas y sus pointsEarned calculados.

**Validates: Requirements 8.4**

### Property 29: Validación de marcadores de API

*Para cualquier* respuesta de la API externa, si los marcadores no son números enteros no negativos, la actualización debe ser rechazada y los datos existentes deben mantenerse sin cambios.

**Validates: Requirements 8.5**

### Property 30: Preservación de datos ante error de sincronización

*Para cualquier* error durante la sincronización con la API externa, los datos de matches en la base de datos deben permanecer sin cambios (no debe haber actualizaciones parciales o corrupción de datos).

**Validates: Requirements 8.8**

### Property 31: Actualización de configuración de scheduler

*Para cualquier* expresión cron válida proporcionada por un administrador, el sistema debe actualizar la configuración del scheduled job y aplicar el nuevo horario.

**Validates: Requirements 8.1.8**

### Property 32: Almacenamiento completo de datos de match

*Para cualquier* match obtenido de la API externa durante la carga de fixture, deben almacenarse homeTeam, awayTeam, matchDate y deadline (donde deadline = matchDate).

**Validates: Requirements 9.2**

### Property 33: Validación de fechas de matches

*Para cualquier* match obtenido de la API externa, si la fecha (matchDate) no es válida o está en formato incorrecto, el match debe ser rechazado y no almacenado.

**Validates: Requirements 9.3**

### Property 34: Deadline igual a fecha de inicio

*Para cualquier* match almacenado en el sistema, el campo deadline debe ser exactamente igual al campo matchDate.

**Validates: Requirements 9.4**

## Error Handling

### Error Categories

#### 1. Authentication Errors

**Firebase Auth Failures**
- Error: Token inválido o expirado
- Firebase Error Code: auth/invalid-credential, auth/user-token-expired
- Message: "Sesión expirada, por favor inicia sesión nuevamente"
- Action: Redirigir a login

**Duplicate Email Registration**
- Error: Email ya existe
- Firebase Error Code: auth/email-already-in-use
- Message: "El email ya está registrado"
- Action: Sugerir login o recuperación de contraseña

**Invalid Credentials**
- Error: Email/password incorrectos
- Firebase Error Code: auth/wrong-password, auth/user-not-found
- Message: "Credenciales inválidas"
- Action: Permitir reintento

**Weak Password**
- Error: Contraseña débil
- Firebase Error Code: auth/weak-password
- Message: "La contraseña debe tener al menos 6 caracteres"
- Action: Mostrar requisitos de contraseña

#### 2. Authorization Errors

**Non-Admin Access to Admin Functions**
- Error: Usuario sin rol admin intenta llamar función administrativa
- Firebase Error Code: functions/permission-denied
- Message: "Acceso denegado: se requieren permisos de administrador"
- Action: Registrar intento en Cloud Logging

**Firestore Security Rules Violation**
- Error: Intento de operación no permitida por Security Rules
- Firebase Error Code: permission-denied
- Message: "No tienes permisos para realizar esta operación"
- Action: Verificar permisos del usuario

#### 3. Validation Errors

**Invalid Prediction Scores**
- Error: Marcadores negativos o no enteros
- Validation: Frontend + Firestore Security Rules
- Message: "Los marcadores deben ser números enteros no negativos"
- Action: Mostrar error en formulario

**Prediction After Deadline**
- Error: Intento de crear/modificar predicción después de deadline
- Validation: Firestore Security Rules
- Firebase Error Code: permission-denied
- Message: "El plazo para predecir este partido ha expirado"
- Action: Deshabilitar formulario

**Missing Required Fields**
- Error: Campos requeridos faltantes
- Validation: Frontend + Firestore Security Rules
- Message: "Campos requeridos: [lista de campos]"
- Action: Validación en frontend

**Duplicate Prediction**
- Error: Usuario intenta crear segunda predicción para mismo match
- Validation: Frontend verifica antes de crear
- Message: "Ya existe una predicción para este partido"
- Action: Sugerir modificar predicción existente

#### 4. External API Errors

**Football API Unavailable**
- Error: API externa no responde o timeout
- Handling: En Firebase Function
- Message: "Servicio de sincronización temporalmente no disponible"
- Action: Mantener datos existentes, reintentar más tarde
- Logging: Cloud Logging con timestamp

**Invalid API Response**
- Error: Respuesta de API con formato inválido
- Handling: En Firebase Function
- Message: "Error al procesar datos de la API externa"
- Action: No actualizar datos, notificar a admin via Cloud Logging
- Logging: Registrar respuesta completa

**API Rate Limiting**
- Error: Exceso de requests a API externa
- Handling: En Firebase Function
- Message: "Límite de sincronizaciones alcanzado, intente más tarde"
- Action: Implementar backoff exponencial en Cloud Scheduler

#### 5. Firestore Errors

**Connection Failures**
- Error: No se puede conectar a Firestore
- Firebase Error Code: unavailable
- Message: "Error de conexión, reintentando..."
- Action: Firestore SDK reintenta automáticamente
- Logging: Cloud Logging

**Permission Denied**
- Error: Security Rules rechazan operación
- Firebase Error Code: permission-denied
- Message: "No tienes permisos para realizar esta operación"
- Action: Verificar autenticación y permisos

**Transaction Failures**
- Error: Fallo en transacción (ej: cálculo de puntos)
- Firebase Error Code: aborted, failed-precondition
- Message: "Error al procesar operación"
- Action: Firestore reintenta automáticamente hasta 5 veces
- Logging: Cloud Logging con detalles de transacción

**Quota Exceeded**
- Error: Límite de lecturas/escrituras excedido
- Firebase Error Code: resource-exhausted
- Message: "Límite de operaciones excedido"
- Action: Implementar rate limiting, optimizar queries

#### 6. Business Logic Errors

**Match Not Found**
- Error: ID de match no existe en Firestore
- Firebase: Document snapshot no existe
- Message: "Partido no encontrado"
- Action: Verificar ID en request

**Prediction Not Found**
- Error: ID de predicción no existe o no pertenece al usuario
- Firebase: Document snapshot no existe o Security Rules bloquean
- Message: "Predicción no encontrada"
- Action: Verificar ownership

**Match Already Finished**
- Error: Intento de modificar match finalizado
- Validation: En Firebase Function
- Message: "No se puede modificar un partido finalizado"
- Action: Solo permitir a admins con confirmación

**Cloud Function Timeout**
- Error: Function excede tiempo límite (60s default, 540s max)
- Firebase Error Code: deadline-exceeded
- Message: "Operación tomó demasiado tiempo"
- Action: Optimizar function, aumentar timeout si es necesario

### Error Handling Strategy

1. **Graceful Degradation**: Si la API externa falla, el sistema sigue funcionando con datos existentes
2. **Transactional Integrity**: Cálculos de puntos en Firestore transactions para evitar estados inconsistentes
3. **User-Friendly Messages**: Errores técnicos se traducen a mensajes comprensibles
4. **Comprehensive Logging**: Todos los errores se registran en Cloud Logging con contexto
5. **Retry Logic**: Errores transitorios (network, timeouts) se reintentan automáticamente por Firebase SDK
6. **Admin Notifications**: Errores críticos (API failures, quota exceeded) notifican a administradores via Cloud Logging
7. **Offline Resilience**: Firestore offline persistence permite operación sin conexión
8. **Security Rules Validation**: Validación a nivel de base de datos previene operaciones inválidas

## Testing Strategy

### Dual Testing Approach

El sistema implementará tanto unit tests como property-based tests para garantizar corrección completa:

- **Unit tests**: Verifican ejemplos específicos, casos edge y condiciones de error
- **Property tests**: Verifican propiedades universales a través de múltiples inputs generados

Ambos tipos de tests son complementarios y necesarios:
- Unit tests capturan bugs concretos y validan comportamientos específicos
- Property tests verifican corrección general y descubren edge cases inesperados

### Property-Based Testing Configuration

**Library**: fast-check (JavaScript/TypeScript property-based testing library)

**Configuration**:
- Mínimo 100 iteraciones por property test (debido a randomización)
- Cada test debe referenciar su propiedad del documento de diseño
- Formato de tag: `// Feature: prode-mundial, Property {number}: {property_text}`

**Ejemplo de Property Test**:
```typescript
import fc from 'fast-check';

// Feature: prode-mundial, Property 15: Puntos por resultado exacto
test('exact prediction earns 3 points', () => {
  fc.assert(
    fc.property(
      fc.integer({ min: 0, max: 10 }), // homeScore
      fc.integer({ min: 0, max: 10 }), // awayScore
      (homeScore, awayScore) => {
        const prediction = { homeScore, awayScore };
        const actualResult = { homeScore, awayScore };
        const points = calculatePoints(prediction, actualResult);
        expect(points).toBe(3);
      }
    ),
    { numRuns: 100 }
  );
});
```

### Unit Testing Strategy

**Framework**: Vitest + React Testing Library

**Coverage Areas**:

1. **Authentication Module**
   - Login con Google OAuth (mock de Firebase Auth)
   - Registro con email/password
   - Validación de email duplicado (mock de Firebase error)
   - Custom claims para admin

2. **Predictions Module**
   - Creación de predicción válida (mock de Firestore)
   - Modificación antes de deadline
   - Rechazo después de deadline (Security Rules)
   - Validación de marcadores negativos
   - Edge case: predicción en el momento exacto del deadline
   - Verificación de unique constraint (userId + matchId)

3. **Scoring Module**
   - Cálculo de 3 puntos por resultado exacto
   - Cálculo de 1 punto por ganador correcto
   - Cálculo de 1 punto por empate correcto
   - No acumulación de 3+1 puntos
   - Edge case: empate 0-0
   - Edge case: goleada (ej: 7-1)

4. **Ranking Module**
   - Ordenamiento correcto con scores variados
   - Manejo de empates en puntaje
   - Cálculo de posición
   - Edge case: todos con 0 puntos
   - Edge case: un solo usuario
   - Realtime updates con Firestore listeners

5. **API Integration**
   - Mock de respuestas exitosas de Football API
   - Manejo de errores (timeout, invalid response, rate limiting)
   - Validación de formato de respuesta

6. **Cloud Scheduler**
   - Configuración de scheduled function
   - Verificación de World Cup Period
   - Ejecución de sincronización programada (mock)

7. **PWA Features**
   - Service worker registration
   - Cache strategies (CacheFirst para assets)
   - Offline detection y fallback
   - Install prompt behavior
   - Update detection y prompt
   - Manifest validation
   - Firestore offline persistence

8. **Design System Components**
   - Button variants (filled, outlined, text)
   - Card rendering con glassmorphism
   - Input validation y estados
   - Modal sheet behavior
   - Theme switching (light/dark)
   - Responsive behavior

9. **Firebase Security Rules**
   - Test de Security Rules con Firebase Emulator
   - Verificación de permisos de lectura/escritura
   - Validación de deadline en predictions
   - Protección de campos admin-only (isAdmin, totalScore, pointsEarned)

### Integration Testing

**Scope**: End-to-end flows críticos con Firebase Emulator Suite

1. **User Journey: Hacer una predicción**
   - Login → Ver matches (Firestore listener) → Seleccionar match → Crear predicción (Firestore write) → Confirmar

2. **Admin Journey: Sincronizar resultados**
   - Login como admin → Trigger sync (Callable Function) → Verificar matches actualizados → Verificar puntos calculados (Firestore trigger) → Verificar ranking actualizado

3. **Scoring Flow**
   - Match finaliza (Firestore update) → Firestore trigger ejecuta → Puntos calculados → Scores actualizados → Ranking actualizado en realtime

4. **PWA Offline Flow**
   - Usuario online → Crea predicción → Pierde conexión → Predicción queda en Firestore offline queue → Recupera conexión → Predicción se sincroniza automáticamente

5. **PWA Install Flow**
   - Usuario visita app → Ve prompt de instalación → Instala → App se abre en standalone mode → Funciona offline con Firestore persistence

**Firebase Emulator Suite**:
```bash
# Instalar emulators
firebase init emulators

# Ejecutar emulators para testing
firebase emulators:start

# Emulators necesarios:
# - Authentication
# - Firestore
# - Functions
# - Hosting (opcional)
```

### Test Data Strategy

**Generators para Property Tests**:
```typescript
// Generador de usuarios
const userArb = fc.record({
  email: fc.emailAddress(),
  name: fc.string({ minLength: 1, maxLength: 50 }),
  isAdmin: fc.boolean(),
});

// Generador de matches
const matchArb = fc.record({
  homeTeam: fc.string({ minLength: 3, maxLength: 30 }),
  awayTeam: fc.string({ minLength: 3, maxLength: 30 }),
  matchDate: fc.date({ min: new Date('2024-01-01') }),
  status: fc.constantFrom('PENDING', 'IN_PROGRESS', 'FINISHED'),
});

// Generador de predicciones
const predictionArb = fc.record({
  homeScore: fc.integer({ min: 0, max: 15 }),
  awayScore: fc.integer({ min: 0, max: 15 }),
});

// Generador de resultados de API
const apiResultArb = fc.record({
  matchId: fc.uuid(),
  homeScore: fc.integer({ min: 0, max: 10 }),
  awayScore: fc.integer({ min: 0, max: 10 }),
  status: fc.constant('FINISHED'),
});
```

**Fixtures para Unit Tests**:
- Usuarios de prueba (admin y no-admin)
- Matches en diferentes estados
- Predicciones con diferentes resultados
- Respuestas mock de Football API
- Mock de Firebase Auth
- Mock de Firestore queries y writes

### Performance Testing

**Targets**:
- Firestore query response time: < 200ms para queries simples
- Firestore write latency: < 100ms
- Firebase Function cold start: < 2s
- Firebase Function warm execution: < 500ms
- Cálculo de puntos para 100 predicciones: < 1s (en Cloud Function)
- Generación de ranking con 1000 usuarios: < 500ms (query ordenada de Firestore)
- PWA First Load: < 3s
- PWA Subsequent Loads (cached): < 1s

**Tools**: 
- Firebase Performance Monitoring (integrado)
- Lighthouse para PWA metrics
- Artillery o k6 para load testing de Cloud Functions

### Security Testing

**Areas**:
1. Firestore Security Rules: Validar que reglas previenen accesos no autorizados
2. XSS: Validar sanitización de inputs en frontend
3. Firebase Auth Security: Validar expiración y renovación de tokens
4. Rate Limiting: Validar límites en Cloud Functions (Firebase App Check)
5. Admin Verification: Validar que custom claims de admin funcionan correctamente
6. HTTPS Only: Validar que Firebase Hosting fuerza HTTPS

**Firebase Security Tools**:
- Firebase Security Rules Unit Testing
- Firebase App Check para proteger contra abuse
- Cloud Functions CORS configuration

### Continuous Integration

**Pipeline**:
1. Lint (ESLint + Prettier)
2. Type Check (TypeScript)
3. Unit Tests (Vitest)
4. Property Tests (fast-check)
5. Integration Tests (con Firebase Emulator)
6. Security Rules Tests
7. Build (Vite)
8. Deploy to Firebase Hosting (staging)

**Firebase CI/CD**:
```bash
# Instalar Firebase CLI
bun add -g firebase-tools

# Login
firebase login

# Deploy
firebase deploy --only hosting,functions,firestore:rules
```

**Coverage Target**: 80% code coverage mínimo

### Test Organization

```
tests/
├── unit/
│   ├── auth/
│   │   ├── firebase-auth.test.ts
│   │   ├── email-auth.test.ts
│   │   └── admin-claims.test.ts
│   ├── predictions/
│   │   ├── create-prediction.test.ts
│   │   ├── update-prediction.test.ts
│   │   └── deadline-validation.test.ts
│   ├── scoring/
│   │   ├── calculate-points.test.ts
│   │   └── update-scores.test.ts
│   ├── ranking/
│   │   ├── generate-ranking.test.ts
│   │   └── calculate-position.test.ts
│   ├── pwa/
│   │   ├── service-worker.test.ts
│   │   ├── cache-strategies.test.ts
│   │   ├── offline-detection.test.ts
│   │   └── install-prompt.test.ts
│   ├── design-system/
│   │   ├── button.test.tsx
│   │   ├── card.test.tsx
│   │   ├── input.test.tsx
│   │   ├── modal.test.tsx
│   │   └── theme.test.ts
│   └── firebase/
│       ├── security-rules.test.ts
│       ├── firestore-queries.test.ts
│       └── functions.test.ts
├── property/
│   ├── auth.property.test.ts
│   ├── predictions.property.test.ts
│   ├── scoring.property.test.ts
│   └── ranking.property.test.ts
├── integration/
│   ├── user-prediction-flow.test.ts
│   ├── admin-sync-flow.test.ts
│   ├── scoring-flow.test.ts
│   └── pwa-offline-flow.test.ts
└── fixtures/
    ├── users.ts
    ├── matches.ts
    ├── api-responses.ts
    └── firebase-mocks.ts
```

### Property Test Implementation Requirements

Cada propiedad de corrección definida en este documento DEBE ser implementada como un ÚNICO property-based test. El test debe:

1. Usar fast-check para generar inputs aleatorios
2. Ejecutar mínimo 100 iteraciones
3. Incluir un comentario con el tag: `// Feature: prode-mundial, Property {N}: {descripción}`
4. Verificar la propiedad universal para todos los inputs generados

**Mapeo de Propiedades a Tests**:
- Property 1-6: `tests/property/auth.property.test.ts`
- Property 7-14: `tests/property/predictions.property.test.ts`
- Property 15-18: `tests/property/scoring.property.test.ts`
- Property 19-26: `tests/property/ranking.property.test.ts`
- Property 27-34: `tests/property/sync.property.test.ts`

## Firebase Configuration and Deployment

### Firebase Project Setup

**Inicialización del proyecto**:
```bash
# Instalar Firebase CLI con bun
bun add -g firebase-tools

# Login
firebase login

# Inicializar proyecto
firebase init

# Seleccionar servicios:
# - Hosting
# - Functions
# - Firestore
# - Authentication
# - Emulators
```

**Estructura de archivos Firebase**:
```
proyecto/
├── firebase.json              # Configuración principal
├── .firebaserc               # Alias de proyectos
├── firestore.rules           # Security Rules
├── firestore.indexes.json    # Índices de Firestore
├── functions/                # Cloud Functions
│   ├── package.json
│   ├── tsconfig.json
│   └── src/
│       ├── index.ts
│       ├── sync.ts
│       ├── scoring.ts
│       └── admin.ts
└── frontend/                 # App React
    └── dist/                 # Build output para Hosting
```

### firebase.json Configuration

```json
{
  "hosting": {
    "public": "frontend/dist",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ],
    "headers": [
      {
        "source": "**/*.@(js|css|woff2)",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "max-age=31536000"
          }
        ]
      }
    ]
  },
  "functions": {
    "source": "functions",
    "runtime": "nodejs20",
    "predeploy": ["bun run lint", "bun run build"]
  },
  "firestore": {
    "rules": "firestore.rules",
    "indexes": "firestore.indexes.json"
  },
  "emulators": {
    "auth": {
      "port": 9099
    },
    "functions": {
      "port": 5001
    },
    "firestore": {
      "port": 8080
    },
    "hosting": {
      "port": 5000
    },
    "ui": {
      "enabled": true,
      "port": 4000
    }
  }
}
```

### Environment Variables

**Frontend** (`.env`):
```bash
VITE_FIREBASE_API_KEY=your_api_key
VITE_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your_project_id
VITE_FIREBASE_STORAGE_BUCKET=your_project.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
VITE_FIREBASE_APP_ID=your_app_id
VITE_FOOTBALL_API_KEY=your_football_api_key
```

**Cloud Functions** (configurado via Firebase CLI):
```bash
firebase functions:config:set football.api_key="your_api_key"
firebase functions:config:set scheduler.timezone="America/Argentina/Buenos_Aires"
```

### Deployment Commands

**Deploy completo**:
```bash
# Build frontend
cd frontend
bun run build

# Deploy todo
firebase deploy
```

**Deploy selectivo**:
```bash
# Solo hosting
firebase deploy --only hosting

# Solo functions
firebase deploy --only functions

# Solo firestore rules
firebase deploy --only firestore:rules

# Solo indexes
firebase deploy --only firestore:indexes
```

**Deploy a staging**:
```bash
# Usar alias de proyecto
firebase use staging
firebase deploy

# Volver a producción
firebase use production
```

### Cloud Scheduler Setup

**Configuración via Firebase Console**:
1. Ir a Cloud Scheduler en Google Cloud Console
2. Crear job:
   - Name: `daily-results-sync`
   - Frequency: `0 0 * * *` (diario a medianoche)
   - Timezone: `America/Argentina/Buenos_Aires`
   - Target: Pub/Sub topic que triggerea la function
   - Payload: `{}`

**Configuración via CLI**:
```bash
# Crear topic de Pub/Sub
gcloud pubsub topics create daily-sync

# Crear scheduled job
gcloud scheduler jobs create pubsub daily-results-sync \
  --schedule="0 0 * * *" \
  --topic=daily-sync \
  --message-body='{}' \
  --time-zone="America/Argentina/Buenos_Aires"
```

### Firebase Admin SDK Setup

**En Cloud Functions** (`functions/src/admin.ts`):
```typescript
import * as admin from 'firebase-admin';

admin.initializeApp();

export const db = admin.firestore();
export const auth = admin.auth();

// Set custom claims para admin
export const setAdminRole = async (uid: string) => {
  await auth.setCustomUserClaims(uid, { admin: true });
};
```

### Monitoring and Logging

**Cloud Logging**:
- Todos los logs de Functions aparecen en Cloud Logging
- Usar `console.log()`, `console.error()` en functions
- Filtrar por severity: ERROR, WARNING, INFO

**Firebase Performance Monitoring**:
```typescript
// En frontend
import { getPerformance } from 'firebase/performance';

const perf = getPerformance();
// Automáticamente trackea métricas de performance
```

**Firebase Analytics** (opcional):
```typescript
import { getAnalytics, logEvent } from 'firebase/analytics';

const analytics = getAnalytics();
logEvent(analytics, 'prediction_created');
```

### Cost Optimization

**Firestore**:
- Usar listeners solo donde se necesita realtime
- Implementar pagination para listas largas
- Cachear queries que no cambian frecuentemente
- Usar `limit()` en queries

**Cloud Functions**:
- Minimizar cold starts (mantener functions warm con scheduled pings)
- Optimizar tamaño de deployment (tree-shaking)
- Usar memory allocation apropiada (default 256MB)

**Hosting**:
- Comprimir assets (Vite lo hace automáticamente)
- Usar CDN de Firebase (incluido)
- Implementar cache headers agresivos para assets estáticos

**Quotas y Límites**:
- Spark Plan (gratis): 50K reads/day, 20K writes/day
- Blaze Plan (pay-as-you-go): Necesario para Cloud Scheduler
- Monitorear uso en Firebase Console
