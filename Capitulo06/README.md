# Integración de geolocalización con Google Play Services Location 21.4.0, permisos sensibles, mapas y sensores en Android API 30–37

## Metadatos

| Dimensión | Detalle |
| :--- | :--- |
| **Duración** | 144 minutos |
| **Complejidad** | Media |
| **Nivel de Bloom** | Aplicar (Apply) |

---

## Descripción General

En este laboratorio práctico, implementarás una arquitectura de telemetría y geolocalización en tiempo real para dispositivos móviles que ejecutan desde Android 11 (API 30) hasta Android 15/16 (API 35–37). Construirás un sistema reactivo y resiliente utilizando **Jetpack Compose**, **Google Play Services Location 21.4.0**, **Google Maps Compose 4.3.3** y el **SensorManager** del sistema operativo. 

El flujo de trabajo cubre la declaración técnica de permisos de ubicación en primer y segundo plano, la solicitud dinámica de los mismos mediante contratos nativos de Compose (`ActivityResultContracts`), la visualización de trayectorias en un mapa interactivo y la integración física del acelerómetro del hardware móvil para centrar automáticamente la visualización mediante un gesto físico de agitación (*shake gesture*).

---

## Objetivos de Aprendizaje

Al finalizar este laboratorio, serás capaz de:
- [ ] **Implementar** el flujo moderno de solicitud de permisos sensibles en tiempo de ejecución (`ACCESS_FINE_LOCATION`, `ACCESS_COARSE_LOCATION` y `ACCESS_BACKGROUND_LOCATION`) adaptado rigurosamente a las restricciones de API 30 a 37.
- [ ] **Configurar e integrar** `FusedLocationProviderClient` para capturar actualizaciones de ubicación periódicas de alta precisión balanceando el consumo energético del hardware.
- [ ] **Renderizar** mapas interactivos con trayectorias dinámicas (`Polyline`) en Jetpack Compose consumiendo flujos de datos asíncronos fríos (`Flow`) y calientes (`StateFlow`).
- [ ] **Conectar** el hardware del acelerómetro mediante `SensorEventListener` encapsulando la telemetría en corrutinas para procesar interacciones físicas directas sobre la vista del mapa.

---

## Prerrequisitos

Para completar este laboratorio con éxito, requieres:
1. **Conocimientos Técnicos Previos**:
   - Arquitectura limpia en Android (Clean Architecture / patrón MVVM).
   - Manejo de asincronía reactiva con Kotlin Coroutines, `StateFlow` y `SharedFlow`.
   - Gestión básica del ciclo de vida en Jetpack Compose (`remember`, `LaunchedEffect`, `DisposableEffect`).
2. **Cuentas y Accesos**:
   - Una cuenta activa en **Google Cloud Console** para la generación de credenciales y habilitación de la SDK de *Maps SDK for Android*.
   - Conexión a Internet sin restricciones de proxy para la descarga de dependencias Maven de Google.

---

## Entorno de Laboratorio

### Requisitos de Hardware Mínimos y Recomendados

| Componente | Requisito Mínimo | Requisito Recomendado |
| :--- | :--- | :--- |
| **Procesador** | Intel Core i7 o AMD Ryzen 7 (11va Gen) | Apple Silicon (M1/M2/M3) o Ryzen 9 / Core i9 |
| **Memoria RAM** | 16 GB DDR4 | 32 GB DDR5 (para IDE + Emulador simultáneos) |
| **Almacenamiento**| SSD con 40 GB de espacio libre | NVMe M.2 con 80 GB de espacio libre dedicado |
| **Dispositivo Físico**| N/A (Emulador con Google Play Services) | Dispositivo Android Físico con API 30+ y giroscopio/acelerómetro |

### Matriz de Software y Herramientas Requeridas

