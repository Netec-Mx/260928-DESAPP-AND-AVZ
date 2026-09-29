# Implementación de arquitectura MVVM + Clean Architecture con ViewModel, StateFlow, Repository, Use Cases e inyección de dependencias con Hilt

## ## Metadatos

| Métrica | Detalle |
| :--- | :--- |
| **Duración** | 216 minutos |
| **Complejidad** | Alta (Hard) |
| **Nivel de Bloom** | Crear (Create) |

---

## ## Descripción General

En este laboratorio, los estudiantes tomarán la base de código visual desarrollada en la práctica anterior y realizarán una reestructuración profunda aplicando **Clean Architecture** y el patrón de presentación **MVVM (Model-View-ViewModel)**. 

Se modularizará y separará el proyecto en tres capas lógicas bien definidas: **Dominio** (casos de uso y entidades puras), **Datos** (repositorios y simuladores de telemetría de geolocalización) y **Presentación** (ViewModels, Compose UI State e inyección de dependencias). El flujo de datos se regirá por un modelo unidireccional estricto (**UDF** - *Unidirectional Data Flow*) mediante el uso de `StateFlow` para el estado persistente de la UI y `SharedFlow` para la gestión de eventos únicos (navegación, alertas y errores). Finalmente, se integrará **Dagger Hilt** para centralizar la inyección de dependencias y se utilizará **JetBrains AI Assistant** de forma guiada para diseñar pruebas de estrés y refactorizar el código de presentación.

---

## ## Objetivos de Aprendizaje

Al finalizar este laboratorio, serás capaz de:
- [ ] **Refactorizar** un codebase heredado dividiendo las clases en paquetes desacoplados que respeten las fronteras de Clean Architecture (`domain`, `data`, `ui`).
- [ ] **Configurar e implementar** inyección de dependencias (DI) robusta mediante módulos de Dagger Hilt para proveer interfaces de repositorio y casos de uso.
- [ ] **Modelar y consumir** estados de UI inmutables a través de `StateFlow` con recolección segura respecto al ciclo de vida (`collectAsStateWithLifecycle`).
- [ ] **Implementar y emitir** eventos de disparo único (*one-off events*) como redirecciones y toasts utilizando `SharedFlow` y `LaunchedEffect` en la capa visual.
- [ ] **Escribir pruebas unitarias** utilizando librerías de testing asíncrono para verificar que los ViewModels respondan adecuadamente a flujos reactivos interrumpidos o fallidos.

---

## ## Prerrequisitos

### 1. Conocimientos Teóricos
- Comprensión de los principios SOLID de diseño orientado a objetos.
- Conocimiento de los conceptos de Clean Architecture: separación de responsabilidades, flujos de dependencia hacia adentro e inversión de dependencias.
- Familiaridad con la programación asíncrona mediante Kotlin Coroutines y el ecosistema reactivo de Kotlin Flows (`Flow`, `StateFlow`, `SharedFlow`).

### 2. Acceso y Entorno Operativo
- Código fuente visual funcional de la práctica de geolocalización y telemetría anterior (o estructura base equivalente de Compose).
- Cuenta activa con acceso a **JetBrains AI Assistant** (Licencia de pago/suscripción individual o corporativa activa, con el plugin "JetBrains AI Assistant" instalado y habilitado en el IDE de Android Studio).

---

## ## Entorno de Laboratorio

### Especificaciones de Hardware Requeridas

| Recurso | Configuración Mínima | Configuración Recomendada |
| :--- | :--- | :--- |
| **Procesador** | Intel Core i7 / AMD Ryzen 7 de 11ª Gen (x86_64) | Apple Silicon (M1/M2/M3 ARM64) o AMD Ryzen 9 de última generación |
| **Memoria RAM** | 16 GB DDR4/DDR5 | 32 GB DDR5 para ejecución concurrente de Gradle + Emulador |
| **Almacenamiento** | 20 GB de espacio libre (SSD SATA III) | 40 GB de espacio libre en Unidad de Estado Sólido NVMe M.2 |
| **Conexión de Red** | Banda ancha de 10 Mbps de bajada | Conexión de fibra simétrica de 50 Mbps o superior |

### Especificaciones de Software y Versiones Exactas

