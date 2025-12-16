# Cómo Conseguir Máquinas Virtuales (VMs) Gratuitas

Para desarrolladores, estudiantes o cualquier persona que necesite un pequeño servidor para experimentar, alojar un bot, un blog personal o una VPN, existen varias opciones para obtener una máquina virtual de forma gratuita. Los principales proveedores de servicios en la nube (Cloud) ofrecen niveles gratuitos ("Free Tiers") que incluyen VMs con recursos limitados, pero que son perfectamente funcionales.

**Nota Importante:** La mayoría de estos servicios requieren una tarjeta de crédito para registrarse. Esto es principalmente para la verificación de identidad y para poder cobrar si excedes los límites del nivel gratuito. No se te cobrará nada si te mantienes dentro de los límites especificados.

---

## 1. Google Cloud Platform (GCP)

Google Cloud ofrece una de las capas gratuitas más conocidas, que incluye una micro instancia de VM de forma permanente.

*   **Instancia Incluida:** `e2-micro`
*   **Recursos:**
    *   2 vCPUs (compartidas, con ráfagas de rendimiento)
    *   1 GB de memoria RAM
    *   30 GB de almacenamiento en disco duro estándar
    *   1 GB de tráfico de red de salida (egress) al mes (excepto a China y Australia).
*   **Disponibilidad:** Esta oferta está limitada a algunas regiones de EE. UU. (como `us-west1`, `us-central1` o `us-east1`). Debes asegurarte de crear tu VM en una de estas regiones.

### ¿Cómo obtenerla?
1.  Ve al sitio web de [Google Cloud](https://cloud.google.com/) y crea una cuenta.
2.  Completa el proceso de registro, incluyendo los datos de tu tarjeta de crédito.
3.  En la consola de GCP, ve al servicio **Compute Engine -> Instancias de VM**.
4.  Crea una nueva instancia.
5.  **Asegúrate de seleccionar una de las regiones elegibles** (ej: `us-east1`).
6.  En la configuración de la máquina, selecciona la serie **E2** y el tipo de máquina **e2-micro**.
7.  Elige un sistema operativo (Debian y Ubuntu son opciones comunes y ligeras).
8.  En el firewall, asegúrate de permitir el tráfico HTTP/HTTPS si planeas alojar un servidor web.
9.  La consola te mostrará una estimación del costo. Si has seleccionado todo correctamente, debería indicarte que está cubierta por el nivel gratuito.

---

## 2. Amazon Web Services (AWS)

AWS, el pionero en la computación en la nube, también ofrece un nivel gratuito durante los primeros 12 meses para nuevos usuarios, y algunas ofertas "siempre gratuitas".

*   **Instancia Incluida:** `t2.micro` (o `t3.micro` dependiendo de la región).
*   **Límite:** 750 horas de uso al mes. Esto es suficiente para mantener una instancia funcionando 24/7 durante todo el mes.
*   **Recursos (t2.micro):**
    *   1 vCPU
    *   1 GB de memoria RAM
    *   Hasta 30 GB de almacenamiento (EBS)
    *   100 GB de tráfico de salida al mes.

### ¿Cómo obtenerla?
1.  Crea una cuenta en [AWS](https://aws.amazon.com/free/).
2.  Ve a la consola de AWS y busca el servicio **EC2**.
3.  Haz clic en "Lanzar instancia".
4.  Cuando elijas la AMI (Amazon Machine Image), asegúrate de marcar la opción **"Free tier eligible"** (Elegible para el nivel gratuito) para filtrar las imágenes de SO compatibles.
5.  En el tipo de instancia, selecciona `t2.micro` y verifica que esté marcada como "Free tier eligible".
6.  Configura el resto de las opciones (red, almacenamiento, etc.) y lanza la instancia.

---

## 3. Oracle Cloud (OCI)

Oracle ofrece la capa gratuita **más generosa** del mercado, y es una opción excelente si necesitas un poco más de potencia. Su oferta "Always Free" es permanente.

*   **Instancias Incluidas:**
    *   **VMs basadas en ARM:** Hasta 4 núcleos de CPU (Ampere A1) y 24 GB de RAM en total. Puedes distribuirlos como una sola VM de 4 núcleos y 24 GB de RAM, o hasta 4 VMs de 1 núcleo y 6 GB de RAM cada una.
    *   **VMs basadas en AMD:** 2 instancias `VM.Standard.E2.1.Micro` con 1/8 de OCPU y 1 GB de RAM cada una.
*   **Recursos Adicionales:**
    *   200 GB de almacenamiento en bloque.
    *   10 TB de tráfico de salida al mes.

### ¿Cómo obtenerla?
1.  Regístrate en [Oracle Cloud Free Tier](https://www.oracle.com/cloud/free/).
2.  Una vez en la consola, ve a **Compute -> Instances**.
3.  Haz clic en "Create Instance".
4.  Al elegir la "Image and shape" (Imagen y forma), busca las que están marcadas como **"Always Free Eligible"**. Puedes elegir una de las micro instancias de AMD o, para más potencia, seleccionar una forma de Ampere (ARM) como `VM.Standard.A1.Flex` y asignar los núcleos y la RAM que necesites dentro de los límites gratuitos.

**Ventaja de Oracle:** La posibilidad de usar procesadores ARM con una cantidad tan generosa de RAM la convierte en la mejor opción para proyectos que requieren más memoria o múltiples núcleos de CPU.

---

## Consideraciones Finales

*   **Rendimiento:** Estas VMs son de bajo rendimiento. Son perfectas para aprender, desarrollar o para aplicaciones ligeras, pero no esperes poder ejecutar cargas de trabajo pesadas.
*   **Ubicación:** La latencia de red depende de la región donde crees tu VM. Elige una que esté geográficamente cerca de ti o de tus usuarios.
*   **Seguridad:** Eres responsable de la seguridad de tu VM. Asegúrate de configurar correctamente el firewall y de mantener el sistema operativo y el software actualizados.
*   **Monitoriza el Uso:** Aunque estés en el nivel gratuito, es una buena práctica configurar alertas de facturación para que el proveedor te notifique si te estás acercando a un límite que podría generar costos.
