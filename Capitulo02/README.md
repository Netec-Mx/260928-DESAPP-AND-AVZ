# Construcción de interfaz declarativa con Jetpack Compose BOM 2026.02.01, Material 3, estado, recomposición, navegación y LazyColumn

## Metadatos

| Métrica | Detalle |
| :--- | :--- |
| **Duración** | 288 minutos |
| **Complejidad** | Alta (Hard) |
| **Nivel de Bloom** | Crear (Create) |

---

## Descripción General

En este laboratorio práctico, transformarás por completo la interfaz de usuario del sistema de rastreo de telemetría, sustituyendo el antiguo paradigma imperativo de vistas XML por una arquitectura de UI declarativa moderna. Utilizarás **Jetpack Compose BOM 2026.02.01** y **Material Design 3** para estructurar un flujo de navegación con tipado seguro (*Type-Safe Navigation*). 

Diseñarás e implementarás una pantalla de listado dinámico basada en `LazyColumn` que renderizará los datos de motores de simulación, aplicando técnicas de *State Hoisting* para separar la lógica de presentación de la renderización visual. Asimismo, construirás una pantalla de detalles interactiva que empleará `remember` y `rememberSaveable` para garantizar la persistencia del estado ante cambios rotacionales y de configuración física del dispositivo.

---

## Objetivos de Aprendizaje

Al finalizar este laboratorio, serás capaz de:
- [ ] Configurar y utilizar el sistema de gestión de dependencias de **Jetpack Compose BOM 2026.02.01** junto con el compilador integrado de Kotlin 2.3.10.
- [ ] Diseñar componentes visuales reactivos utilizando las guías de diseño de **Material Design 3** (`Scaffold`, `Card`, `TopAppBar` y tipografías).
- [ ] Implementar un flujo de navegación tipado utilizando la biblioteca de **Compose Navigation** sin exponer rutas basadas en cadenas de texto propensas a errores.
- [ ] Optimizar el rendimiento de listas complejas mediante `LazyColumn` empleando claves únicas (`key`) e índices de optimización de recomposición.
- [ ] Gestionar el estado mutacional de la interfaz de manera resiliente ante la recreación del ciclo de vida utilizando `rememberSaveable` y savers personalizados.

---

## Prerrequisitos

Para completar con éxito este laboratorio, requieres:
1. **Código Base Previo:** Tener completado el proyecto base estructurado de la Práctica 1 (donde se definieron las configuraciones iniciales del SDK de Android 35 y la estructura de paquetes de `com.example.advancedtracker`).
2. **Acceso a Herramientas:**
   * **JetBrains AI Assistant (v242.23339):** Requiere una suscripción activa comercial o educativa configurada dentro de Android Studio para asistencia de refactorización y generación de pruebas guiadas.
3. **Conexión de Red:** Acceso irrestricto a Internet para la descarga de artefactos desde el repositorio Maven de Google.

---

## Entorno de Laboratorio

### Requisitos de Hardware

| Componente | Especificación Mínima | Especificación Recomendada |
| :--- | :--- | :--- |
| **Procesador** | Intel Core i7 / AMD Ryzen 7 (11va Gen o Sup.) | Apple Silicon (M1/M2/M3) o Intel/AMD Core i9 |
| **Memoria RAM** | 16 GB DDR4/DDR5 | 32 GB DDR5 |
| **Disco Duro** | 40 GB libres en SSD | 80 GB libres en NVMe M.2 SSD |
| **Dispositivo** | Emulador API 30+ (x86_64) | Dispositivo físico Android con API 35 (USB Debugging) |

### Requisitos de Software y Librerías