| Software / Herramienta | Edición y Arquitectura | Versión Exacta | Enlace de Descarga / Fuente Oficial |
| :--- | :--- | :--- | :--- |
| **Android Studio** | Ladybug, 64 bits (x86_64 o ARM64) | `2024.2.1 Patch 3` | [Android Studio Developer Portal](https://developer.android.com/studio) |
| **Eclipse Temurin JDK** | LTS OpenJDK, 64 bits (x86_64/ARM64) | `17.0.10+7` | [Adoptium Foundation](https://adoptium.net/temurin/releases/?version=17) |
| **Kotlin Compiler / Gradle Plugin** | Kotlin Gradle Plugin (KGP) / Compiler | `2.1.10` | [Kotlin Releases](https://kotlinlang.org/) |
| **Android Gradle Plugin (AGP)** | Build Tools Integration | `8.8.0` | [Android Developer Build Tools](https://developer.android.com/studio/releases/gradle-plugin) |
| **Jetpack Compose BOM** | Bill of Materials (BOM) | `2026.02.01` | [Google Maven Repository](https://maven.google.com/) |
| **Dagger Hilt** | Core DI & Compiler Integration | `2.51.1` | [Dagger Hilt Guide](https://dagger.dev/hilt/) |
| **Hilt Navigation Compose** | Compose Navigation Integration | `1.2.0` | [Google Maven Repository](https://maven.google.com/) |
| **JetBrains AI Assistant** | IDE Plugin Suite | `242.23339` | [JetBrains Marketplace](https://plugins.jetbrains.com/plugin/17777-jetbrains-ai-assistant) |

---

## ## Instrucciones Paso a Paso

El proyecto base de esta práctica se encuentra en el paquete base de la aplicación: `com.example.advancedtracker`. 
Asegúrate de que la configuración global de tu SDK tenga definidos los siguientes valores:
*   `compileSdk = 35`
*   `minSdk = 30`
*   `targetSdk = 35`
*   `JVM Target = "17"`

---

### Paso 1: Configuración de Dependencias de Hilt y Arquitectura en el Proyecto

En este paso, configurarás los archivos de construcción de Gradle para incorporar los plugins y dependencias necesarias para la inyección de dependencias con Dagger Hilt y el procesamiento de anotaciones con KSP.

1. Abre el archivo de configuración global de plugins de Gradle de nivel de proyecto (`build.gradle.kts` en el directorio raíz):

```kotlin
// build.gradle.kts (Raíz del Proyecto)
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.compose) apply false
    // Plugin de Hilt para la inyección de dependencias
    id("com.google.dagger.hilt.android") version "2.51.1" apply false
    // Herramienta KSP para procesar anotaciones con rendimiento optimizado
    id("com.google.devtools.ksp") version "2.1.10-1.0.29" apply false
}
```

2. Abre el archivo de configuración de dependencias de nivel de módulo (`app/build.gradle.kts`):

```kotlin
// app/build.gradle.kts
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
    id("com.google.dagger.hilt.android")
    id("com.google.devtools.ksp")
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
            isMinifyEnabled = false
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
        freeCompilerArgs += listOf("-opt-in=kotlinx.coroutines.ExperimentalCoroutinesApi")
    }
    buildFeatures {
        compose = true
    }
}

dependencies {
    // Jetpack Compose BOM
    val composeBom = platform("androidx.compose:compose-bom:2026.02.01")
    implementation(composeBom)
    androidTestImplementation(composeBom)

    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.ui:ui-graphics")
    implementation("androidx.compose.ui:ui-tooling-preview")
    implementation("androidx.compose.material3:material3")
    
    // Lifecycle components y recolección segura en Compose
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.8.7")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.7")
    implementation("androidx.lifecycle:lifecycle-runtime-compose:2.8.7")

    // Dagger Hilt para Inyección de Dependencias
    implementation("com.google.dagger:hilt-android:2.51.1")
    ksp("com.google.dagger:hilt-compiler:2.51.1")
    implementation("androidx.hilt:hilt-navigation-compose:1.2.0")

    // Coroutines de Kotlin para concurrencia asíncrona
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.10.1")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.10.1")

    // Componentes de prueba
    testImplementation("junit:junit:4.13.2")
    testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.10.1")
    testImplementation("org.mockito:mockito-core:5.11.0")
    testImplementation("org.mockito.kotlin:mockito-kotlin:5.2.1")
}
```

3. Sincroniza el proyecto pulsando **Sync Now** en la barra superior de Android Studio.

#### Verificación de la compilación de dependencias:
Ejecuta la tarea de Gradle de verificación en la terminal embebida de Android Studio para asegurar que los scripts se resuelvan sin problemas de dependencias:
```bash
./gradlew dependencies --configuration kotlinCompilerPluginClasspath
```
**Salida esperada de la terminal:**
Debe listar las dependencias de compilación y confirmar `BUILD SUCCESSFUL` sin conflictos de versiones del compilador de Kotlin ni de KSP.

---

### Paso 2: Creación de la Estructura de Paquetes (Clean Architecture)

En este paso, estructurarás los directorios de tu proyecto para separar de manera clara los componentes lógicos del sistema bajo el paquete base `com.example.advancedtracker`.

1. Dirígete a la pestaña de exploración de proyecto (*Project Tool Window*) de Android Studio.
2. Haz clic derecho sobre el paquete raíz `com.example.advancedtracker` y crea los siguientes directorios para definir las capas arquitectónicas. La estructura final debe lucir exactamente así:

```text
com.example.advancedtracker/
│
├── data/
│   ├── datasource/
│   └── repository/
│
├── domain/
│   ├── model/
│   ├── repository/
│   └── usecase/
│
├── di/
│
└── ui/
    ├── telemetry/
    └── theme/
```

3. Asegúrate de que no haya clases huérfanas en directorios incorrectos. Esta segmentación garantiza que la capa de `domain` sea agnóstica a frameworks externos como Hilt, Room, Retrofit o Jetpack Compose.

---

### Paso 3: Definición del Dominio (Modelos y Casos de Uso)

Aquí construirás el núcleo lógico de tu negocio (capa *Domain*). Esta capa contiene entidades de datos inmutables, la definición abstracta del repositorio mediante interfaces e implementaciones puras de casos de uso sin elementos del framework de Android.

1. Crea la clase inmutable de modelo de dominio `TelemetryData.kt` en el paquete `com.example.advancedtracker.domain.model`:

```kotlin
// TelemetryData.kt
package com.example.advancedtracker.domain.model

import java.util.Date

/**
 * Representa la entidad de datos de telemetría inmutable pura del negocio.
 * Es completamente independiente de bases de datos locales o respuestas HTTP.
 */
data class TelemetryData(
    val id: String,
    val latitude: Double,
    val longitude: Double,
    val speedKmh: Double,
    val timestamp: Date,
    val isMocked: Boolean
)
```

2. Crea la interfaz del repositorio `TelemetryRepository.kt` en el paquete `com.example.advancedtracker.domain.repository`:

```kotlin
// TelemetryRepository.kt
package com.example.advancedtracker.domain.repository

import com.example.advancedtracker.domain.model.TelemetryData
import kotlinx.coroutines.flow.Flow

/**
 * Define el contrato de abstracción para la recuperación y envío de telemetría.
 * Implementa el principio de inversión de dependencias de SOLID.
 */
interface TelemetryRepository {
    
    /**
     * Provee un flujo continuo de actualizaciones de telemetría en tiempo real.
     */
    fun observeTelemetry(): Flow<TelemetryData>

    /**
     * Almacena o envía un registro específico de telemetría.
     */
    suspend fun saveTelemetry(data: TelemetryData): Result<Unit>
    
    /**
     * Fuerza la actualización del stream de telemetría.
     */
    suspend fun forceRefresh()
}
```

3. Crea el caso de uso para observar las lecturas en tiempo real en `GetTelemetryStreamUseCase.kt` dentro del paquete `com.example.advancedtracker.domain.usecase`:

```kotlin
// GetTelemetryStreamUseCase.kt
package com.example.advancedtracker.domain.usecase

import com.example.advancedtracker.domain.model.TelemetryData
import com.example.advancedtracker.domain.repository.TelemetryRepository
import kotlinx.coroutines.flow.Flow
import javax.inject.Inject

/**
 * Caso de uso específico para suscribirse al stream reactivo de geolocalización.
 * Inyecta el contrato del repositorio, desacoplando la implementación concreta.
 */
class GetTelemetryStreamUseCase @Inject constructor(
    private val repository: TelemetryRepository
) {
    operator fun invoke(): Flow<TelemetryData> {
        return repository.observeTelemetry()
    }
}
```

4. Crea el caso de uso para el almacenamiento local/remoto en `SaveTelemetryUseCase.kt` dentro del paquete `com.example.advancedtracker.domain.usecase`:

```kotlin
// SaveTelemetryUseCase.kt
package com.example.advancedtracker.domain.usecase

import com.example.advancedtracker.domain.model.TelemetryData
import com.example.advancedtracker.domain.repository.TelemetryRepository
import javax.inject.Inject

/**
 * Caso de uso encargado de la lógica y validaciones previas al almacenamiento de la telemetría.
 */
class SaveTelemetryUseCase @Inject constructor(
    private val repository: TelemetryRepository
) {
    suspend operator fun invoke(data: TelemetryData): Result<Unit> {
        // Validación de negocio: Evitar almacenar coordenadas aberrantes o imposibles (v.g. Latitudes fuera del rango [-90, 90])
        if (data.latitude !in -90.0..90.0 || data.longitude !in -180.0..180.0) {
            return Result.failure(IllegalArgumentException("Coordenadas geográficas inválidas"))
        }
        return repository.saveTelemetry(data)
    }
}
```

**Verificación:**
Asegúrate de que no haya ninguna importación del tipo `import android.*` ni de librerías de UI en las clases creadas en este paso. El código debe ser Kotlin puro.

---

### Paso 4: Implementación de la Capa de Datos (Repositorio y Fuentes de Datos)

En esta capa implementarás la lógica de infraestructura para simular la recolección de telemetría mediante flujos asíncronos concurrentes.

1. Crea la clase de implementación del repositorio en `TelemetryRepositoryImpl.kt` dentro de `com.example.advancedtracker.data.repository`:

```kotlin
// TelemetryRepositoryImpl.kt
package com.example.advancedtracker.data.repository

import com.example.advancedtracker.domain.model.TelemetryData
import com.example.advancedtracker.domain.repository.TelemetryRepository
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.sync.Mutex
import kotlinx.coroutines.sync.withLock
import java.util.Date
import java.util.UUID
import javax.inject.Inject
import javax.inject.Singleton

/**
 * Implementación concreta del repositorio. Realiza la simulación de lecturas de sensores GPS.
 * Utiliza @Singleton para asegurar una única fuente de verdad en memoria.
 */
@Singleton
class TelemetryRepositoryImpl @Inject constructor() : TelemetryRepository {

    private val mutex = Mutex()
    private val storedTelemetryList = mutableListOf<TelemetryData>()

    // Coordenadas base (Santiago de Chile, como ejemplo geográfico de prueba)
    private var baseLatitude = -33.4489
    private var baseLongitude = -70.6693

    override fun observeTelemetry(): Flow<TelemetryData> = flow {
        var count = 0
        while (true) {
            // Simular perturbaciones leves de localización de un dispositivo en movimiento
            val randomOffsetLat = ((-100..100).random()) * 0.0001
            val randomOffsetLng = ((-100..100).random()) * 0.0001
            val instantSpeed = (20..110).random().toDouble()

            val telemetrySample = TelemetryData(
                id = UUID.randomUUID().toString(),
                latitude = baseLatitude + randomOffsetLat,
                longitude = baseLongitude + randomOffsetLng,
                speedKmh = instantSpeed,
                timestamp = Date(),
                isMocked = true
            )

            emit(telemetrySample)
            count++

            // Simular recolección continua cada 3 segundos
            delay(3000)
        }
    }

    override suspend fun saveTelemetry(data: TelemetryData): Result<Unit> {
        return mutex.withLock {
            try {
                // Simulación de retraso de almacenamiento (operación de entrada y salida asíncrona)
                delay(150)
                storedTelemetryList.add(data)
                Result.success(Unit)
            } catch (e: Exception) {
                Result.failure(e)
            }
        }
    }

    override suspend fun forceRefresh() {
        // En una implementación real, dispararía una actualización de la fuente GPS.
        delay(500)
    }
}
```

---

### Paso 5: Inyección de Dependencias con Dagger Hilt

Configurarás los mecanismos de inyección de dependencias para enlazar las interfaces declaradas en la capa de dominio con sus implementaciones de la capa de datos de forma automática mediante módulos Hilt.

1. Crea la clase base de la aplicación `TrackerApplication.kt` en el paquete raíz `com.example.advancedtracker`:

```kotlin
// TrackerApplication.kt
package com.example.advancedtracker

import android.app.Application
import dagger.hilt.android.HiltAndroidApp

/**
 * Punto de entrada de la aplicación que desencadena la generación de código de Hilt.
 */
@HiltAndroidApp
class TrackerApplication : Application()
```

2. Configura el archivo `AndroidManifest.xml` en tu directorio `app/src/main` para registrar esta clase. Este paso es fundamental para inicializar el contenedor de inyección de dependencias de la aplicación:

```xml
<!-- AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.advancedtracker">

    <!-- Declaración del nombre de la clase de aplicación para enlazar Hilt -->
    <application
        android:name=".TrackerApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Material3.DayNight.NoActionBar">
        
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:theme="@style/Theme.Material3.DayNight.NoActionBar">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

3. Crea el módulo de inyección de dependencias `RepositoryModule.kt` en el paquete `com.example.advancedtracker.di`:

```kotlin
// RepositoryModule.kt
package com.example.advancedtracker.di

import com.example.advancedtracker.data.repository.TelemetryRepositoryImpl
import com.example.advancedtracker.domain.repository.TelemetryRepository
import dagger.Binds
import dagger.Module
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent
import javax.inject.Singleton

/**
 * Módulo de Hilt encargado de proveer de forma inyectable la abstracción de Repositorio.
 */
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {

    @Binds
    @Singleton
    abstract fun bindTelemetryRepository(
        telemetryRepositoryImpl: TelemetryRepositoryImpl
    ): TelemetryRepository
}
```

---

### Paso 6: Implementación de la Capa de Presentación (ViewModel, StateFlow, SharedFlow)

En este paso diseñarás la lógica de presentación con un flujo de control estrictamente unidireccional (UDF) y eventos persistentes y de un solo disparo (*one-off*).

1. Define los estados inmutables en `TelemetryUiState.kt` en el paquete `com.example.advancedtracker.ui.telemetry`:

```kotlin
// TelemetryUiState.kt
package com.example.advancedtracker.ui.telemetry

import com.example.advancedtracker.domain.model.TelemetryData

/**
 * Representación inmutable del estado completo de la interfaz visual.
 */
data class TelemetryUiState(
    val isLoading: Boolean = false,
    val currentTelemetry: TelemetryData? = null,
    val lastError: String? = null,
    val isTrackingActive: Boolean = false,
    val hasUnsavedChanges: Boolean = false
)
```

2. Define la jerarquía sellada de eventos de disparo único en `TelemetryUiEvent.kt` dentro del paquete `com.example.advancedtracker.ui.telemetry`:

```kotlin
// TelemetryUiEvent.kt
package com.example.advancedtracker.ui.telemetry

/**
 * Eventos no persistentes orientados a disparar acciones transitorias de interfaz
 * como alertas temporales (Toasts, Snackbars) o navegación a otras pantallas.
 */
sealed interface TelemetryUiEvent {
    data class ShowToastMessage(val message: String) : TelemetryUiEvent
    data class NavigateToDetails(val telemetryId: String) : TelemetryUiEvent
    object NavigationBack : TelemetryUiEvent
}
```

3. Crea e implementa el ViewModel avanzado `TelemetryViewModel.kt` en `com.example.advancedtracker.ui.telemetry`:

```kotlin
// TelemetryViewModel.kt
package com.example.advancedtracker.ui.telemetry

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.advancedtracker.domain.model.TelemetryData
import com.example.advancedtracker.domain.usecase.GetTelemetryStreamUseCase
import com.example.advancedtracker.domain.usecase.SaveTelemetryUseCase
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.Job
import kotlinx.coroutines.flow.MutableSharedFlow
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.SharedFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asSharedFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.catch
import kotlinx.coroutines.flow.update
import kotlinx.coroutines.launch
import javax.inject.Inject

/**
 * Gestiona el estado de la UI y los flujos de comunicación con el negocio de telemetría.
 * Implementa el patrón de Flujo de Datos Unidireccional (UDF).
 */
@HiltViewModel
class TelemetryViewModel @Inject constructor(
    private val getTelemetryStreamUseCase: GetTelemetryStreamUseCase,
    private val saveTelemetryUseCase: SaveTelemetryUseCase
) : ViewModel() {

    // Estado interno mutable encapsulado (UDF)
    private val _uiState = MutableStateFlow(TelemetryUiState())
    // Estado expuesto inmutable para observación segura en Compose
    val uiState: StateFlow<TelemetryUiState> = _uiState.asStateFlow()

    // Flujo para la propagación segura de eventos que se consumen una sola vez (como Toasts o navegaciones)
    private val _uiEvent = MutableSharedFlow<TelemetryUiEvent>()
    val uiEvent: SharedFlow<TelemetryUiEvent> = _uiEvent.asSharedFlow()

    private var trackingJob: Job? = null

    init {
        // Arranca por defecto observando los cambios
        startTracking()
    }

    /**
     * Inicia de forma asíncrona la recolección del flujo reactivo del caso de uso.
     */
    fun startTracking() {
        if (trackingJob?.isActive == true) return

        _uiState.update { it.copy(isLoading = true, isTrackingActive = true, lastError = null) }

        trackingJob = viewModelScope.launch {
            getTelemetryStreamUseCase()
                .catch { exception ->
                    _uiState.update { 
                        it.copy(
                            isLoading = false, 
                            lastError = exception.localizedMessage ?: "Error desconocido de telemetría"
                        ) 
                    }
                    _uiEvent.emit(TelemetryUiEvent.ShowToastMessage("Fallo crítico del stream de geolocalización"))
                }
                .collect { telemetryData ->
                    _uiState.update { 
                        it.copy(
                            isLoading = false,
                            currentTelemetry = telemetryData
                        )
                    }
                }
        }
    }

    /**
     * Cancela la recolección activa del flujo y actualiza el estado de la UI.
     */
    fun stopTracking() {
        trackingJob?.cancel()
        _uiState.update { it.copy(isTrackingActive = false, isLoading = false) }
        viewModelScope.launch {
            _uiEvent.emit(TelemetryUiEvent.ShowToastMessage("Seguimiento pausado por el usuario"))
        }
    }

    /**
     * Guarda la lectura actual de telemetría haciendo uso del caso de uso correspondiente.
     */
    fun persistCurrentTelemetry() {
        val currentData = _uiState.value.currentTelemetry ?: return
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true) }
            val result = saveTelemetryUseCase(currentData)
            _uiState.update { it.copy(isLoading = false) }

            result.fold(
                onSuccess = {
                    _uiEvent.emit(TelemetryUiEvent.ShowToastMessage("Lectura grabada localmente de manera segura"))
                },
                onFailure = { error ->
                    _uiEvent.emit(TelemetryUiEvent.ShowToastMessage("Error al guardar: ${error.message}"))
                }
            )
        }
    }

    /**
     * Emite un evento asíncrono para navegar al panel de detalle.
     */
    fun navigateToTelemetryDetail() {
        val currentId = _uiState.value.currentTelemetry?.id ?: return
        viewModelScope.launch {
            _uiEvent.emit(TelemetryUiEvent.NavigateToDetails(currentId))
        }
    }

    override fun onCleared() {
        super.onCleared()
        trackingJob?.cancel() // Se asegura de cancelar las corrutinas activas al destruirse el ViewModel
    }
}
```

---

### Paso 7: Refactorización y Enlace de la Interfaz con Jetpack Compose

Conectaremos la interfaz visual de la aplicación con la estructura reactiva del ViewModel inyectando la referencia mediante Hilt y consumiendo el estado respetando el ciclo de vida.

1. Abre el archivo de actividad principal `MainActivity.kt` en el paquete `com.example.advancedtracker`:

```kotlin
// MainActivity.kt
package com.example.advancedtracker

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.ui.Modifier
import com.example.advancedtracker.ui.telemetry.TelemetryScreen
import com.example.advancedtracker.ui.theme.AdvancedTrackerTheme
import dagger.hilt.android.AndroidEntryPoint

/**
 * Punto de entrada visual del ciclo de vida de Android.
 * La anotación @AndroidEntryPoint es requerida por Hilt para la inyección en componentes del sistema.
 */
@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            AdvancedTrackerTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    TelemetryScreen()
                }
            }
        }
    }
}
```

2. Si aún no cuentas con un archivo de estilos básico de Jetpack Compose en tu proyecto, crea el paquete `com.example.advancedtracker.ui.theme` con la clase `Theme.kt`:

```kotlin
// Theme.kt
package com.example.advancedtracker.ui.theme

