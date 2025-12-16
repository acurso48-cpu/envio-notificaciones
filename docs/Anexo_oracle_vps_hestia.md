# Tutorial: Crear una VPS Gratuita en Oracle Cloud para Instalar Hestia CP

Este documento es una guía paso a paso para configurar una máquina virtual (VPS) en la capa gratuita de Oracle Cloud (OCI), instalar la última versión de Ubuntu y prepararla para la instalación de Hestia Control Panel, un potente panel de control de hosting de código abierto.

La oferta "Always Free" de Oracle es ideal para este propósito, ya que proporciona recursos generosos (especialmente las instancias ARM) que son más que suficientes para alojar múltiples sitios web ligeros.

---

### Requisitos Previos

*   **Cuenta de Oracle Cloud:** Necesitas haberte registrado en [Oracle Cloud Free Tier](https://www.oracle.com/cloud/free/).
*   **Tarjeta de Crédito:** Oracle requiere una tarjeta de crédito para la verificación de identidad. No se te cobrará nada si te mantienes dentro de los límites de la capa gratuita.
*   **Un par de claves SSH:** OCI utiliza claves SSH para la autenticación segura en lugar de contraseñas. Si no tienes una, el propio proceso de creación te permite generar y descargar una.

---

### Paso 1: Navegar a la Creación de Instancias

1.  Inicia sesión en tu consola de Oracle Cloud.
2.  Haz clic en el menú de hamburguesa (tres líneas) en la esquina superior izquierda.
3.  Navega a **Compute -> Instances**.

### Paso 2: Iniciar la Creación de la Instancia

1.  Asegúrate de estar en un compartimento que desees usar (el compartimento `root` está bien para empezar).
2.  Haz clic en el botón **"Create instance"**.

### Paso 3: Configurar la Imagen y la Forma (La parte más importante)

Aquí es donde definimos el hardware y el sistema operativo de nuestra VM.

1.  **Name:** Dale un nombre a tu instancia, por ejemplo, `hestia-server`.

2.  **Image and shape:** Haz clic en **"Edit"**.
    *   **Image:** Haz clic en **"Change Image"**. Selecciona **Canonical Ubuntu** y elige la última versión LTS disponible (por ejemplo, `22.04`). **Importante:** Siempre verifica los [requisitos del sistema de Hestia CP](https://hestiacp.com/docs/server-administration/os-support.html) para asegurarte de que la versión de Ubuntu es compatible.
    *   **Shape:** Haz clic en **"Change Shape"**. Selecciona la serie **Ampere** (ARM). Haz clic en `VM.Standard.A1.Flex` y marca la casilla **"Always Free Eligible"**. Puedes asignar los recursos que desees dentro del límite gratuito. Para Hestia CP, una configuración de **2 OCPU y 4 GB de RAM** es un excelente punto de partida y está bien dentro de los límites gratuitos (que permiten hasta 4 OCPU y 24 GB de RAM).

### Paso 4: Configurar la Red (Abrir Puertos)

Este paso es crucial para que Hestia CP funcione correctamente.

1.  **Primary network:** Puedes dejar que OCI cree una nueva red virtual en la nube (VCN) por ti o seleccionar una existente.
2.  **Subnet:** Asegúrate de que **"Assign a public IPv4 address"** esté seleccionado.
3.  **Security Rules:** Aquí es donde abrimos los puertos. Crea un **Nuevo grupo de seguridad de red** (New network security group) y añade las siguientes **reglas de entrada (Ingress Rules)**. Para cada regla, el **Source CIDR** debe ser `0.0.0.0/0` para permitir el acceso desde cualquier lugar de internet.

| Protocolo | Rango de Puertos de Destino | Descripción                                    |
| :-------- | :-------------------------- | :--------------------------------------------- |
| TCP       | 22                          | SSH (Acceso a la terminal)                     |
| TCP       | 80                          | HTTP (Tráfico web)                             |
| TCP       | 443                         | HTTPS (Tráfico web seguro)                     |
| TCP       | 8083                        | Hestia CP Panel (Puerto de administración)      |
| TCP       | 25, 465, 587                | SMTP (Para enviar correos)                     |
| TCP       | 110, 995                    | POP3 (Para recibir correos)                    |
| TCP       | 143, 993                    | IMAP (Para recibir correos)                    |
| TCP       | 20, 21                      | FTP (Si planeas usarlo)                        |

### Paso 5: Añadir tus Claves SSH

1.  En la sección **"Add SSH keys"**, tienes dos opciones:
    *   **Paste public keys:** Si ya tienes un par de claves SSH, pega aquí el contenido de tu clave pública (`.pub`).
    *   **Generate a key pair for me:** Si no tienes claves, selecciona esta opción. OCI creará un par y te pedirá que **descargues la clave privada**. **Guarda este archivo `.key` en un lugar seguro, ya que no podrás volver a descargarlo.**

### Paso 6: Crear y Conectar a la Instancia

1.  Haz clic en el botón **"Create"** en la parte inferior.
2.  La instancia comenzará a provisionarse. Espera a que su estado cambie a **"Running"** (se pondrá de color verde).
3.  Una vez que esté en ejecución, en la página de detalles de la instancia, encontrarás la **"Public IPv4 Address"**. Cópiala.
4.  Abre una terminal (en Windows puedes usar PowerShell, CMD o WSL) y conéctate usando el siguiente comando. Reemplaza `</path/to/private_key.key>` con la ruta a tu archivo de clave privada y `<public_ip>` con la IP que copiaste.

    ```bash
    ssh -i </path/to/private_key.key> ubuntu@<public_ip>
    ```

    *El nombre de usuario por defecto para las imágenes de Ubuntu en OCI es `ubuntu`.*

### Paso 7: Instalar Hestia Control Panel

Ahora que estás dentro de tu servidor a través de SSH, estás listo para instalar Hestia.

1.  Actualiza los paquetes del sistema:
    ```bash
    sudo apt update && sudo apt upgrade -y
    ```
2.  Descarga el script de instalación de Hestia:
    ```bash
    wget https://raw.githubusercontent.com/hestiacp/hestiacp/release/install/hst-install.sh
    ```
3.  Ejecuta el script de instalación. El modificador `--interactive no` acepta las opciones por defecto, que son adecuadas para una instalación estándar. Si quieres personalizar, simplemente ejecuta `sudo bash hst-install.sh`.
    ```bash
    sudo bash hst-install.sh --interactive no --email tu@email.com --password 'TuContraseñaSegura' --hostname tu.dominio.com -f
    ```
    *Espera a que el proceso termine. Puede tardar entre 10 y 15 minutos.*

4.  Al finalizar, el instalador te mostrará en pantalla la URL de acceso, el usuario (`admin`) y la contraseña. **Guarda estos datos**.

### Paso 8: Acceder a tu Panel de Hestia

1.  Abre tu navegador web y ve a la dirección que te proporcionó el instalador. Será algo como:
    `https://<public_ip>:8083`

2.  Tu navegador te mostrará una advertencia de seguridad porque Hestia usa un certificado SSL autofirmado inicialmente. Acepta el riesgo y continúa.

3.  Inicia sesión con el usuario `admin` y la contraseña que te dio el instalador.

¡Felicidades! Ya tienes un servidor VPS totalmente funcional con un panel de control profesional, todo de forma gratuita gracias a Oracle Cloud.
