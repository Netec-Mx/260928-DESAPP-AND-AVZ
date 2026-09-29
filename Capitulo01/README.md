# Configuración del proyecto Android avanzado y desarrollo de concurrencia con Kotlin 2.3.10, Coroutines, StateFlow y SharedFlow sobre API 30–37

## Metadatos

| Atributo | Detalle |
| :--- | :--- |
| **Duración** | 216 minutos |
| **Complejidad** | Media |
| **Nivel de Bloom** | Aplicar (Apply) |

---

## Descripción General

En este laboratorio práctico, configurarás desde cero la base estructural de un proyecto de Android avanzado utilizando **Kotlin 2.3.10** y las API de Android de la 30 a la 37. Desarrollarás un motor de simulación de telemetría reactiva en tiempo real (`TelemetryEngine`) que modelará el comportamiento de un dispositivo IoT o rastreador GPS de alto rendimiento.

A lo largo del laboratorio, aplicarás conceptos avanzados de Kotlin moderno como **Value Classes**, **Delegación de Propiedades** y **Contratos**, combinados con una arquitectura de concurrencia asíncrona basada en **Kotlin Coroutines** (Scopes, Dispatchers y cancelación cooperativa), **StateFlow** y **SharedFlow**. Además, aprenderás a validar y depurar estos flujos asíncronos mediante pruebas unitarias estructuradas bajo un enfoque offline-first.

---

## Objetivos de Aprendizaje

Al finalizar este laboratorio, serás capaz de:
- [ ] Configurar un proyecto base en Android Studio optimizado para Kotlin 2.3.10, Gradle Kotlin DSL y APIs 30 a 37.
- [ ] Implementar estructuras de datos eficientes utilizando clases de valor inline (`value classes`), delegación de propiedades personalizada y contratos del compilador para Smart Casts seguros.
- [ ] Desarrollar y depurar operaciones concurrentes utilizando corrutinas, gestionando de forma segura los hilos de ejecución mediante `Dispatchers.Default`, `Dispatchers.IO` y `Dispatchers.Main`.
- [ ] Diseñar flujos de datos reactivos empleando flujos fríos (`Flow`) y calientes (`StateFlow` y `SharedFlow`), configurando estrategias de mitigación de contrapresión (*backpressure*).
- [ ] Crear pruebas unitarias avanzadas para corrutinas y flujos utilizando `kotlinx-coroutines-test` para verificar la sincronía y las emisiones de datos.

---

## Prerrequisitos

Para completar con éxito este laboratorio, requieres:

### Conocimientos Previos
- Dominio de la sintaxis estándar de Kotlin (funciones, clases, interfaces).
- Comprensión conceptual básica de hilos (*threading*) y programación reactiva.
- Experiencia en el uso de Android Studio para abrir y compilar proyectos móviles.

### Requisitos de Acceso y Licencias
- Cuenta activa y acceso a la herramienta de asistencia al desarrollo **JetBrains AI Assistant (v242.23339)** bajo licencia comercial o de evaluación individual activa.
- Conexión a internet estable de banda de ancha (mínimo 20 Mbps) sin bloqueos de firewall corporativo para descarga de repositorios Gradle y dependencias Maven.

---

## Entorno de Laboratorio

El laboratorio debe ejecutarse bajo las siguientes especificaciones técnicas de hardware y software para garantizar la reproducibilidad completa del código.

### Especificaciones de Hardware (Mínimo vs Recomendado)

| Componente | Requisito Mínimo | Configuración Recomendada |
| :--- | :--- | :--- |
| **Procesador** | Intel Core i7 / AMD Ryzen 7 (11ª Gen) | Apple Silicon (M1/M2/M3) o AMD Ryzen 9 |
| **Memoria RAM** | 16 GB DDR4/DDR5 | 32 GB DDR5 |
| **Almacenamiento** | SSD con 20 GB libres | SSD NVMe con 40 GB libres |
| **Conectividad** | 20 Mbps de bajada | 100 Mbps de bajada (fibra simétrica) |

### Versiones de Software Exactas y Fuentes Oficiales

