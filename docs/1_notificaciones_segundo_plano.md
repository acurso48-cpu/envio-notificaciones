# Guía para Enviar Notificaciones Push en Segundo Plano

Las notificaciones push son una herramienta fundamental para mantener a los usuarios involucrados (engagement) con tu aplicación. Poder enviar notificaciones incluso cuando la aplicación no está en uso (en segundo plano o cerrada) es crucial. Firebase Cloud Messaging (FCM) es el servicio estándar y gratuito para gestionar esto tanto en Android como en iOS.

Aquí te explicamos cómo funcionan y las dos arquitecturas principales para enviarlas.

---

## Arquitectura Básica de Notificaciones Push

El flujo siempre involucra tres componentes:

1.  **La Aplicación Cliente (Tu app Android/iOS):** Es responsable de obtener un "token de registro" único de FCM y enviarlo a tu backend para almacenarlo.
2.  **El Backend de FCM:** Los servidores de Google que reciben la solicitud de envío y se encargan de entregar el mensaje al dispositivo correcto.
3.  **Tu Servidor de Aplicación:** Es el cerebro que decide cuándo, a quién y qué notificación enviar. Este puede ser la propia consola de Firebase, una Cloud Function o un servidor completamente personalizado.

El proceso general es:

*   Tu app obtiene un token de FCM y lo registra en tu servidor.
*   Un evento ocurre (ej: nuevo mensaje en un chat, una oferta especial) y tu servidor decide enviar una notificación.
*   Tu servidor hace una petición a la API de FCM, indicando el token del dispositivo de destino y el contenido del mensaje.
*   FCM se encarga de la entrega al dispositivo.

---

## Opción 1: Enviar desde la Consola de Firebase o Cloud Functions

Este es el enfoque más simple y es ideal para empezar, para campañas de marketing o para lógica que vive enteramente dentro del ecosistema de Google.

### ¿Cómo funciona?

*   **Consola de Firebase:** A través de la sección "Cloud Messaging" o "In-App Messaging" en la consola de Firebase, puedes redactar notificaciones y enviarlas a todos los usuarios de tu app, a segmentos específicos (basados en Analytics) o a un dispositivo de prueba usando su token.
*   **Cloud Functions:** Puedes escribir funciones (en Node.js, Python, Go, etc.) que se ejecutan en los servidores de Google y se disparan por eventos. Por ejemplo, puedes tener una función que se ejecute cada vez que se crea un nuevo documento en una colección de Firestore y envíe una notificación al usuario correspondiente.

### Pros:
*   **Simplicidad:** No necesitas gestionar tu propia infraestructura de servidor.
*   **Integración:** Totalmente integrado con otros servicios de Firebase como Analytics, Firestore y Authentication.
*   **Costo:** Generoso plan gratuito para empezar.

### Contras:
*   **Menos Flexibilidad:** Estás limitado a la lógica que puedes ejecutar en Cloud Functions o a los envíos manuales desde la consola. No es ideal si la lógica de notificación depende de una base de datos o servicios externos a Google.

---

## Opción 2: Enviar desde un Servidor Propio

Este es el enfoque más potente y flexible, utilizado por la mayoría de las aplicaciones a escala. Te da control total sobre la lógica de envío de notificaciones.

### ¿Cómo funciona?

1.  **Obtener y Guardar el Token:** Tu aplicación Android/iOS, al iniciarse, solicita el token de FCM. Una vez recibido, lo envía a tu servidor (a través de una API REST que tú creas) y lo almacenas en tu base de datos, asociándolo a un usuario específico.

