# Manual: Implementación de Notificaciones con Firebase Paso a Paso

Este documento detalla todos los pasos realizados para integrar Firebase Cloud Messaging (FCM) en el proyecto, con el objetivo de habilitar notificaciones push que funcionen en segundo plano.

---

### Paso 1: Configuración del Entorno de Firebase y Gradle

El primer paso fue preparar el proyecto para que pudiera comunicarse con los servicios de Firebase.

**1.1. Plugin de Google Services (Nivel de Proyecto):**
Se añadió el plugin de Google Services al archivo `build.gradle.kts` raíz. Este plugin es esencial para que Gradle pueda procesar el archivo `google-services.json` y configurar la conexión con Firebase.

*   **Archivo:** `build.gradle.kts`
*   **Código añadido:**
    ```kotlin
    plugins {
        // ...
        id("com.google.gms.google-services") version "4.4.1" apply false
    }
    ```

**1.2. Dependencias y Plugin de App:**
Se configuró el archivo `build.gradle.kts` del módulo `app`:
*   Se aplicó el plugin `com.google.gms.google-services`.
*   Se añadió la **Firebase Bill of Materials (BoM)**, que gestiona las versiones de todas las dependencias de Firebase para asegurar la compatibilidad entre ellas.
*   Se incluyeron las dependencias específicas para **Firebase Cloud Messaging (`firebase-messaging-ktx`)** y **Firebase Analytics (`firebase-analytics-ktx`)**.
*   Se activó **`viewBinding`** para facilitar la interacción con la UI de forma segura.

*   **Archivo:** `app/build.gradle.kts`
*   **Cambios clave:**
    ```kotlin
    plugins {
        id("com.google.gms.google-services")
        // ...
    }

    android {
        // ...
        buildFeatures {
            viewBinding = true
        }
    }

    dependencies {
        // ...
        implementation(platform("com.google.firebase:firebase-bom:32.7.4"))
        implementation("com.google.firebase:firebase-messaging-ktx")
        implementation("com.google.firebase:firebase-analytics-ktx")
        // ...
    }
    ```

---

### Paso 2: Creación del Servicio de Mensajería

El componente principal para recibir notificaciones es un `Service` que hereda de `FirebaseMessagingService`.

*   **Archivo Creado:** `app/src/main/java/com/kuvuni/envionotificaciones/MyFirebaseMessagingService.kt`

**Funcionalidad Clave:**
1.  **`onMessageReceived(remoteMessage)`:** Este método es el corazón del servicio. Se activa **siempre** que llega un mensaje de FCM con un **payload de "datos"**, sin importar si la app está en primer o segundo plano. La lógica implementada extrae el `title` y el `body` de los datos y llama a `showNotification()` para mostrar una notificación local.
2.  **`onNewToken(token)`:** Se dispara cuando se genera por primera vez o se actualiza el token de registro del dispositivo. Aquí es donde se debería añadir la lógica para enviar este token al backend de tu aplicación.
3.  **`showNotification(title, body)`:** Un método privado que se encarga de construir y mostrar una notificación en el dispositivo, creando un `NotificationChannel` si es necesario (para Android 8.0+).

---

### Paso 3: Actualización del `AndroidManifest.xml`

Se realizaron cambios cruciales en el manifiesto para registrar los nuevos componentes y solicitar los permisos necesarios.

*   **Archivo:** `app/src/main/AndroidManifest.xml`

**Cambios Realizados:**
1.  **Permiso de Notificaciones:** Se añadió el permiso `POST_NOTIFICATIONS`, obligatorio para mostrar notificaciones en Android 13 (API 33) y versiones superiores.
    ```xml
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
    ```
2.  **Registro del Servicio:** Se declaró el `MyFirebaseMessagingService` para que el sistema Android lo reconozca y pueda entregarle los eventos de mensajería de Firebase.
    ```xml
    <service
        android:name=".MyFirebaseMessagingService"
        android:exported="false">
        <intent-filter>
            <action android:name="com.google.firebase.MESSAGING_EVENT" />
        </intent-filter>
    </service>
    ```
3.  **Icono de Notificación por Defecto:** Se creó un icono vectorial simple (`ic_notification.xml`) y se estableció como el icono por defecto para las notificaciones de Firebase a través de metadatos, asegurando que siempre haya un icono visible.
    ```xml
    <meta-data
        android:name="com.google.firebase.messaging.default_notification_icon"
        android:resource="@drawable/ic_notification" />
    ```

---

### Paso 4: Lógica de la `MainActivity` y UI

Se actualizó la `MainActivity` y su layout para gestionar los permisos y facilitar las pruebas.

**4.1. `MainActivity.kt`:**
*   **ViewBinding:** Se implementó `ViewBinding` para acceder a las vistas del layout de forma segura.
*   **Solicitud de Permisos:** Se utilizó `registerForActivityResult` para crear un lanzador de solicitud de permisos. La función `askForNotificationPermission()` comprueba si la app se ejecuta en Android 13+ y si el permiso ya ha sido concedido; si no, lo solicita.
*   **Obtención de Token:** La función `getToken()` utiliza `FirebaseMessaging.getInstance().token` para obtener el token de registro del dispositivo. Este token se muestra en el Logcat, en un `Toast`, y se copia automáticamente al portapapeles para facilitar su uso en herramientas de prueba como Postman.

**4.2. `activity_main.xml`:**
*   Se simplificó el layout a un `LinearLayout`.
*   Se añadió un único `Button` (`@+id/button_get_token`) que, al ser pulsado, ejecuta la función `getToken()`.

---

### Resumen del Flujo y Próximos Pasos

1.  **Al iniciar la app**, se solicita al usuario el permiso para mostrar notificaciones.
2.  El usuario puede **pulsar el botón "Obtener y Copiar Token"**. Esto mostrará el token de FCM y lo copiará al portapapeles.
3.  Para **probar una notificación en segundo plano**, la app debe estar cerrada o en segundo plano. Luego, se debe enviar una petición POST a `https://fcm.googleapis.com/fcm/send` con los encabezados de autorización correctos y un cuerpo JSON que contenga el token del dispositivo y un payload de `data` (no de `notification`).

    ```json
    {
      "to": "EL_TOKEN_COPIADO_DEL_DISPOSITIVO",
      "data": {
        "title": "Prueba desde Servidor",
        "body": "¡Esto funciona en segundo plano!"
      }
    }
    ```
4.  El `MyFirebaseMessagingService` recibirá este payload, `onMessageReceived` se ejecutará y el método `showNotification` creará y mostrará la notificación en el dispositivo.
