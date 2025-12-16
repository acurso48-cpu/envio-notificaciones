# Guía: Crear una VM Local con Ubuntu y Hestia CP en macOS

Esta guía te mostrará cómo crear un entorno de servidor local completo en tu Mac. Configuraremos una máquina virtual (VM) con Ubuntu Server y le instalaremos Hestia Control Panel. Esto es ideal para desarrolladores que desean un entorno de pruebas de hosting aislado y potente sin costo alguno.

---

### Herramientas Necesarias

1.  **Software de Virtualización:** Necesitas un programa para ejecutar máquinas virtuales.
    *   **Para Macs con procesador Intel:** [VirtualBox](https://www.virtualbox.org/wiki/Downloads) es una opción gratuita y muy popular.
    *   **Para Macs con procesador Apple Silicon (M1/M2/M3):** [UTM](https://mac.getutm.app/) es una excelente opción gratuita y de código abierto que utiliza la tecnología QEMU. La puedes descargar desde su sitio web o comprarla en la App Store para apoyar al desarrollador.

2.  **Imagen de Ubuntu Server:** Descarga la última versión LTS (Long-Term Support) de [Ubuntu Server](https://ubuntu.com/download/server). Asegúrate de descargar la versión correcta para tu Mac:
    *   **Intel Mac:** Elige la versión `x86_64` o `AMD64`.
    *   **Apple Silicon Mac:** Elige la versión `ARM64`.

--- 

### Paso 1: Crear y Configurar la Máquina Virtual

(Las instrucciones se centrarán en **VirtualBox**, pero los conceptos son transferibles a UTM).

1.  **Abre VirtualBox** y haz clic en **"Nueva"**.
2.  **Nombre y Sistema Operativo:**
    *   **Nombre:** Dale un nombre descriptivo, como `Hestia-Dev-Server`.
    *   **Imagen ISO:** Selecciona la imagen de Ubuntu Server que descargaste.
    *   VirtualBox debería detectar el sistema operativo automáticamente.
3.  **Hardware:**
    *   **Memoria base (RAM):** Asigna al menos **2048 MB** (2 GB). Si tu Mac tiene mucha RAM, 4096 MB (4 GB) es ideal.
    *   **Procesadores (CPU):** Asigna al menos **2 CPUs**.
4.  **Disco Duro Virtual:**
    *   Selecciona **"Crear un Disco Duro Virtual Ahora"**.
    *   Asigna un tamaño de al menos **25 GB**.
    *   Puedes dejar el tipo de disco y el almacenamiento dinámico como están por defecto.
5.  **Revisar y Terminar:** Revisa el resumen y haz clic en "Terminar".

### Paso 2: Ajustar la Configuración de Red (¡Muy Importante!)

Antes de iniciar la VM, necesitamos que pueda comunicarse con tu red local.

1.  Selecciona tu nueva VM en la lista de VirtualBox y haz clic en **"Configuración"**.
2.  Ve a la pestaña **"Red"**.
3.  En **"Conectado a:"**, cambia `NAT` por **`Adaptador Puente (Bridged Adapter)`**.
4.  En **"Nombre"**, asegúrate de que esté seleccionado tu adaptador de red principal (ej: `en0: Wi-Fi`).

    *¿Por qué hacemos esto?* El modo `Adaptador Puente` hace que tu VM se comporte como un dispositivo físico más en tu red local. Obtendrá su propia dirección IP de tu router, y podrás acceder a ella desde tu Mac u otros dispositivos en la misma red.

### Paso 3: Instalar Ubuntu Server

1.  Inicia tu VM. Debería arrancar automáticamente desde la imagen ISO de Ubuntu.
2.  Sigue el asistente de instalación:
    *   **Idioma y teclado:** Elige tus preferencias.
    *   **Red:** Debería detectar tu red y obtener una dirección IP automáticamente gracias al adaptador puente.
    *   **Proxy:** Déjalo en blanco (a menos que uses uno).
    *   **Particionado de disco:** Usa la opción guiada por defecto ("Use an entire disk").
    *   **Configuración del perfil:** Define tu nombre, el nombre del servidor y un nombre de usuario y contraseña. Recuerda estos datos.
    *   **Instalar Servidor OpenSSH:** **¡MUY IMPORTANTE!** Cuando te pregunte, asegúrate de marcar la casilla **`Install OpenSSH server`**. Esto es esencial para poder conectarte a la VM desde la terminal de tu Mac.
    *   **Snaps populares:** No necesitas seleccionar ninguno.
3.  La instalación comenzará. Una vez que termine, te pedirá que reinicies. La VM se reiniciará.

### Paso 4: Conectarse a la VM vía SSH

1.  Una vez que la VM se haya reiniciado, verás una pantalla de login de terminal. No necesitas hacer login aquí.
2.  Para encontrar la dirección IP de tu VM, puedes iniciar sesión una vez en la consola de VirtualBox y escribir el comando:
    ```bash
    ip a
    ```
    Busca la dirección IP bajo la interfaz `enp0s3` (o similar). Será algo como `192.168.1.XX`.

3.  Abre la aplicación **Terminal** en tu Mac.
4.  Conéctate a la VM usando el comando `ssh`. Reemplaza `<usuario>` y `<ip_de_la_vm>` con los datos que definiste durante la instalación.
    ```bash
    ssh <usuario>@<ip_de_la_vm>
    ```

¡Ya estás dentro de tu servidor Ubuntu local!

### Paso 5: Instalar Hestia Control Panel

Ahora, dentro de la sesión SSH, ejecuta los siguientes comandos.

1.  **Actualiza el sistema:**
    ```bash
    sudo apt update && sudo apt upgrade -y
    ```
2.  **Descarga el script de instalación de Hestia:**
    ```bash
    wget https://raw.githubusercontent.com/hestiacp/hestiacp/release/install/hst-install.sh
    ```
3.  **Ejecuta el script de instalación:**
    ```bash
    sudo bash hst-install.sh
    ```
    Sigue las instrucciones del asistente. Te pedirá que confirmes la instalación de los componentes, que introduzcas un email de administrador y un nombre de host (hostname). Para un entorno local, puedes usar algo como `srv1.local`.

4.  El proceso tardará unos minutos. Al finalizar, te mostrará la URL de acceso, el usuario (`admin`) y una contraseña. **Guarda estos datos.**

### Paso 6: Acceder a Hestia CP desde tu Mac

1.  Abre un navegador web en tu Mac (Safari, Chrome, etc.).
2.  Navega a la dirección IP de tu VM en el puerto 8083:
    `https://<ip_de_la_vm>:8083`

3.  Verás una advertencia de seguridad por el certificado autofirmado. Procede a la página.
4.  Inicia sesión con el usuario `admin` y la contraseña que te proporcionó el instalador.

¡Listo! Ahora tienes un completo panel de control de hosting ejecutándose localmente en tu Mac, perfecto para desarrollar y probar sitios web antes de subirlos a un servidor de producción.
