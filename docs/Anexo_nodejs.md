# Anexo: Creando un Servidor de Notificaciones con Node.js

En los pasos anteriores, preparamos nuestra aplicación Android para **recibir** notificaciones. Ahora, vamos a crear la otra mitad del sistema: un **servidor** que decida **cuándo y qué enviar**. Este es el verdadero cerebro detrás de las notificaciones push.

Usaremos **Node.js** para esta tarea, ya que nos permite crear un servidor simple y eficiente con muy poco código.

---

### ¿Qué es Node.js y por qué lo usamos?

Node.js es un entorno que nos permite ejecutar código JavaScript en un servidor, fuera del navegador. Es perfecto para este caso porque:

1.  **Es ligero y rápido.**
2.  **Maneja muy bien las tareas de red**, como hacer peticiones a la API de Firebase.
3.  Tiene un gestor de paquetes llamado **npm** que facilita enormemente la instalación de herramientas, como el SDK oficial de Firebase para servidores.

En resumen, con Node.js construiremos un pequeño programa que se conectará a Firebase y le dará la orden de enviar una notificación a un dispositivo específico.

---

### ¿Qué necesitarán los alumnos?

*   **Node.js instalado:** Deben tenerlo en su sistema. Se puede descargar desde [nodejs.org](https://nodejs.org/).
*   **El proyecto de Firebase:** El mismo que ya usaron para configurar la app de Android.

---

### Paso 1: Preparar el Entorno del Servidor

Nuestro servidor vivirá en una carpeta separada, fuera del proyecto de Android.

1.  **Crea una carpeta:** En un lugar fácil de encontrar (como el Escritorio o tu carpeta de Proyectos), crea una nueva carpeta llamada `mi-servidor-fcm`.

2.  **Abre la terminal en esa carpeta:** Navega con la terminal hasta la carpeta que acabas de crear.
    ```bash
    # Ejemplo si la creaste en el escritorio
    cd Desktop/mi-servidor-fcm
    ```

3.  **Inicializa el proyecto de Node.js:** Ejecuta el siguiente comando. Esto creará un archivo `package.json`, que es como el cerebro de un proyecto de Node.js.
    ```bash
    npm init -y
    ```

### Paso 2: Conectar el Servidor con Firebase

Ahora, le daremos a nuestro servidor las herramientas y las credenciales para hablar con Firebase.

1.  **Instala el Firebase Admin SDK:** En la misma terminal, ejecuta:
    ```bash
    npm install firebase-admin
    ```
    Esto descargará el código oficial de Google para controlar Firebase desde un servidor.

2.  **Obtén la Clave Privada del Servidor:**
    A diferencia de la app de Android que usa el `google-services.json`, un servidor necesita una **clave privada** para demostrar que tiene permisos de administrador.

    *   Ve a la **Consola de Firebase** -> **Configuración del proyecto** (el engranaje).
    *   Ve a la pestaña **"Cuentas de servicio"**.
    *   Haz clic en el botón **"Generar nueva clave privada"**.
    *   Se descargará un archivo JSON. **Renómbralo a `serviceAccountKey.json`**.
    *   **Mueve este archivo** a la carpeta de tu servidor (`mi-servidor-fcm`).

    > **¡ADVERTENCIA DE SEGURIDAD PARA LOS ALUMNOS!**
    > Esta clave es como la contraseña de administrador de tu proyecto de Firebase. ¡Es súper secreta! Nunca debe subirse a un repositorio público como GitHub. Si usan Git, deben añadir `serviceAccountKey.json` al archivo `.gitignore`.

### Paso 3: Crear el Script de Envío

Este es el archivo que contendrá la lógica para enviar el mensaje.

1.  Dentro de la carpeta `mi-servidor-fcm`, crea un archivo llamado `enviarNotificacion.js`.
2.  Pega el siguiente código en él:

```javascript
// Paso 1: Importar las herramientas necesarias
const admin = require("firebase-admin");
const serviceAccount = require("./serviceAccountKey.json");

// Paso 2: Conectar nuestro script con el proyecto de Firebase
admin.initializeApp({
  credential: admin.credential.cert(serviceAccount)
});

console.log("¡Conexión con Firebase establecida!");

// Paso 3: Definir a quién le enviaremos la notificación
// ¡¡¡AQUÍ PEGARÁS EL TOKEN DE TU APP!!!
const registrationToken = "REEMPLAZA_ESTO_CON_EL_TOKEN_DE_TU_DISPOSITIVO";

// Paso 4: Escribir el mensaje que queremos enviar
// Es muy importante usar la clave "data" para que la notificación
// sea procesada por nuestro servicio en segundo plano.
const message = {
  data: {
    title: "🚀 ¡Notificación desde el Servidor!",
    body: "¡Lo lograste! Esto fue enviado con Node.js."
  },
  token: registrationToken
};

// Paso 5: Enviar el mensaje usando el SDK de Firebase
console.log("Intentando enviar la notificación...");
admin.messaging().send(message)
  .then((response) => {
    // Si todo sale bien, Firebase nos devuelve un ID de mensaje
    console.log("Notificación enviada con éxito:", response);
  })
  .catch((error) => {
    // Si algo falla, veremos el error aquí
    console.log("Error al enviar la notificación:", error);
  });
```

### Paso 4: ¡Probando el Flujo Completo!

Este es el momento de la verdad, donde conectamos todo el trabajo.

1.  **Ejecuta la app `EnvioNotificaciones`** en tu emulador o dispositivo físico.
2.  Pulsa el botón **"Obtener y Copiar Token"**. El token de tu dispositivo ya está en el portapapeles.
3.  **Pon la app en segundo plano** (ve al inicio) o ciérrala por completo. Esto es crucial para probar que la notificación llega cuando la app no está activa.
4.  **Abre el archivo `enviarNotificacion.js`** en tu editor de código (como Visual Studio Code).
5.  **Pega el token** que copiaste de la app, reemplazando el texto `"REEMPLAZA_ESTO_CON_EL_TOKEN_DE_TU_DISPOSITIVO"`.
6.  Guarda el archivo `enviarNotificacion.js`.
7.  Vuelve a tu **terminal**, que debe estar en la carpeta `mi-servidor-fcm`.
8.  Ejecuta el script con el siguiente comando:
    ```bash
    node enviarNotificacion.js
    ```

Si todo ha ido bien, verás en la terminal un mensaje de éxito y, lo más importante, **¡la notificación deberá aparecer en tu dispositivo o emulador!**

Este script básico es el punto de partida. En una aplicación real, este código se ejecutaría no manualmente, sino en respuesta a un evento, como un nuevo mensaje en una base de datos o una petición a una API.
