# 1. Objetivo
<img width="2752" height="1536" alt="image" src="https://github.com/user-attachments/assets/ca27d09d-eba3-4800-8605-d0b6b6f7686a" />

#2. Marco Teórico
<img width="1678" height="937" alt="image" src="https://github.com/user-attachments/assets/f151cfee-c5b2-42ff-802e-1b25b3403084" />

# 3. Tecnologías Utilizadas
<img width="2752" height="1536" alt="image" src="https://github.com/user-attachments/assets/79c33d95-367b-4e02-aa37-5a4ad4a87b53" />

# 4. Desarrollo
<img width="1202" height="1309" alt="image" src="https://github.com/user-attachments/assets/9ffbc912-3027-45fc-8173-0e6bf80eb410" />

# 5. Implementación
<img width="1307" height="1203" alt="image" src="https://github.com/user-attachments/assets/5f5f03d2-f1d8-425c-bee6-dd8d14da267a" />

---

Este documento presenta el proyecto **"Mega Pizzaplex"**, estructurado bajo los estándares de ingeniería de software descritos en las fuentes y utilizando como base técnica el plan de implementación proporcionado en el repositorio de GitHub.

### **Índice de Contenido**

1.  **Objetivo**
2.  **Marco Teórico**
3.  **Tecnologías Utilizadas**
4.  **Desarrollo**
    *   4.1 Arquitectura del Sistema
    *   4.2 Diseño de UI/UX e Identidad Visual
    *   4.3 Modelo de Datos y Flujo de Usuario
5.  **Implementación**
    *   5.1 Fases del Proyecto
    *   5.2 Seguridad y Reglas de Negocio
6.  **Conclusiones**

---

### **1. Objetivo**
El objetivo principal es desarrollar una **aplicación multiplataforma** (Android, iOS y Windows) para un servicio de comida a domicilio tipo Uber Eats o Rappi. El sistema debe estar **inspirado visualmente en *FNaF Security Breach***, permitiendo a los usuarios realizar órdenes en línea que son comunicadas a preparadores y repartidores en tiempo real.

### **2. Marco Teórico**
El proyecto se fundamenta en los principios de la **Ingeniería de Software**, definida como la aplicación de un enfoque sistemático, disciplinado y cuantificable al desarrollo y mantenimiento de sistemas. Se adopta un **ciclo de vida ágil**, que promueve la retroalimentación continua del cliente y el desarrollo incremental en plazos cortos. La arquitectura base será el patrón **Clean Architecture + Feature First**, dividiendo el sistema en capas de Presentación, Dominio y Datos para garantizar la escalabilidad y el mantenimiento.

### **3. Tecnologías Utilizadas**
Para el desarrollo se han seleccionado herramientas modernas que permiten una solución multiplataforma robusta:
*   **Flutter (Dart):** Framework para el desarrollo de la interfaz de usuario.
*   **Firebase:** Suite de backend que incluye *Authentication* para la gestión de usuarios, *Cloud Firestore* como base de datos NoSQL, *Storage* para archivos y *Cloud Messaging* para notificaciones push.
*   **Antigravity (Google):** Editor de código principal.
*   **Git y GitHub:** Para el control de versiones y gestión del repositorio.

### **4. Desarrollo**

#### **4.1 Arquitectura del Sistema**
Se utiliza una estructura de **capas desacopladas** (Presentation, Domain, Data) que facilita la separación de responsabilidades entre los roles de administrador, cliente y repartidor.

#### **4.2 Diseño de UI/UX e Identidad Visual**
La estética se centra en un **tema oscuro (#1E1E1E)** con acentos de neón basados en los personajes del Mega Pizzaplex (Glamrock Freddy, Roxanne Wolf, etc.). El estilo busca ser futurista y dinámico, empleando microanimaciones e íconos dinámicos en pantallas clave como el *Home*, el Carrito y el Seguimiento de pedidos.

#### **4.3 Modelo de Datos y Flujo de Usuario**
La base de datos se organiza en **11 tablas lógicas** que cubren el ciclo completo del servicio:
*   **Usuarios y Direcciones:** Gestión de perfiles y múltiples domicilios.
*   **Catálogo:** Estructura jerárquica de restaurantes, categorías y productos.
*   **Ciclo de Pedido:** Entidad central que coordina al cliente, restaurante y repartidor, pasando por estados desde "pendiente" hasta "entregado".
*   **Pagos y Reseñas:** Registro de transacciones financieras y retroalimentación post-entrega.

### **5. Implementación**

#### **5.1 Fases del Proyecto**
El despliegue se divide en ocho fases críticas:
1.  **Preparación:** Configuración de entornos y arquitectura.
2.  **UI Base:** Creación del tema global y pantallas de login.
3.  **Backend Base:** Implementación de autenticación y reglas de Firestore.
4.  **Módulo Cliente:** Desarrollo del catálogo y flujo de compra.
5.  **Módulo Admin:** Dashboard para gestión de menús y pedidos.
6.  **Módulo Reparto:** Seguimiento por geolocalización.
7.  **Testing:** Pruebas unitarias, de integración y de seguridad.
8.  **Deploy:** Generación de binarios para Android (APK), iOS y Windows (EXE).

#### **5.2 Seguridad y Reglas de Negocio**
Se establecen **reglas de Firestore estrictas** donde el cliente solo accede a sus propios datos, mientras que el administrador tiene acceso extendido. Se incluyen medidas como la sanitización de entradas, protección de pagos y validación de escritura para evitar la escalación de privilegios.

### **6. Conclusiones**
El plan de implementación del **Mega Pizzaplex** establece una base profesional que combina un diseño temático atractivo con una **arquitectura escalable y segura**. El uso de Firebase y Flutter asegura que el producto sea funcional en múltiples dispositivos, permitiendo una evolución desde un Producto Mínimo Viable (MVP) hasta una plataforma comercial completa preparada para expansión futura.