2.  **Usar el Admin SDK de Firebase:** En tu backend (escrito en Node.js, Java, Python, C#, etc.), integras el **Firebase Admin SDK**. Este SDK te permite autenticarte con tus credenciales de servidor de Firebase.

3.  **Construir y Enviar el Mensaje:** Cuando tu lógica de negocio lo requiera, usas el Admin SDK para construir el objeto del mensaje. Puedes especificar el token del dispositivo de destino, el título, el cuerpo y, lo más importante, un **payload de datos**.

    *Ejemplo de envío simple con el Admin SDK en Node.js:*
    ```javascript
    // Importar el Admin SDK
    const admin = require('firebase-admin');

    // Lógica para inicializar la app con tus credenciales...

    const registrationToken = 'EL_TOKEN_QUE_GUARDASTE_DEL_DISPOSITIVO';

    const message = {
      notification: {
        title: '¡Nueva Oferta!',
        body: 'Aprovecha un 20% de descuento en tu próxima compra.'
      },
      token: registrationToken
    };

    // Enviar el mensaje
    admin.messaging().send(message)
      .then((response) => {
        console.log('Mensaje enviado con éxito:', response);
      })
      .catch((error) => {
        console.log('Error enviando el mensaje:', error);
      });
    ```

### Pros:
*   **Control Total:** Puedes implementar cualquier lógica de negocio, por compleja que sea. Puedes consultar tus propias bases de datos, conectarte a APIs de terceros, etc.
*   **Escalabilidad:** La arquitectura puede crecer junto con tu aplicación.

### Contras:
*   **Más Complejidad:** Requiere que desarrolles y mantengas un backend, así como la API para que los clientes registren sus tokens.
*   **Costo de Infraestructura:** Eres responsable de los costos asociados a tu servidor.

---

## Recomendaciones Clave para Android

1.  **Usa Mensajes de Datos para Tareas en Segundo Plano.**
    FCM tiene dos tipos de mensajes: **Notificación** y **Datos**.
    *   **Mensajes de Notificación:** Si la app está en segundo plano, el sistema operativo se encarga de mostrar la notificación automáticamente en la bandeja del sistema. Tu código en `onMessageReceived` **NO se ejecuta**.
    *   **Mensajes de Datos (Recomendado):** Estos mensajes **SIEMPRE** se entregan al método `onMessageReceived` de tu `FirebaseMessagingService`, sin importar si la app está en primer o segundo plano. Esto te da control total. Desde ahí, puedes leer el payload de datos y crear una notificación personalizada, ejecutar una tarea de sincronización o lo que necesites.

    *Payload Mixto (Notificación + Datos):* Si envías ambos, en segundo plano se muestra la notificación y los datos solo se entregan si el usuario pulsa sobre ella. Por eso, para lógica de fondo, **usa solo mensajes de datos**.

2.  **Gestiona el Ciclo de Vida de los Tokens.**
    El token de FCM puede cambiar (por reinstalación de la app, restauración, etc.). Debes sobrescribir el método `onNewToken` en tu `FirebaseMessagingService` para detectar cuándo se genera un nuevo token y enviarlo a tu servidor para mantenerlo actualizado.

3.  **Crea Canales de Notificación (Android 8.0+).**
    A partir de Android Oreo, todas las notificaciones deben asignarse a un canal. Esto permite a los usuarios gestionar qué tipos de notificaciones quieren recibir de tu app. Define tus canales la primera vez que se inicie la aplicación.

4.  **Utiliza la Prioridad Correcta.**
    Puedes establecer una prioridad a tus mensajes (`high` o `normal`). Las notificaciones de alta prioridad están diseñadas para despertar a los dispositivos en modo de bajo consumo (Doze mode), pero su uso excesivo puede llevar a que el sistema las degrade. Úsalas solo para alertas importantes y urgentes.

5.  **Aprovecha la Suscripción a Temas (Topics).**
    Si quieres enviar una notificación a un gran grupo de usuarios (ej: todos los interesados en "deportes"), no envíes un mensaje individual a cada token. En su lugar, haz que los clientes se suscriban a un tema (`Firebase.messaging.subscribeToTopic("sports")`) y luego envía un único mensaje a ese tema. Es mucho más eficiente.

6.  **Seguridad:** Nunca expongas tu clave de servidor de Firebase en el lado del cliente. Toda la lógica de envío debe residir en un entorno seguro (tu servidor o Cloud Functions).
