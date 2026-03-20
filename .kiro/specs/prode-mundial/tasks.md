# Plan de Implementación: Prode Mundial

## Descripción General

Implementación de una aplicación web full-stack de prode (predicción de resultados) del mundial de fútbol. El sistema permite a usuarios autenticarse, realizar predicciones sobre partidos, calcular puntajes automáticamente y visualizar un ranking en tiempo real. La aplicación utiliza React con Vite, un design system inspirado en Apple (iOS/macOS), Firebase como backend serverless, Firestore como base de datos NoSQL, y está configurada como PWA para funcionamiento offline.

**Stack**: React 18+ con Vite, TypeScript, Firebase Functions (serverless), Cloud Firestore (NoSQL), Firebase Authentication, Firebase Hosting, API-Football, Cloud Scheduler, Bun (package manager), Vitest, fast-check.

## Tareas

- [x] 1. Configurar estructura del proyecto y dependencias
  - Crear estructura de directorios para frontend (React + Vite) y functions (Firebase Functions)
  - Configurar TypeScript para ambos proyectos
  - Instalar dependencias core con Bun: React, Vite, Firebase SDK, Firebase Functions
  - Configurar Vite con plugin PWA
  - Configurar ESLint y Prettier
  - Inicializar proyecto Firebase (firebase init)
  - _Requisitos: Todos los requisitos dependen de esta base_

- [-] 2. Configurar Firestore y Security Rules
  - [-] 2.1 Crear colecciones de Firestore y definir estructura de datos
    - Definir colección users con campos: id, email, name, photoURL, isAdmin, totalScore
    - Definir colección matches con campos: id, homeTeam, awayTeam, homeScore, awayScore, matchDate, deadline, status, externalId
    - Definir colección predictions con campos: id, userId, matchId, homeScore, awayScore, pointsEarned
    - Definir colección config/scheduler con campos: cronExpression, isActive, worldCupStart, worldCupEnd
    - Establecer relaciones mediante referencias: userId → users/{userId}, matchId → matches/{matchId}
    - Crear índices compuestos en firestore.indexes.json
    - _Requisitos: 1.2, 1.3, 1.4, 1.5, 2.1, 3.1, 4.1, 8.1_
  
  - [ ]* 2.2 Escribir property test para validación de estructura de datos
    - **Property 32: Almacenamiento completo de datos de match**
    - **Valida: Requisitos 9.2**