import androidx.compose.foundation.isSystemInDarkTheme
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.darkColorScheme
import androidx.compose.material3.lightColorScheme
import androidx.compose.runtime.Composable

private val DarkColorScheme = darkColorScheme()
private val LightColorScheme = lightColorScheme()

@Composable
fun AdvancedTrackerTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val colorScheme = if (darkTheme) DarkColorScheme else LightColorScheme

    MaterialTheme(
        colorScheme = colorScheme,
        content = content
    )
}
```

3. Diseña el Composable de pantalla principal `TelemetryScreen.kt` en `com.example.advancedtracker.ui.telemetry`. El composable utilizará el helper `collectAsStateWithLifecycle` para asegurar una recolección del flujo optimizada para el ciclo de vida:

```kotlin
// TelemetryScreen.kt
package com.example.advancedtracker.ui.telemetry

import android.widget.Toast
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.width
import androidx.compose.material3.Button
import androidx.compose.material3.ButtonDefaults
import androidx.compose.material3.CircularProgressIndicator
import androidx.compose.material3.Card
import androidx.compose.material3.CardDefaults
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.getValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import androidx.hilt.navigation.compose.hiltViewModel
import androidx.lifecycle.compose.collectAsStateWithLifecycle

@Composable
fun TelemetryScreen(
    modifier: Modifier = Modifier,
    viewModel: TelemetryViewModel = hiltViewModel()
) {
    val context = LocalContext.current
    
    // Recolectamos el estado de manera segura para evitar fugas y consumo innecesario de recursos
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    // Escuchamos de manera segura los eventos asíncronos puntuales mediante LaunchedEffect
    LaunchedEffect(key1 = true) {
        viewModel.uiEvent.collect { event ->
            when (event) {
                is TelemetryUiEvent.ShowToastMessage -> {
                    Toast.makeText(context, event.message, Toast.LENGTH_SHORT).show()
                }
                is TelemetryUiEvent.NavigateToDetails -> {
                    Toast.makeText(context, "Navegando al detalle de telemetría: ${event.telemetryId}", Toast.LENGTH_SHORT).show()
                    // Aquí iría el controlador de navegación real (NavController)
                }
                is TelemetryUiEvent.NavigationBack -> {
                    // Acción de retroceso
                }
            }
        }
    }

    Scaffold(
        modifier = modifier.fillMaxSize()
    ) { paddingValues ->
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(paddingValues)
                .padding(24.dp),
            horizontalAlignment = Alignment.CenterHorizontally,
            verticalArrangement = Arrangement.Center
        ) {
            Text(
                text = "Telemetría en Tiempo Real",
                style = MaterialTheme.typography.headlineMedium,
                fontWeight = FontWeight.Bold,
                color = MaterialTheme.colorScheme.primary
            )
            
            Spacer(modifier = Modifier.height(16.dp))

            // Indicador de carga
            if (uiState.isLoading) {
                CircularProgressIndicator()
                Spacer(modifier = Modifier.height(16.dp))
            }

            // Visualización del estado actual de geolocalización
            Card(
                modifier = Modifier.fillMaxWidth(),
                colors = CardDefaults.cardColors(containerColor = MaterialTheme.colorScheme.surfaceVariant),
                elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
            ) {
                Column(modifier = Modifier.padding(16.dp)) {
                    val telemetry = uiState.currentTelemetry
                    if (telemetry != null) {
                        Text(text = "ID: ${telemetry.id}", style = MaterialTheme.typography.bodySmall)
                        Spacer(modifier = Modifier.height(4.dp))
                        Text(text = "Latitud: ${telemetry.latitude}", style = MaterialTheme.typography.bodyLarge)
                        Text(text = "Longitud: ${telemetry.longitude}", style = MaterialTheme.typography.bodyLarge)
                        Spacer(modifier = Modifier.height(8.dp))
                        Text(
                            text = "Velocidad: ${telemetry.speedKmh} Km/H", 
                            style = MaterialTheme.typography.titleMedium,
                            fontWeight = FontWeight.Bold,
                            color = MaterialTheme.colorScheme.secondary
                        )
                        Spacer(modifier = Modifier.height(4.dp))
                        Text(text = "Fecha: ${telemetry.timestamp}", style = MaterialTheme.typography.bodyMedium)
                    } else {
                        Text(
                            text = "Buscando coordenadas satelitales activas...",
                            style = MaterialTheme.typography.bodyMedium,
                            fontWeight = FontWeight.Light
                        )
                    }
                }
            }

            Spacer(modifier = Modifier.height(24.dp))

            // Bloque de botones de interacción del usuario
            Row(
                modifier = Modifier.fillMaxWidth(),
                horizontalArrangement = Arrangement.SpaceEvenly
            ) {
                if (uiState.isTrackingActive) {
                    Button(
                        onClick = { viewModel.stopTracking() },
                        colors = ButtonDefaults.buttonColors(containerColor = MaterialTheme.colorScheme.error)
                    ) {
                        Text("Pausar GPS")
                    }
                } else {
                    Button(onClick = { viewModel.startTracking() }) {
                        Text("Iniciar GPS")
                    }
                }

                Spacer(modifier = Modifier.width(8.dp))

                Button(
                    onClick = { viewModel.persistCurrentTelemetry() },
                    enabled = uiState.currentTelemetry != null
                ) {
                    Text("Guardar Lectura")
                }
            }

            Spacer(modifier = Modifier.height(16.dp))

            Button(
                onClick = { viewModel.navigateToTelemetryDetail() },
                modifier = Modifier.fillMaxWidth(),
                colors = ButtonDefaults.buttonColors(containerColor = MaterialTheme.colorScheme.tertiary),
                enabled = uiState.currentTelemetry != null
            ) {
                Text("Ver Detalles Completos")
            }
        }
    }
}
```

---

### Paso 8: Configuración y Optimización Guiada por JetBrains AI Assistant (Refactorización y Testing)

En este paso, utilizarás el **JetBrains AI Assistant** (un asistente persistente integrado en tu IDE) para refactorizar la lógica del ViewModel y generar pruebas automatizadas avanzadas. 

#### 1. Contexto de uso de IA y Configuración de Pruebas Unitarias
El asistente opera en base a directrices o *instrucciones del sistema* del IDE y acepta mensajes directos del usuario (*prompts*). 

Para generar las pruebas de forma óptima, abre la ventana de chat del **JetBrains AI Assistant** en el lateral derecho de Android Studio. En primer lugar, copia el contenido de la clase `TelemetryViewModel` en el portapapeles o asegúrate de que el archivo esté abierto en el editor principal para que la IA tenga contexto completo del archivo seleccionado.

#### 2. Instrucción de Refactorización (Prompt)
Envía la siguiente instrucción de entrada en la barra de chat:

> **Prompt:**  
> *"Actúa como un desarrollador experto en Android. Analiza la clase `TelemetryViewModel.kt` abierta. Genera una batería completa de pruebas unitarias utilizando el framework de pruebas asíncronas de Kotlin Coroutines (`TestScope`, `UnconfinedTestDispatcher` y `runTest`). Utiliza Mockito para simular el comportamiento de los casos de uso `GetTelemetryStreamUseCase` y `SaveTelemetryUseCase`. Necesito cubrir dos escenarios fundamentales:*
> *1. Flujo de datos exitoso que recolecta y actualiza progresivamente el `TelemetryUiState` con datos simulados.*
> *2. Comportamiento ante un fallo crítico en el stream de datos del caso de uso de telemetría (caso adverso), verificando que el estado actualice el valor `lastError` y emita el evento transitorio `TelemetryUiEvent.ShowToastMessage` mediante el canal correspondiente. Devuelve la clase de prueba de manera limpia y lista para compilar en `src/test/java`."*

#### 3. Implementación de la Prueba Unitaria Generada
Crea un archivo llamado `TelemetryViewModelTest.kt` en el directorio de pruebas locales de la aplicación (`app/src/test/java/com/example/advancedtracker/ui/telemetry/TelemetryViewModelTest.kt`):

```kotlin
// TelemetryViewModelTest.kt
package com.example.advancedtracker.ui.telemetry