| Herramienta / API | Versión de Referencia | Licencia / Rol | Enlace de Descarga / Fuente |
| :--- | :--- | :--- | :--- |
| **Android Studio** | Ladybug (2024.2.1 Patch 3) | Propietario (Gratuito) | [Android Studio Downloads](https://developer.android.com/studio) |
| **JDK** | Eclipse Temurin 17.0.10+7 | GPLv2+CE | [Adoptium Releases](https://adoptium.net/temurin/releases/?version=17) |
| **Kotlin Compiler** | 2.3.10 | Apache 2.0 | [Kotlin Releases](https://github.com/JetBrains/kotlin/releases) |
| **Compose BOM** | 2026.02.01 | Apache 2.0 | [Google Maven Repository](https://maven.google.com/) |
| **Compose Navigation** | 2.8.8 | Apache 2.0 | [Google Maven Repository](https://maven.google.com/) |
| **Kotlinx Serialization**| 1.7.3 | Apache 2.0 | [Kotlinx Serialization Repo](https://github.com/Kotlin/kotlinx.serialization) |

---

## Instrucciones Paso a Paso

### Paso 1: Configurar el entorno de construcción Gradle para Jetpack Compose

**Objetivo:** Configurar el archivo de construcción a nivel de aplicación para soportar el compilador de Jetpack Compose integrado en Kotlin 2.3.10, deshabilitar la generación de layouts XML antiguos y añadir el ecosistema Jetpack Compose BOM 2026.02.01 de manera centralizada.

**Instrucciones:**

1. Abre el archivo `build.gradle.kts` de tu módulo principal (usualmente `app/build.gradle.kts`).
2. Reemplaza la sección de `plugins` y añade los plugins correspondientes a Kotlin Compose y Kotlinx Serialization para habilitar la navegación tipada:

```kotlin
plugins {
    id("com.android.application") version "8.4.0"
    id("org.jetbrains.kotlin.android") version "2.3.10"
    id("org.jetbrains.kotlin.plugin.compose") version "2.3.10"
    id("org.jetbrains.kotlin.plugin.serialization") version "2.3.10"
}
```

3. Modifica la configuración dentro del bloque `android` para especificar el uso de Compose, el nivel del SDK (compileSdk/targetSdk = 35) y la versión de Java:

```kotlin
android {
    compileSdk = 35

    defaultConfig {
        applicationId = "com.example.advancedtracker"
        minSdk = 30
        targetSdk = 35
        versionCode = 1
        versionName = "1.0"

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
    }
    buildFeatures {
        compose = true
    }
}
```

4. Agrega las dependencias de Compose utilizando el BOM (Bill of Materials) `2026.02.01` y añade las dependencias de navegación y serialización:

```kotlin
dependencies {
    // Importar Compose BOM de forma unificada
    val composeBom = platform("androidx.compose:compose-bom:2026.02.01")
    implementation(composeBom)
    androidTestImplementation(composeBom)

    // Componentes Core de Compose
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.ui:ui-graphics")
    implementation("androidx.compose.ui:ui-tooling-preview")
    implementation("androidx.compose.material3:material3")
    
    // Soporte para Activity e integración con ViewModels
    implementation("androidx.activity:activity-compose:1.9.3")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.7")

    // Navegación con tipado seguro
    implementation("androidx.navigation:navigation-compose:2.8.8")
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.3")

    // Herramientas de Depuración
    debugImplementation("androidx.compose.ui:ui-tooling")
    debugImplementation("androidx.compose.ui:ui-test-manifest")

    // Pruebas Unitarias y de UI
    testImplementation("junit:junit:4.13.2")
    androidTestImplementation("androidx.compose.ui:ui-test-junit4")
}
```

5. Sincroniza el proyecto seleccionando **Sync Project with Gradle Files**.

**Resultado esperado de compilación:** El proyecto debe compilarse limpiamente sin advertencias de colisión entre la versión del compilador de Compose y Kotlin, debido a que el compilador se autogestiona directamente por el plugin oficial `org.jetbrains.kotlin.plugin.compose` en la versión de Kotlin 2.3.10.

**Verificación:** Ejecuta `./gradlew assembleDebug` en la terminal embebida de Android Studio y asegúrate de recibir un estado `BUILD SUCCESSFUL`.

---

### Paso 2: Definir el modelo de datos y la fuente de datos simulada de telemetría

**Objetivo:** Crear el modelo de datos de dominio que representará la telemetría de un motor de simulación y una fuente de datos in-memory con soporte de flujos de datos.

**Instrucciones:**

1. Crea un archivo denominado `TelemetryEngine.kt` dentro del paquete `com.example.advancedtracker.domain.model`:

```kotlin
package com.example.advancedtracker.domain.model

import kotlinx.serialization.Serializable

@Serializable
data class TelemetryEngine(
    val id: String,
    val name: String,
    val model: String,
    val status: EngineStatus,
    val coreTemperature: Double,
    val fuelLevel: Float, // 0.0f a 1.0f
    val currentRpm: Int,
    val failureRisk: Double // Porcentaje 0.0 a 100.0
)

enum class EngineStatus {
    ACTIVE,
    STANDBY,
    CRITICAL_OVERHEAT,
    MAINTENANCE
}
```

2. Crea la clase repositorio simulada para proveer datos a la interfaz reactiva. Ubícala en `com.example.advancedtracker.data.repository.TelemetryRepository.kt`:

```kotlin
package com.example.advancedtracker.data.repository

import com.example.advancedtracker.domain.model.EngineStatus
import com.example.advancedtracker.domain.model.TelemetryEngine
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.map

class TelemetryRepository {
    private val _engines = MutableStateFlow<List<TelemetryEngine>>(initialEngines)
    val engines: Flow<List<TelemetryEngine>> = _engines.asStateFlow()

    fun getEngineById(id: String): Flow<TelemetryEngine?> {
        return _engines.map { list -> list.firstOrNull { it.id == id } }
    }

    fun updateEngineTelemetry(id: String, temperature: Double, rpm: Int, fuel: Float) {
        val currentList = _engines.value.toMutableList()
        val index = currentList.indexOfFirst { it.id == id }
        if (index != -1) {
            val old = currentList[index]
            val newStatus = when {
                temperature > 105.0 -> EngineStatus.CRITICAL_OVERHEAT
                fuel < 0.05f -> EngineStatus.MAINTENANCE
                else -> EngineStatus.ACTIVE
            }
            currentList[index] = old.copy(
                coreTemperature = temperature,
                currentRpm = rpm,
                fuelLevel = fuel,
                status = newStatus,
                failureRisk = calculateRisk(temperature, rpm)
            )
            _engines.value = currentList
        }
    }

    private fun calculateRisk(temp: Double, rpm: Int): Double {
        var baseRisk = 5.0
        if (temp > 90.0) baseRisk += 35.0
        if (temp > 105.0) baseRisk += 45.0
        if (rpm > 8000) baseRisk += 15.0
        return baseRisk.coerceAtMost(100.0)
    }

    companion object {
        private val initialEngines = listOf(
            TelemetryEngine("ENG-001", "Core Thruster Alpha", "TX-900V", EngineStatus.ACTIVE, 78.5, 0.85f, 5400, 12.4),
            TelemetryEngine("ENG-002", "Auxiliary Turbine Beta", "AT-450", EngineStatus.STANDBY, 42.1, 0.99f, 0, 1.5),
            TelemetryEngine("ENG-003", "Main Reactor Gamma", "RX-Hyperion", EngineStatus.CRITICAL_OVERHEAT, 112.4, 0.42f, 9200, 89.9),
            TelemetryEngine("ENG-004", "Hydraulic Pump Delta", "HP-77X", EngineStatus.MAINTENANCE, 55.0, 0.02f, 1200, 45.0)
        )
    }
}
```

**Resultado esperado:** Las clases se compilarán de manera independiente. El repositorio simula de forma efectiva un flujo unidireccional de datos con cálculo de estados críticos sobre la marcha.

---

### Paso 3: Configurar la navegación tipada segura (Type-Safe Navigation)

**Objetivo:** Crear la definición de rutas utilizando la nueva arquitectura declarativa basada en anotaciones `@Serializable` evitando strings de concatenación manuales.

**Instrucciones:**

1. Define los objetos de ruta serializables en un archivo denominado `TelemetryScreens.kt` dentro del paquete `com.example.advancedtracker.ui.navigation`:

```kotlin
package com.example.advancedtracker.ui.navigation

import kotlinx.serialization.Serializable

@Serializable
object EngineListDestination

@Serializable
data class EngineDetailDestination(val engineId: String)
```

**Resultado esperado:** El compilador de Kotlinx Serialization autogenerará las clases auxiliares internas requeridas para deserializar argumentos dinámicos de manera automatizada al cambiar de pantalla.

---

### Paso 4: Construir la pantalla de listado (ListScreen) con LazyColumn y Material 3

**Objetivo:** Crear una interfaz de usuario fluida y eficiente que filtre los motores mediante un motor de búsqueda dinámico local y renderice tarjetas dinámicas optimizadas.

**Instrucciones:**

1. Crea el archivo de UI `EngineListScreen.kt` en `com.example.advancedtracker.ui.screens.list`:

```kotlin
package com.example.advancedtracker.ui.screens.list

import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.dp
import com.example.advancedtracker.domain.model.EngineStatus
import com.example.advancedtracker.domain.model.TelemetryEngine

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun EngineListScreen(
    engines: List<TelemetryEngine>,
    onEngineClick: (String) -> Unit,
    modifier: Modifier = Modifier
) {
    var searchQuery by remember { mutableStateOf("") }
    
    val filteredEngines = remember(engines, searchQuery) {
        if (searchQuery.isBlank()) {
            engines
        } else {
            engines.filter {
                it.name.contains(searchQuery, ignoreCase = true) ||
                it.id.contains(searchQuery, ignoreCase = true)
            }
        }
    }

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Telemetría de Motores") },
                colors = TopAppBarDefaults.topAppBarColors(
                    containerColor = MaterialTheme.colorScheme.primaryContainer,
                    titleContentColor = MaterialTheme.colorScheme.onPrimaryContainer
                )
            )
        },
        modifier = modifier
    ) { innerPadding ->
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(innerPadding)
        ) {
            // Buscador dinámico de motores
            OutlinedTextField(
                value = searchQuery,
                onValueChange = { searchQuery = it },
                label = { Text("Filtrar por nombre o ID") },
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(16.dp),
                singleLine = true
            )

            if (filteredEngines.isEmpty()) {
                Box(
                    modifier = Modifier.fillMaxSize(),
                    contentAlignment = Alignment.Center
                ) {
                    Text(
                        text = "No se encontraron motores.",
                        style = MaterialTheme.typography.bodyLarge,
                        color = MaterialTheme.colorScheme.outline
                    )
                }
            } else {
                LazyColumn(
                    contentPadding = PaddingValues(bottom = 16.dp),
                    verticalArrangement = Arrangement.spacedBy(8.dp),
                    modifier = Modifier.fillMaxSize()
                ) {
                    items(
                        items = filteredEngines,
                        key = { it.id } // Optimización crucial para reconciliación en LazyColumn
                    ) { engine ->
                        EngineItemCard(
                            engine = engine,
                            onClick = { onEngineClick(engine.id) }
                        )
                    }
                }
            }
        }
    }
}

@Composable
fun EngineItemCard(
    engine: TelemetryEngine,
    onClick: () -> Unit,
    modifier: Modifier = Modifier
) {
    Card(
        modifier = modifier
            .fillMaxWidth()
            .padding(horizontal = 16.dp)
            .clickable { onClick() },
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.surfaceVariant
        )
    ) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp),
            horizontalArrangement = Arrangement.SpaceBetween,
            verticalAlignment = Alignment.CenterVertically
        ) {
            Column(modifier = Modifier.weight(1f)) {
                Text(
                    text = engine.name,
                    style = MaterialTheme.typography.titleMedium,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
                Text(
                    text = "ID: ${engine.id} | Modelo: ${engine.model}",
                    style = MaterialTheme.typography.bodySmall,
                    color = MaterialTheme.colorScheme.outline
                )
                Spacer(modifier = Modifier.height(8.dp))
                LinearProgressIndicator(
                    progress = { engine.fuelLevel },
                    modifier = Modifier.fillMaxWidth().height(4.dp),
                    color = MaterialTheme.colorScheme.primary,
                    trackColor = MaterialTheme.colorScheme.surface
                )
            }

            Spacer(modifier = Modifier.width(16.dp))

            Column(horizontalAlignment = Alignment.End) {
                StatusBadge(status = engine.status)
                Spacer(modifier = Modifier.height(4.dp))
                Text(
                    text = "${engine.coreTemperature}°C",
                    style = MaterialTheme.typography.titleSmall,
                    color = if (engine.status == EngineStatus.CRITICAL_OVERHEAT) {
                        Color.Red
                    } else {
                        MaterialTheme.colorScheme.onSurfaceVariant
                    }
                )
            }
        }
    }
}

@Composable
fun StatusBadge(status: EngineStatus) {
    val (color, text) = when (status) {
        EngineStatus.ACTIVE -> Color(0xFF2E7D32) to "ACTIVE"
        EngineStatus.STANDBY -> Color(0xFFEF6C00) to "STANDBY"
        EngineStatus.CRITICAL_OVERHEAT -> Color(0xFFC62828) to "CRITICAL"
        EngineStatus.MAINTENANCE -> Color(0xFF1565C0) to "SERVICE"
    }

    Surface(
        color = color.copy(alpha = 0.15f),
        contentColor = color,
        shape = MaterialTheme.shapes.small
    ) {
        Text(
            text = text,
            style = MaterialTheme.typography.labelSmall,
            modifier = Modifier.padding(horizontal = 8.dp, vertical = 4.dp)
        )
    }
}
```

**Resultado esperado:** La lista se renderizará eficientemente utilizando `remember` para recalcular los filtros únicamente cuando la entrada de texto cambie o la lista base de motores sufra una actualización asíncrona.

---

### Paso 5: Diseñar la pantalla de detalle (DetailScreen) con estados mutables resilientes

**Objetivo:** Desarrollar un panel donde el operador controle y simule métricas físicas. Emplearemos `rememberSaveable` para asegurar que el valor ingresado por el usuario persista ante la rotación de pantalla.

**Instrucciones:**

1. Crea el archivo `EngineDetailScreen.kt` en `com.example.advancedtracker.ui.screens.detail`:

```kotlin
package com.example.advancedtracker.ui.screens.detail

import androidx.compose.foundation.layout.*
import androidx.compose.foundation.text.KeyboardOptions
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.automirrored.filled.ArrowBack
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.runtime.saveable.rememberSaveable
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.text.input.KeyboardType
import androidx.compose.ui.unit.dp
import com.example.advancedtracker.domain.model.TelemetryEngine

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun EngineDetailScreen(
    engine: TelemetryEngine?,
    onNavigateBack: () -> Unit,
    onSaveMetrics: (temperature: Double, rpm: Int, fuel: Float) -> Unit,
    modifier: Modifier = Modifier
) {
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text(engine?.name ?: "Buscando Motor...") },
                navigationIcon = {
                    IconButton(onClick = onNavigateBack) {
                        Icon(
                            imageVector = Icons.AutoMirrored.Filled.ArrowBack,
                            contentDescription = "Volver"
                        )
                    }
                },
                colors = TopAppBarDefaults.topAppBarColors(
                    containerColor = MaterialTheme.colorScheme.primaryContainer,
                    titleContentColor = MaterialTheme.colorScheme.onPrimaryContainer
                )
            )
        },
        modifier = modifier
    ) { innerPadding ->
        if (engine == null) {
            Box(
                modifier = Modifier
                    .fillMaxSize()
                    .padding(innerPadding),
                contentAlignment = androidx.compose.ui.Alignment.Center
            ) {
                CircularProgressIndicator()
            }
        } else {
            // Uso obligatorio de rememberSaveable para sobrevivir a recreaciones de Activity/Rotaciones
            var inputTemp by rememberSaveable { mutableStateOf(engine.coreTemperature.toString()) }
            var inputRpm by rememberSaveable { mutableStateOf(engine.currentRpm.toString()) }
            var inputFuel by rememberSaveable { mutableStateOf(engine.fuelLevel.toString()) }

            var isError by remember { mutableStateOf(false) }

            Column(
                modifier = Modifier
                    .fillMaxSize()
                    .padding(innerPadding)
                    .padding(16.dp),
                verticalArrangement = Arrangement.spacedBy(16.dp)
            ) {
                Card(
                    modifier = Modifier.fillMaxWidth(),
                    colors = CardDefaults.cardColors(
                        containerColor = MaterialTheme.colorScheme.secondaryContainer
                    )
                ) {
                    Column(modifier = Modifier.padding(16.dp)) {
                        Text(
                            text = "ESTADO DE RIESGO: ${engine.failureRisk}%",
                            style = MaterialTheme.typography.titleLarge,
                            color = if (engine.failureRisk > 50.0) Color(0xFFD32F2F) else MaterialTheme.colorScheme.onSecondaryContainer
                        )
                        Text(
                            text = "Modelo de turbina validado: ${engine.model}",
                            style = MaterialTheme.typography.bodyMedium
                        )
                    }
                }

                Divider()

                Text(
                    text = "Panel de Modulación Física (Simulador)",
                    style = MaterialTheme.typography.titleMedium
                )

                OutlinedTextField(
                    value = inputTemp,
                    onValueChange = { inputTemp = it },
                    label = { Text("Temperatura de Núcleo (°C)") },
                    keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Decimal),
                    modifier = Modifier.fillMaxWidth(),
                    isError = isError
                )

                OutlinedTextField(
                    value = inputRpm,
                    onValueChange = { inputRpm = it },
                    label = { Text("Velocidad de Operación (RPM)") },
                    keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Number),
                    modifier = Modifier.fillMaxWidth(),
                    isError = isError
                )

                OutlinedTextField(
                    value = inputFuel,
                    onValueChange = { inputFuel = it },
                    label = { Text("Nivel de Carburante (0.0 a 1.0)") },
                    keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Decimal),
                    modifier = Modifier.fillMaxWidth(),
                    isError = isError
                )

                if (isError) {
                    Text(
                        text = "Datos de entrada incorrectos. Asegúrese de ingresar valores numéricos válidos.",
                        color = MaterialTheme.colorScheme.error,
                        style = MaterialTheme.typography.bodySmall
                    )
                }

                Button(
                    onClick = {
                        val parsedTemp = inputTemp.toDoubleOrNull()
                        val parsedRpm = inputRpm.toIntOrNull()
                        val parsedFuel = inputFuel.toFloatOrNull()

                        if (parsedTemp != null && parsedRpm != null && parsedFuel != null && parsedFuel in 0.0f..1.0f) {
                            isError = false
                            onSaveMetrics(parsedTemp, parsedRpm, parsedFuel)
                        } else {
                            isError = true
                        }
                    },
                    modifier = Modifier.fillMaxWidth()
                ) {
                    Text("Actualizar Consola de Simulación")
                }
            }
        }
    }
}
```

**Resultado esperado:** El formulario acepta entradas dinámicas del operador y realiza la simulación local de cambios de estado críticos de riesgo de fallo. Ante cambios de orientación de pantalla, el estado sin guardar persistirá en la memoria de restauración.

---

### Paso 6: Unificar la navegación mediante NavHost en MainActivity

**Objetivo:** Reemplazar el layout XML de la Activity principal por la infraestructura moderna de navegación declarativa Compose vinculando el listado y el detalle.

**Instrucciones:**

1. Abre el archivo `MainActivity.kt` de tu ruta `com.example.advancedtracker` y reestructúralo completamente:

```kotlin
package com.example.advancedtracker

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.ui.Modifier
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController
import androidx.navigation.toRoute
import com.example.advancedtracker.data.repository.TelemetryRepository
import com.example.advancedtracker.ui.navigation.EngineDetailDestination
import com.example.advancedtracker.ui.navigation.EngineListDestination
import com.example.advancedtracker.ui.screens.detail.EngineDetailScreen
import com.example.advancedtracker.ui.screens.list.EngineListScreen

class MainActivity : ComponentActivity() {

    // Repositorio in-memory compartido (en la siguiente práctica se migrará a Hilt + Room)
    private val telemetryRepository = TelemetryRepository()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MaterialTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    val navController = rememberNavController()

                    // Recolectar flujos reactivos de datos directamente en el ciclo de Compose
                    val enginesState by telemetryRepository.engines.collectAsState(initial = emptyList())

                    NavHost(
                        navController = navController,
                        startDestination = EngineListDestination
                    ) {
                        // Ruta 1: Pantalla del Listado de Motores
                        composable<EngineListDestination> {
                            EngineListScreen(
                                engines = enginesState,
                                onEngineClick = { id ->
                                    navController.navigate(EngineDetailDestination(engineId = id))
                                }
                            )
                        }

                        // Ruta 2: Pantalla de Detalle de Motores con Tipado Seguro
                        composable<EngineDetailDestination> { backStackEntry ->
                            val args = backStackEntry.toRoute<EngineDetailDestination>()
                            val selectedEngineFlow = telemetryRepository.getEngineById(args.engineId)
                            val engineDetail by selectedEngineFlow.collectAsState(initial = null)

                            EngineDetailScreen(
                                engine = engineDetail,
                                onNavigateBack = {
                                    navController.popBackStack()
                                },
                                onSaveMetrics = { temp, rpm, fuel ->
                                    telemetryRepository.updateEngineTelemetry(
                                        id = args.engineId,
                                        temperature = temp,
                                        rpm = rpm,
                                        fuel = fuel
                                    )
                                    navController.popBackStack()
                                }
                            )
                        }
                    }
                }
            }
        }
    }
}
```

2. Remueve completamente cualquier archivo XML relacionado con la UI antigua de la carpeta `res/layout` si existía.

**Resultado esperado:** Al lanzar la aplicación, se inicia directamente el árbol de composición sin inflar vistas tradicionales. Al pulsar un motor del listado, la navegación con tipado seguro intercepta el ID, abre la pantalla detallada y actualiza el repositorio global de forma síncrona tras la confirmación de la consola de modulación.

---

## Validación y Pruebas

Para garantizar que el laboratorio se ha completado siguiendo las directrices del curso y con una calidad técnica excelente, se definen los siguientes mecanismos de validación.

### Caso de Prueba Adversario: Prueba de Resiliencia de Estado ante Rotación

Este escenario valida que el estado de edición temporal (no guardado) introducido en los formularios no sea destruido por la reconstrucción de la actividad, demostrando la diferencia entre `remember` y `rememberSaveable`.

#### Instrucciones de Ejecución:
1. Inicie el emulador o conecte un dispositivo físico compatible con API 30+.
2. Seleccione el motor **"Core Thruster Alpha"** de la lista principal.
3. Modifique la temperatura a un valor inestable (por ejemplo: `109.8`). No pulse el botón "Actualizar Consola".
4. Utilizando los controles de desarrollo o físicos, **rote la orientación del dispositivo** (de Portrait a Landscape).
5. Inspeccione visualmente el campo de texto de Temperatura.

#### Comportamiento Esperado (Criterio de Aceptación):
* **Correcto:** El cuadro de texto sigue conservando el valor manual de `109.8` debido a que `rememberSaveable` retiene la referencia en el Bundle de restauración.
* **Fallo Crítico:** El valor regresa automáticamente al valor de lectura inicial `78.5` (lo que indica que se utilizó `remember` en lugar de `rememberSaveable` perdiéndose la entrada del operador).

### Matriz de Criterios de Evaluación y Pruebas Métricas

| ID de Prueba | Componente Evaluado | Acción / Entrada | Evidencia Técnica Esperada | Resultado Exitoso (Métrico) |
| :--- | :--- | :--- | :--- | :--- |
| **VAL-001** | Integración de Compilación | `./gradlew clean build` | Generación de APK sin advertencias de colisiones de Kotlin compiler. | El ejecutable Gradle reporta un estado de salida cero `0`. |
| **VAL-002** | Navegación Tipada | Clic en elemento de la lista. | Intercepción de clase `EngineDetailDestination` mediante deserializador seguro. | Desplazamiento instantáneo al ID del motor sin crasheos de casting. |
| **VAL-003** | Buscador Dinámico | Filtro de entrada "TX" o "TX-900V". | Recomputación mediante la cláusula `remember(engines, searchQuery)`. | El árbol reduce el listado instantáneamente mostrando únicamente el motor correspondiente. |
| **VAL-004** | Modificación Reactiva | Guardado de métricas térmicas > 105.0°C. | Recomposición inteligente en la lista tras popBackStack. | El Badge del motor cambia de estado visual "ACTIVE" a "CRITICAL" con color rojo. |

---

## Solución de Problemas

### Problema 1: Error de compilación por falta de un plugin de serialización compatible con Kotlin 2.3.10
* **Síntoma:** El compilador de Kotlin arroja un error en tiempo de construcción: `Serializer class EngineDetailDestination is not defined or compiler plugin is missing`.
* **Causa:** El plugin de serialización `org.jetbrains.kotlin.plugin.serialization` no está declarado en el bloque `plugins` de tu archivo Gradle a nivel de módulo, o su versión no está alineada con el compilador de Kotlin principal (2.3.10).
* **Solución:** Asegúrate de declarar el plugin con la misma versión de Kotlin en tu archivo `app/build.gradle.kts`:
  ```kotlin
  plugins {
      id("com.android.application") version "8.4.0"
      id("org.jetbrains.kotlin.android") version "2.3.10"
      id("org.jetbrains.kotlin.plugin.compose") version "2.3.10"
      id("org.jetbrains.kotlin.plugin.serialization") version "2.3.10"
  }
  ```
  Luego, ejecuta una limpieza profunda del proyecto desde la terminal de Android Studio ejecutando `./gradlew clean` para forzar la regeneración de los serializadores.

### Problema 2: Error en tiempo de ejecución (Crash) al navegar utilizando Navigation Compose
* **Síntoma:** La aplicación se cierra de manera repentina cuando intentas navegar desde el listado a la pantalla de detalle, mostrando un error de tipo `java.lang.IllegalArgumentException: Cannot serialize class EngineDetailDestination`.
* **Causa:** Te has saltado la anotación `@Serializable` en la declaración de las clases de destino dentro de `TelemetryScreens.kt` o estás importando una clase incorrecta de serialización que no pertenece al ecosistema de Kotlinx.
* **Solución:** Abre el archivo `TelemetryScreens.kt` y verifica minuciosamente que tengas importada exactamente la anotación `kotlinx.serialization.Serializable`. Debe lucir de la siguiente manera:
  ```kotlin
  import kotlinx.serialization.Serializable

  @Serializable
  data class EngineDetailDestination(val engineId: String)
  ```
  Evita usar otras anotaciones de serialización heredadas (como Gson, Jackson o Moshi) para gestionar rutas tipadas en Compose Navigation.

---

## Limpieza

Para liberar almacenamiento local de dependencias residuales e impedir conflictos en ejecuciones posteriores de compilación, ejecuta los siguientes pasos:

1. Ejecuta el comando de limpieza de Gradle desde la terminal interna:
```bash
./gradlew clean
```
2. Elimina los directorios de construcción locales autogenerados en tu estructura física:
```bash
rm -rf .gradle build/ app/build/
```
3. Realiza un **Invalidate Caches / Restart** desde la barra superior de menús de Android Studio (`File -> Invalidate Caches...`) para eliminar índices corruptos o desactualizados.

---

## Resumen

En este laboratorio avanzado has logrado una evolución sustancial de arquitectura de UI:

* **Eliminación del XML:** Remplazaste la herencia pesada del sistema de vistas XML clásico por una interfaz ligera declarativa de Kotlin a través de funciones con anotaciones `@Composable`.
* **Material Design 3:** Implementaste un sistema robusto de tarjetas interactivas e indicadores de progreso alineados a los lineamientos modernos de Material Design.
* **Navegación con Tipado Seguro:** Configuraste rutas tipadas que eliminan los riesgos de paso de datos entre pantallas, usando la serialización oficial de Jetpack Navigation.
* **Resiliencia de Estados:** Integraste con éxito `remember` para mejorar las búsquedas reactivas en memoria y `rememberSaveable` para evitar pérdidas involuntarias de datos ingresados por el operador del motor de simulación.

### Recursos Oficiales para Lectura Adicional
* [Jetpack Compose BOM - Versiones y Fechas de Lanzamiento](https://developer.android.com/develop/ui/compose/bom)
* [Navegación seguras con tipado en Jetpack Compose](https://developer.android.com/guide/navigation/design/type-safety?hl=es-419)
* [Gestión de estados en Jetpack Compose](https://developer.android.com/develop/ui/compose/state?hl=es-419)
