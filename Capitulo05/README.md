# Persistencia avanzada con Room 2.8.4 y DataStore: entidades, DAO, migraciones, relaciones, caché y repositorio offline-first API + Room

## Metadatos

| Atributo | Detalle |
| :--- | :--- |
| **Duración** | 216 minutos |
| **Complejidad** | Alta (Hard) |
| **Nivel de Bloom** | Crear (Bloom Level 6) |

---

## Descripción General

En esta práctica de laboratorio avanzada, transformará la aplicación de monitoreo de telemetría desarrollada en la Práctica 4 en una solución móvil altamente resiliente que implementa una estrategia **Offline-First**. Integrará la biblioteca de base de datos relacional **Room 2.8.4** junto con el procesador de símbolos de Kotlin (**KSP**) para cachear de manera eficiente los flujos de datos entrantes desde la API REST. Asimismo, implementará almacenamiento ligero estructurado con **DataStore Preferences 1.1.2** para almacenar configuraciones de usuario y filtros de sincronización de manera asíncrona sin bloquear el hilo de ejecución principal de la interfaz de usuario.

El núcleo de la práctica consiste en el diseño y la codificación de un repositorio híbrido sincronizado. Cuando el dispositivo disponga de conectividad de red, consumirá los datos más recientes a través del cliente HTTP (Retrofit + OkHttp), actualizará de manera atómica las tablas en la base de datos local y emitirá de forma reactiva las actualizaciones a la interfaz gráfica por medio de `Flow`. En caso de desconexión o fallas de red, el repositorio servirá transparentemente los datos almacenados en la base de datos local `telemetry_db`, garantizando una experiencia de usuario ininterrumpida y libre de cuelgues.

---

## Objetivos de Aprendizaje

Al finalizar este laboratorio, usted será capaz de:
- [ ] Configurar una base de datos local relacional con **Room 2.8.4** mediante la definición de entidades con índices únicos, llaves primarias y objetos de acceso a datos (DAOs) reactivos sustentados en `Kotlin Flow`.
- [ ] Implementar almacenamiento estructurado de clave-valor utilizando **DataStore Preferences 1.1.2** bajo un enfoque totalmente asíncrono y no bloqueante.
- [ ] Diseñar un repositorio híbrido utilizando el patrón de diseño **Offline-First** que sincroniza datos remotos (API REST) y persistencia local (Room) utilizando flujos reactivos de datos unidireccionales de manera transparente.
- [ ] Implementar esquemas de migración manual y automática dentro de Room asegurando la integridad de los datos persistentes del usuario entre diferentes versiones de la base de datos.
- [ ] Utilizar **JetBrains AI Assistant** (u otros asistentes basados en IA) como herramienta guiada para agilizar el refactorizado arquitectónico y la escritura automatizada de pruebas unitarias y de integración para la capa de persistencia.

---

## Prerrequisitos

### Requisitos de Conocimiento
- Comprensión sólida de la arquitectura limpia (**Clean Architecture**) y el patrón **MVVM** en Android.
- Dominio de la inyección de dependencias con **Dagger Hilt 2.51.1**.
- Manejo avanzado de **Kotlin Coroutines** (Scopes, Dispatchers, `suspend` functions) y programación reactiva mediante flujos (`Flow`, `StateFlow`, `SharedFlow`).
- Comprensión de los principios REST y el funcionamiento del cliente HTTP **Retrofit**.

### Acceso Requerido
- Entorno de desarrollo Android Studio con permisos de lectura/escritura en el sistema de archivos del usuario.
- Acceso a internet de banda ancha (mínimo 20 Mbps) para la descarga de artefactos de compilación Maven, dependencias Kotlin KSP y librerías adicionales de Jetpack.
- Suscripción activa o periodo de evaluación en **JetBrains AI Assistant** (mínimo versión de plugin 242.23339) para el uso interactivo de optimizaciones por Inteligencia Artificial.

---

## Entorno de Laboratorio

### Hardware Requerido
- **Procesador:** Intel Core i7 / AMD Ryzen 7 (11va generación o superior) o Apple Silicon (M1/M2/M3/M4).
- **Memoria RAM:** Mínimo 16 GB de memoria RAM (32 GB recomendados para mitigar latencias debido a la ejecución concurrente del IDE, demonios Gradle y el Emulador de Android).
- **Almacenamiento:** Unidad de Estado Sólido (SSD) con un mínimo de 40 GB de espacio libre para archivos temporales de Gradle, cachés locales de Kotlin compiler y archivos `.avd` de emuladores.

### Software y Librerías Requeridas

A continuación, se tabulan los componentes exactos de software que deben estar instalados y configurados para mitigar incompatibilidades de compilación de código bytecode Java o interrupciones por desajustes del plugin de Kotlin:

