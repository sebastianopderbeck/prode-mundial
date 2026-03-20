# Requirements Document

## Introduction

Sistema de prode (predicción de resultados) del mundial que permite a usuarios de un mismo grupo de trabajo competir prediciendo los resultados de los partidos. El sistema otorga puntos según la precisión de las predicciones y mantiene un ranking de participantes.

## Glossary

- **Prode_System**: El sistema completo de predicción de resultados del mundial
- **User**: Persona que participa en el prode realizando predicciones
- **Administrator**: Usuario con permisos especiales para gestionar el sistema, sincronizar datos y cargar información manualmente
- **Match**: Partido del mundial entre dos selecciones
- **Prediction**: Predicción de un usuario sobre el resultado de un partido específico
- **Exact_Result**: Resultado que coincide exactamente con el marcador final (ej: 2-1)
- **Winner**: Equipo que gana el partido
- **Score**: Puntaje acumulado por un usuario basado en sus predicciones
- **Ranking**: Lista ordenada de usuarios según su puntaje total
- **Match_Deadline**: Momento límite para realizar predicciones (inicio del partido)
- **Football_API**: API externa que provee información actualizada sobre partidos y resultados del mundial
- **Scheduled_Job**: Tarea automatizada que se ejecuta en intervalos programados sin intervención manual
- **World_Cup_Period**: Período de tiempo durante el cual se disputa el mundial de fútbol

## Requirements

### Requirement 1: Registro y Autenticación de Usuarios

**User Story:** Como usuario, quiero registrarme e iniciar sesión en el sistema, para poder participar en el prode con mis compañeros de trabajo.

#### Acceptance Criteria

1. THE Prode_System SHALL permitir que un User se autentique mediante Google OAuth
2. WHEN un User se autentica con Google OAuth, THE Prode_System SHALL capturar el email del User para identificación
3. THE Prode_System SHALL permitir que un User se registre con email y contraseña como método alternativo
4. THE Prode_System SHALL autenticar a un User mediante email y contraseña como método alternativo
5. WHEN un User intenta registrarse con un email ya existente, THE Prode_System SHALL retornar un mensaje de error
6. THE Prode_System SHALL mantener la sesión del User activa hasta que cierre sesión

### Requirement 1.1: Gestión de Roles de Administrador

**User Story:** Como administrador del sistema, quiero tener permisos especiales, para poder gestionar el fixture y sincronizar resultados.

#### Acceptance Criteria

1. THE Prode_System SHALL asignar el rol de Administrator a usuarios específicos mediante configuración
2. THE Prode_System SHALL identificar si un User autenticado tiene rol de Administrator
3. THE Prode_System SHALL mostrar funcionalidades administrativas únicamente a usuarios con rol de Administrator
4. THE Prode_System SHALL impedir que usuarios sin rol de Administrator accedan a funcionalidades administrativas

### Requirement 2: Visualización de Partidos

**User Story:** Como usuario, quiero ver la lista de partidos del mundial, para saber qué partidos puedo predecir.

#### Acceptance Criteria

1. THE Prode_System SHALL mostrar todos los Match del mundial con fecha, hora y equipos
2. THE Prode_System SHALL indicar para cada Match si el User ya realizó una Prediction
3. THE Prode_System SHALL mostrar el estado de cada Match (pendiente, en curso, finalizado)
4. WHEN un Match ha finalizado, THE Prode_System SHALL mostrar el resultado final

### Requirement 3: Creación de Predicciones

**User Story:** Como usuario, quiero predecir el resultado de los partidos, para acumular puntos según mis aciertos.

#### Acceptance Criteria

1. WHEN un User selecciona un Match pendiente, THE Prode_System SHALL permitir ingresar una Prediction con el marcador de ambos equipos
2. WHILE un Match no ha alcanzado su Match_Deadline, THE Prode_System SHALL permitir que el User modifique su Prediction
3. WHEN un Match alcanza su Match_Deadline, THE Prode_System SHALL bloquear la creación o modificación de Prediction para ese Match
4. THE Prode_System SHALL validar que los marcadores ingresados sean números enteros no negativos
5. THE Prode_System SHALL confirmar al User cuando una Prediction se guarda exitosamente

### Requirement 4: Cálculo de Puntaje por Predicción Exacta

**User Story:** Como usuario, quiero recibir 3 puntos cuando adivino el resultado exacto, para obtener la máxima recompensa por mi precisión.

#### Acceptance Criteria

1. WHEN un Match finaliza y una Prediction coincide con el Exact_Result, THE Prode_System SHALL otorgar 3 puntos al User
2. THE Prode_System SHALL considerar Exact_Result cuando ambos marcadores coinciden exactamente
3. WHEN se otorgan puntos por Exact_Result, THE Prode_System SHALL actualizar el Score del User inmediatamente

### Requirement 5: Cálculo de Puntaje por Ganador

**User Story:** Como usuario, quiero recibir 1 punto cuando adivino el ganador pero no el resultado exacto, para obtener puntos parciales por mi predicción.

#### Acceptance Criteria

1. WHEN un Match finaliza y una Prediction identifica correctamente el Winner pero no el Exact_Result, THE Prode_System SHALL otorgar 1 punto al User
2. THE Prode_System SHALL identificar el Winner como el equipo con mayor marcador
3. WHEN un Match termina en empate y la Prediction también predice empate sin ser Exact_Result, THE Prode_System SHALL otorgar 1 punto al User
4. THE Prode_System SHALL otorgar únicamente los puntos por Exact_Result cuando ambos criterios aplican (no sumar 1+3 puntos)