| Software / Dependencia | Versión Exacta | Arquitectura / Canal | Licencia | URL de Descarga Oficial |
| :--- | :--- | :--- | :--- | :--- |
| **Android Studio** | Ladybug (2024.2.1 Patch 3) | x86_64 / ARM64 | Freeware (JetBrains / Google) | [Android Studio Downloads](https://developer.android.com/studio) |
| **Eclipse Temurin JDK** | 17.0.10+7 | x64 / AArch64 | GPLv2 with Classpath Exception | [Adoptium Temurin](https://adoptium.net/temurin/releases/) |
| **Kotlin Compiler** | 2.3.10 | Native | Apache License 2.0 | [Kotlin Releases](https://github.com/JetBrains/kotlin/releases) |
| **Google Play Services Location** | 21.4.0 | Android Library | Propietaria de Google | [Google Maven Repository](https://maven.google.com/web/index.html) |
| **Google Maps Compose** | 4.3.3 | Compose Extension | Apache License 2.0 | [Google Maps Compose GitHub](https://github.com/googlemaps/android-maps-compose) |
| **JetBrains AI Assistant** | 242.23339 | IDE Plugin | Comercial / Suscripción | [JetBrains AI](https://www.jetbrains.com/ai/) |

### Configuración Global del Entorno Gradle

Asegúrate de que tu proyecto herede las siguientes variables globales en su archivo de configuración:

* **Paquete de Aplicación Base**: `com.example.advancedtracker`
* **Compilación del SDK**: `compileSdk = 35`
* **SDK Mínimo**: `minSdk = 30`
* **SDK Objetivo**: `targetSdk = 35`
* **Destino de Máquina Virtual**: `JVM Target = 17`

---

## Instrucciones Paso a Paso

### Paso 1: Configurar Dependencias, Manifiesto y API Key de Google Maps

**Objetivo**: Establecer los cimientos del proyecto configurando de manera correcta el archivo de dependencias de Gradle, declarando los permisos granulares y configurando de forma segura la API Key de Google Maps en el Manifiesto de Android.

#### Instrucciones

1. Abre tu archivo `build.gradle.kts` (Módulo: `app`) e inserta dentro del bloque `dependencies` las librerías necesarias. Garantiza que no existan colisiones de versiones utilizando las constantes requeridas:

```kotlin
// build.gradle.kts (Módulo :app)
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.compose.compiler)
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
        vectorDrawables {
            useSupportLibrary = true
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    kotlinOptions {
        jvmTarget = "17"
    }
    buildFeatures {
        compose = true
    }
}

dependencies {
    // Core Android y Compose BOM
    implementation(platform("androidx.compose:compose-bom:2026.02.01"))
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.ui:ui-graphics")
    implementation("androidx.compose.ui:ui-tooling-preview")
    implementation("androidx.compose.material3:material3")
    implementation("androidx.activity:activity-compose:1.10.0")
    
    // Google Play Services - Ubicación
    implementation("com.google.android.gms:play-services-location:21.4.0")
    
    // Google Maps para Jetpack Compose
    implementation("com.google.maps.android:maps-compose:4.3.3")
    
    // Coroutines & Lifecycle
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.10.1")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.7")
    implementation("androidx.lifecycle:lifecycle-runtime-compose:2.8.7")
    
    // Testing
    testImplementation("junit:junit:4.13.2")
    androidTestImplementation("androidx.test.ext:junit:1.2.1")
    androidTestImplementation("androidx.test.espresso:espresso-core:3.6.1")
    androidTestImplementation(platform("androidx.compose:compose-bom:2026.02.01"))
    androidTestImplementation("androidx.compose.ui:ui-test-junit4")
}
```

2. Configura tu API Key en la Consola de Google Cloud. Dirígete a [Google Cloud Console](https://console.cloud.google.com/), habilita **Maps SDK for Android** y genera una credencial de tipo *API Key*.

3. Modifica tu archivo `AndroidManifest.xml` (ubicado en `app/src/main/AndroidManifest.xml`) agregando la declaración obligatoria de los permisos de geolocalización avanzada, segundo plano y telemetría de red, junto a los metadatos de la API Key:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.advancedtracker">

    <!-- Permisos de Ubicación requeridos -->
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    
    <!-- Permiso requerido para rastreo en background si el usuario lo decide (Android 10+) -->
    <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
    
    <!-- Permisos de red e Internet para cargar el mapa -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="Advanced Telemetry Tracker"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Material3.DayNight.NoActionBar">
        
        <!-- Credenciales de Google Maps API -->
        <meta-data
            android:name="com.google.android.geo.API_KEY"
            android:value="REPLACE_WITH_YOUR_API_KEY" />

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

#### Resultado Esperado
El proyecto debe sincronizar con Gradle sin emitir errores de dependencias en conflicto. El compilador de Kotlin resolverá todas las dependencias asociadas a la versión Compose BOM `2026.02.01` utilizando la versión del compilador Kotlin `2.3.10`.

#### Verificación
Ejecuta la tarea de verificación de Gradle desde la terminal embebida de Android Studio para constatar que el árbol de dependencias es consistente:
```bash
./gradlew app:dependencies --configuration releaseRuntimeClasspath | grep -E "play-services-location|maps-compose"
```
Debes visualizar de salida las versiones exactas (`21.4.0` y `4.3.3`) acopladas correctamente al árbol.

---

### Paso 2: Diseñar la Capa de Datos y el Sensor de Movimiento (Acelerómetro)

**Objetivo**: Crear un flujo reactivo infinito (`callbackFlow`) que registre y filtre los eventos físicos del hardware acelerómetro para detectar un movimiento físico de agitación (*shake gesture*) del dispositivo móvil, aislando la lógica de sensores dentro del ciclo de vida del framework de Android.

#### Instrucciones

1. Crea el paquete `com.example.advancedtracker.data.sensor`.
2. Implementa la clase `ShakeDetector` que encapsulará el comportamiento físico utilizando el acelerómetro nativo mediante `SensorManager`. Usaremos Kotlin Coroutines con un flujo `Flow<Unit>` frío para procesar los eventos en tiempo real:

```kotlin
// File: app/src/main/java/com/example/advancedtracker/data/sensor/ShakeDetector.kt
package com.example.advancedtracker.data.sensor

import android.content.Context
import android.hardware.Sensor
import android.hardware.SensorEvent
import android.hardware.SensorEventListener
import android.hardware.SensorManager
import kotlinx.coroutines.channels.awaitClose
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.callbackFlow
import kotlin.math.sqrt

class ShakeDetector(private val context: Context) {

    private val sensorManager = context.getSystemService(Context.SENSOR_SERVICE) as SensorManager
    private val accelerometer: Sensor? = sensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER)

    /**
     * Retorna un Flow frío que emite una señal de tipo Unit cada vez que detecta un gesto
     * físico de agitación por encima del umbral de fuerza g definido (SHAKE_THRESHOLD_G).
     */
    fun detectShakeFlow(shakeThresholdG: Float = 2.7f, intervalMs: Long = 500L): Flow<Unit> = callbackFlow {
        if (accelerometer == null) {
            close(IllegalStateException("El sensor acelerómetro no se encuentra disponible en este hardware."))
            return@callbackFlow
        }

        var lastShakeTime = 0Long

        val listener = object : SensorEventListener {
            override fun onSensorChanged(event: SensorEvent?) {
                if (event == null) return
                if (event.sensor.type == Sensor.TYPE_ACCELEROMETER) {
                    val x = event.values[0]
                    val y = event.values[1]
                    val z = event.values[2]

                    // Cálculo del vector de gravedad
                    val gX = x / SensorManager.GRAVITY_EARTH
                    val gY = y / SensorManager.GRAVITY_EARTH
                    val gZ = z / SensorManager.GRAVITY_EARTH

                    // Fuerza G total resultante
                    val gForce = sqrt(gX * gX + gY * gY + gZ * gZ)

                    if (gForce > shakeThresholdG) {
                        val currentTime = System.currentTimeMillis()
                        if (currentTime - lastShakeTime > intervalMs) {
                            lastShakeTime = currentTime
                            trySend(Unit) // Emite de manera segura al flujo reactivo
                        }
                    }
                }
            }

            override fun onAccuracyChanged(sensor: Sensor?, accuracy: Int) {
                // No se requiere para este caso de uso
            }
        }

        // Registrar el Listener en el despachador de eventos del SensorManager
        sensorManager.registerListener(
            listener,
            accelerometer,
            SensorManager.SENSOR_DELAY_UI
        )

        // Limpieza automática cuando el colector de corrutinas cancela el flujo
        awaitClose {
            sensorManager.unregisterListener(listener)
        }
    }
}
```

#### Resultado Esperado
La clase `ShakeDetector` aislará la API imperativa `SensorEventListener` de Android dentro de un flujo funcional asíncrono y auto-gestionable. Si el colector cancela la corrutina asociada, el listener se desregistrará de inmediato del hardware para prevenir memory leaks y drenaje de batería.

#### Verificación
Crea un test unitario básico que instancie la lógica utilizando dobles de prueba (*mocks*) o verifica que el flujo compile adecuadamente sin llamadas de métodos deprecated del sistema operativo.

---

### Paso 3: Construir el Gestor de Ubicación y Flujo de Permisos Reactivo

**Objetivo**: Implementar un servicio robusto de localización usando `FusedLocationProviderClient` de Google Play Services Location `21.4.0` que proporcione actualizaciones en tiempo real a través de un `Flow` de coordenadas.

#### Instrucciones

1. Crea el paquete `com.example.advancedtracker.data.location`.
2. Define la clase `LocationTracker` encargada de gestionar los requests del GPS integrando los requisitos técnicos del SDK 30–37. Configura las peticiones con precisión milimétrica usando `Priority.PRIORITY_HIGH_ACCURACY`.

```kotlin
// File: app/src/main/java/com/example/advancedtracker/data/location/LocationTracker.kt
package com.example.advancedtracker.data.location

import android.annotation.SuppressLint
import android.content.Context
import android.location.Location
import android.os.Looper
import com.google.android.gms.location.FusedLocationProviderClient
import com.google.android.gms.location.LocationCallback
import com.google.android.gms.location.LocationRequest
import com.google.android.gms.location.LocationResult
import com.google.android.gms.location.LocationServices
import com.google.android.gms.location.Priority
import kotlinx.coroutines.channels.awaitClose
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.callbackFlow

class LocationTracker(private val context: Context) {

    private val fusedLocationClient: FusedLocationProviderClient =
        LocationServices.getFusedLocationProviderClient(context)

    /**
     * Flujo asíncrono frío que emite coordenadas de ubicación geográfica actualizadas.
     */
    @SuppressLint("MissingPermission")
    fun fetchLocationUpdates(intervalMs: Long = 4000L, fastestIntervalMs: Long = 2000L): Flow<Location> = callbackFlow {
        // Validación de seguridad de compilación y runtime para Android Studio
        if (!context.hasLocationPermissions()) {
            close(SecurityException("Permisos de ubicación insuficientes."))
            return@callbackFlow
        }

        // Construcción moderna del LocationRequest siguiendo lineamientos de la API 30+
        val locationRequest = LocationRequest.Builder(Priority.PRIORITY_HIGH_ACCURACY, intervalMs)
            .setMinUpdateIntervalMillis(fastestIntervalMs)
            .build()

        val callback = object : LocationCallback() {
            override fun onLocationResult(result: LocationResult) {
                for (location in result.locations) {
                    trySend(location) // Envia la coordenada al colector activo
                }
            }
        }

        fusedLocationClient.requestLocationUpdates(
            locationRequest,
            callback,
            Looper.getMainLooper()
        )

        // Limpieza de subscripción de localización para optimizar consumo energético
        awaitClose {
            fusedLocationClient.removeLocationUpdates(callback)
        }
    }
}

// Extensión para verificar permisos
fun Context.hasLocationPermissions(): Boolean {
    val fine = androidx.core.content.ContextCompat.checkSelfPermission(
        this, android.Manifest.permission.ACCESS_FINE_LOCATION
    ) == android.content.pm.PackageManager.PERMISSION_GRANTED
    val coarse = androidx.core.content.ContextCompat.checkSelfPermission(
        this, android.Manifest.permission.ACCESS_COARSE_LOCATION
    ) == android.content.pm.PackageManager.PERMISSION_GRANTED
    return fine || coarse
}
```

#### Resultado Esperado
Un wrapper asíncrono y reactivo alrededor del cliente oficial de Google Play Services que se actualice cada 4 segundos, emitiendo un flujo estructurado de datos que puede ser cancelado de forma transparente por el ciclo de vida de la UI de Jetpack Compose.

#### Verificación
Asegura que el compilador no arroje advertencias (*warnings*) relacionadas con APIs deprecadas como `LocationRequest()` por constructor tradicional. El uso de `LocationRequest.Builder` es obligatorio para API 31+.

---

### Paso 4: Implementar el ViewModel para la Telemetría y Geolocalización

**Objetivo**: Diseñar un `ViewModel` robusto que orqueste de manera concurrente el estado del mapa, procese el historial de trayectorias, consuma los flujos del acelerómetro y el GPS, y exponga canales seguros de comunicación unidireccional (UDF - *Unidirectional Data Flow*).

#### Instrucciones

1. Crea el paquete `com.example.advancedtracker.ui.viewmodel`.
2. Define la estructura de estado para la UI:

```kotlin
// File: app/src/main/java/com/example/advancedtracker/ui/viewmodel/TelemetryUiState.kt
package com.example.advancedtracker.ui.viewmodel

import android.location.Location
import com.google.android.gms.maps.model.LatLng

data class TelemetryUiState(
    val currentPosition: LatLng? = null,
    val routeHistory: List<LatLng> = emptyList(),
    val totalDistanceMeters: Float = 0f,
    val speedKmh: Float = 0f,
    val isTracking: Boolean = false,
    val lastKnownAccuracy: Float = 0f
)
```

3. Implementa la clase `TelemetryViewModel`. Integrará de forma coordinada el `LocationTracker` y el `ShakeDetector`:

```kotlin
// File: app/src/main/java/com/example/advancedtracker/ui/viewmodel/TelemetryViewModel.kt
package com.example.advancedtracker.ui.viewmodel

import android.app.Application
import android.location.Location
import androidx.lifecycle.AndroidViewModel
import androidx.lifecycle.viewModelScope
import com.example.advancedtracker.data.location.LocationTracker
import com.example.advancedtracker.data.sensor.ShakeDetector
import com.google.android.gms.maps.model.LatLng
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

class TelemetryViewModel(application: Application) : AndroidViewModel(application) {

    private val locationTracker = LocationTracker(application)
    private val shakeDetector = ShakeDetector(application)

    private val _uiState = MutableStateFlow(TelemetryUiState())
    val uiState: StateFlow<TelemetryUiState> = _uiState.asStateFlow()

    // Evento de disparo unidireccional caliente para notificar a la UI que debe re-centrar la cámara
    private val _eventCameraCenter = MutableSharedFlow<LatLng>(replay = 0)
    val eventCameraCenter: SharedFlow<LatLng> = _eventCameraCenter.asSharedFlow()

    private var trackingJob: Job? = null
    private var shakeJob: Job? = null

    init {
        startSensorMonitoring()
    }

    /**
     * Inicia de forma activa la recopilación de datos desde el GPS.
     */
    fun startTracking() {
        if (_uiState.value.isTracking) return

        _uiState.update { it.copy(isTracking = true) }
        trackingJob = viewModelScope.launch {
            locationTracker.fetchLocationUpdates()
                .catch { exception ->
                    _uiState.update { it.copy(isTracking = false) }
                }
                .collect { location ->
                    processNewLocation(location)
                }
        }
    }

    /**
     * Cancela el seguimiento asíncrono deteniendo el flujo del GPS.
     */
    fun stopTracking() {
        trackingJob?.cancel()
        trackingJob = null
        _uiState.update { it.copy(isTracking = false) }
    }

    private fun startSensorMonitoring() {
        shakeJob = viewModelScope.launch {
            shakeDetector.detectShakeFlow()
                .catch { /* No-op: Evitar crash si el emulador no simula el sensor */ }
                .collect {
                    _uiState.value.currentPosition?.let { currentLatLng ->
                        // Emitir el evento de recentrado a la UI de forma asíncrona
                        _eventCameraCenter.emit(currentLatLng)
                    }
                }
        }
    }

    private fun processNewLocation(location: Location) {
        val newPoint = LatLng(location.latitude, location.longitude)
        _uiState.update { currentState ->
            val updatedHistory = currentState.routeHistory + newPoint
            
            // Cálculo incremental de la distancia
            var incrementalDistance = currentState.totalDistanceMeters
            if (currentState.currentPosition != null) {
                val results = FloatArray(1)
                Location.distanceBetween(
                    currentState.currentPosition.latitude,
                    currentState.currentPosition.longitude,
                    location.latitude,
                    location.longitude,
                    results
                )
                incrementalDistance += results[0]
            }

            currentState.copy(
                currentPosition = newPoint,
                routeHistory = updatedHistory,
                totalDistanceMeters = incrementalDistance,
                speedKmh = (location.speed * 3.6f), // Conversión de m/s a Km/h
                lastKnownAccuracy = location.accuracy
            )
        }
    }

    override fun onCleared() {
        super.onCleared()
        stopTracking()
        shakeJob?.cancel()
    }
}
```

#### Resultado Esperado
El `TelemetryViewModel` centralizará la lógica de negocio y encapsulará todo el ciclo de vida, permitiendo a la capa composable suscribirse únicamente al estado `uiState` y al canal de eventos de cámara `eventCameraCenter` de forma independiente del ciclo de vida físico del móvil.

---

### Paso 3: Diseñar el Flujo Moderno de Solicitud de Permisos en Compose

> *Nota de Arquitectura*: En concordancia con las lecciones previas y la evolución del sistema a partir de la API 30+, implementaremos un flujo que distinga la denegación simple de la denegación permanente, mostrando un cuadro de diálogo educativo (*Rationale*) altamente pedagógico y permitiendo la redirección directa a la configuración del sistema operativo.

#### Instrucciones

1. Crea el paquete `com.example.advancedtracker.ui.components`.
2. Implementa el componente de diálogo `PermissionRationaleDialog`:

```kotlin
// File: app/src/main/java/com/example/advancedtracker/ui/components/PermissionRationaleDialog.kt
package com.example.advancedtracker.ui.components

import androidx.compose.material3.AlertDialog
import androidx.compose.material3.Button
import androidx.compose.material3.Text
import androidx.compose.material3.TextButton
import androidx.compose.runtime.Composable

@Composable
fun PermissionRationaleDialog(
    onDismiss: () -> Unit,
    onConfirm: () -> Unit
) {
    AlertDialog(
        onDismissRequest = onDismiss,
        title = { Text(text = "Acceso a la Ubicación Requerido") },
        text = {
            Text(text = "Para poder registrar la trayectoria en tiempo real y mostrar tu avance en el mapa interactivo, la aplicación necesita acceso a tu ubicación precisa (GPS). Esta información no es compartida con terceros y se procesa localmente.")
        },
        confirmButton = {
            Button(onClick = onConfirm) {
                Text(text = "Conceder Permiso")
            }
        },
        dismissButton = {
            TextButton(onClick = onDismiss) {
                Text(text = "Más Tarde")
            }
        }
    )
}
```

3. Diseña la pantalla contenedora de permisos `PermissionGate` que interceptará la vista del mapa si los privilegios requeridos por el SDK no han sido otorgados:

```kotlin
// File: app/src/main/java/com/example/advancedtracker/ui/components/PermissionGate.kt
package com.example.advancedtracker.ui.components

import android.Manifest
import android.content.Context
import android.content.Intent
import android.net.Uri
import android.provider.Settings
import androidx.activity.compose.rememberLauncherForActivityResult
import androidx.activity.result.contract.ActivityResultContracts
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.unit.dp
import com.example.advancedtracker.data.location.hasLocationPermissions

@Composable
fun PermissionGate(
    modifier: Modifier = Modifier,
    onPermissionsGranted: () -> Unit,
    content: @Composable () -> Unit
) {
    val context = LocalContext.current
    var hasPermissions by remember { mutableStateOf(context.hasLocationPermissions()) }
    var showRationale by remember { mutableStateOf(false) }
    var hasPermanentlyDenied by remember { mutableStateOf(false) }

    val permissionLauncher = rememberLauncherForActivityResult(
        contract = ActivityResultContracts.RequestMultiplePermissions()
    ) { permissionsMap ->
        val fineGranted = permissionsMap[Manifest.permission.ACCESS_FINE_LOCATION] ?: false
        val coarseGranted = permissionsMap[Manifest.permission.ACCESS_COARSE_LOCATION] ?: false
        
        if (fineGranted || coarseGranted) {
            hasPermissions = true
            onPermissionsGranted()
        } else {
            hasPermissions = false
            hasPermanentlyDenied = true
        }
    }

    LaunchedEffect(Unit) {
        if (hasPermissions) {
            onPermissionsGranted()
        }
    }

    if (hasPermissions) {
        content()
    } else {
        Column(
            modifier = modifier
                .fillMaxSize()
                .padding(32.dp),
            verticalArrangement = Arrangement.Center,
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            Text(
                text = "Permisos Requeridos",
                style = MaterialTheme.typography.headlineMedium,
                color = MaterialTheme.colorScheme.error,
                modifier = Modifier.padding(bottom = 16.dp)
            )
            
            Text(
                text = "Para continuar con la telemetría geográfica avanzada, se necesita la autorización de geolocalización.",
                textAlign = TextAlign.Center,
                modifier = Modifier.padding(bottom = 32.dp)
            )

            Button(onClick = {
                val activity = context as? android.app.Activity
                val shouldShow = activity?.let {
                    androidx.core.app.ActivityCompat.shouldShowRequestPermissionRationale(
                        it, Manifest.permission.ACCESS_FINE_LOCATION
                    )
                } ?: false

                if (shouldShow) {
                    showRationale = true
                } else {
                    permissionLauncher.launch(
                        arrayOf(
                            Manifest.permission.ACCESS_FINE_LOCATION,
                            Manifest.permission.ACCESS_COARSE_LOCATION
                        )
                    )
                }
            }) {
                Text("Autorizar Ubicación")
            }

            if (hasPermanentlyDenied) {
                Spacer(modifier = Modifier.height(16.dp))
                TextButton(onClick = {
                    val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS).apply {
                        data = Uri.fromParts("package", context.packageName, null)
                        flags = Intent.FLAG_ACTIVITY_NEW_TASK
                    }
                    context.startActivity(intent)
                }) {
                    Text("Configuración del Sistema")
                }
            }
        }
    }

    if (showRationale) {
        PermissionRationaleDialog(
            onDismiss = { showRationale = false },
            onConfirm = {
                showRationale = false
                permissionLauncher.launch(
                    arrayOf(
                        Manifest.permission.ACCESS_FINE_LOCATION,
                        Manifest.permission.ACCESS_COARSE_LOCATION
                    )
                )
            }
        )
    }
}
```

#### Resultado Esperado
Un flujo reactivo e inquebrantable de control de seguridad. Si el usuario decide rechazar el permiso de manera recurrente, se le presentará el botón explícito para forzar su redirección al menú del sistema, satisfaciendo los criterios de diseño exigidos por Google Play Store en el rastreo de geolocalización sensible.

---

### Paso 5: Construir la UI del Mapa Interactivo y Controladores con Jetpack Compose

**Objetivo**: Integrar la librería Google Maps Compose `4.3.3` dentro de la interfaz declarativa. Vincularás el controlador de la cámara para que reciba las señales asíncronas del `SharedFlow` emitidas por la agitación física del acelerómetro y grafique de forma progresiva la trayectoria del usuario.

#### Instrucciones

1. Crea el componente visual principal de mapa `TelemetryMapScreen`:

```kotlin
// File: app/src/main/java/com/example/advancedtracker/ui/components/TelemetryMapScreen.kt
package com.example.advancedtracker.ui.components

import androidx.compose.animation.AnimatedVisibility
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.dp
import androidx.lifecycle.compose.collectAsStateWithLifecycle
import com.example.advancedtracker.ui.viewmodel.TelemetryViewModel
import com.google.android.gms.maps.CameraUpdateFactory
import com.google.android.gms.maps.model.CameraPosition
import com.google.android.gms.maps.model.LatLng
import com.google.maps.android.compose.*
import kotlinx.coroutines.flow.collectLatest

@Composable
fun TelemetryMapScreen(
    viewModel: TelemetryViewModel,
    modifier: Modifier = Modifier
) {
    val state by viewModel.uiState.collectAsStateWithLifecycle()
    
    // Configuración inicial por defecto (ejemplo: Latitud/Longitud central de referencia)
    val defaultPos = LatLng(0.0, 0.0)
    val cameraPositionState = rememberCameraPositionState {
        position = CameraPosition.fromLatLngZoom(defaultPos, 15f)
    }

    // Escuchar activamente el SharedFlow caliente de recentrado del mapa mediante acelerómetro
    LaunchedEffect(key1 = viewModel) {
        viewModel.eventCameraCenter.collectLatest { targetLatLng ->
            cameraPositionState.animate(
                update = CameraUpdateFactory.newLatLngZoom(targetLatLng, 17f),
                durationMs = 1000
            )
        }
    }

    Box(modifier = modifier.fillMaxSize()) {
        GoogleMap(
            modifier = Modifier.fillMaxSize(),
            cameraPositionState = cameraPositionState,
            uiSettings = MapUiSettings(zoomControlsEnabled = false, myLocationButtonEnabled = true),
            properties = MapProperties(
                isMyLocationEnabled = state.currentPosition != null,
                maxZoomPreference = 20f,
                minZoomPreference = 3f
            )
        ) {
            // Dibujar la trayectoria histórica recolectada por el GPS de alta precisión
            if (state.routeHistory.size > 1) {
                Polyline(
                    points = state.routeHistory,
                    clickable = false,
                    color = Color(0xFF1E88E5),
                    width = 12f
                )
            }

            // Marcador visual de posición actual con precisión
            state.currentPosition?.let { current ->
                Marker(
                    state = MarkerState(position = current),
                    title = "Posición Actual",
                    snippet = "Precisión: ${"%.1f".format(state.lastKnownAccuracy)}m"
                )
            }
        }

        // Overlay de Telemetría Flotante
        Column(
            modifier = Modifier
                .align(Alignment.TopCenter)
                .padding(16.dp)
                .clip(RoundedCornerShape(12.dp))
                .background(MaterialTheme.colorScheme.surfaceVariant.copy(alpha = 0.9f))
                .padding(16.dp)
                .fillMaxWidth(),
            verticalArrangement = Arrangement.spacedBy(4.dp)
        ) {
            Text(
                text = "Métricas de Telemetría",
                style = MaterialTheme.typography.titleMedium,
                color = MaterialTheme.colorScheme.onSurfaceVariant
            )
            Divider(color = MaterialTheme.colorScheme.onSurfaceVariant.copy(alpha = 0.2f))
            
            Row(
                modifier = Modifier.fillMaxWidth(),
                horizontalArrangement = Arrangement.SpaceBetween
            ) {
                Text(text = "Distancia: ${"%.2f".format(state.totalDistanceMeters / 1000f)} Km")
                Text(text = "Velocidad: ${"%.1f".format(state.speedKmh)} Km/h")
            }
            Text(text = "Último Punto: ${state.currentPosition?.let { "${"%.5f".format(it.latitude)}, ${"%.5f".format(it.longitude)}" } ?: "Buscando GPS..."}")
        }

        // Panel de control inferior (Play / Stop Tracking)
        Box(
            modifier = Modifier
                .align(Alignment.BottomCenter)
                .padding(bottom = 24.dp)
        ) {
            Button(
                onClick = {
                    if (state.isTracking) {
                        viewModel.stopTracking()
                    } else {
                        viewModel.startTracking()
                    }
                },
                colors = ButtonDefaults.buttonColors(
                    containerColor = if (state.isTracking) MaterialTheme.colorScheme.error else MaterialTheme.colorScheme.primary
                ),
                elevation = ButtonDefaults.buttonElevation(defaultElevation = 8.dp)
            ) {
                Text(text = if (state.isTracking) "Detener Rastreo" else "Iniciar Rastreo")
            }
        }
    }
}
```

2. Integra la lógica global de la aplicación en el punto de entrada de la actividad `MainActivity`:

```kotlin
// File: app/src/main/java/com/example/advancedtracker/MainActivity.kt
package com.example.advancedtracker

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.viewModels
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.ui.Modifier
import com.example.advancedtracker.ui.components.PermissionGate
import com.example.advancedtracker.ui.components.TelemetryMapScreen
import com.example.advancedtracker.ui.viewmodel.TelemetryViewModel

class MainActivity : ComponentActivity() {

    private val telemetryViewModel: TelemetryViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MaterialTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    PermissionGate(
                        onPermissionsGranted = {
                            // Los permisos están concedidos. El ViewModel comenzará a responder.
                        }
                    ) {
                        TelemetryMapScreen(
                            viewModel = telemetryViewModel,
                            modifier = Modifier.fillMaxSize()
                        )
                    }
                }
            }
        }
    }
}
```

---

## Validación y Pruebas

Para garantizar que el motor de telemetría y geolocalización responde de acuerdo a las especificaciones técnicas requeridas, realizaremos una simulación de pruebas que incluye una prueba de inyección de datos adversarios.

### Procedimiento de Prueba Interactiva en Emulador

1. Inicia un emulador Android con soporte para Google Play Services de API 33 o superior.
2. Abre la consola de desarrollador del emulador haciendo clic en los tres puntos (`...`) (*Extended Controls*).
3. Dirígete a la pestaña **Location** e importa un archivo de trayecto GPX, o define manualmente varios puntos agregándolos a la cola de simulación.
4. Presiona el botón de **Play** de la simulación del emulador en una velocidad constante de `2x` o `5x`.
5. En la aplicación ejecutándose en el emulador, pulsa el botón **Iniciar Rastreo**.
6. **Verificación visual del mapa**: Observa que el marcador de posición dibuje progresivamente una polilínea azul de alta precisión sobre las calles recorridas y que el HUD superior actualice la velocidad de forma reactiva, junto al kilometraje total.

### Escenario de Prueba Adversaria: Inyección de Coordenadas Erráticas o Telemetría Inestable

Para probar la resiliencia técnica de la capa de datos de la aplicación ante escenarios donde los servicios del GPS entregan coordenadas imposibles o fraudulentas (por ejemplo, coordenadas con altos índices de error de exactitud), introduciremos una regla de validación de telemetría en el colector del `ViewModel`.

#### Procedimiento de Validación
Modifica temporalmente la función `processNewLocation` del `TelemetryViewModel` para filtrar puntos de baja calidad. Inserta una restricción que rechace coordenadas con un margen de error mayor a 40 metros (este filtrado evita saltos bruscos provocados por rebotes de señal en rascacielos o pérdida transitoria de cobertura GPS):

```kotlin
// Modificación de prueba en TelemetryViewModel.kt para validación adversaria
private fun processNewLocation(location: Location) {
    // Escenario de Defensa: Ignorar si la precisión reportada por el sensor GPS supera los 40 metros
    if (location.accuracy > 40.0f) {
         // Registro de advertencia interna en consola. La telemetría descarta el punto fraudulento
         android.util.Log.w("TelemetryTracker", "Coordenada ignorada debido a baja precisión: ${location.accuracy}m")
         return
    }
    
    // Proceso normal de telemetría
    val newPoint = LatLng(location.latitude, location.longitude)
    // ... lógica remanente
}
```

Simula una señal defectuosa en la herramienta *Extended Controls* del emulador incrementando manualmente el parámetro `Accuracy` (precisión) en un valor de `100` o superior. 

**Resultado de Validación**: La terminal de *Logcat* de Android Studio debe registrar de forma continua las advertencias filtradas, demostrando que la polilínea y los cálculos de distancia permanecen inalterables ante lecturas erróneas, garantizando la resiliencia de la app en producción.

---

## Solución de Problemas

A continuación, se describen los dos problemas más comunes que pueden ocurrir durante la ejecución y despliegue del proyecto, detallando sus causas de origen y las acciones técnicas necesarias para resolverlos.

### Problema 1: El mapa se muestra completamente en gris o con rejillas sin cargar imágenes de satélite

* **Síntoma**: El contenedor del mapa renderiza correctamente en pantalla con las opciones de control de zoom pero el fondo permanece de color gris claro o con marcas de agua de Google, sin descargar texturas geográficas.
* **Causa**: La credencial `API_KEY` colocada en el archivo `AndroidManifest.xml` es inválida, no tiene los permisos activos para consumir la SDK de *Maps SDK for Android* en la Google Cloud Console, o las firmas SHA-1 de depuración del proyecto no fueron añadidas en la configuración de restricciones de la credencial en la consola web de Google.
* **Resolución**:
  1. Accede a tu consola de [Google Cloud Console](https://console.cloud.google.com/).
  2. Verifica que el proyecto tenga la API denominada **Maps SDK for Android** habilitada en la pestaña de APIs & Services.
  3. Compara el string exacto de la credencial y asegúrate de reemplazar `"REPLACE_WITH_YOUR_API_KEY"` en el `AndroidManifest.xml` con la llave autogenerada.
  4. Si la llave tiene restricciones de IP o de aplicación, inhabilítalas temporalmente durante la sesión del laboratorio para comprobar la conectividad del emulador.

### Problema 2: El gesto de agitación (Shake Gesture) no recentra la cámara en el mapa o genera fugas de memoria

* **Síntoma**: Al agitar de forma rápida el dispositivo físico o presionar el botón de simulación de aceleración en el emulador, la cámara de Google Maps no realiza ninguna transición animada hacia la ubicación actual del usuario. Adicionalmente, el rendimiento de la aplicación se degrada después de abrir y cerrar repetidas veces la pantalla.
* **Causa**: El hardware del acelerómetro no está disponible o simulado, el listener del `SensorManager` no está liberando recursos debido a una recolección fallida del flujo frío de Kotlin Coroutines, o no se ha iniciado formalmente la recolección asíncrona mediante el constructor apropiado de Jetpack Compose (`LaunchedEffect`).
* **Resolución**:
  1. Si ejecutas en emulador, asegúrate de activar la simulación de movimiento físico abriendo *Extended Controls* -> *Virtual Sensors* y rotando el dispositivo en los ejes tridimensionales.
  2. Asegúrate de consumir el flujo `eventCameraCenter` de forma asíncrona y segura mediante `collectLatest` en un bloque `LaunchedEffect`. Esto asegura que los hilos de render de la UI reciban y actualicen de forma inmediata el estado visual del mapa.
  3. Revisa la clase `ShakeDetector` y valida que el bloque `awaitClose` esté presente. Esto remueve de manera explícita el listener mediante `sensorManager.unregisterListener(listener)`.

---

## Limpieza

Al finalizar el laboratorio, es indispensable limpiar el entorno local para asegurar un uso óptimo del almacenamiento del sistema de desarrollo:

1. Realiza una limpieza completa del caché y los artefactos binarios construidos por Gradle:
   ```bash
   ./gradlew clean
   ```
2. Cierra el Emulador Android y remueve las ubicaciones simuladas para liberar el procesador de tu equipo.
3. Si utilizaste un dispositivo físico, accede al menú *Ajustes* -> *Aplicaciones* -> *Advanced Telemetry Tracker* y procede a desinstalar la aplicación para desvincular los procesos en background de la batería del dispositivo.

---

## Resumen

En este laboratorio, has integrado y consolidado con éxito múltiples APIs avanzadas de Android en una sola aplicación de telemetría moderna y reactiva:

- **Estrategia Moderna de Permisos**: Diseñaste un flujo completo para la gestión de permisos en tiempo de ejecución compatible con APIs del nivel 30 al 37, incorporando una lógica que despliega un diálogo explicativo de negocio (*Rationale*) y una ruta alternativa hacia los ajustes del sistema ante una denegación permanente.
- **Geolocalización Asíncrona**: Consumiste coordenadas geográficas periódicas mediante `FusedLocationProviderClient` de Google Play Services Location `21.4.0`, exponiéndolas de forma reactiva y asíncrona mediante flujos de Kotlin Coroutines (`Flow` y `callbackFlow`).
- **Visualización en Mapa Interactivos**: Representaste trayectorias históricas y dinámicas en tiempo real mediante el renderizado de `Polyline` utilizando la biblioteca moderna Google Maps Compose `4.3.3`.
- **Integración con Sensores de Movimiento**: Conectaste de manera concurrente el acelerómetro nativo del hardware para implementar interacciones físicas fluidas (*Shake Gesture*), procesando la señal y coordinando el movimiento de la cámara a través del ViewModel mediante canales de comunicación asíncronos unidireccionales (`SharedFlow`).

### Recursos Adicionales para Profundizar

* [Documentación Oficial de Maps SDK for Android en Compose](https://github.com/googlemaps/android-maps-compose)
* [Guía Oficial de Geolocalización en Google Play Services](https://developers.google.com/android/guides/releases)
* [Buenas Prácticas para el Manejo de Permisos en Android (Google Developers)](https://developer.android.com/training/permissions/requesting)
* [Guía de Kotlin Coroutines y CallbackFlow asíncronos](https://kotlinlang.org/docs/flow.html#callbackflow)