import com.example.advancedtracker.domain.model.TelemetryData
import com.example.advancedtracker.domain.usecase.GetTelemetryStreamUseCase
import com.example.advancedtracker.domain.usecase.SaveTelemetryUseCase
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.ExperimentalCoroutinesApi
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.flow.flowOf
import kotlinx.coroutines.test.UnconfinedTestDispatcher
import kotlinx.coroutines.test.resetMain
import kotlinx.coroutines.test.runTest
import kotlinx.coroutines.test.setMain
import org.junit.After
import org.junit.Assert.assertEquals
import org.junit.Assert.assertNotNull
import org.junit.Assert.assertTrue
import org.junit.Before
import org.junit.Test
import org.mockito.Mockito
import org.mockito.kotlin.any
import org.mockito.kotlin.mock
import org.mockito.kotlin.whenever
import java.util.Date

@OptIn(ExperimentalCoroutinesApi::class)
class TelemetryViewModelTest {

    private val testDispatcher = UnconfinedTestDispatcher()

    private val getTelemetryStreamUseCase: GetTelemetryStreamUseCase = mock()
    private val saveTelemetryUseCase: SaveTelemetryUseCase = mock()

    private lateinit var viewModel: TelemetryViewModel

    @Before
    fun setUp() {
        // Configuramos el despachador de pruebas para simular el hilo principal (Main) de Android
        Dispatchers.setMain(testDispatcher)
    }