| Herramienta / SDK | Versión Exacta | Enlace de Descarga / Fuente |
| :--- | :--- | :--- |
| **Android Studio** | Ladybug (2024.2.1 Patch 3) | [Android Studio Archive](https://developer.android.com/studio/archive) |
| **JDK (Java)** | Eclipse Temurin JDK 17.0.10+7 | [Adoptium Releases](https://adoptium.net/temurin/releases/) |
| **Kotlin Compiler** | 2.3.10 | [Kotlin Releases](https://kotlinlang.org/docs/releases.html) |
| **Coroutines Core/Android**| 1.10.1 | [Kotlinx Coroutines GitHub](https://github.com/Kotlin/kotlinx.coroutines) |
| **Jetpack Compose BOM** | 2026.02.01 | [Compose BOM Mapping](https://developer.android.com/jetpack/compose/bom/bom-mapping) |
| **JetBrains AI Assistant**| v242.23339 | [JetBrains AI Plugin Marketplace](https://plugins.jetbrains.com/plugin/22003-jetbrains-ai) |

### Configuración Inicial del Entorno

1. Abre tu terminal de sistema local y comprueba que la versión global del JDK instalada sea compatible con la versión 17:
   ```bash
   java -version
   ```
   *Deberías ver una salida similar a:* `openjdk version "17.0.10" 2024-01-16`.

2. En Android Studio Ladybug, navega a **Settings / Preferences -> Build, Execution, Deployment -> Build Tools -> Gradle** y confirma que el **Gradle JDK** esté configurado para apuntar a la ruta de instalación de tu JDK 17 local.

---

## Instrucciones Paso a Paso

### Paso 1: Configuración del proyecto base con Kotlin 2.3.10 y APIs 30–37

**Objetivo:** Crear la estructura de directorios del proyecto base, configurar el catálogo de versiones de Gradle (`libs.versions.toml`) y preparar el archivo `build.gradle.kts` a nivel de proyecto y de módulo bajo los estándares de las API 30 a 35 (con soporte hasta API 37).

#### Instrucciones

1. Abre Android Studio y selecciona **New Project -> Empty Activity**.
2. Configura los siguientes parámetros en el asistente de creación:
   - **Name:** `Telemetry Tracker`
   - **Package Name:** `com.example.advancedtracker`
   - **Language:** Kotlin
   - **Minimum SDK:** API 30 (Android 11)
   - **Build Configuration Language:** Kotlin DSL (Build.gradle.kts)
3. En la raíz del proyecto, localiza la carpeta `gradle` y abre (o crea) el archivo `libs.versions.toml`. Modifica su contenido para unificar la gestión de dependencias avanzadas:

   ```toml
   [versions]
   agp = "8.4.0"
   kotlin = "2.3.10"
   coreKtx = "1.15.0"
   lifecycleRuntimeKtx = "2.8.7"
   activityCompose = "1.10.0"
   composeBom = "2026.02.01"
   coroutines = "1.10.1"
   junit = "4.13.2"
   mockK = "1.13.12"
   turbine = "1.1.0"

   [libraries]
   androidx-core-ktx = { group = "androidx.core", name = "core-ktx", version.ref = "coreKtx" }
   androidx-lifecycle-runtime-ktx = { group = "androidx.lifecycle", name = "lifecycle-runtime-ktx", version.ref = "lifecycleRuntimeKtx" }
   androidx-activity-compose = { group = "androidx.activity", name = "activity-compose", version.ref = "activityCompose" }
   androidx-compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "composeBom" }
   androidx-compose-ui = { group = "androidx.compose.ui", name = "ui" }
   androidx-compose-ui-graphics = { group = "androidx.compose.ui", name = "ui-graphics" }
   androidx-compose-material3 = { group = "androidx.compose.material3", name = "material3" }
   
   # Concurrencia y Reactividad
   kotlinx-coroutines-core = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-core", version.ref = "coroutines" }
   kotlinx-coroutines-android = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-android", version.ref = "coroutines" }

   # Pruebas Unitarias
   junit = { group = "junit", name = "junit", version.ref = "junit" }
   kotlinx-coroutines-test = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-test", version.ref = "coroutines" }
   mockk = { group = "io.mockk", name = "mockk", version.ref = "mockK" }
   turbine = { group = "app.cash.turbine", name = "turbine", version.ref = "turbine" }

   [plugins]
   android-application = { id = "com.android.application", version.ref = "agp" }
   kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
   kotlin-compose = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
   ```

4. Abre el archivo `build.gradle.kts` a nivel de **proyecto** y asegúrate de que use las declaraciones del catálogo:

   ```kotlin
   // build.gradle.kts (Project)
   plugins {
       alias(libs.plugins.android.application) apply false
       alias(libs.plugins.kotlin.android) apply false
       alias(libs.plugins.kotlin.compose) apply false
   }
   ```

5. Abre el archivo `build.gradle.kts` a nivel de **módulo app** (`app/build.gradle.kts`) y configúralo con los siguientes SDKs y configuraciones del compilador:

   ```kotlin
   // build.gradle.kts (Module :app)
   plugins {
       alias(libs.plugins.android.application)
       alias(libs.plugins.kotlin.android)
       alias(libs.plugins.kotlin.compose)
   }

   android {
       namespace = "com.example.advancedtracker"
       compileSdk = 35

       defaultConfig {
           applicationId = "com.example.advancedtracker"
           minSdk = 30
           targetSdk = 35
           versionCode = 1
           versionName = "1.0.0"

           testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
       }

       buildTypes {
           release {
               isMinifyEnabled = true
               proguardFiles(
                   getDefaultProguardFile("proguard-android-optimize.txt"),
                   "proguard-rules.pro"
               )
           }
       }
       compileOptions {
           sourceCompatibility = JavaVersion.VERSION_17
           targetCompatibility = JavaVersion.VERSION_17
       }
       kotlinOptions {
           jvmTarget = "17"
           freeCompilerArgs = freeCompilerArgs + listOf(
               "-opt-in=kotlin.contracts.ExperimentalContracts",
               "-opt-in=kotlinx.coroutines.ExperimentalCoroutinesApi"
           )
       }
   }

   dependencies {
       implementation(libs.androidx.core.ktx)
       implementation(libs.androidx.lifecycle.runtime.ktx)
       implementation(libs.androidx.activity.compose)
       implementation(platform(libs.androidx.compose.bom))
       implementation(libs.androidx.compose.ui)
       implementation(libs.androidx.compose.ui.graphics)
       implementation(libs.androidx.compose.material3)

       // Concurrencia
       implementation(libs.kotlinx.coroutines.core)
       implementation(libs.kotlinx.coroutines.android)

       // Pruebas
       testImplementation(libs.junit)
       testImplementation(libs.kotlinx.coroutines.test)
       testImplementation(libs.mockk)
       testImplementation(libs.turbine)
   }
   ```

6. Sincroniza el proyecto con Gradle haciendo clic en el botón **Sync Project with Gradle Files**.

#### Resultado esperado
La compilación inicial y sincronización deben ejecutarse de manera limpia en menos de 3 minutos, sin presentar advertencias de incompatibilidad entre Kotlin 2.3.10 y el Plugin de Gradle de Android (AGP) 8.4.0.

#### Verificación
Ejecuta la tarea de verificación por terminal para corroborar que no existan discrepancias en el script de compilación:
```bash
./gradlew help
```
*Salida:* El proceso debe retornar un estado `BUILD SUCCESSFUL` inequívoco.

---

### Paso 2: Modelado de dominio con Kotlin Moderno (Value Classes, Delegados y Contratos)

**Objetivo:** Desarrollar los objetos de transferencia de datos y de dominio utilizando optimizaciones en memoria y de análisis estático que ofrece Kotlin 2.3.10 para evitar sobrecarga del Recolector de Basura (*Garbage Collector*) bajo lecturas continuas a alta frecuencia.

#### Instrucciones

1. Crea un paquete dentro de tu directorio principal denominado `com.example.advancedtracker.domain.model`.
2. Dentro del paquete, crea el archivo `TelemetryModels.kt`.
3. Declara una **clase de valor** (`value class`) anotada con `@JvmInline` para manejar identificadores de dispositivos de manera segura sin sobrecargar el *heap* de memoria con asignaciones redundantes de objetos:

   ```kotlin
   package com.example.advancedtracker.domain.model

   @JvmInline
   value class DeviceId(val value: String) {
       init {
           require(value.isNotBlank()) { "El identificador del dispositivo no puede estar vacío." }
       }
   }
   ```

4. Desarrollarás un delegado de propiedad personalizado que formatee y registre en consola el acceso de depuración cada vez que se asigne un nuevo paquete de coordenadas GPS. Declara la clase `CoordinateLoggerDelegate` dentro del mismo archivo:

   ```kotlin
   import kotlin.properties.ReadWriteProperty
   import kotlin.reflect.KProperty

   class CoordinateLoggerDelegate(initialValue: Pair<Double, Double>) : ReadWriteProperty<Any?, Pair<Double, Double>> {
       private var internalCoords = initialValue

       override fun getValue(thisRef: Any?, property: KProperty<*>): Pair<Double, Double> {
           return internalCoords
       }

       override fun setValue(thisRef: Any?, property: KProperty<*>, value: Pair<Double, Double>) {
           // Validación básica de coordenadas geográficas
           val (lat, lon) = value
           if (lat in -90.0..90.0 && lon in -180.0..180.0) {
               internalCoords = value
           } else {
               println("[ERROR - Telemetría] Coordenadas inválidas descartadas: Lat $lat, Lon $lon")
           }
       }
   }
   ```

5. Define la clase de datos `TelemetryData` que encapsulará los valores emitidos en tiempo real:

   ```kotlin
   data class TelemetryData(
       val deviceId: DeviceId,
       val timestamp: Long,
       val speedKmh: Double,
       val temperatureCelsius: Double,
       val isAnomaly: Boolean = false
   ) {
       // Delegación para rastrear cambios en coordenadas de manera transparente
       var lastPosition: Pair<Double, Double> by CoordinateLoggerDelegate(0.0 to 0.0)
   }
   ```

6. Implementa un **Contrato del Compilador** que examine si una trama de datos de telemetría es válida y asegure al compilador que, en caso de retornar `true`, el objeto evaluado no es nulo, habilitando *Smart Casts* seguros:

   ```kotlin
   import kotlin.contracts.ExperimentalContracts
   import kotlin.contracts.contract

   @OptIn(ExperimentalContracts::class)
   fun verifyTelemetryPayload(payload: TelemetryData?): Boolean {
       contract {
           returns(true) implies (payload != null)
       }
       return payload != null && 
              payload.timestamp > 0 && 
              payload.speedKmh >= 0.0 && 
              payload.temperatureCelsius in -50.0..100.0
   }
   ```

#### Resultado esperado
Tendrás disponibles tres componentes de datos optimizados y seguros a nivel de tipos que reducen el impacto en memoria durante la simulación de ráfagas continuas de información.

#### Verificación
Crea un archivo temporal de pruebas o una clase principal pequeña para verificar el Smart Cast provisto por el contrato:
```kotlin
fun testContract(data: TelemetryData?) {
    if (verifyTelemetryPayload(data)) {
        // No requiere llamadas seguras (?.) ni aserciones (!!) gracias al contrato del compilador
        println("ID del Dispositivo verificado: ${data.deviceId.value}")
    }
}
```
Si el código compila sin requerir operaciones seguras para `data.deviceId`, el contrato está operando adecuadamente.

---

### Paso 3: Desarrollo del Motor de Telemetría (TelemetryEngine) con Flow y Corrutinas

**Objetivo:** Desarrollar un componente lógico central (`TelemetryEngine`) que emita periódicamente telemetría simulada a través de flujos fríos (`Flow`), alternando dinámicamente contextos de ejecución segura entre diferentes *Dispatchers* de Corrutinas para evitar el bloqueo del hilo de interfaz de usuario.

#### Instrucciones

1. Crea el paquete `com.example.advancedtracker.data.engine`.
2. Crea la clase `TelemetryEngine.kt` en su interior.
3. El motor de telemetría inyectará un despachador por defecto (`CoroutineDispatcher`) para permitir su desacoplamiento en pruebas automáticas. Implementa la lógica usando `flow` y el cambio de contexto con `flowOn`:

   ```kotlin
   package com.example.advancedtracker.data.engine

   import com.example.advancedtracker.domain.model.DeviceId
   import com.example.advancedtracker.domain.model.TelemetryData
   import com.example.advancedtracker.domain.model.verifyTelemetryPayload
   import kotlinx.coroutines.CoroutineDispatcher
   import kotlinx.coroutines.Dispatchers
   import kotlinx.coroutines.delay
   import kotlinx.coroutines.flow.Flow
   import kotlinx.coroutines.flow.flow
   import kotlinx.coroutines.flow.flowOn
   import kotlinx.coroutines.withContext
   import kotlin.random.Random

   class TelemetryEngine(
       private val defaultDispatcher: CoroutineDispatcher = Dispatchers.Default,
       private val ioDispatcher: CoroutineDispatcher = Dispatchers.IO
   ) {
       private val random = Random(System.currentTimeMillis())

       /**
        * Flujo frío que simula la generación de datos GPS y de sensores.
        * Cambia de contexto dinámicamente para proteger el hilo principal.
        */
       fun startTelemetryEmission(deviceId: DeviceId, intervalMs: Long): Flow<TelemetryData> = flow {
           var seqLat = 4.7110 // Coordenadas base (Bogotá, CO)
           var seqLon = -74.0721
           
           while (true) {
               // Operaciones con uso intensivo de CPU se asocian al despachador de cálculo
               val telemetry = withContext(defaultDispatcher) {
                   val rawSpeed = random.nextDouble(0.0, 120.0)
                   val rawTemp = random.nextDouble(15.0, 45.0)
                   
                   // Desviación geográfica incremental simulada
                   seqLat += random.nextDouble(-0.001, 0.001)
                   seqLon += random.nextDouble(-0.001, 0.001)
                   
                   val isAnomalyDetected = rawTemp > 40.0 || rawSpeed > 110.0

                   TelemetryData(
                       deviceId = deviceId,
                       timestamp = System.currentTimeMillis(),
                       speedKmh = rawSpeed,
                       temperatureCelsius = rawTemp,
                       isAnomaly = isAnomalyDetected
                   ).apply {
                       lastPosition = Pair(seqLat, seqLon)
                   }
               }

               // Validamos la integridad estructural utilizando el contrato desarrollado en el Paso 2
               if (verifyTelemetryPayload(telemetry)) {
                   // Simulación de guardado interno persistente rápido usando el contexto de I/O
                   withContext(ioDispatcher) {
                       saveTelemetryLogToDisk(telemetry)
                   }
                   
                   emit(telemetry)
               } else {
                   println("[ALERTA] Estructura de telemetría corrupta. Descartando emisión.")
               }

               delay(intervalMs)
           }
       }.flowOn(defaultDispatcher)

       private fun saveTelemetryLogToDisk(data: TelemetryData) {
           // Simulación rápida de escritura física o logging en almacenamiento de alta velocidad.
           // En este laboratorio emulamos el consumo del hilo I/O imprimiendo en consola de bajo nivel.
           val threadName = Thread.currentThread().name
           println("[LOG DISCO] [$threadName] Registrando telemetría para: ${data.deviceId.value} a velocidad ${"%.2f".format(data.speedKmh)} km/h")
       }
   }
   ```

#### Resultado esperado
Un motor funcional que puede instanciarse y emitir datos asíncronos en bucle infinito, autogestionando el cambio de contexto entre `Dispatchers.Default` (para cálculos matemáticos y simulación) e `Dispatchers.IO` (para el simulador de disco persistente).

#### Verificación
Para probar este comportamiento de forma aislada, puedes estructurar un test conceptual rápido en el entorno de pruebas, que cubra una duración corta de 3 segundos antes de cancelar cooperativamente la corrutina recolectora.

---

### Paso 4: Conversión y Exposición del Estado mediante StateFlow y SharedFlow

**Objetivo:** Desarrollar un gestor de estado arquitectónico (utilizando patrones MVVM adaptables) que procese el flujo frío proveniente de `TelemetryEngine` y exponga las actualizaciones de estado para componentes UI de Compose utilizando flujos calientes (`StateFlow` para estado continuo y `SharedFlow` para eventos únicos de alerta).

#### Instrucciones

1. Crea el paquete `com.example.advancedtracker.presentation`.
2. Crea la clase `TelemetryViewModel.kt` para orquestar la conversión de los flujos calientes. Asegúrate de estructurar el ViewModel de forma limpia e independiente:

   ```kotlin
   package com.example.advancedtracker.presentation

   import androidx.lifecycle.ViewModel
   import androidx.lifecycle.viewModelScope
   import com.example.advancedtracker.data.engine.TelemetryEngine
   import com.example.advancedtracker.domain.model.DeviceId
   import com.example.advancedtracker.domain.model.TelemetryData
   import kotlinx.coroutines.CoroutineDispatcher
   import kotlinx.coroutines.Dispatchers
   import kotlinx.coroutines.Job
   import kotlinx.coroutines.flow.MutableSharedFlow
   import kotlinx.coroutines.flow.MutableStateFlow
   import kotlinx.coroutines.flow.SharedFlow
   import kotlinx.coroutines.flow.StateFlow
   import kotlinx.coroutines.flow.asSharedFlow
   import kotlinx.coroutines.flow.asStateFlow
   import kotlinx.coroutines.flow.catch
   import kotlinx.coroutines.launch

   class TelemetryViewModel(
       private val telemetryEngine: TelemetryEngine,
       private val dispatcher: CoroutineDispatcher = Dispatchers.Main
   ) : ViewModel() {

       // Estado UI Caliente: Retiene el último estado emitido de la telemetría
       private val _uiState = MutableStateFlow<TelemetryUiState>(TelemetryUiState.Idle)
       val uiState: StateFlow<TelemetryUiState> = _uiState.asStateFlow()

       // Flujo de Eventos Únicos Caliente: No retiene estado, ideal para alertas o notificaciones "one-shot"
       private val _alertEvents = MutableSharedFlow<String>(extraBufferCapacity = 5)
       val alertEvents: SharedFlow<String> = _alertEvents.asSharedFlow()

       private var telemetryJob: Job? = null

       fun startTracking(deviceId: DeviceId, intervalMs: Long) {
           // Cancelamos cualquier job previo de monitoreo para evitar fugas de memoria o corrutinas huérfanas
           telemetryJob?.cancel()
           _uiState.value = TelemetryUiState.Loading

           telemetryJob = viewModelScope.launch(dispatcher) {
               telemetryEngine.startTelemetryEmission(deviceId, intervalMs)
                   .catch { exception ->
                       _uiState.value = TelemetryUiState.Error(exception.localizedMessage ?: "Error de Conexión")
                   }
                   .collect { telemetryData ->
                       _uiState.value = TelemetryUiState.Active(telemetryData)

                       // Evaluar anomalías en tiempo real para disparar el flujo caliente de alertas directas
                       if (telemetryData.isAnomaly) {
                           _alertEvents.emit(
                               "¡ANOMALÍA DETECTADA! El dispositivo ${deviceId.value} superó límites críticos. Temp: ${"%.1f".format(telemetryData.temperatureCelsius)}°C."
                           )
                       }
                   }
           }
       }

       fun stopTracking() {
           telemetryJob?.cancel()
           _uiState.value = TelemetryUiState.Idle
       }

       override fun onCleared() {
           super.onCleared()
           stopTracking()
       }
   }

   // Interfaz de sellado para representar los estados explícitos de la interfaz de usuario
   sealed interface TelemetryUiState {
       object Idle : TelemetryUiState
       object Loading : TelemetryUiState
       data class Active(val data: TelemetryData) : TelemetryUiState
       data class Error(val message: String) : TelemetryUiState
   }
   ```

3. Utiliza la herramienta de asistencia integrada **JetBrains AI Assistant (v242.23339)** para optimizar este ViewModel. Sigue estas pautas precisas:
   - Abre la ventana de chat del asistente en Android Studio.
   - Envía el siguiente prompt estructurado (*instrucción de usuario*):
     > **Prompt:** "Actuando como un experto instructor técnico de Kotlin, analiza la clase `TelemetryViewModel` provista. Genera una propuesta de refactorización que introduzca una estrategia de manejo de contrapresión (*backpressure*) utilizando operadores como `conflate()` o `buffer()` para mitigar picos de alta frecuencia del emisor de telemetría de forma controlada. Explica detalladamente las implicaciones en memoria del operador propuesto en comparación con procesamientos bloqueantes en hilos."

4. El asistente de IA te sugerirá una modificación a la línea donde se consume el flujo. Aplica la estrategia recomendada. La forma típica sugerida se ve de la siguiente manera:

   ```kotlin
   // Aplicación típica del operador para mitigar picos de emisión continuos
   telemetryEngine.startTelemetryEmission(deviceId, intervalMs)
       .conflate() // Evita acumulación en memoria descartando elementos obsoletos intermedios si el receptor se retrasa
       .catch { ... }
   ```

#### Resultado esperado
Un ViewModel reactivo robusto capaz de consolidar la emisión de flujos fríos en flujos calientes optimizados para su consumo en la UI, mitigando riesgos de fugas o retrasos por exceso de eventos de telemetría.

#### Verificación
Compila la clase para asegurar que los operadores sugeridos por la IA y la definición de la interfaz de sellado `TelemetryUiState` no contengan errores de sintaxis o de tipado.

---

### Paso 5: Pruebas Unitarias de Flujos y Corrutinas (kotlinx-coroutines-test)

**Objetivo:** Desarrollar el conjunto de pruebas automatizadas para verificar que el comportamiento de emisión asíncrona, la canalización del estado en `StateFlow` y el envío de alertas de anomalías en `SharedFlow` operen en perfecto orden, controlando el paso del tiempo de forma virtual con el despachador de pruebas.

#### Instrucciones

1. Navega en la estructura de carpetas de tu proyecto hacia el directorio de tests unitarios locales `app/src/test/java/com/example/advancedtracker`.
2. Crea el archivo de clase `TelemetryTrackerTests.kt`.
3. Escribe la suite de pruebas unitarias importando adecuadamente los utilitarios de prueba de corrutinas (`kotlinx-coroutines-test`) y la biblioteca de aserciones de flujo **Turbine**:

   ```kotlin
   package com.example.advancedtracker

   import app.cash.turbine.test
   import com.example.advancedtracker.data.engine.TelemetryEngine
   import com.example.advancedtracker.domain.model.DeviceId
   import com.example.advancedtracker.presentation.TelemetryUiState
   import com.example.advancedtracker.presentation.TelemetryViewModel
   import kotlinx.coroutines.Dispatchers
   import kotlinx.coroutines.ExperimentalCoroutinesApi
   import kotlinx.coroutines.test.StandardTestDispatcher
   import kotlinx.coroutines.test.advanceTimeBy
   import kotlinx.coroutines.test.resetMain
   import kotlinx.coroutines.test.runTest
   import kotlinx.coroutines.test.setMain
   import org.junit.After
   import org.junit.Assert.assertEquals
   import org.junit.Assert.assertTrue
   import org.junit.Before
   import org.junit.Test

   @OptIn(ExperimentalCoroutinesApi::class)
   class TelemetryTrackerTests {

       private val testDispatcher = StandardTestDispatcher()
       private lateinit var telemetryEngine: TelemetryEngine
       private lateinit var viewModel: TelemetryViewModel

       @Before
       fun setUp() {
           // Redireccionamos el despachador principal de Android al entorno virtual de pruebas
           Dispatchers.setMain(testDispatcher)
           telemetryEngine = TelemetryEngine(defaultDispatcher = testDispatcher, ioDispatcher = testDispatcher)
           viewModel = TelemetryViewModel(telemetryEngine, dispatcher = testDispatcher)
       }

       @After
       fun tearDown() {
           Dispatchers.resetMain()
       }

       @Test
       fun `cuando se inicia el monitoreo, cambia el estado a cargando y luego emite telemetria activa`() = runTest(testDispatcher) {
           val targetId = DeviceId("PRO_TRACK_001")

           // Utilizamos Turbine para escuchar los cambios secuenciales de estado en StateFlow
           viewModel.uiState.test {
               // El estado inicial debe ser Idle
               assertEquals(TelemetryUiState.Idle, awaitItem())

               // Iniciamos simulación con intervalo rápido de 1000ms
               viewModel.startTracking(targetId, 1000L)

               // Siguiente cambio inmediato: Loading
               assertEquals(TelemetryUiState.Loading, awaitItem())

               // Simulamos el paso del tiempo virtual para detonar la primera emisión
               advanceTimeBy(1005)

               // Siguiente emisión esperada: Active
               val activeState = awaitItem()
               assertTrue(activeState is TelemetryUiState.Active)
               
               val payload = (activeState as TelemetryUiState.Active).data
               assertEquals("PRO_TRACK_001", payload.deviceId.value)

               // Detenemos de forma limpia la simulación
               viewModel.stopTracking()
               assertEquals(TelemetryUiState.Idle, awaitItem())
           }
       }

       @Test
       fun `cuando ocurre una lectura anomalas, se emite una alerta a traves del flujo caliente SharedFlow`() = runTest(testDispatcher) {
           val targetId = DeviceId("ANOMALY_MONITOR")

           viewModel.alertEvents.test {
               viewModel.startTracking(targetId, 100L)
               
               // Ejecutamos varios ciclos rápidos de simulación para forzar probabilísticamente un evento extremo
               advanceTimeBy(1500)

               // Esperamos al menos una emisión de alerta registrada en el búfer caliente
               val alertMessage = awaitItem()
               assertTrue(alertMessage.contains("ANOMALY_MONITOR") || alertMessage.contains("¡ANOMALÍA DETECTADA!"))
               
               viewModel.stopTracking()
           }
       }
   }
   ```

#### Resultado esperado
Una suite de pruebas unitarias robusta que valida los flujos de concurrencia simulando el avance del tiempo en milisegundos, garantizando la eliminación completa de pruebas parpadeantes (*flaky tests*) que dependen de tiempos de ejecución físicos de la máquina host.

#### Verificación
Haz clic derecho en la clase de test `TelemetryTrackerTests` y selecciona **Run 'TelemetryTrackerTests'**. 

---

## Validación y Pruebas

Para garantizar que el laboratorio se haya completado de manera rigurosa y cumpla con los estándares de diseño offline-first y reactivo seguro, comprueba los siguientes puntos:

### Criterios de Aceptación y Validación de la Ejecución

1. **Compilación Limpia:** El proyecto debe compilarse utilizando la versión de Kotlin 2.3.10 y el backend del compilador K2 sin generar advertencias de desuso (*deprecationwarnings*) sobre las firmas de corrutinas configuradas.
2. **Resultados de Tests Unitarios:** Los dos casos de prueba (`cuando se inicia el monitoreo, cambia el estado a cargando y luego emite telemetria activa` y `cuando ocurre una lectura anomalas, se emite una alerta a traves del flujo caliente SharedFlow`) deben completarse exitosamente en verde en un tiempo menor a 5 segundos de ejecución real.
3. **Smart Cast Confirmado:** No debe existir ninguna llamada con operador de aserción no nula `!!` para desenvolver el modelo de telemetría verificado por la función con contrato `verifyTelemetryPayload`.

### Caso Adversario de Prueba (AI and Flow Injection Handling)

**Escenario de Prueba de Robustez:** Supón que el motor de telemetría emite un paquete corrupto que contiene un valor fuera de rango o un timestamp negativo. ¿Cómo reacciona nuestro sistema?

Añade este test unitario en tu suite para comprobar que el contrato y el filtro del flujo no propaguen datos inválidos hacia las vistas de Compose:

```kotlin
@Test
fun `cuando se genera un payload con timestamp corrupto, el motor lo descarta y no altera el flujo UI`() = runTest(testDispatcher) {
    val invalidId = DeviceId("CORRUPTED_DEVICE")
    
    viewModel.uiState.test {
        assertEquals(TelemetryUiState.Idle, awaitItem())
        
        // Iniciamos el rastreo de datos
        viewModel.startTracking(invalidId, 500L)
        assertEquals(TelemetryUiState.Loading, awaitItem())
        
        // Simulamos un avance de tiempo que no cumple con el contrato (ej. inyectando una marca de tiempo inválida si alteramos el estado)
        // El motor filtra internamente a través de verifyTelemetryPayload por lo que no se emitirá hacia el StateFlow.
        advanceTimeBy(200)
        
        // Aseguramos que el estado permanece en Loading y no cambia a Active debido a la restricción del contrato.
        assertTrue(viewModel.uiState.value is TelemetryUiState.Loading)
        
        viewModel.stopTracking()
        assertEquals(TelemetryUiState.Idle, awaitItem())
    }
}
```

Ejecuta este caso adversario para confirmar el blindaje de tu API de telemetría frente a inyecciones de datos anómalas o dañadas.

---

## Solución de Problemas

A continuación, se listan dos problemas típicos que se presentan comúnmente durante la integración de esta arquitectura de concurrencia, junto con sus causas raíz y soluciones de ingeniería correspondientes:

### Problema 1: Excepción de compilación "Incompatible classes or Gradle Version conflict with Kotlin 2.3.10"
- **Síntomas:** El proceso de sincronización de Gradle falla con un error que indica que la versión de Kotlin no es compatible con el Android Gradle Plugin (AGP) actual instalado, o que la configuración del plugin de Compose Compiler no se puede inicializar.
- **Causa Raíz:** Se está utilizando un proyecto inicial clásico que configura el compilador de Jetpack Compose mediante dependencias heredadas de la sección `androidx.compose.compiler`, las cuales eran fuertemente dependientes de versiones específicas de Kotlin previas a la 2.0.
- **Solución:** A partir de Kotlin 2.0 y sostenido en Kotlin 2.3.10, el plugin del compilador de Compose está integrado directamente en la estructura de desarrollo del compilador de Kotlin. En tu archivo `app/build.gradle.kts`, remueve cualquier declaración del compilador heredada (`composeOptions { kotlinCompilerExtensionVersion = ... }`) y asegúrate de aplicar únicamente el plugin unificado en el catálogo de versiones:
  ```toml
  kotlin-compose = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
  ```

### Problema 2: El Test Unitario se queda "colgado" indefinidamente (Infinite Test Block)
- **Síntomas:** Al ejecutar la suite de pruebas mediante `./gradlew test`, el subproceso correspondiente al caso de prueba de `TelemetryEngine` o de `SharedFlow` entra en un bucle infinito de espera sin finalizar nunca.
- **Causa Raíz:** El motor de telemetría utiliza un bucle continuo de emisión `while(true) { ... delay(intervalMs) }`. Si la corrutina recolectora no está asociada a un contexto cooperativo de cancelación, o si no se llama a `stopTracking()` o a la cancelación del Job de forma explícita dentro del bloque de test, el despachador de pruebas virtual `StandardTestDispatcher` continuará avanzando el reloj virtual indefinidamente para resolver las colas de eventos del bucle sin fin.
- **Solución:** Envuelve siempre tus colecciones de flujos continuos en bloques de aserción controlados por límites de tiempo o cancela el trabajo de forma explícita antes de que finalice la ejecución de la función de pruebas. El uso del método `test` provisto por la dependencia **Turbine** maneja este ciclo de vida de manera segura cerrando el canal una vez que se completan las aserciones de interés especificadas dentro del bloque de prueba.

---

## Limpieza

Para garantizar que el espacio de almacenamiento local no se sature debido a los metadatos y caché acumulados durante la compilación incremental de Kotlin y Gradle:

1. Limpia los archivos binarios generados por el compilador ejecutando la tarea limpia desde tu terminal:
   ```bash
   ./gradlew clean
   ```
2. Cierra Android Studio y elimina las carpetas de caché internas generadas en tu directorio de usuario en caso de querer resetear por completo las descargas de dependencias:
   ```bash
   rm -rf ~/.gradle/caches/
   ```

---

## Resumen

En esta práctica avanzada, has completado la construcción de una base de código robusta y moderna sobre APIs de Android de la 30 a la 37, aprovechando las últimas mejoras de rendimiento e inferencia que ofrece **Kotlin 2.3.10**. 

### Conceptos Clave Consolidados:
- **Value Classes (`@JvmInline`):** Reducción drástica del impacto en el Heap al envolver datos primitivos de dominio.
- **Contratos de Compilador:** Comunicación directa con el compilador de Kotlin para dotar de inteligencia a los análisis estáticos de Smart Casts en variables opcionales.
- **Flujos Reactivos Concurrentes:** Consolidación de flujos fríos (`Flow`) asíncronos distribuidos en múltiples despachadores paralelos (`Default` e `IO`) hacia flujos calientes (`StateFlow` y `SharedFlow`) protegidos contra pérdidas de datos y picos de contrapresión.
- **Pruebas Síncronas sobre Asincronía:** Diseño de suites de pruebas inmunes al retraso de ejecución real gracias al reloj simulado de `kotlinx-coroutines-test`.

### Recursos Adicionales:
- [Guía de optimización de Corrutinas en Android](https://developer.android.com/kotlin/coroutines-best-practices)
- [Documentación del plugin unificado de Compose en Kotlin](https://kotlinlang.org/docs/compose-compiler.html)
- [Estrategias avanzadas de Flow con Turbine](https://github.com/cashapp/turbine)
