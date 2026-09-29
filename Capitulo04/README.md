# Integración de API REST con Retrofit 3.0.0 y OkHttp BOM 5.5.0: GET, POST, autenticación, serialización, interceptores y manejo de estados de red

## Metadatos

| Atributo | Detalle |
| :--- | :--- |
| **Duración** | 216 minutos |
| **Complejidad** | Media |
| **Nivel de Bloom** | Aplicar |

## Descripción General

En este laboratorio de nivel avanzado, migrarás el sistema de simulación local del rastreador telemetry a un pipeline de red real consumiendo servicios REST. Configurarás Retrofit 3.0.0-alpha junto con OkHttp BOM 5.5.0-alpha, implementando interceptores para el registro de trazas y la inserción dinámica de cabeceras de autorización Bearer. Estructurarás un wrapper genérico de estados de red (`NetworkResult`) que convertirá las respuestas HTTP en estados reactivos para su consumo seguro en la UI con Jetpack Compose.

## Objetivos de Aprendizaje

Al finalizar este laboratorio, serás capaz de:
- [ ] Configurar el archivo de dependencias de Gradle aplicando OkHttp BOM 5.5.0-alpha y Retrofit 3.0.0-alpha integrando de forma óptima Kotlinx Serialization.
- [ ] Diseñar e implementar un interceptor de red para la inyección automatizada de cabeceras de autenticación (Bearer Token) y logs detallados.
- [ ] Construir un wrapper genérico de respuestas asíncronas (`NetworkResult`) para mapear de forma segura estados de éxito, carga y error.
- [ ] Crear el servicio de API, su repositorio asociado y conectarlo al ViewModel para emitir estados reactivos bidireccionales mediante StateFlow.

## Prerrequisitos

Para completar este laboratorio con éxito, se requiere:
*   Haber completado satisfactoriamente la Práctica 3 (Arquitectura limpia con Hilt y DataStore).
*   Comprensión de los principios fundamentales de la arquitectura MVVM y Clean Architecture.
*   Conocimientos de programación concurrente utilizando Kotlin Coroutines, StateFlow y SharedFlow.
*   Conexión a internet activa para la descarga automatizada de dependencias y la posterior validación contra el servidor de pruebas.

## Entorno de Laboratorio

Asegúrate de que tu estación de trabajo cumple con las siguientes especificaciones:

### Requisitos de Hardware

| Componente | Especificación Mínima | Especificación Recomendada |
| :--- | :--- | :--- |
| **Procesador** | Intel Core i7 / AMD Ryzen 7 (11ª Gen) | Apple Silicon (M1/M2/M3) o equivalente x86 |
| **Memoria RAM** | 16 GB | 32 GB (para una ejecución fluida de emuladores) |
| **Almacenamiento**| SSD con 40 GB libres | SSD NVMe con 60 GB libres |

### Requisitos de Software