    @After
    fun tearDown() {
        Dispatchers.resetMain()
    }

    @Test
    fun `cuando inicia tracking, el estado de UI se actualiza con coordenadas exitosas`() = runTest {
        // GIVEN
        val mockData = TelemetryData(
            id = "test-123",
            latitude = -12.0463,
            longitude = -77.0427,
            speedKmh = 55.5,
            timestamp = Date(),
            isMocked = true
        )
        whenever(getTelemetryStreamUseCase()).thenReturn(flowOf(mockData))

        // WHEN
        viewModel = TelemetryViewModel(getTelemetryStreamUseCase, saveTelemetryUseCase)
        viewModel.startTracking()

        // THEN
        val state = viewModel.uiState.value
        assertEquals(mockData, state.currentTelemetry)
        assertEquals(false, state.isLoading)
        assertEquals(null, state.lastError)
    }

    @Test
    fun `cuando falla el stream del GPS, el estado actualiza el error de manera controlada`() = runTest {
        // GIVEN (Simulamos un fallo inesperado del sensor mediante una excepción en el flow)
        val errorMessage = "Fallo de conexión de satélites GPS"
        whenever(getTelemetryStreamUseCase()).thenReturn(flow {
            throw RuntimeException(errorMessage)
        })

        // WHEN
        viewModel = TelemetryViewModel(getTelemetryStreamUseCase, saveTelemetryUseCase)
        viewModel.startTracking()

        // THEN
        val state = viewModel.uiState.value
        assertNotNull(state.lastError)
        assertTrue(state.lastError!!.contains(errorMessage))
        assertEquals(false, state.isLoading)
    }
}
```

---

## ## Validación y Pruebas

Para garantizar que el proceso de refactorización y la inyección de dependencias cumplan de forma robusta con la arquitectura diseñada, se realizarán tres niveles de validación: compilación/generación de código Hilt, pruebas de caja negra simulando fallas críticas de componentes, y la ejecución automatizada de la suite de pruebas unitarias.

### 1. Pruebas Unitarias Automatizadas
Ejecuta la batería de pruebas de software locales a través de la terminal integrada del IDE:

```bash
./gradlew testDebugUnitTest
```

**Salida exitosa esperada en la consola:**
```text
> Task :app:testDebugUnitTest