- [ ] 3. Implementar Design System (Apple-inspired)
  - [ ] 3.1 Crear tokens de diseño (colores, tipografía, espaciado, sombras)
    - Definir paleta de colores Apple (Primary #007AFF, Success #34C759, etc.)
    - Configurar tipografía SF Pro Display/Text con fallback a system-ui
    - Definir sistema de espaciado basado en 4px
    - Definir border radius y sombras estilo iOS
    - _Requisitos: Todos los requisitos de UI_
  
  - [ ] 3.2 Crear componentes base del design system
    - Implementar Button (filled, outlined, text variants)
    - Implementar Card con glassmorphism opcional
    - Implementar Input estilo iOS con label flotante
    - Implementar Modal sheet-style de iOS
    - Implementar List grouped style de iOS
    - _Requisitos: 1.3, 2.1, 3.1, 6.1_
  
  - [ ]* 3.3 Escribir unit tests para componentes del design system
    - Test de Button variants y estados
    - Test de Card rendering
    - Test de Input validation
    - Test de Modal behavior
    - Test de theme switching (light/dark)
    - _Requisitos: Todos los requisitos de UI_

- [ ] 4. Implementar autenticación con Firebase Authentication
  - [ ] 4.1 Configurar Firebase Authentication en frontend
    - Instalar Firebase SDK con Bun (firebase)
    - Configurar Firebase app con credenciales del proyecto
    - Configurar Google OAuth provider (signInWithPopup)
    - Habilitar Email/Password authentication en Firebase Console
    - Crear documento de usuario en Firestore tras autenticación exitosa
    - _Requisitos: 1.1, 1.2_
  
  - [ ] 4.2 Implementar registro y login con email/password
    - Usar createUserWithEmailAndPassword para registro
    - Usar signInWithEmailAndPassword para login
    - Crear documento en users collection tras registro exitoso
    - Manejar errores de Firebase Auth (email-already-in-use, weak-password, etc.)
    - _Requisitos: 1.3, 1.4, 1.5_
  
  - [ ] 4.3 Configurar custom claims para administradores
    - Crear Cloud Function para asignar custom claim isAdmin
    - Verificar custom claims en frontend (user.getIdTokenResult())
    - Proteger Cloud Functions verificando context.auth.token.admin
    - _Requisitos: 1.1.2, 1.1.3, 1.1.4, 1.6_
  
  - [ ]* 4.4 Escribir property test para autenticación OAuth
    - **Property 1: Email capturado en autenticación OAuth**
    - **Valida: Requisitos 1.2**
  
  - [ ]* 4.5 Escribir property test para round-trip de credenciales
    - **Property 2: Round-trip de autenticación con credenciales**
    - **Valida: Requisitos 1.4**
  
  - [ ]* 4.6 Escribir property test para rechazo de emails duplicados
    - **Property 3: Rechazo de emails duplicados**
    - **Valida: Requisitos 1.5**
  
  - [ ]* 4.7 Escribir property test para persistencia de sesión
    - **Property 4: Persistencia de sesión con Firebase Auth**
    - **Valida: Requisitos 1.6**
  
  - [ ]* 4.8 Escribir property test para identificación de rol admin
    - **Property 5: Identificación correcta de rol admin**
    - **Valida: Requisitos 1.1.2**
  
  - [ ]* 4.9 Escribir property test para restricción de funciones admin
    - **Property 6: Restricción de funciones administrativas**
    - **Valida: Requisitos 1.1.3**

- [ ] 5. Implementar UI de autenticación en frontend
  - [ ] 5.1 Crear AuthProvider con Context API y Firebase Auth
    - Implementar contexto con user, isAdmin, loginWithGoogle, loginWithEmail, logout
    - Integrar Firebase Auth (signInWithPopup, signInWithEmailAndPassword)
    - Verificar custom claims para isAdmin
    - Manejar estado de autenticación con onAuthStateChanged
    - Habilitar persistencia offline de Firestore (enableIndexedDbPersistence)
    - _Requisitos: 1.1, 1.2, 1.3, 1.4, 1.6_
  
  - [ ] 5.2 Crear LoginPage con estilo Apple
    - Diseñar formulario con inputs estilo iOS
    - Agregar botón de Google OAuth con estilo iOS
    - Implementar manejo de errores de Firebase Auth
    - Agregar animaciones suaves (iOS-style transitions)
    - _Requisitos: 1.1, 1.3, 1.4_

- [ ] 6. Checkpoint - Verificar autenticación funcional
  - Asegurar que todos los tests pasen, preguntar al usuario si surgen dudas.

- [ ] 7. Implementar servicio de integración con Football API
  - [ ] 7.1 Crear FootballAPIService para obtener fixtures y resultados
    - Configurar cliente HTTP (axios) con API-Football
    - Implementar fetchWorldCupFixtures() para obtener lista de partidos
    - Implementar fetchMatchResults() para obtener resultados actualizados
    - Implementar validateAPIResponse() para validar formato de respuesta
    - Manejar errores: timeout, 500, 503, 429 con retry logic
    - _Requisitos: 8.2, 8.3, 8.5, 8.8, 9.1, 9.3_
  
  - [ ]* 7.2 Escribir property test para validación de marcadores de API
    - **Property 29: Validación de marcadores de API**
    - **Valida: Requisitos 8.5**
  
  - [ ]* 7.3 Escribir property test para preservación de datos ante error
    - **Property 30: Preservación de datos ante error de sincronización**
    - **Valida: Requisitos 8.8**
  
  - [ ]* 7.4 Escribir unit tests para manejo de errores de API
    - Test de timeout handling
    - Test de respuestas 500, 503, 429
    - Test de formato inválido de respuesta
    - Test de retry logic con backoff exponencial
    - _Requisitos: 8.8, 9.6_

- [ ] 8. Implementar carga automática de fixture
  - [ ] 8.1 Crear Cloud Function para sincronizar matches desde API
    - Implementar syncMatches como Callable Function (httpsCallable)
    - Llamar a FootballAPIService para obtener fixtures
    - Almacenar matches en Firestore collection con batch writes
    - Establecer deadline = matchDate
    - Validar fechas válidas antes de almacenar
    - Verificar custom claim admin en context.auth.token
    - _Requisitos: 9.1, 9.2, 9.3, 9.4_
  
  - [ ] 8.2 Crear UI admin para carga manual de fixture
    - Crear botón de sincronización en AdminPanel
    - Llamar a Cloud Function syncMatches con httpsCallable
    - Mostrar loading state y resultados
    - _Requisitos: 9.5_
  
  - [ ]* 8.3 Escribir property test para completitud de información de matches
    - **Property 7: Completitud de información de matches**
    - **Valida: Requisitos 2.1**
  
  - [ ]* 8.4 Escribir property test para deadline igual a matchDate
    - **Property 34: Deadline igual a fecha de inicio**
    - **Valida: Requisitos 9.4**
  
  - [ ]* 8.5 Escribir property test para validación de fechas
    - **Property 33: Validación de fechas de matches**
    - **Valida: Requisitos 9.3**

- [ ] 9. Implementar visualización de partidos en frontend
  - [ ] 9.1 Crear MatchList y MatchCard components con Firestore realtime listeners
    - Implementar MatchList que muestra todos los matches con onSnapshot
    - Usar query ordenada por matchDate (orderBy('matchDate', 'asc'))
    - Implementar MatchCard con equipos, fecha, hora, estado
    - Mostrar indicador si usuario ya predijo (verificar en predictions collection)
    - Mostrar resultado final si match está FINISHED
    - Aplicar estilo Apple: Card con border radius 12px, sombra sutil
    - _Requisitos: 2.1, 2.2, 2.3, 2.4_
  
  - [ ]* 9.2 Escribir property test para indicador de predicción existente
    - **Property 8: Indicador de predicción existente**
    - **Valida: Requisitos 2.2**
  
  - [ ]* 9.3 Escribir property test para visibilidad de estado de match
    - **Property 9: Visibilidad de estado de match**
    - **Valida: Requisitos 2.3**
  
  - [ ]* 9.4 Escribir property test para visibilidad de resultado final
    - **Property 10: Visibilidad de resultado final**
    - **Valida: Requisitos 2.4**

- [ ] 10. Checkpoint - Verificar visualización de partidos
  - Asegurar que todos los tests pasen, preguntar al usuario si surgen dudas.

- [ ] 11. Implementar creación y modificación de predicciones
  - [ ] 11.1 Crear Firestore Security Rules para predictions
    - Implementar reglas que validen deadline no alcanzado
    - Validar marcadores >= 0 y enteros en Security Rules
    - Validar que userId coincida con auth.uid
    - Prevenir modificación de pointsEarned (solo Cloud Functions)
    - Implementar validación de unique constraint (userId + matchId) en frontend
    - _Requisitos: 3.1, 3.2, 3.3, 3.4, 10.1, 10.2, 10.3, 10.4_
  
  - [ ] 11.2 Implementar operaciones de predicciones en frontend
    - Usar setDoc para crear predicción en Firestore
    - Usar updateDoc para modificar predicción existente
    - Verificar deadline en frontend antes de enviar
    - Usar query para obtener predicciones del usuario (where('userId', '==', uid))
    - Usar query para verificar predicción existente antes de crear
    - _Requisitos: 3.1, 3.2, 7.1_
  
  - [ ]* 11.3 Escribir property test para creación antes de deadline
    - **Property 11: Creación de predicción antes de deadline**
    - **Valida: Requisitos 3.1**
  
  - [ ]* 11.4 Escribir property test para modificación antes de deadline
    - **Property 12: Modificación de predicción antes de deadline**
    - **Valida: Requisitos 3.2**
  
  - [ ]* 11.5 Escribir property test para bloqueo después de deadline
    - **Property 13: Bloqueo de predicciones después de deadline**
    - **Valida: Requisitos 3.3, 10.1, 10.2, 10.3, 10.4**
  
  - [ ]* 11.6 Escribir property test para validación de marcadores no negativos
    - **Property 14: Validación de marcadores no negativos**
    - **Valida: Requisitos 3.4**

- [ ] 12. Implementar UI de predicciones en frontend
  - [ ] 12.1 Crear PredictionForm component con Firestore
    - Implementar modal sheet-style de iOS para ingresar predicción
    - Validar marcadores >= 0 en frontend
    - Validar deadline no alcanzado antes de enviar
    - Usar setDoc/updateDoc para guardar en Firestore
    - Mostrar confirmación al guardar exitosamente
    - Permitir modificación si deadline no pasó
    - _Requisitos: 3.1, 3.2, 3.3, 3.4, 3.5_
  
  - [ ] 12.2 Crear PredictionList component con realtime listeners
    - Usar onSnapshot para escuchar predicciones del usuario en tiempo real
    - Query: where('userId', '==', currentUser.uid)
    - Mostrar resultado real vs predicción para matches finalizados
    - Mostrar puntos obtenidos por cada match
    - Aplicar estilo Apple: Lista agrupada estilo iOS
    - _Requisitos: 7.1, 7.2, 7.3_
  
  - [ ]* 12.3 Escribir property test para recuperación completa de predicciones
    - **Property 23: Recuperación completa de predicciones propias**
    - **Valida: Requisitos 7.1**

- [ ] 13. Checkpoint - Verificar predicciones funcionales
  - Asegurar que todos los tests pasen, preguntar al usuario si surgen dudas.

- [ ] 14. Implementar motor de cálculo de puntajes
  - [ ] 14.1 Crear Cloud Function para calcular puntajes
    - Implementar calculatePoints(prediction, actualResult) con lógica:
      - 3 puntos si homeScore y awayScore coinciden exactamente
      - 1 punto si ganador coincide pero resultado no es exacto
      - 1 punto si ambos son empate pero marcadores difieren
      - 0 puntos en otros casos
    - Crear Firestore trigger onUpdate para matches collection
    - Cuando match.status cambia a FINISHED, calcular puntos para todas las predictions
    - Usar batch writes para actualizar predictions.pointsEarned
    - Crear Firestore trigger onUpdate para predictions collection
    - Cuando prediction.pointsEarned se actualiza, actualizar user.totalScore con transaction
    - _Requisitos: 4.1, 4.2, 4.3, 5.1, 5.2, 5.3, 5.4_
  
  - [ ]* 14.2 Escribir property test para puntos por resultado exacto
    - **Property 15: Puntos por resultado exacto**
    - **Valida: Requisitos 4.1, 4.2**
  
  - [ ]* 14.3 Escribir property test para actualización inmediata de score
    - **Property 16: Actualización inmediata de score total**
    - **Valida: Requisitos 4.3, 8.6**
  
  - [ ]* 14.4 Escribir property test para puntos por ganador sin exacto
    - **Property 17: Puntos por acertar ganador sin resultado exacto**
    - **Valida: Requisitos 5.1, 5.2, 5.3**
  
  - [ ]* 14.5 Escribir property test para no acumulación de puntos
    - **Property 18: No acumulación de puntos exacto + ganador**
    - **Valida: Requisitos 5.4**
  
  - [ ]* 14.6 Escribir property test para score total como suma
    - **Property 26: Score total como suma de puntos**
    - **Valida: Requisitos 7.4**
  
  - [ ]* 14.7 Escribir unit tests para casos edge de scoring
    - Test de empate 0-0
    - Test de goleada (ej: 7-1)
    - Test de empate predicho vs empate real con marcadores diferentes
    - Test de predicción incorrecta (0 puntos)
    - _Requisitos: 4.1, 5.1, 5.3_

- [ ] 15. Implementar sincronización automática de resultados
  - [ ] 15.1 Crear Cloud Function para sincronización manual
    - Crear Callable Function syncResults (solo admin)
    - Llamar a FootballAPIService.fetchMatchResults()
    - Actualizar matches con status FINISHED y marcadores usando batch writes
    - Los triggers de Firestore calcularán puntos automáticamente
    - Retornar resumen de matches actualizados
    - _Requisitos: 8.1, 8.2, 8.3, 8.4, 8.6_
  
  - [ ]* 15.2 Escribir property test para actualización de matches desde API
    - **Property 27: Actualización de matches desde API**
    - **Valida: Requisitos 8.3**
  
  - [ ]* 15.3 Escribir property test para cálculo tras actualización
    - **Property 28: Cálculo de puntos tras actualización de match**
    - **Valida: Requisitos 8.4**

- [ ] 16. Implementar scheduled job para sincronización diaria
  - [ ] 16.1 Crear Scheduled Function con Cloud Scheduler
    - Crear Scheduled Function con pubsub.schedule('0 0 * * *')
    - Configurar timezone: 'America/Argentina/Buenos_Aires'
    - Implementar lógica de sincronización (llamar a FootballAPIService)
    - Verificar World Cup Period consultando config/scheduler en Firestore
    - Ejecutar sincronización solo durante World Cup Period
    - Actualizar matches y dejar que triggers calculen puntos
    - _Requisitos: 8.1.1, 8.1.2, 8.1.3, 8.1.4, 8.1.5, 8.1.7_
  
  - [ ] 16.2 Crear UI admin para configurar scheduler
    - Crear formulario en AdminPanel para actualizar config/scheduler
    - Actualizar documento en Firestore (cronExpression, worldCupStart, worldCupEnd)
    - Nota: cambio de cron requiere redeploy de Cloud Function
    - _Requisitos: 8.1.8_
  
  - [ ]* 16.3 Escribir property test para actualización de configuración
    - **Property 31: Actualización de configuración de scheduler**
    - **Valida: Requisitos 8.1.8**
  
  - [ ]* 16.4 Escribir unit tests para scheduler
    - Test de configuración de scheduled function
    - Test de verificación de World Cup Period
    - Test de ejecución programada (mock)
    - _Requisitos: 8.1.1, 8.1.2_

- [ ] 17. Implementar AdminPanel en frontend
  - [ ] 17.1 Crear AdminPanel component (solo visible para admins)
    - Implementar botón de sincronización manual con estilo iOS
    - Llamar a Cloud Function syncResults con httpsCallable
    - Implementar formulario de carga manual de resultados (updateDoc en Firestore)
    - Implementar configuración de scheduled job (actualizar config/scheduler)
    - Mostrar estados: loading, success, error
    - Aplicar estilo Apple: Cards con glassmorphism, botones con estados claros
    - _Requisitos: 8.1, 8.7, 8.1.8, 9.5_

- [ ] 18. Checkpoint - Verificar sincronización y scoring
  - Asegurar que todos los tests pasen, preguntar al usuario si surgen dudas.

- [ ] 19. Implementar ranking de usuarios
  - [ ] 19.1 Implementar queries de ranking en frontend con Firestore
    - Usar query ordenada: orderBy('totalScore', 'desc'), orderBy('name', 'asc')
    - Usar onSnapshot para actualización en tiempo real del ranking
    - Calcular posición de cada usuario en frontend (index + 1)
    - Implementar query para obtener posición del usuario actual
    - _Requisitos: 6.1, 6.2, 6.4_
  
  - [ ] 19.2 Crear componente de ranking con realtime updates
    - Implementar RankingTable que escucha cambios en users collection
    - Mostrar ranking actualizado automáticamente cuando cambian scores
    - _Requisitos: 6.1, 6.4_
  
  - [ ]* 19.3 Escribir property test para ordenamiento descendente
    - **Property 19: Ordenamiento descendente de ranking**
    - **Valida: Requisitos 6.1**
  
  - [ ]* 19.4 Escribir property test para completitud de información
    - **Property 20: Completitud de información en ranking**
    - **Valida: Requisitos 6.2**
  
  - [ ]* 19.5 Escribir property test para actualización de ranking
    - **Property 21: Actualización de ranking tras cálculo de puntos**
    - **Valida: Requisitos 6.3**
  
  - [ ]* 19.6 Escribir property test para cálculo de posición
    - **Property 22: Cálculo correcto de posición en ranking**
    - **Valida: Requisitos 6.4**
  
  - [ ]* 19.7 Escribir unit tests para casos edge de ranking
    - Test con todos los usuarios con 0 puntos
    - Test con un solo usuario
    - Test con empates en puntaje
    - _Requisitos: 6.1, 6.4_

- [ ] 20. Implementar RankingTable en frontend
  - [ ] 20.1 Crear RankingTable component con realtime listeners
    - Mostrar lista de usuarios con posición, nombre y score
    - Usar onSnapshot para actualización en tiempo real
    - Ordenar por score descendente con query de Firestore
    - Highlight del usuario actual con color accent
    - Aplicar estilo Apple: Lista agrupada estilo iOS, animaciones suaves
    - _Requisitos: 6.1, 6.2, 6.3, 6.4_
  
  - [ ]* 20.2 Escribir property test para visibilidad de puntos en predicciones
    - **Property 24: Visibilidad de puntos en predicciones de matches finalizados**
    - **Valida: Requisitos 7.2**
  
  - [ ]* 20.3 Escribir property test para visibilidad de resultado real
    - **Property 25: Visibilidad de resultado real junto a predicción**
    - **Valida: Requisitos 7.3**

- [ ] 21. Checkpoint - Verificar ranking funcional
  - Asegurar que todos los tests pasen, preguntar al usuario si surgen dudas.

- [ ] 22. Configurar PWA (Progressive Web App)
  - [ ] 22.1 Configurar Vite PWA Plugin con Workbox
    - Instalar vite-plugin-pwa y workbox-window con Bun
    - Configurar plugin en vite.config.ts con registerType: 'autoUpdate'
    - Configurar workbox con estrategias de cache:
      - CacheFirst para assets estáticos (CSS, JS, fonts, images)
      - Firestore maneja su propio cache offline automáticamente
    - Nota: No cachear API calls, Firestore tiene persistencia offline nativa
    - _Requisitos: Todos los requisitos (soporte offline)_
  
  - [ ] 22.2 Crear manifest.json para PWA
    - Definir name, short_name, description, start_url
    - Configurar display: standalone, theme_color: #007AFF
    - Agregar iconos en múltiples tamaños (72x72 a 512x512)
    - _Requisitos: Todos los requisitos (instalación)_
  
  - [ ] 22.3 Generar iconos PWA en múltiples tamaños
    - Crear iconos: 72x72, 96x96, 128x128, 144x144, 152x152, 192x192, 384x384, 512x512
    - Colocar en public/icons/
    - _Requisitos: Todos los requisitos (instalación)_

- [ ] 23. Implementar componentes PWA en frontend
  - [ ] 23.1 Crear hooks PWA
    - Implementar useOnlineStatus() para detectar conexión
    - Implementar useInstallPrompt() para manejar instalación
    - Implementar useServiceWorker() para lifecycle del SW
    - _Requisitos: Todos los requisitos (offline support)_
  
  - [ ] 23.2 Crear componentes PWA
    - Implementar InstallPrompt component (banner de instalación estilo iOS)
    - Implementar OfflineIndicator component (indicador de estado offline)
    - Implementar UpdatePrompt component (prompt de actualización estilo iOS)
    - _Requisitos: Todos los requisitos (offline support)_
  
  - [ ] 23.3 Habilitar persistencia offline de Firestore
    - Llamar a enableIndexedDbPersistence en inicialización de Firestore
    - Manejar errores (multiple tabs, browser no soportado)
    - Firestore sincroniza automáticamente writes offline al recuperar conexión
    - No necesita queue manual, Firestore lo maneja nativamente
    - _Requisitos: 3.1, 3.2 (offline support)_
  
  - [ ]* 23.4 Escribir unit tests para PWA features
    - Test de service worker registration
    - Test de cache strategies
    - Test de offline detection
    - Test de install prompt behavior
    - Test de update detection
    - Test de Firestore offline persistence
    - _Requisitos: Todos los requisitos (offline support)_

- [ ] 24. Implementar manejo de errores y validaciones
  - [ ] 24.1 Implementar Firestore Security Rules comprehensivas
    - Validar campos requeridos, tipos, rangos en Security Rules
    - Retornar permission-denied con validación fallida
    - Validar deadline en predictions
    - Proteger campos admin-only (isAdmin, totalScore, pointsEarned)
    - _Requisitos: 3.4, 8.5, 9.3_
  
  - [ ] 24.2 Implementar manejo de errores global en Cloud Functions
    - Crear error handler para Cloud Functions
    - Mapear errores técnicos a mensajes user-friendly
    - Implementar logging comprehensivo con Cloud Logging
    - Implementar retry logic con backoff exponencial para API externa
    - _Requisitos: 8.8, 9.6_
  
  - [ ] 24.3 Implementar manejo de errores en frontend
    - Crear ErrorBoundary component para React
    - Mostrar mensajes de error con estilo Apple (alert sheet)
    - Implementar toast notifications para feedback
    - Manejar errores de Firebase Auth (auth/email-already-in-use, etc.)
    - Manejar errores de Firestore (permission-denied, unavailable, etc.)
    - _Requisitos: 3.5, 8.8_

- [ ] 25. Implementar accesibilidad y optimizaciones de performance
  - [ ] 25.1 Agregar soporte de accesibilidad (WCAG 2.1 AA)
    - Agregar ARIA labels en componentes interactivos
    - Implementar navegación por teclado completa
    - Asegurar contraste de colores mínimo 4.5:1
    - Implementar focus indicators visibles (outline estilo iOS)
    - Respetar preferencias de sistema (prefers-reduced-motion, prefers-color-scheme)
    - _Requisitos: Todos los requisitos de UI_
  
  - [ ] 25.2 Optimizar performance del frontend
    - Implementar lazy loading de componentes con React.lazy()
    - Implementar code splitting por rutas
    - Optimizar imágenes (WebP con fallback)
    - Configurar fonts con font-display: swap
    - Implementar debouncing en inputs
    - Implementar memoización con React.memo() en componentes pesados
    - _Requisitos: Todos los requisitos (performance)_

- [ ] 26. Configurar testing y CI/CD
  - [ ] 26.1 Configurar Vitest y React Testing Library con Bun
    - Configurar vitest.config.ts
    - Configurar @testing-library/react y @testing-library/jest-dom
    - Configurar fast-check para property-based testing
    - Configurar Firebase Emulator Suite para testing
    - Crear fixtures de prueba (users, matches, predictions)
    - Usar Bun para ejecutar tests (bun test)
    - _Requisitos: Todos los requisitos (calidad)_
  
  - [ ] 26.2 Configurar pipeline de CI con Firebase
    - Configurar lint (ESLint + Prettier)
    - Configurar type check (TypeScript)
    - Configurar ejecución de unit tests y property tests con Bun
    - Configurar Firestore Security Rules tests
    - Configurar build (Vite)
    - Configurar deploy a Firebase Hosting (staging)
    - Target: 80% code coverage mínimo
    - _Requisitos: Todos los requisitos (calidad)_

- [ ] 27. Checkpoint final - Integración completa
  - Ejecutar todos los tests (unit, property, integration)
  - Verificar que todas las funcionalidades estén integradas
  - Verificar accesibilidad y performance
  - Asegurar que la PWA funcione offline
  - Preguntar al usuario si surgen dudas o ajustes necesarios.

## Notas

- Las tareas marcadas con `*` son opcionales y pueden omitirse para un MVP más rápido
- Cada tarea referencia requisitos específicos para trazabilidad
- Los checkpoints aseguran validación incremental
- Los property tests validan propiedades universales de corrección
- Los unit tests validan ejemplos específicos y casos edge
- El diseño sigue los lineamientos de Apple (iOS/macOS) definidos en Figma
- La aplicación es una PWA con soporte offline completo
- Firebase maneja autenticación, base de datos, functions y hosting
- Firestore proporciona persistencia offline automática
- Bun se usa como package manager para mejor performance
- Cloud Scheduler ejecuta sincronizaciones programadas
- Security Rules validan permisos a nivel de base de datos