### Requirement 6: Visualización de Ranking

**User Story:** Como usuario, quiero ver el ranking de todos los participantes, para saber mi posición y comparar mi desempeño con mis compañeros.

#### Acceptance Criteria

1. THE Prode_System SHALL mostrar un Ranking con todos los User ordenados por Score descendente
2. THE Prode_System SHALL mostrar para cada User en el Ranking su nombre y Score total
3. THE Prode_System SHALL actualizar el Ranking automáticamente cuando se calculan nuevos puntos
4. THE Prode_System SHALL indicar la posición del User actual en el Ranking

### Requirement 7: Visualización de Predicciones Propias

**User Story:** Como usuario, quiero ver mis predicciones y los puntos obtenidos, para hacer seguimiento de mi desempeño.

#### Acceptance Criteria

1. THE Prode_System SHALL mostrar al User todas sus Prediction realizadas
2. WHEN un Match ha finalizado, THE Prode_System SHALL mostrar junto a la Prediction los puntos obtenidos
3. THE Prode_System SHALL mostrar el resultado real del Match junto a la Prediction del User
4. THE Prode_System SHALL calcular y mostrar el Score total acumulado por el User

### Requirement 8: Sincronización Automática de Resultados de Partidos

**User Story:** Como administrador del sistema, quiero sincronizar automáticamente los resultados reales de los partidos desde la web, para que el sistema calcule los puntajes sin intervención manual.

#### Acceptance Criteria

1. THE Prode_System SHALL mostrar un botón de sincronización de resultados únicamente a usuarios administradores
2. WHEN un administrador presiona el botón de sincronización, THE Prode_System SHALL obtener los resultados actualizados de Match desde una API externa de fútbol
3. WHEN se obtienen resultados desde la API externa, THE Prode_System SHALL actualizar el estado y marcador de cada Match finalizado
4. WHEN se actualiza el resultado de un Match, THE Prode_System SHALL calcular los puntos para todas las Prediction de ese Match
5. THE Prode_System SHALL validar que los resultados obtenidos de la API contengan marcadores válidos (números enteros no negativos)
6. WHEN se calcula el puntaje de un Match, THE Prode_System SHALL actualizar el Score de todos los User afectados
7. THE Prode_System SHALL permitir a un administrador ingresar manualmente el resultado de un Match como método alternativo
8. IF la sincronización con la API externa falla, THEN THE Prode_System SHALL mostrar un mensaje de error al administrador y mantener los datos existentes

### Requirement 8.1: Sincronización Automática Programada de Resultados

**User Story:** Como administrador del sistema, quiero que los resultados de los partidos se sincronicen automáticamente todos los días durante el mundial, para evitar la necesidad de sincronización manual constante.

#### Acceptance Criteria

1. WHILE el sistema está en el World_Cup_Period, THE Prode_System SHALL ejecutar un Scheduled_Job diariamente para sincronizar resultados
2. THE Scheduled_Job SHALL ejecutarse automáticamente una vez por día sin intervención manual
3. WHEN el Scheduled_Job se ejecuta, THE Prode_System SHALL obtener los resultados actualizados de Match desde la Football_API
4. WHEN el Scheduled_Job obtiene resultados desde la Football_API, THE Prode_System SHALL actualizar el estado y marcador de cada Match finalizado
5. WHEN el Scheduled_Job actualiza el resultado de un Match, THE Prode_System SHALL calcular los puntos para todas las Prediction de ese Match
6. THE Prode_System SHALL mantener disponible el botón de sincronización manual para casos urgentes o cuando se requiera actualización inmediata
7. IF el Scheduled_Job falla al sincronizar, THEN THE Prode_System SHALL registrar el error en logs y reintentar en la próxima ejecución programada
8. THE Prode_System SHALL permitir a un Administrator configurar el horario de ejecución del Scheduled_Job

### Requirement 9: Carga Automática de Fixture del Mundial

**User Story:** Como administrador del sistema, quiero que el fixture del mundial se cargue automáticamente desde una fuente externa, para que los usuarios puedan realizar sus predicciones sin necesidad de carga manual.

#### Acceptance Criteria

1. WHEN el Prode_System se inicializa, THE Prode_System SHALL obtener automáticamente la lista completa de Match del mundial desde una API externa de fútbol
2. THE Prode_System SHALL almacenar para cada Match los equipos participantes, fecha, hora y Match_Deadline
3. THE Prode_System SHALL validar que cada Match obtenido contenga fecha y hora válidas
4. THE Prode_System SHALL establecer el Match_Deadline como el momento de inicio del Match
5. THE Prode_System SHALL permitir a un administrador actualizar manualmente el fixture como método alternativo
6. IF la carga automática desde la API externa falla, THEN THE Prode_System SHALL permitir al administrador cargar el fixture manualmente

### Requirement 10: Prevención de Predicciones Tardías

**User Story:** Como usuario, quiero que el sistema bloquee predicciones después del inicio del partido, para garantizar la equidad del juego.

#### Acceptance Criteria

1. WHEN la hora actual alcanza o supera el Match_Deadline, THE Prode_System SHALL impedir la creación de nuevas Prediction para ese Match
2. WHEN la hora actual alcanza o supera el Match_Deadline, THE Prode_System SHALL impedir la modificación de Prediction existentes para ese Match
3. IF un User intenta crear o modificar una Prediction después del Match_Deadline, THEN THE Prode_System SHALL mostrar un mensaje indicando que el plazo ha expirado
4. THE Prode_System SHALL verificar el Match_Deadline antes de aceptar cualquier operación sobre Prediction