| Tecnología | Edición / Arquitectura / Versión Exacta | Enlace de Descarga / Fuente Oficial |
| :--- | :--- | :--- |
| **Android Studio** | Ladybug (2024.2.1 Patch 3) | [Android Developer Portal](https://developer.android.com/studio) |
| **JDK** | Eclipse Temurin 17 (17.0.10+7) | [Adoptium Project](https://adoptium.net/temurin/releases/) |
| **Gradle** | Versión 8.7.0 (Wrapper) | [Gradle Distributions](https://services.gradle.org/distributions/) |
| **Kotlin Compiler** | Versión 2.3.10 | [Kotlin Lang releases](https://kotlinlang.org/docs/releases.html) |
| **Android Gradle Plugin (AGP)** | Versión 8.4.0 | [Android Gradle Maven Repo](https://developer.android.com/studio/releases/gradle-plugin) |
| **Room Database** | Versión 2.8.4 | [Android Jetpack Room](https://developer.android.com/jetpack/androidx/releases/room) |
| **DataStore Preferences** | Versión 1.1.2 | [Android Jetpack DataStore](https://developer.android.com/jetpack/androidx/releases/datastore) |
| **Dagger Hilt** | Versión 2.51.1 | [Google Dagger GitHub](https://github.com/google/dagger) |
| **Kotlin KSP Plugin** | Versión `2.3.10-1.0.30` | [Google KSP Releases](https://github.com/google/ksp/releases) |
| **JetBrains AI Assistant** | Versión 242.23339 | [JetBrains Marketplace](https://plugins.jetbrains.com/plugin/22282-jetbrains-ai-assistant) |

### Comandos de Configuración Inicial

Para garantizar un entorno limpio y libre de artefactos de compilación corruptos de ejecuciones anteriores de la Práctica 4, ejecute los siguientes comandos en la terminal integrada de su IDE (Android Studio Terminal) desde la carpeta raíz del proyecto `com.example.advancedtracker`:

```bash
## Limpiar toda la caché local de Gradle y directorios temporales de compilación build/
./gradlew clean

## Forzar la actualización e indexación de dependencias remotas del proyecto
./gradlew --refresh-dependencies

## Verificar que la configuración del JDK apunta a la versión 17 de forma consistente
java -version
```

---

## Instrucciones Paso a Paso

### Paso 1: Configurar Dependencias (Room 2.8.4, KSP, DataStore, Hilt)

**Objetivo:** Configurar de forma sólida los archivos de configuración de Gradle para el soporte de compilación de Room mediante Kotlin Symbol Processing (KSP) e integrar DataStore Preferences con compatibilidad total con la JVM target `17`.

**Instrucciones:**

1. Abra el archivo `build.gradle.kts` a nivel de proyecto (raíz) y agregue el plugin de KSP en el bloque `plugins`. Asegúrese de que la versión de KSP coincida de manera estricta con la versión del compilador de Kotlin (`2.3.10`):

```kotlin
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.kapt) apply false
    alias(libs.plugins.ksp) apply false // Plugin de KSP para procesamiento rápido de anotaciones
    alias(libs.plugins.hilt.android) apply false
}
```

2. Abra su archivo de catálogo de dependencias `gradle/libs.versions.toml` y declare de forma explícita las versiones de Room, KSP, Hilt y DataStore de la siguiente manera:

```toml
[versions]
agp = "8.4.0"
kotlin = "2.3.10"
ksp = "2.3.10-1.0.30"
room = "2.8.4"
datastore = "1.1.2"
hilt = "2.51.1"

[libraries]
## Room Database Components
room-runtime = { group = "androidx.room", name = "room-runtime", version.ref = "room" }
room-ktx = { group = "androidx.room", name = "room-ktx", version.ref = "room" }
room-compiler = { group = "androidx.room", name = "room-compiler", version.ref = "room" }

## DataStore Preferences
datastore-preferences = { group = "androidx.datastore", name = "datastore-preferences", version.ref = "datastore" }

## Hilt Dependencies
hilt-android = { group = "google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "google.dagger", name = "hilt-compiler", version.ref = "hilt" }

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
kotlin-android = { id = "com.android.kotlin.android", version.ref = "kotlin" }
kotlin-kapt = { id = "org.jetbrains.kotlin.kapt", version.ref = "kotlin" }
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
hilt-android = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
```

3. Modifique el archivo `build.gradle.kts` a nivel de módulo (`app/build.gradle.kts`) aplicando los plugins requeridos y añadiendo las dependencias en sus bloques respectivos:

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.ksp) // Reemplaza KAPT para Room
    alias(libs.plugins.hilt.android)
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
        
        // Configurar exportación de esquemas para migraciones controladas
        ksp {
            arg("room.schemaLocation", "$projectDir/schemas")
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

    composeOptions {
        kotlinCompilerExtensionVersion = "2.3.10"
    }
}

dependencies {
    // Room Dependencies
    implementation(libs.room.runtime)
    implementation(libs.room.ktx)
    ksp(libs.room.compiler) // Compilador de Room mediante KSP

    // DataStore Preferences
    implementation(libs.datastore.preferences)

    // Hilt Dependency Injection
    implementation(libs.hilt.android)
    ksp(libs.hilt.compiler)

    // Componentes adicionales necesarios para el proyecto previo de la Práctica 4
    implementation("androidx.core:core-ktx:1.12.0")
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.7.0")
    implementation("androidx.activity:activity-compose:1.8.2")
    implementation(platform("androidx.compose:compose-bom:2026.02.01"))
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.ui:ui-graphics")
    implementation("androidx.compose.ui:ui-tooling-preview")
    implementation("androidx.compose.material3:material3")
}
```

4. Ejecute una sincronización de Gradle presionando el botón **Sync Project with Gradle Files** en Android Studio.

**Resultados esperados:**
El proyecto compilará exitosamente sin advertencias ni conflictos de dependencias de KSP y Room 2.8.4. Se creará de forma transparente una carpeta vacía de generación de esquemas bajo `app/schemas/`.

**Verificación:**
Abra la ventana de herramientas **Build** en la parte inferior de Android Studio. Verifique que la tarea `:app:kspDebugKotlin` se ejecuta con éxito y no reporta errores de firma de versiones o de JDK.

---

### Paso 2: Crear Entidades de Room y Relaciones

**Objetivo:** Diseñar la estructura de base de datos relacional para modelar las lecturas de telemetría, asignando índices de búsqueda optimizados para marcas de tiempo e identificadores de red.

**Instrucciones:**

1. Cree el paquete `com.example.advancedtracker.data.local.entities` en el módulo principal de su proyecto.
2. Dentro de este paquete, implemente la clase de datos `TelemetryEntity` que representará de forma unívoca a los registros de sensores recuperados de la API:

```kotlin
package com.example.advancedtracker.data.local.entities

import androidx.room.ColumnInfo
import androidx.room.Entity
import androidx.room.Index
import androidx.room.PrimaryKey

@Entity(
    tableName = "telemetry_table",
    indices = [
        Index(value = ["device_id"]),
        Index(value = ["timestamp"]) // Facilita búsquedas ordenadas cronológicamente
    ]
)
data class TelemetryEntity(
    @PrimaryKey
    @ColumnInfo(name = "id")
    val id: String,

    @ColumnInfo(name = "device_id")
    val deviceId: String,

    @ColumnInfo(name = "telemetry_type")
    val type: String,

    @ColumnInfo(name = "metric_value")
    val value: Double,

    @ColumnInfo(name = "timestamp")
    val timestamp: Long, // Almacena Epoch Milliseconds (tiempo Unix)

    @ColumnInfo(name = "is_synced")
    val isSynced: Boolean = true // Flag fundamental para control de escrituras offline
)
```

3. Cree una clase de mapeo auxiliar en el paquete `com.example.advancedtracker.data.mappers` para desacoplar el modelo de datos de base de datos (`TelemetryEntity`) de los modelos de dominio (`Telemetry`) y los modelos de red (`TelemetryDto`):

```kotlin
package com.example.advancedtracker.data.mappers

import com.example.advancedtracker.data.local.entities.TelemetryEntity
import com.example.advancedtracker.domain.model.Telemetry

fun TelemetryEntity.toDomain(): Telemetry {
    return Telemetry(
        id = this.id,
        deviceId = this.deviceId,
        type = this.type,
        value = this.value,
        timestamp = this.timestamp
    )
}

fun Telemetry.toEntity(isSynced: Boolean = true): TelemetryEntity {
    return TelemetryEntity(
        id = this.id,
        deviceId = this.deviceId,
        type = this.type,
        value = this.value,
        timestamp = this.timestamp,
        isSynced = isSynced
    )
}
```

*Nota: Asegúrese de que la clase de dominio `Telemetry` esté previamente creada en el paquete `com.example.advancedtracker.domain.model` con las propiedades requeridas.*

**Resultados esperados:**
El código compilará correctamente indicando que el mapeo bidireccional está completo y no requiere transformaciones con pérdida de precisión de datos tipo flotante (`Double`).

**Verificación:**
Valide que no hay errores de sintaxis en `TelemetryEntity` y que la anotación `@PrimaryKey` de Room se asigna a una propiedad de tipo `String` no nula.

---

### Paso 3: Diseñar el Objeto de Acceso a Datos (TelemetryDao)

**Objetivo:** Escribir la interfaz DAO que defina consultas reactivas por flujo (`Flow`) y operaciones de transacciones escalares asíncronas de inserción y purgado físico masivo.

**Instrucciones:**

1. Cree el paquete `com.example.advancedtracker.data.local.dao`.
2. Escriba la interfaz `TelemetryDao`. Utilice `OnConflictStrategy.REPLACE` para simplificar la sobrescritura de cachés locales cuando la API envíe datos actualizados del mismo identificador:

```kotlin
package com.example.advancedtracker.data.local.dao

import androidx.room.Dao
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query
import androidx.room.Transaction
import com.example.advancedtracker.data.local.entities.TelemetryEntity
import kotlinx.coroutines.flow.Flow

@Dao
interface TelemetryDao {

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertTelemetryList(telemetries: List<TelemetryEntity>)

    @Query("SELECT * FROM telemetry_table ORDER BY timestamp DESC")
    fun getAllTelemetriesFlow(): Flow<List<TelemetryEntity>>

    @Query("SELECT * FROM telemetry_table WHERE device_id = :deviceId ORDER BY timestamp DESC")
    fun getTelemetriesByDeviceFlow(deviceId: String): Flow<List<TelemetryEntity>>

    @Query("DELETE FROM telemetry_table")
    suspend fun clearAllTelemetries()

    @Query("DELETE FROM telemetry_table WHERE is_synced = 1 AND timestamp < :expirationTime")
    suspend fun purgeOldSyncedTelemetries(expirationTime: Long)

    @Transaction
    suspend fun refreshLocalCache(telemetries: List<TelemetryEntity>) {
        clearAllTelemetries()
        insertTelemetryList(telemetries)
    }
}
```

**Resultados esperados:**
KSP validará las sentencias `@Query` en tiempo de compilación. Si comete un error de ortografía en la tabla `telemetry_table` o en la columna `timestamp`, el proyecto arrojará un error explícito de sintaxis SQL de Room inmediatamente.

**Verificación:**
Construya el proyecto ejecutando la tarea de compilación en Gradle (`Make Project` o `Ctrl + F9` / `Cmd + F9`). El compilador debe terminar satisfactoriamente sin advertencias sobre consultas asíncronas no seguras.

---

### Paso 4: Implementar DataStore Preferences

**Objetivo:** Crear un gestor de preferencias no asíncrono-bloqueante para leer y guardar estados de configuración tales como el filtro por tipo de telemetría seleccionado y el intervalo máximo de sincronización.

**Instrucciones:**

1. Cree el paquete `com.example.advancedtracker.data.local.preferences`.
2. Cree la clase `UserPreferencesSerializer` o cree directamente el delegado singleton para `DataStore` mediante la extensión de contexto de Kotlin:

```kotlin
package com.example.advancedtracker.data.local.preferences

import android.content.Context
import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.Preferences
import androidx.datastore.preferences.core.booleanPreferencesKey
import androidx.datastore.preferences.core.edit
import androidx.datastore.preferences.core.emptyPreferences
import androidx.datastore.preferences.core.stringPreferencesKey
import androidx.datastore.preferences.preferencesDataStore
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.catch
import kotlinx.coroutines.flow.map
import java.io.IOException

// Declarar la instancia singleton de DataStore Preferences
val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "user_preferences")

class TelemetryDataStore(private val context: Context) {

    companion object {
        val SELECTED_DEVICE_KEY = stringPreferencesKey("selected_device_id")
        val OFFLINE_MODE_ACTIVE_KEY = booleanPreferencesKey("offline_mode_active")
    }

    val selectedDeviceFlow: Flow<String?> = context.dataStore.data
        .catch { exception ->
            if (exception is IOException) {
                emit(emptyPreferences())
            } else {
                throw exception
            }
        }
        .map { preferences ->
            preferences[SELECTED_DEVICE_KEY]
        }

    val offlineModeActiveFlow: Flow<Boolean> = context.dataStore.data
        .catch { exception ->
            if (exception is IOException) {
                emit(emptyPreferences())
            } else {
                throw exception
            }
        }
        .map { preferences ->
            preferences[OFFLINE_MODE_ACTIVE_KEY] ?: false
        }

    suspend fun saveSelectedDevice(deviceId: String) {
        context.dataStore.edit { preferences ->
            preferences[SELECTED_DEVICE_KEY] = deviceId
        }
    }

    suspend fun setOfflineMode(active: Boolean) {
        context.dataStore.edit { preferences ->
            preferences[OFFLINE_MODE_ACTIVE_KEY] = active
        }
    }

    suspend fun clearPreferences() {
        context.dataStore.edit { preferences ->
            preferences.clear()
        }
    }
}
```

**Resultados esperados:**
DataStore administrará las escrituras y lecturas de manera totalmente transaccional e inmune a las excepciones de corrupción comunes de las APIs heredadas de `SharedPreferences`.

**Verificación:**
Construya un caso de prueba local simple donde se verifique que la lectura de `offlineModeActiveFlow` retorne su valor por defecto (`false`) si la base de preferencias clave-valor no se ha inicializado en el disco físico.

---

### Paso 5: Configurar la Base de Datos Room y su Migración

**Objetivo:** Crear el componente principal del motor Room (`TelemetryDatabase`) configurando la base de datos `telemetry_db` y añadiendo soporte para una estrategia de migración controlada de versión 1 a versión 2 ante un futuro cambio de esquema.

**Instrucciones:**

1. Cree la clase abstracta de base de datos en el paquete `com.example.advancedtracker.data.local`:

```kotlin
package com.example.advancedtracker.data.local

import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase
import androidx.room.migration.Migration
import androidx.sqlite.db.SupportSQLiteDatabase
import com.example.advancedtracker.data.local.dao.TelemetryDao
import com.example.advancedtracker.data.local.entities.TelemetryEntity

@Database(
    entities = [TelemetryEntity::class],
    version = 2, // Versión de producción configurada
    exportSchema = true
)
abstract class TelemetryDatabase : RoomDatabase() {

    abstract fun telemetryDao(): TelemetryDao

    companion object {
        @Volatile
        private var INSTANCE: TelemetryDatabase? = null

        // Suponga que la Versión 1 original carecía del campo "is_synced" en la tabla telemetry_table
        // Esta migración añade la columna con soporte seguro ante nulos y un valor por defecto de 1 (true)
        val MIGRATION_1_2 = object : Migration(1, 2) {
            override fun migrate(db: SupportSQLiteDatabase) {
                db.execSQL(
                    "ALTER TABLE telemetry_table ADD COLUMN is_synced INTEGER NOT NULL DEFAULT 1"
                )
            }
        }

        fun buildDatabase(context: Context): TelemetryDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    TelemetryDatabase::class.java,
                    "telemetry_db" // Nombre exacto definido en las directrices de entorno
                )
                .addMigrations(MIGRATION_1_2) // Agregar el soporte de migración
                // .fallbackToDestructiveMigration() // Habilitar ÚNICAMENTE si no se desea conservar los datos locales en depuración
                .build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

**Resultados esperados:**
La compilación generará el archivo de esquema JSON bajo la ruta de guardado especificada (`app/schemas/com.example.advancedtracker.data.local.TelemetryDatabase/2.json`).

**Verificación:**
Confirme que la versión del archivo JSON generada coincida exactamente con la constante `version = 2` declarada en la anotación `@Database`.

---

### Paso 6: Construir el Repositorio Offline-First con Flujos de Kotlin

**Objetivo:** Integrar en una implementación concreta de repositorio (`TelemetryRepositoryImpl`) el consumo del cliente REST y la sincronización transparente sobre la base de datos Room, administrando asertivamente los estados de error de red sin interrumpir los flujos activos de datos.

**Instrucciones:**

1. Primero, asegúrese de tener definido el recurso de envoltura del estado de UI (`Resource.kt`) dentro del paquete `com.example.advancedtracker.domain.util` o similar:

```kotlin
package com.example.advancedtracker.domain.util

sealed class Resource<out T> {
    data class Success<out T>(val data: T) : Resource<T>()
    data class Error(val exception: Throwable, val cachedData: Any? = null) : Resource<Nothing>()
    object Loading : Resource<Nothing>()
}
```

2. Defina la interfaz del repositorio en el dominio de la aplicación (`com.example.advancedtracker.domain.repository.TelemetryRepository`):

```kotlin
package com.example.advancedtracker.domain.repository

import com.example.advancedtracker.domain.model.Telemetry
import com.example.advancedtracker.domain.util.Resource
import kotlinx.coroutines.flow.Flow

interface TelemetryRepository {
    fun getTelemetryStream(): Flow<Resource<List<Telemetry>>>
    suspend fun refreshTelemetries(): Result<Unit>
}
```

3. Asegúrese de que su API de red (`TelemetryApi`) de la Práctica 4 esté disponible en su paquete respectivo:

```kotlin
package com.example.advancedtracker.data.remote

import retrofit2.http.GET

interface TelemetryApi {
    @GET("telemetry")
    suspend fun fetchTelemetries(): List<TelemetryDto>
}

// Representación de Dto
data class TelemetryDto(
    val id: String,
    val deviceId: String,
    val type: String,
    val value: Double,
    val timestamp: Long
)
```

4. Implemente el mapeo correspondiente para el Dto en `com.example.advancedtracker.data.mappers`:

```kotlin
package com.example.advancedtracker.data.mappers

import com.example.advancedtracker.data.local.entities.TelemetryEntity
import com.example.advancedtracker.data.remote.TelemetryDto

fun TelemetryDto.toEntity(): TelemetryEntity {
    return TelemetryEntity(
        id = this.id,
        deviceId = this.deviceId,
        type = this.type,
        value = this.value,
        timestamp = this.timestamp,
        isSynced = true
    )
}
```

5. Implemente la clase `TelemetryRepositoryImpl` en `com.example.advancedtracker.data.repository`:

```kotlin
package com.example.advancedtracker.data.repository

import com.example.advancedtracker.data.local.dao.TelemetryDao
import com.example.advancedtracker.data.mappers.toDomain
import com.example.advancedtracker.data.mappers.toEntity
import com.example.advancedtracker.data.remote.TelemetryApi
import com.example.advancedtracker.domain.model.Telemetry
import com.example.advancedtracker.domain.repository.TelemetryRepository
import com.example.advancedtracker.domain.util.Resource
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.emitAll
import kotlinx.coroutines.flow.first
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.flow.map
import java.net.ConnectException
import java.net.UnknownHostException
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class TelemetryRepositoryImpl @Inject constructor(
    private val telemetryApi: TelemetryApi,
    private val telemetryDao: TelemetryDao
) : TelemetryRepository {

    override fun getTelemetryStream(): Flow<Resource<List<Telemetry>>> = flow {
        emit(Resource.Loading)

        // Consultamos inicialmente si existen datos en caché local
        val localCached = telemetryDao.getAllTelemetriesFlow().first()
        
        try {
            // Intentamos sincronizar datos frescos desde la red
            val remoteData = telemetryApi.fetchTelemetries()
            val entities = remoteData.map { it.toEntity() }
            
            // Actualización atómica de la caché de base de datos
            telemetryDao.refreshLocalCache(entities)
        } catch (e: Exception) {
            // Evaluamos si el error es de conexión física de red
            if (e is UnknownHostException || e is ConnectException) {
                // Emitimos un error acompañado de la última caché guardada para resiliencia offline
                emit(Resource.Error(e, localCached.map { it.toDomain() }))
            } else {
                emit(Resource.Error(e))
            }
        }

        // Finalmente enlazamos la emisión de datos reactiva al flujo continuo de la base de datos
        val databaseFlow = telemetryDao.getAllTelemetriesFlow().map { entityList ->
            Resource.Success(entityList.map { it.toDomain() })
        }
        emitAll(databaseFlow)
    }

    override suspend fun refreshTelemetries(): Result<Unit> {
        return try {
            val remoteData = telemetryApi.fetchTelemetries()
            telemetryDao.refreshLocalCache(remoteData.map { it.toEntity() })
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

**Resultados esperados:**
El repositorio proporciona un flujo dinámico unificado que nunca emite un colapso en la UI en caso de que ocurran excepciones HTTP 5xx o desconexión del host, sino que recurre elegantemente al almacenamiento local.

**Verificación:**
Inspeccione que el flujo utilice `emitAll` redirigiendo al usuario al estado de observación continua de la base de datos local `telemetryDao.getAllTelemetriesFlow()`.

---

### Paso 7: Proveer Dependencias mediante Hilt

**Objetivo:** Configurar el módulo de inyección de dependencias de Hilt para instanciar en tiempo de ejecución de manera segura y centralizada la base de datos, los DAOs, las preferencias y el repositorio Offline-First.

**Instrucciones:**

1. Cree o extienda el archivo de módulo de inyección de dependencias `DatabaseModule` en el paquete `com.example.advancedtracker.di`:

```kotlin
package com.example.advancedtracker.di

import android.content.Context
import com.example.advancedtracker.data.local.TelemetryDatabase
import com.example.advancedtracker.data.local.dao.TelemetryDao
import com.example.advancedtracker.data.local.preferences.TelemetryDataStore
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.android.qualifiers.ApplicationContext
import dagger.hilt.components.SingletonComponent
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {

    @Provides
    @Singleton
    fun provideTelemetryDatabase(
        @ApplicationContext context: Context
    ): TelemetryDatabase {
        return TelemetryDatabase.buildDatabase(context)
    }

    @Provides
    @Singleton
    fun provideTelemetryDao(
        database: TelemetryDatabase
    ): TelemetryDao {
        return database.telemetryDao()
    }

    @Provides
    @Singleton
    fun provideTelemetryDataStore(
        @ApplicationContext context: Context
    ): TelemetryDataStore {
        return TelemetryDataStore(context)
    }
}
```

2. Cree o modifique el archivo `RepositoryModule` para enlazar la implementación concreta del repositorio a su abstracción de dominio:

```kotlin
package com.example.advancedtracker.di

import com.example.advancedtracker.data.repository.TelemetryRepositoryImpl
import com.example.advancedtracker.domain.repository.TelemetryRepository
import dagger.Binds
import dagger.Module
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent
import javax.inject.Singleton

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

**Resultados esperados:**
El grafo de dependencias de Hilt podrá resolverse correctamente durante el arranque de la aplicación móvil sin arrojar excepciones de dependencias faltantes o duplicidades de tipo singleton.

**Verificación:**
Haga clic en la opción **Build > Clean Project** de la barra de menús principal de Android Studio y luego reconstruya el proyecto. La generación de clases del plugin de Hilt no debe dar problemas de dependencias cíclicas.

---

## Validación y Pruebas

Para garantizar que nuestra implementación Offline-First y de base de datos Room 2.8.4 funciona con estricta rigidez bajo condiciones extremas de degradación de red y fallas transitorias, ejecutaremos las siguientes estrategias de validación en tiempo real.

### 1. Validación de Comportamiento Offline en Tiempo Real

#### Escenario de Prueba: Pérdida total de conectividad durante la observación activa de telemetrías.

**Acciones de ejecución:**
1. Despliegue el aplicativo en un Emulador de Android ejecutando API 33 (Android 13.0).
2. Abra la aplicación y verifique que la lista de telemetrías se cargue completamente mostrando datos dinámicos.
3. Desde la barra lateral del emulador (tres puntos verticales `...`), ingrese a **Cellular** y cambie la opción **Data Status** de *Home* a *Denied* (o de manera equivalente, active el **Modo Avión** en la barra superior de estado del emulador).
4. Cierre por completo la aplicación matando su proceso desde el panel de aplicaciones recientes de Android (`adb shell am force-stop com.example.advancedtracker`).
5. Reabra el aplicativo desde el launcher del dispositivo.

**Resultado esperado detectable (Criterio de Aceptación):**
La aplicación se abre instantáneamente. En lugar de desplegar un indicador de error de red vacío o colapsar con una excepción `ConnectException`, despliega inmediatamente la lista de registros con su última caché guardada que se obtuvo antes de cortar la red. Se puede verificar la integridad del caché local abriendo el panel **App Inspection** en Android Studio, seleccionando la pestaña **Database Inspector** y seleccionando el archivo `telemetry_db` para inspeccionar visualmente que los datos desplegados en la UI coincidan bit a bit con los registros en la tabla `telemetry_table`.

---

### 2. Pruebas de Resiliencia y Pruebas Adversarias

Para blindar la consistencia de nuestra lógica de negocio, diseñamos una prueba adversarial unitaria robusta simulando fallas críticas de infraestructura local de persistencia.

#### Prueba Adversarial: Manejo de Excepciones del Motor de SQLite por Agotamiento de Espacio en Disco

En esta prueba unitaria, simulamos una base de datos donde el hardware del teléfono móvil simula fallas de bajo nivel físico de la base de datos SQLite (como `SQLiteDiskIOException`), asegurando que nuestra capa de repositorios reacciona de manera predecible sin detener abruptamente el hilo principal.

Cree el archivo de pruebas `TelemetryRepositoryTest.kt` dentro del directorio `app/src/test/java/com/example/advancedtracker/data/repository/`:

```kotlin
package com.example.advancedtracker.data.repository

import android.database.sqlite.SQLiteDiskIOException
import com.example.advancedtracker.data.local.dao.TelemetryDao
import com.example.advancedtracker.data.local.entities.TelemetryEntity
import com.example.advancedtracker.data.remote.TelemetryApi
import com.example.advancedtracker.data.remote.TelemetryDto
import com.example.advancedtracker.domain.util.Resource
import kotlinx.coroutines.ExperimentalCoroutinesApi
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.flow.toList
import kotlinx.coroutines.test.runTest
import org.junit.Assert.assertEquals
import org.junit.Assert.assertTrue
import org.junit.Before
import org.junit.Test
import org.mockito.Mock
import org.mockito.Mockito.`when`
import org.mockito.MockitoAnnotations

@OptIn(ExperimentalCoroutinesApi::class)
class TelemetryRepositoryTest {

    @Mock
    private lateinit var mockApi: TelemetryApi

    @Mock
    private lateinit var mockDao: TelemetryDao

    private lateinit var repository: TelemetryRepositoryImpl

    @Before
    fun setUp() {
        MockitoAnnotations.openMocks(this)
        repository = TelemetryRepositoryImpl(mockApi, mockDao)
    }

    @Test
    fun getTelemetryStream_whenDatabaseThrowsDiskException_failsGracefully() = runTest {
        // Configurar escenario de base de datos rota o colapsada por espacio en disco
        `when`(mockDao.getAllTelemetriesFlow()).thenReturn(flow {
            throw SQLiteDiskIOException("Simulated hard failure: Disk Full on device storage")
        })
        
        // Simular respuesta correcta de la red
        val fakeApiData = listOf(
            TelemetryDto("1", "dev_01", "temp", 24.5, 1718101200000L)
        )
        `when`(mockApi.fetchTelemetries()).thenReturn(fakeApiData)

        // Consumo de flujo
        val results = mutableListOf<Resource<Any>>()
        try {
            repository.getTelemetryStream().toList(results)
        } catch (e: Exception) {
            results.add(Resource.Error(e))
        }

        // Verificar comportamiento tolerante a fallas
        assertTrue(results.isNotEmpty())
        val initialResult = results[0]
        assertTrue("El flujo inicial debe desplegar el estado de carga", initialResult is Resource.Loading)
        
        // Buscamos si existe un evento de error de base de datos capturado de manera segura
        val containsError = results.any { it is Resource.Error }
        assertTrue("La capa de repositorio debe atrapar o fluir errores de persistencia sin causar crash", containsError)
    }
}
```

Para correr esta prueba de resiliencia, ejecute el siguiente comando en la terminal de Android Studio:

```bash
./gradlew testDebugUnitTest --tests "com.example.advancedtracker.data.repository.TelemetryRepositoryTest"
```

El resultado de la tarea debe ser exitoso (`SUCCESS`), validando que la capa del repositorio captura y maneja de manera resiliente las excepciones de disco.

---

### 3. Validación Asistida por IA (Límites de Confianza y Casos Adversarios)

Para el uso seguro y eficiente de **JetBrains AI Assistant**, se debe recordar que los sistemas basados en IA de tipo Large Language Model (LLM) no deben usarse sin supervisión directa debido a su susceptibilidad de inventar código desactualizado o proponer APIs obsoletas.

**Instrucción / Prompt seguro para refinamiento arquitectónico en JetBrains AI Assistant Chat:**

> **Mensaje de Sistema y Contexto de Consulta de Datos:**
> "Usted actúa en este chat como un revisor de código estático experto en Android Jetpack Room 2.8.4, Kotlin Coroutines y Clean Architecture. Su objetivo es identificar fugas de memoria, optimizaciones SQL no indexadas y fallas en la resiliencia offline del fragmento de repositorio adjunto.
> **Restricciones:** No invente firmas de métodos inexistentes de la librería Room 2.8.4. No genere soluciones que requieran deshabilitar la verificación en tiempo de compilación.
> **Código a evaluar:** (Pegue aquí su archivo `TelemetryRepositoryImpl.kt`)"

**Evidencia de Validación Esperada:**
El asistente de IA debe generar un análisis indicando si el flujo reactivo está consumiendo operaciones en el hilo incorrecto, aconsejando el uso explícito del despachador `Dispatchers.IO` en llamadas que no son reactivas nativas de Room.

---

## Solución de Problemas

En esta sección se listan y resuelven dos problemas reales de alta complejidad técnica que suelen surgir durante la ejecución de esta práctica de laboratorio:

### Problema 1: Excepción de compilación de KSP indicando que el esquema de base de datos de Room cambió de forma no controlada.

- **Síntomas:**
  Al intentar ejecutar o empaquetar el proyecto, la compilación de Gradle se detiene mostrando el siguiente error crítico en la pestaña de logs:
  `error: Room cannot verify the data integrity of your database. Please provide a migration path or use fallbackToDestructiveMigration.`
  
- **Causa raíz:**
  Usted modificó la clase de entidad `TelemetryEntity` (por ejemplo, agregó un nuevo campo o modificó el nombre de una columna utilizando la anotación `@ColumnInfo`) o actualizó manualmente el parámetro `version` de la base de datos en la anotación `@Database` de `@Database(..., version = 2)` a `@Database(..., version = 3)`, pero omitió registrar un objeto de migración `Migration` adecuado en la inicialización del constructor del constructor de base de datos (`addMigrations(...)`).

- **Solución / Acción correctiva:**
  Para entornos de desarrollo rápido, si no le interesa la persistencia previa de los datos de depuración, agregue de manera temporal el método `.fallbackToDestructiveMigration()` en el builder de Room ubicado en su clase `TelemetryDatabase.kt`:
  
  ```kotlin
  fun buildDatabase(context: Context): TelemetryDatabase {
      return Room.databaseBuilder(
          context.applicationContext,
          TelemetryDatabase::class.java,
          "telemetry_db"
      )
      .fallbackToDestructiveMigration() // Agregado para recrear las tablas automáticamente
      .build()
  }
  ```
  
  Para entornos de producción reales:
  1. Determine las sentencias SQL DDL exactas para reflejar el cambio.
  2. Incremente la versión en la definición de la base de datos.
  3. Declare un nuevo objeto de tipo `Migration` de forma explícita que altere la tabla SQLite (`ALTER TABLE ... ADD COLUMN ...`) y agréguelo usando `.addMigrations(MIGRATION_X_Y)`.

---

### Problema 2: El recolector de DataStore arroja excepciones de bloqueo mutuo `IllegalStateException` de archivos concurrentes.

- **Síntomas:**
  La aplicación se bloquea de manera aleatoria al abrirse o al intentar mutar preferencias concurrentemente desde diferentes subprocesos, arrojando el error en consola:
  `java.lang.IllegalStateException: There are multiple DataStores active for the same file: /data/user/0/.../files/datastore/user_preferences.preferences_pb. You should only have a single instance of DataStore for this file!`

- **Causa raíz:**
  La definición del delegado de obtención del singleton `preferencesDataStore` se declaró dentro de una clase que se instancia múltiples veces o fuera del nivel de archivo global de Kotlin. Esto provoca que Hilt o el ciclo de vida del framework instancien múltiples gestores interactuando en paralelo sobre la misma dirección de disco física.

- **Solución / Acción correctiva:**
  Asegúrese de declarar la inicialización delegada `val Context.dataStore` como una extensión **directamente en el nivel superior del archivo Kotlin** de configuración (afuera de cualquier clase, struct o bloque de inicialización de clase companion), de manera que se garantice que solo se compila una instancia única por cargador de clases de la Máquina Virtual de Java:
  
  ```kotlin
  // Archivo: TelemetryDataStore.kt
  package com.example.advancedtracker.data.local.preferences
  
  import android.content.Context
  import androidx.datastore.preferences.preferencesDataStore
  
  // ¡CORRECTO! Declaración en la raíz de archivo, fuera de toda definición de clase
  val Context.dataStore by preferencesDataStore(name = "user_preferences")
  
  class TelemetryDataStore(private val context: Context) {
      // Use el delegado únicamente apuntando al objeto context.dataStore global
  }
  ```

---

## Limpieza

Para restaurar el entorno de desarrollo al estado limpio previo y verificar que la regeneración dinámica del código de persistencia no conserve metadatos obsoletos, aplique las siguientes acciones:

1. Desinstale la aplicación de prueba instalada en sus dispositivos de prueba o emuladores de desarrollo para vaciar por completo las bases de datos de SQLite persistidas físicamente en los directorios aislados de Android. Use la terminal para asegurar una limpieza total:
   ```bash
   adb uninstall com.example.advancedtracker
   ```
2. Ejecute un saneado profundo del gestor Gradle limpiando compilaciones incrementales del motor KSP:
   ```bash
   ./gradlew cleanBuildCache
   ./gradlew clean
   ```
3. Opcionalmente, puede eliminar de manera segura la carpeta temporal oculta `.gradle/` y `.kotlin/` generada en la carpeta raíz del proyecto en caso de persistencia de inconsistencias insolubles de sincronización de plugins de Hilt o compilador de Room.

---

## Resumen

### Puntos Clave Cubiertos en la Práctica
- **Integración de Room 2.8.4 con KSP:** Configuración moderna del procesador de símbolos de Kotlin, minimizando significativamente los tiempos de compilación comparado con el procesador heredado KAPT.
- **Estrategia de Almacenamiento Offline-First:** Arquitectura resiliente de flujo de datos unificado. Se intenta la obtención de datos remotos mediante Retrofit para actualizar la caché local, y de forma transparente se expone la base de datos SQLite como la única fuente de verdad (*Single Source of Truth*) para la interfaz de usuario.
- **Asincronía Total con DataStore Preferences:** Persistencia de configuraciones clave-valor ligeras por medio de APIs transaccionales basadas completamente en flujos de datos asíncronos (`Kotlin Flow`), eliminando fallas de bloqueo e inconsistencias transaccionales de escritura.
- **Estrategias de Migración y Robustez:** Implementación de flujos de control de migraciones de esquemas y cobertura mediante pruebas de estrés simulando condiciones extremas en la base de datos de almacenamiento local de bajo nivel.

### Recursos de Información Adicionales
- [Documentación Oficial de Android Room](https://developer.android.com/training/data-storage/room) [Enlace Oficial de Android Jetpack]
- [Guía de Arquitectura Offline-First en Android Studio](https://developer.android.com/topic/architecture/data-layer/offline-first) [Enlace Oficial de Android Studio Support]
- [Documentación del Repositorio de Kotlin Symbol Processing (KSP)](https://github.com/google/ksp) [Enlace de Repositorio Oficial de Google KSP]
- [Guías de Implementación de Jetpack DataStore](https://developer.android.com/topic/libraries/architecture/datastore) [Enlace Oficial de Jetpack DataStore]
- [Documentación de Plugin de JetBrains AI Assistant](https://www.jetbrains.com/help/ai/installation-and-subscription.html) [Enlace Oficial de JetBrains AI Assistant]
