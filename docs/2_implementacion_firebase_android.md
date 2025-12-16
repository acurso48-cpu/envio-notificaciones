# Tutorial: Implementar Notificaciones Push con Firebase en Android

Esta guía te mostrará cómo integrar Firebase Cloud Messaging (FCM) en tu aplicación Android para enviar y recibir notificaciones push, con un enfoque especial en la gestión de mensajes cuando la aplicación está en segundo plano o cerrada.

---

### Paso 1: Configurar tu Proyecto en la Consola de Firebase

1.  **Ve a la [Consola de Firebase](https://console.firebase.google.com/)** y crea un nuevo proyecto (o selecciona uno existente).
2.  Dentro de tu proyecto, haz clic en el ícono de Android para añadir una nueva aplicación.
3.  **Registra tu app:**
    *   **Nombre del paquete de Android:** Búscalo en tu archivo `app/build.gradle.kts` bajo `applicationId`. Suele ser algo como `com.kuvuni.envionotificaciones`.
    *   **Apodo de la app (opcional):** Un nombre fácil de recordar, como "Envio Notificaciones App".
    *   **Certificado de firma de depuración SHA-1 (opcional pero recomendado):** Esto es necesario para servicios como Google Sign-In, pero también es una buena práctica. Puedes obtenerlo en Android Studio abriendo la pestaña "Gradle" a la derecha, navegando a **app -> Tasks -> android** y ejecutando `signingReport`.
4.  **Descarga el archivo de configuración:** Descarga el archivo `google-services.json` que te proporciona Firebase.

---

### Paso 2: Configurar el Proyecto de Android

1.  **Mueve el archivo `google-services.json`** que acabas de descargar a la carpeta raíz de tu módulo `app` (`app/`).

2.  **Añade las dependencias de Gradle:** Abre tus archivos `build.gradle.kts` y asegúrate de que contengan las siguientes configuraciones.

    *   **Archivo `build.gradle.kts` a nivel de proyecto (`/build.gradle.kts`):**
        ```kotlin
        // Top-level build file where you can add configuration options common to all sub-projects/modules.
        plugins {
            id("com.android.application") version "8.2.0" apply false
            id("org.jetbrains.kotlin.android") version "1.9.0" apply false
            // Asegúrate de que este plugin esté aquí
            id("com.google.gms.google-services") version "4.4.1" apply false
        }
        ```

    *   **Archivo `build.gradle.kts` a nivel de app (`/app/build.gradle.kts`):**
        ```kotlin
        plugins {
            id("com.android.application")
            id("org.jetbrains.kotlin.android")
            // Añade este plugin al principio
            id("com.google.gms.google-services")
        }

        dependencies {
            // ... otras dependencias

            // Firebase BoM (Bill of Materials) - Gestiona las versiones de las librerías de Firebase
            implementation(platform("com.google.firebase:firebase-bom:32.7.4"))

            // Dependencia de Firebase Cloud Messaging
            implementation("com.google.firebase:firebase-messaging-ktx")

            // Dependencia de Analytics (recomendada)
            implementation("com.google.firebase:firebase-analytics-ktx")
        }
        ```

3.  **Sincroniza tu proyecto con Gradle** haciendo clic en "Sync Now".

---

### Paso 3: Crear el Servicio para Recibir Mensajes

Crea un nuevo archivo Kotlin llamado `MyFirebaseMessagingService.kt`. Este servicio se ejecutará en segundo plano para escuchar los mensajes entrantes de FCM.

```kotlin
package com.kuvuni.envionotificaciones

import android.app.NotificationChannel
import android.app.NotificationManager
import android.content.Context
import android.os.Build
import android.util.Log
import androidx.core.app.NotificationCompat
import com.google.firebase.messaging.FirebaseMessagingService
import com.google.firebase.messaging.RemoteMessage

class MyFirebaseMessagingService : FirebaseMessagingService() {

    // Se llama cuando la app recibe un mensaje, sin importar si está en primer o segundo plano.
    // Esto funciona gracias a que usamos mensajes de "Datos" (data messages).
    override fun onMessageReceived(remoteMessage: RemoteMessage) {
        super.onMessageReceived(remoteMessage)

        Log.d(TAG, "From: ${remoteMessage.from}")

        // Los mensajes de notificación solo se reciben aquí si la app está en primer plano.
        // Para un control total en segundo plano, usaremos mensajes de datos.
        remoteMessage.notification?.let {
            Log.d(TAG, "Notification Message Body: ${it.body}")
        }

        // Comprobar si el mensaje contiene un payload de datos.
        remoteMessage.data.isNotEmpty().let {
            Log.d(TAG, "Data Message payload: " + remoteMessage.data)
            // Aquí procesamos los datos y creamos nuestra propia notificación.
            showNotification(remoteMessage.data["title"], remoteMessage.data["body"])
        }
    }

    // Se llama cuando se genera un nuevo token de registro o cuando el existente cambia.
    override fun onNewToken(token: String) {
        super.onNewToken(token)
        Log.d(TAG, "Refreshed token: $token")

        // TODO: Enviar este token a tu servidor para poder enviar notificaciones
        // a este dispositivo específico.
        sendTokenToServer(token)
    }

    private fun sendTokenToServer(token: String) {
        // Lógica para enviar el token a tu backend (API REST, etc.)
    }

    private fun showNotification(title: String?, body: String?) {
        val channelId = "default_channel_id"
        val notificationManager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager

        // Crear canal de notificación para Android 8.0+
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(channelId, "Default Channel", NotificationManager.IMPORTANCE_DEFAULT)
            notificationManager.createNotificationChannel(channel)
        }

        val notificationBuilder = NotificationCompat.Builder(this, channelId)
            .setSmallIcon(R.drawable.ic_launcher_foreground) // Asegúrate de tener este drawable
            .setContentTitle(title)
            .setContentText(body)
            .setAutoCancel(true) // La notificación se cierra al pulsarla
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)

        // Usamos un ID único para cada notificación para que no se sobreescriban
        notificationManager.notify(System.currentTimeMillis().toInt(), notificationBuilder.build())
    }

    companion object {
        private const val TAG = "MyFirebaseMsgService"
    }
}
```

### Paso 4: Declarar el Servicio en `AndroidManifest.xml`

Abre tu `AndroidManifest.xml` (en `app/src/main`) y añade el servicio dentro de la etiqueta `<application>`.

```xml
<application ...>

    <activity ...>... </activity>

    <!-- Declara tu servicio de Firebase -->
    <service
        android:name=".MyFirebaseMessagingService"
        android:exported="false">
        <intent-filter>
            <action android:name="com.google.firebase.MESSAGING_EVENT" />
        </intent-filter>
    </service>

</application>
```

---

### Paso 5: Lógica y UI en la App

#### Permiso de Notificaciones (Android 13+)

Necesitas solicitar permiso explícitamente para mostrar notificaciones.

**1. `AndroidManifest.xml`:**
```xml
<manifest ...>
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
    ...
</manifest>
```

**2. `MainActivity.kt`:**
Modifica tu `MainActivity.kt` para solicitar el permiso y para obtener y mostrar el token de FCM (muy útil para pruebas).

```kotlin
package com.kuvuni.envionotificaciones

import android.Manifest
import android.content.pm.PackageManager
import android.os.Build
import android.os.Bundle
import android.util.Log
import android.widget.Button
import android.widget.Toast
import androidx.activity.result.contract.ActivityResultContracts
import androidx.appcompat.app.AppCompatActivity
import androidx.core.content.ContextCompat
import com.google.firebase.messaging.FirebaseMessaging

class MainActivity : AppCompatActivity() {

    private val requestPermissionLauncher = registerForActivityResult(
        ActivityResultContracts.RequestPermission()
    ) { isGranted: Boolean ->
        if (isGranted) {
            Toast.makeText(this, "Permiso de notificaciones concedido", Toast.LENGTH_SHORT).show()
        } else {
            Toast.makeText(this, "Permiso de notificaciones denegado", Toast.LENGTH_SHORT).show()
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        askForNotificationPermission()

        val getTokenButton: Button = findViewById(R.id.button_get_token)
        getTokenButton.setOnClickListener {
            FirebaseMessaging.getInstance().token.addOnCompleteListener { task ->
                if (!task.isSuccessful) {
                    Log.w(TAG, "Fetching FCM registration token failed", task.exception)
                    return@addOnCompleteListener
                }

                // Obtener nuevo token de registro de FCM
                val token = task.result

                // Log y mostrarlo
                Log.d(TAG, "FCM Token: $token")
                Toast.makeText(baseContext, "FCM Token: $token", Toast.LENGTH_LONG).show()
            }
        }
    }

    private fun askForNotificationPermission() {
        // Solo se aplica a Android 13 (API 33) o superior
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            when {
                ContextCompat.checkSelfPermission(
                    this,
                    Manifest.permission.POST_NOTIFICATIONS
                ) == PackageManager.PERMISSION_GRANTED -> {
                    // Permiso ya concedido
                }
                shouldShowRequestPermissionRationale(Manifest.permission.POST_NOTIFICATIONS) -> {
                    // Muestra una UI explicando por qué necesitas el permiso (opcional)
                    // y luego solicita el permiso.
                    requestPermissionLauncher.launch(Manifest.permission.POST_NOTIFICATIONS)
                }
                else -> {
                    // Solicita directamente el permiso
                    requestPermissionLauncher.launch(Manifest.permission.POST_NOTIFICATIONS)
                }
            }
        }
    }

    companion object {
        private const val TAG = "MainActivity"
    }
}
```

**3. `activity_main.xml`:**
Añade un botón a tu layout para poder obtener el token fácilmente.

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/button_get_token"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Obtener Token FCM"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

---

### Paso 6: Probar las Notificaciones en Segundo Plano

Para que `onMessageReceived` se ejecute en segundo plano, debes enviar un mensaje de **"Datos"**.

1.  Ejecuta tu aplicación. Concede el permiso y pulsa el botón "Obtener Token FCM". Copia el token que aparece en el Logcat.
2.  Pon la aplicación en segundo plano o ciérrala.
3.  En Firebase, abre tu proyecto. En Messaging, crea una campaña y prueba (tienes que poner el token del Logcat.
    


Si todo está configurado correctamente, deberías ver aparecer una notificación en tu dispositivo, creada por el código que escribiste en `showNotification()`.

---

### Mejoras de Arquitectura

La implementación anterior es funcional, pero en una aplicación real, se pueden hacer varias mejoras:

1.  **ViewModel y Coroutines:** Mueve la lógica para obtener el token de la `MainActivity` a un `ViewModel`. Usa coroutines de Kotlin para gestionar las llamadas asíncronas de forma más limpia.

2.  **Repositorio:** Crea una clase `Repository` que sea la única fuente de verdad para los datos. El `ViewModel` le pediría el token al repositorio, y el `MyFirebaseMessagingService` usaría este mismo repositorio para enviar el token actualizado al servidor. Esto centraliza la lógica de red.

3.  **Inyección de Dependencias (Hilt/Koin):** En lugar de instanciar clases directamente, usa un framework como Hilt para "inyectar" las dependencias (como el `Repository`) en tu `Activity`, `ViewModel` y `Service`. Esto facilita las pruebas y desacopla tu código.

4.  **Modelo de Datos:** Define clases `data class` para representar el payload de tus notificaciones. Usa una librería como `Gson` o `Moshi` para deserializar los datos de `remoteMessage.data` a tus objetos de Kotlin de forma segura.