| Herramienta / Librería | Versión Exacta | Enlace Oficial / Origen |
| :--- | :--- | :--- |
| **IDE** | Android Studio Ladybug (2024.2.1 Patch 3) | [Android Studio](https://developer.android.com/studio) |
| **JDK** | Eclipse Temurin JDK 17 (17.0.10+7) | [Adoptium Temurin](https://adoptium.net/temurin/releases/?version=17) |
| **Kotlin Compiler** | 2.3.10 | [Kotlin Lang](https://kotlinlang.org/) |
| **OkHttp BOM** | 5.5.0-alpha (5.0.0-alpha.14 / 5.5.0-alpha) | [OkHttp Square](https://square.github.io/okhttp/) |
| **Retrofit** | 3.0.0-alpha1 | [Retrofit Square](https://github.com/square/retrofit) |
| **Kotlinx Serialization**| 1.6.2 | [Kotlinx Serialization](https://github.com/Kotlin/kotlinx.serialization) |
| **Hilt DI** | 2.51.1 | [Dagger Hilt](https://dagger.dev/hilt/) |
| **JetBrains AI Assistant**| 242.23339 | Requiere suscripción activa a JetBrains AI Pro |

### Comandos de Configuración Inicial

Para garantizar la consistencia en el laboratorio, asegúrate de utilizar el paquete base definido para el proyecto: `com.example.advancedtracker`. 

Verifica la versión del compilador de Kotlin en tu archivo raíz `build.gradle.kts` o en tu catálogo de dependencias `libs.versions.toml`:

```kotlin
// Archivo: build.gradle.kts (Raíz de proyecto)
plugins {
    kotlin("android") version "2.3.10" apply false
    kotlin("plugin.serialization") version "2.3.10" apply false
}
```

---

## Instrucciones Paso a Paso

### Paso 1: Configuración de Dependencias de Red en Gradle (BOM y Plugins)

**Objetivo:** Configurar el catálogo de dependencias del proyecto para incorporar Retrofit 3.0.0-alpha y OkHttp BOM 5.5.0-alpha garantizando la compatibilidad con Kotlin 2.3.10.

**Instrucciones:**

1. Abre tu archivo `libs.versions.toml` (dentro de la carpeta `gradle/`) y actualiza o añade las siguientes definiciones en las secciones correspondientes:

```toml
[versions]
retrofit = "3.0.0-alpha1"
okhttp = "5.0.0-alpha.14" # Representando la versión correspondiente bajo la estrategia BOM 5.5.0-alpha
kotlinxSerializationJson = "1.6.2"

[libraries]
## OkHttp Platform (BOM)
okhttp-bom = { group = "com.squareup.okhttp3", name = "okhttp-bom", version.ref = "okhttp" }
okhttp = { group = "com.squareup.okhttp3", name = "okhttp" }
okhttp-logging = { group = "com.squareup.okhttp3", name = "logging-interceptor" }

## Retrofit
retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
retrofit-converter-serialization = { group = "com.squareup.retrofit2", name = "converter-kotlinx-serialization", version.ref = "retrofit" }

## Kotlinx Serialization
kotlinx-serialization-json = { group = "org.jetbrains.kotlinx", name = "kotlinx-serialization-json", version.ref = "kotlinxSerializationJson" }
```

2. Abre tu archivo `build.gradle.kts` del módulo `:app` y aplica el plugin de serialización junto con las nuevas librerías:

```kotlin
// Archivo: app/build.gradle.kts
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.kapt)
    alias(libs.plugins.dagger.hilt.android)
    // Plugin de compilación de serialización de Kotlin
    id("org.jetbrains.kotlinx.serialization") version "2.3.10"
}

android {
    compileSdk = 35

    defaultConfig {
        applicationId = "com.example.advancedtracker"
        minSdk = 30
        targetSdk = 35
        versionCode = 1
        versionName = "1.0"

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
}

dependencies {
    // Importación del BOM de OkHttp para heredar versiones consistentes
    implementation(platform(libs.okhttp.bom))
    implementation(libs.okhttp)
    implementation(libs.okhttp.logging)

    // Retrofit 3.0.0-alpha y adaptador nativo
    implementation(libs.retrofit)
    implementation(libs.retrofit.converter.serialization)

    // Serialización nativa
    implementation(libs.kotlinx.serialization.json)

    // JUnit y MockWebServer para pruebas unitarias de la API
    testImplementation("com.squareup.okhttp3:mockwebserver:5.0.0-alpha.14")
    testImplementation("junit:junit:4.13.2")
    testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.8.0")
}
```

3. Agrega el permiso de red en el archivo `app/src/main/AndroidManifest.xml` justo encima de la etiqueta `<application>`:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

4. Ejecuta un **Gradle Sync** en tu IDE para validar la correcta resolución de todas las librerías.

**Resultado Esperado:** Sincronización exitosa del proyecto. No debe presentarse ninguna colisión entre el compilador de Kotlin 2.3.10 y el plugin de serialización.

**Verificación:** Ejecuta una compilación del proyecto utilizando la terminal embebida de Android Studio para descartar errores silenciosos:

```bash
./gradlew compileDebugSources --dry-run
```

---

### Paso 2: Implementación de la Capa de Modelado de Datos (DTOs Serializable)

**Objetivo:** Diseñar las clases de transferencia de datos (DTO) anotadas para el mapeo seguro y eficiente usando `kotlinx.serialization`.

**Instrucciones:**

1. Crea el paquete `com.example.advancedtracker.data.model` si no existe.
2. Añade un nuevo archivo de Kotlin con el nombre `TelemetryDto.kt` para representar el payload JSON del servidor:

```kotlin
package com.example.advancedtracker.data.model

import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class TelemetryDto(
    @SerialName("id") val id: String,
    @SerialName("deviceId") val deviceId: String,
    @SerialName("latitude") val latitude: Double,
    @SerialName("longitude") val longitude: Double,
    @SerialName("speed") val speed: Double,
    @SerialName("timestamp") val timestamp: Long,
    @SerialName("status") val status: String
)
```

3. Diseña el DTO de respuesta para la autenticación en el archivo `AuthResponseDto.kt` dentro del mismo paquete:

```kotlin
package com.example.advancedtracker.data.model

import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class AuthResponseDto(
    @SerialName("accessToken") val accessToken: String,
    @SerialName("refreshToken") val refreshToken: String,
    @SerialName("expiresIn") val expiresIn: Long
)
```

**Resultado Esperado:** Creación correcta de las clases de datos autogeneradas por el plugin de compilación de Kotlinx Serialization.

**Verificación:** Ejecuta el análisis estático o de compilación para garantizar que no existan errores de serialización ausentes en los esquemas DTO:

```bash
./gradlew compileDebugKotlin
```

---

### Paso 3: Diseño del Manejador Genérico de Estados de Red (NetworkResult)

**Objetivo:** Crear un wrapper genérico que encapsule las llamadas HTTP y maneje errores de forma consistente bajo una arquitectura limpia.

**Instrucciones:**

1. En el paquete `com.example.advancedtracker.data.network`, crea el archivo `NetworkResult.kt`.
2. Define la estructura genérica basada en interfaces selladas de Kotlin para forzar el control exhaustivo de flujos en la capa UI:

```kotlin
package com.example.advancedtracker.data.network

import retrofit2.HttpException
import java.io.IOException

sealed interface NetworkResult<out T> {
    data class Success<out T>(val data: T) : NetworkResult<T>
    data class Error(val code: Int, val message: String?, val exception: Throwable? = null) : NetworkResult<Nothing>
    data object Loading : NetworkResult<Nothing>
}

/**
 * Ejecutor seguro de operaciones de red que atrapa excepciones de bajo nivel
 * y las transforma en estados controlados bajo NetworkResult.
 */
suspend fun <T> safeApiCall(apiCall: suspend () -> T): NetworkResult<T> {
    return try {
        NetworkResult.Success(apiCall())
    } catch (e: HttpException) {
        // Excepciones devueltas por el servidor (4xx, 5xx)
        NetworkResult.Error(code = e.code(), message = e.message(), exception = e)
    } catch (e: IOException) {
        // Fallos de conectividad (sin red, DNS caídos, timeouts)
        NetworkResult.Error(code = -1, message = "Error de conexión. Verifica tu internet.", exception = e)
    } catch (e: Exception) {
        // Cualquier otra excepción no controlada
        NetworkResult.Error(code = -999, message = e.localizedMessage ?: "Error inesperado", exception = e)
    }
}
```

**Resultado Esperado:** Un utilitario robusto que evite la propagación de excepciones inestables a lo largo de las corrutinas de tu aplicación.

**Verificación:** Revisa la compilación de la interfaz sellada. Debe ser accesible de manera genérica para cualquier tipo de datos devuelto por la API.

---

### Paso 4: Creación del Interceptor de Autenticación Dinámico de OkHttp

**Objetivo:** Desarrollar un interceptor personalizado que inyecte de manera transparente el token de seguridad Bearer recuperado desde las preferencias locales.

**Instrucciones:**

1. Crea el archivo `AuthInterceptor.kt` en el paquete `com.example.advancedtracker.data.network`.
2. Escribe la lógica para inyectar dinámicamente el token si está disponible en tus preferencias de DataStore. El código debe evitar bucles recursivos al consultar recursos protegidos:

```kotlin
package com.example.advancedtracker.data.network

import okhttp3.Interceptor
import okhttp3.Response
import kotlinx.coroutines.runBlocking
import kotlinx.coroutines.flow.firstOrNull
import com.example.advancedtracker.data.repository.UserPreferencesRepository
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class AuthInterceptor @Inject constructor(
    private val preferencesRepository: UserPreferencesRepository
) : Interceptor {

    override fun intercept(chain: Interceptor.Chain): Response {
        val originalRequest = chain.request()
        
        // Evitar inyección de token en rutas públicas como login/registro para evitar recursión
        if (originalRequest.url.encodedPath.contains("/auth/login") || 
            originalRequest.url.encodedPath.contains("/auth/refresh")) {
            return chain.proceed(originalRequest)
        }

        // Obtención de token síncrono bloqueante solo durante el ciclo de red en segundo plano
        val token = runBlocking {
            preferencesRepository.userTokenFlow.firstOrNull()
        }

        val requestBuilder = originalRequest.newBuilder()
            .header("Accept", "application/json")
            .header("User-Agent", "Advanced-Telemetry-Tracker/1.0.0")

        if (!token.isNullOrBlank()) {
            requestBuilder.header("Authorization", "Bearer $token")
        }

        return chain.proceed(requestBuilder.build())
    }
}
```

> **Nota de Contexto Técnico:** El uso de `runBlocking` dentro de un interceptor de OkHttp se considera correcto porque OkHttp ejecuta estas llamadas dentro de hilos específicos de su propio pool de hilos de red, sin bloquear nunca el hilo principal de la UI.

**Resultado Esperado:** Un interceptor integrado capaz de adjuntar cabeceras `Authorization` en todas las peticiones salientes.

**Verificación:** Pruebas unitarias o logs de OkHttp demostrarán que el flujo de cabeceras se ejecuta de manera síncrona con el token recuperado de DataStore.

---

### Paso 5: Definición del Servicio Retrofit y Módulo de Inyección de Dependencias de Hilt

**Objetivo:** Configurar la API y el módulo Hilt central para proveer instancias seguras de Retrofit y OkHttp.

**Instrucciones:**

1. Define la interfaz de endpoints REST del servidor. Crea `TelemetryApiService.kt` en el paquete `com.example.advancedtracker.data.network`:

```kotlin
package com.example.advancedtracker.data.network

import com.example.advancedtracker.data.model.TelemetryDto
import retrofit2.http.Body
import retrofit2.http.GET
import retrofit2.http.POST
import retrofit2.http.Query

interface TelemetryApiService {

    @GET("telemetry")
    suspend fun getTelemetryHistory(
        @Query("limit") limit: Int = 50
    ): List<TelemetryDto>

    @POST("telemetry")
    suspend fun sendTelemetry(
        @Body telemetry: TelemetryDto
    ): TelemetryDto
}
```

2. Implementa la configuración de inyección de dependencias con Hilt en el archivo `com/example/advancedtracker/di/NetworkModule.kt`:

```kotlin
package com.example.advancedtracker.di

import com.example.advancedtracker.data.network.AuthInterceptor
import com.example.advancedtracker.data.network.TelemetryApiService
import com.jakewharton.retrofit2.converter.kotlinx.serialization.asConverterFactory
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent
import kotlinx.serialization.json.Json
import okhttp3.MediaType.Companion.toMediaType
import okhttp3.OkHttpClient
import okhttp3.logging.HttpLoggingInterceptor
import retrofit2.Retrofit
import java.util.concurrent.TimeUnit
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    private const val BASE_URL = "https://api.example.com/v1/" // Puerto estándar HTTPS 443 por defecto

    @Provides
    @Singleton
    fun provideJsonConfiguration(): Json {
        return Json {
            ignoreUnknownKeys = true // Ignora campos adicionales del JSON que no existan en el DTO
            coerceInputValues = true  // Recupera datos por defecto de Kotlin si el servidor envía nulls incompatibles
            encodeDefaults = true
        }
    }

    @Provides
    @Singleton
    fun provideOkHttpClient(
        authInterceptor: AuthInterceptor
    ): OkHttpClient {
        val loggingInterceptor = HttpLoggingInterceptor().apply {
            level = HttpLoggingInterceptor.Level.BODY
        }

        return OkHttpClient.Builder()
            .connectTimeout(15, TimeUnit.SECONDS)
            .readTimeout(15, TimeUnit.SECONDS)
            .writeTimeout(15, TimeUnit.SECONDS)
            .addInterceptor(authInterceptor)
            .addInterceptor(loggingInterceptor)
            .build()
    }

    @Provides
    @Singleton
    fun provideRetrofit(
        okHttpClient: OkHttpClient,
        json: Json
    ): Retrofit {
        val contentType = "application/json".toMediaType()
        
        return Retrofit.Builder()
            .baseUrl(BASE_URL)
            .client(okHttpClient)
            .addConverterFactory(json.asConverterFactory(contentType))
            .build()
    }

    @Provides
    @Singleton
    fun provideTelemetryApiService(retrofit: Retrofit): TelemetryApiService {
        return retrofit.create(TelemetryApiService::class.java)
    }
}
```

**Resultado Esperado:** Un grafo de inyección limpio donde Hilt expone tanto el cliente HTTP altamente robusto como la interfaz del proxy de Retrofit.

**Verificación:** Compila el proyecto. Si existe algún fallo en la inyección de dependencias de Hilt, la compilación de la tarea de procesamiento de anotaciones (Kapt o KSP) fallará inmediatamente arrojando detalles en consola.

---

### Paso 6: Construcción del Repositorio de Red y Vinculación con el ViewModel

**Objetivo:** Implementar la lógica del repositorio que gestionará la fuente de datos remota e integrarlo en la UI reactiva usando StateFlow.

**Instrucciones:**

1. Define la interfaz del repositorio `TelemetryRepository.kt` en el paquete `com.example.advancedtracker.domain.repository`:

```kotlin
package com.example.advancedtracker.domain.repository

import com.example.advancedtracker.data.model.TelemetryDto
import com.example.advancedtracker.data.network.NetworkResult
import kotlinx.coroutines.flow.Flow

interface TelemetryRepository {
    suspend fun getTelemetryData(): NetworkResult<List<TelemetryDto>>
    suspend fun sendTelemetryData(telemetry: TelemetryDto): NetworkResult<TelemetryDto>
}
```

2. Implementa el repositorio en `TelemetryRepositoryImpl.kt` en el paquete `com.example.advancedtracker.data.repository`:

```kotlin
package com.example.advancedtracker.data.repository

import com.example.advancedtracker.data.model.TelemetryDto
import com.example.advancedtracker.data.network.NetworkResult
import com.example.advancedtracker.data.network.TelemetryApiService
import com.example.advancedtracker.data.network.safeApiCall
import com.example.advancedtracker.domain.repository.TelemetryRepository
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class TelemetryRepositoryImpl @Inject constructor(
    private val apiService: TelemetryApiService
) : TelemetryRepository {

    override suspend fun getTelemetryData(): NetworkResult<List<TelemetryDto>> {
        return safeApiCall { apiService.getTelemetryHistory() }
    }

    override suspend fun sendTelemetryData(telemetry: TelemetryDto): NetworkResult<TelemetryDto> {
        return safeApiCall { apiService.sendTelemetry(telemetry) }
    }
}
```

3. Actualiza el ViewModel (`TelemetryViewModel.kt` en `com.example.advancedtracker.ui.viewmodel`) para que gestione y exponga el estado de red a través de un `StateFlow`:

```kotlin
package com.example.advancedtracker.ui.viewmodel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.advancedtracker.data.model.TelemetryDto
import com.example.advancedtracker.data.network.NetworkResult
import com.example.advancedtracker.domain.repository.TelemetryRepository
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch
import javax.inject.Inject

@HiltViewModel
class TelemetryViewModel @Inject constructor(
    private val repository: TelemetryRepository
) : ViewModel() {

    private val _telemetryState = MutableStateFlow<NetworkResult<List<TelemetryDto>>>(NetworkResult.Loading)
    val telemetryState: StateFlow<NetworkResult<List<TelemetryDto>>> = _telemetryState.asStateFlow()

    init {
        fetchTelemetry()
    }

    fun fetchTelemetry() {
        viewModelScope.launch {
            _telemetryState.value = NetworkResult.Loading
            val result = repository.getTelemetryData()
            _telemetryState.value = result
        }
    }

    fun postTelemetry(telemetry: TelemetryDto) {
        viewModelScope.launch {
            _telemetryState.value = NetworkResult.Loading
            repository.sendTelemetryData(telemetry)
            // Actualizar la lista después de enviar un registro nuevo
            fetchTelemetry()
        }
    }
}
```

**Resultado Esperado:** Un flujo unidireccional de datos que expone las transiciones de estados (Loading -> Success/Error) de manera predecible para Jetpack Compose.

**Verificación:** Asegúrate de que el código compila perfectamente sin errores de ligadura de tipos en la arquitectura MVVM.

---

## Validación y Pruebas

Para garantizar el cumplimiento de los estándares de seguridad y resiliencia estipulados para Android 30 a 37, implementaremos pruebas de red automatizadas utilizando un servidor local ficticio (`MockWebServer`). Esto nos permitirá simular interacciones reales y controlar escenarios adversos.

### Creación de Pruebas Unitarias de Red Integradas

Crea la clase de prueba unitaria en la ruta del set de pruebas `app/src/test/java/com/example/advancedtracker/data/network/TelemetryApiTest.kt`:

```kotlin
package com.example.advancedtracker.data.network

import com.example.advancedtracker.data.model.TelemetryDto
import com.example.advancedtracker.data.repository.TelemetryRepositoryImpl
import com.example.advancedtracker.domain.repository.TelemetryRepository
import com.jakewharton.retrofit2.converter.kotlinx.serialization.asConverterFactory
import kotlinx.coroutines.runBlocking
import kotlinx.serialization.json.Json
import okhttp3.MediaType.Companion.toMediaType
import okhttp3.OkHttpClient
import okhttp3.mockwebserver.MockResponse
import okhttp3.mockwebserver.MockWebServer
import org.junit.After
import org.junit.Assert.*
import org.junit.Before
import org.junit.Test
import retrofit2.Retrofit
import java.util.concurrent.TimeUnit

class TelemetryApiTest {

    private lateinit var mockWebServer: MockWebServer
    private lateinit var repository: TelemetryRepository
    private val json = Json { ignoreUnknownKeys = true }

    @Before
    fun setUp() {
        mockWebServer = MockWebServer()
        mockWebServer.start()

        val okHttpClient = OkHttpClient.Builder()
            .connectTimeout(1, TimeUnit.SECONDS)
            .readTimeout(1, TimeUnit.SECONDS)
            .build()

        val retrofit = Retrofit.Builder()
            .baseUrl(mockWebServer.url("/"))
            .client(okHttpClient)
            .addConverterFactory(json.asConverterFactory("application/json".toMediaType()))
            .build()

        val apiService = retrofit.create(TelemetryApiService::class.java)
        repository = TelemetryRepositoryImpl(apiService)
    }

    @After
    fun tearDown() {
        mockWebServer.shutdown()
    }

    @Test
    fun `getTelemetryHistory returns success on 200 HTTP code`() = runBlocking {
        // GIVEN: El servidor Mock responde exitosamente con una lista JSON
        val mockResponse = MockResponse()
            .setResponseCode(200)
            .setBody("""
                [
                    {
                        "id": "tel_01",
                        "deviceId": "device_x",
                        "latitude": 19.4326,
                        "longitude": -99.1332,
                        "speed": 62.5,
                        "timestamp": 1700000000,
                        "status": "normal"
                    }
                ]
            """.trimIndent())
        mockWebServer.enqueue(mockResponse)

        // WHEN: Consultamos la telemetría a través del repositorio
        val result = repository.getTelemetryData()

        // THEN: El estado de retorno es Success y contiene la información deserializada
        assertTrue(result is NetworkResult.Success)
        val data = (result as NetworkResult.Success).data
        assertEquals(1, data.size)
        assertEquals("tel_01", data[0].id)
        assertEquals(19.4326, data[0].latitude, 0.0001)
    }

    @Test
    fun `getTelemetryHistory returns NetworkResult_Error on 500 Server Failure`() = runBlocking {
        // GIVEN: El servidor lanza un error interno
        val mockResponse = MockResponse()
            .setResponseCode(500)
            .setBody("Internal Server Error")
        mockWebServer.enqueue(mockResponse)

        // WHEN: Ejecutamos la petición
        val result = repository.getTelemetryData()

        // THEN: Se captura el error como una instancia estructurada NetworkResult.Error
        assertTrue(result is NetworkResult.Error)
        val error = result as NetworkResult.Error
        assertEquals(500, error.code)
    }
}
```

### Casos de Validación Adversarios (Casos Esquina)

#### Caso A: Ruta Inexistente (Error 404)
Al mapear respuestas HTTP mediante el interceptor y convertidor, el backend simulado podría responder con un recurso web inexistente.

*   **Evidencia Esperada:** El método `safeApiCall` captura la excepción `HttpException` con código `404`, generando un `NetworkResult.Error` que conserva los metadatos de diagnóstico pero no corrompe la estabilidad del hilo de renderizado de UI.

#### Caso B: Payload Corrupto o Inyección de Código Malicioso
Si el payload JSON devuelto por el servidor contiene datos maliciosos o claves nulas que no coinciden con la definición estricta de las clases del modelo, el sistema debe ser resiliente.

*   **Evidencia Esperada:** Gracias a la directiva `coerceInputValues = true` y `ignoreUnknownKeys = true` de nuestra instancia de `Json`, las propiedades corruptas no esperadas se ignoran, y los campos faltantes con valores por defecto en Kotlin se instancian en lugar de lanzar una excepción fatal de serialización.

---

## Solución de Problemas

A continuación, se documentan los dos errores más comunes que ocurren durante la adopción inicial de Retrofit 3.0.0-alpha y la suite OkHttp BOM avanzada:

### Problema 1: Excepción `IllegalArgumentException: Unable to create converter for class` en tiempo de ejecución
*   **Síntomas:** La aplicación se cierra al intentar instanciar o usar el objeto `TelemetryApiService`. Se lee en la traza de logs de Logcat: `IllegalArgumentException: Unable to create converter for com.example.advancedtracker.data.model.TelemetryDto`.
*   **Causa:** No se configuró correctamente la factoría de conversión de serialización al inicializar Retrofit, o falta la anotación `@Serializable` en la clase del modelo DTO que se recibe o se envía.
*   **Solución:** 
    1. Asegúrate de que las clases del DTO utilicen de forma explícita la anotación `@Serializable` al inicio del archivo.
    2. Revisa que en el módulo de inyección de Hilt (`NetworkModule`), la inicialización de Retrofit declare explícitamente el conversor de tipos usando:
    ```kotlin
    .addConverterFactory(json.asConverterFactory("application/json".toMediaType()))
    ```

### Problema 2: Bucles infinitos de llamadas HTTP de redirección o token vencido
*   **Síntomas:** El log del interceptor de red muestra peticiones infinitas que se disparan en ráfaga (High CPU Load), agotando el pool de conexiones.
*   **Causa:** El `AuthInterceptor` está inyectando cabeceras de seguridad de forma indiscriminada en la API, afectando también las rutas de inicio de sesión (`/auth/login`) o de refresco de tokens (`/auth/refresh`). Cuando el token de seguridad expira, estas rutas responden con código `401 Unauthorized`, lo que reinicia incorrectamente el ciclo de autenticación y reintenta la misma petición de manera indefinida.
*   **Solución:** Utiliza exclusiones explícitas basadas en la URL o el encabezado dentro de la lógica del interceptor. Valida la firma del endpoint de autenticación antes de adjuntar el encabezado Bearer, tal como se muestra a continuación:
    ```kotlin
    if (originalRequest.url.encodedPath.contains("/auth/login") || 
        originalRequest.url.encodedPath.contains("/auth/refresh")) {
        return chain.proceed(originalRequest)
    }
    ```

---

## Limpieza

Para prevenir colisiones con las clases generadas en la próxima práctica de la suite, realiza un saneamiento completo de los archivos compilados del proyecto:

1. Ejecuta la herramienta de limpieza de Gradle desde la consola integrada de Android Studio:
   ```bash
   ./gradlew clean
   ```
2. Desde el menú superior, haz clic en **File > Invalidate Caches...**, marca la opción "Clear file system cache and Local History" y presiona **Invalidate and Restart**.
3. Asegúrate de cerrar cualquier proceso de ejecución en background del emulador para liberar el socket local antes de proceder con el siguiente módulo formativo.

---

## Resumen

En este laboratorio, has implementado una arquitectura de red robusta, escalable y segura que aprovecha las capacidades de Retrofit 3.0.0-alpha y la suite OkHttp BOM 5.5.0-alpha:

- **Estructuración del Pipeline de Datos:** Configuraste con éxito el catálogo de dependencias `libs.versions.toml` evitando colisiones entre librerías.
- **Inyección de Token Dinámica:** Creaste un interceptor transparente (`AuthInterceptor`) que inyecta credenciales desde Jetpack DataStore a demanda, protegiendo las rutas privadas y eludiendo de manera eficiente la inyección recursiva en URLs públicas.
- **Manejo Resiliente de Red:** Diseñaste la estructura de `NetworkResult` para encapsular llamadas asíncronas y convertirlas en estados reactivos listos para consumirse con flujos unidireccionales (StateFlow) en vistas Compose.

### Recursos Adicionales
*   [Especificaciones oficiales de diseño de Retrofit](https://square.github.io/retrofit/)
*   [Guía oficial de Kotlinx Serialization](https://github.com/Kotlin/kotlinx.serialization)
*   [Técnicas profesionales para interceptores de OkHttp](https://square.github.io/okhttp/features/interceptors/)