com.example.advancedtracker.ui.telemetry.TelemetryViewModelTest > cuando inicia tracking, el estado de UI se actualiza con coordenadas exitosas PASSED
com.example.advancedtracker.ui.telemetry.TelemetryViewModelTest > cuando falla el stream del GPS, el estado actualiza el error de manera controlada PASSED

BUILD SUCCESSFUL in 4s
7 actionable tasks: 7 executed
```

### 2. Prueba de Robustez ante Entradas Corruptas (Caso Adverso)
Abre la suite de pruebas unitarias y agrega una verificación específica para constatar el comportamiento del sistema cuando el caso de uso `SaveTelemetryUseCase` rechaza coordenadas inválidas (fuera de límites).

Agrega la siguiente prueba dentro del archivo `TelemetryViewModelTest.kt` para evaluar la inyección de reglas de negocio en cascada:

```kotlin
@Test
fun `cuando se intenta guardar una lectura con coordenadas invalidas, se retorna error de negocio`() = runTest {
    // GIVEN: Una lectura corrupta con latitud fuera del rango estándar planetario [-90.0, 90.0]
    val corruptTelemetry = TelemetryData(
        id = "corrupt-id",
        latitude = 150.0,  // Latitud imposible
        longitude = -45.0,
        speedKmh = 0.0,
        timestamp = Date(),
        isMocked = true
    )
    val realSaveUseCase = SaveTelemetryUseCase(repository = mock()) // Usamos la regla de negocio real del caso de uso
    
    // WHEN
    val result = realSaveUseCase(corruptTelemetry)
    
    // THEN
    assertTrue(result.isFailure)
    assertEquals("Coordenadas geográficas inválidas", result.exceptionOrNull()?.message)
}
```
Ejecuta nuevamente `./gradlew testDebugUnitTest` para confirmar que las tres pruebas del sistema pasen exitosamente.

---

## ## Solución de Problemas

A continuación, se listan dos problemas típicos y realistas que pueden surgir al integrar Clean Architecture e inyectar dependencias con Hilt en proyectos complejos, detallando su origen exacto y cómo resolverlos:

### Problema 1: Error de compilación con Hilt al compilar la app (`Hilt: Ink/Type elements cannot be parsed or resolved`)
*   **Sintomatología:** Al compilar el proyecto, Gradle aborta la tarea de compilación con mensajes crípticos referidos a la generación de archivos `@HiltAndroidApp` o `@AndroidEntryPoint`, indicando fallos en el análisis de tipos.
*   **Causa Raíz:** Esto suele deberse a la ausencia del registro correcto de la clase personalizada `TrackerApplication` en el manifiesto principal de la aplicación (`AndroidManifest.xml`), o bien a que la versión de Kotlin instalada no es del todo compatible con la versión del compilador de KSP asignada en `build.gradle.kts`.
*   **Resolución:** 
    1. Abre tu archivo `AndroidManifest.xml` y verifica que el elemento `<application>` contenga la propiedad `android:name=".TrackerApplication"`.
    2. Compara el mapeo de la versión exacta de tu plugin de Kotlin con el de KSP. En este laboratorio se utiliza Kotlin `2.1.10` combinado con la versión KSP `2.1.10-1.0.29`. Si cambias la versión de Kotlin de tu sistema, ajusta adecuadamente la firma de la dependencia de KSP según la matriz oficial en GitHub de Google KSP.

### Problema 2: El flujo de datos en Compose no responde a cambios de estado o "fuga" lecturas tras volver a segundo plano
*   **Sintomatología:** El panel visual `TelemetryScreen` deja de recibir de forma reactiva las coordenadas actualizadas simuladas por el repositorio, o bien continúa consumiendo ciclos de CPU y simulando lecturas asíncronas en segundo plano cuando el usuario sale de la pantalla de la app.
*   **Causa Raíz:** Se está utilizando un recolector de flujos de Compose tradicional como `collectAsState()` en lugar de las extensiones modernas del ciclo de vida, lo cual provoca que las corrutinas de flujo permanezcan atadas de forma ininterrumpida al ciclo de vida global de la aplicación.
*   **Resolución:** Reemplaza cualquier instancia de la llamada de suscripción tradicional en Compose por:
    ```kotlin
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    ```
    Asegúrate de importar correctamente la dependencia en tu clase de interfaz visual: `androidx.lifecycle.compose.collectAsStateWithLifecycle`. Esta API detiene la recolección del flujo reactivo de forma activa cuando la pantalla deja de ser visible.

---

## ## Limpieza

Para dejar el entorno de desarrollo optimizado y libre de cachés o archivos intermedios generados por procesos de compilación anteriores de Kotlin o Hilt, sigue estos pasos:

1. Desde el menú superior de Android Studio, haz clic en **Build > Clean Project**.
2. Abre la terminal embebida y ejecuta la limpieza manual de la caché del demonio de Gradle y directorios temporales de pruebas:
```bash
./gradlew clean
```
3. (Opcional) Si experimentas bloqueos intermitentes debido a las anotaciones de KSP, invalida las cachés del IDE dirigiéndote a **File > Invalidate Caches...**, marca la opción *Clear file system cache and Local History* y pulsa en **Invalidate and Restart**.

---

## ## Resumen

En este laboratorio, has rediseñado con éxito la arquitectura de una aplicación de geolocalización utilizando un enfoque profesional y mantenible:

- **Clean Architecture:** Desacoplaste el núcleo de negocio de los componentes del framework, dividiendo la solución en capas de **Dominio** (entidades y casos de uso inyectables), **Datos** (repositorio asíncrono con flujos de Kotlin) y **Presentación**.
- **MVVM y UDF:** Centralizaste las actualizaciones de estado mediante una clase de estado inmutable (`TelemetryUiState`) y una clase que encapsula eventos de disparo único (`TelemetryUiEvent`), consumidas de forma reactiva en Jetpack Compose mediante la API segura `collectAsStateWithLifecycle`.
- **Inyección de Dependencias:** Configuraste **Dagger Hilt** desde la raíz con `TrackerApplication`, agilizando la inicialización y eliminando el acoplamiento manual mediante fábricas de ViewModels complejas.
- **Optimización con IA:** Utilizaste **JetBrains AI Assistant** para crear una suite de pruebas unitarias asíncronas asertiva que valida el comportamiento del sistema ante fallos del GPS o coordenadas inválidas.

### Recursos de Aprendizaje Recomendados
- [Documentación Oficial de Android sobre Arquitectura de UI](https://developer.android.com/topic/architecture/ui-layer)
- [Guía de Aprendizaje de Dagger Hilt en Android](https://developer.android.com/training/dependency-injection/hilt-android)
- [Estrategia de Recolección Segura con StateFlow en Compose](https://medium.com/androiddevelopers/consuming-flows-safely-in-jetpack-compose-c1bfb0598152)
