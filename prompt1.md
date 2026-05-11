# "Actúa como un administrador de bases de datos en el proyecto "MegaPizzaplex", que es una aplicación para ordenar comida a domicilio. Muestra la estructura que sería recomendable seguir para simular todo el proceso que se lleva a cabo al utilizar la aplicación"

---

Aquí está la estructura recomendada de base de datos para **MegaPizzaplex**, modelada como un ERD completo que cubre todo el ciclo de vida de un pedido:El esquema cubre **11 tablas** que simulan el ciclo completo. Aquí un resumen de los grupos lógicos:

<img width="1440" height="1626" alt="image" src="https://github.com/user-attachments/assets/678a64d7-70bc-49c8-addf-487da8c6b243" />

**Gestión de usuarios y direcciones** — `USUARIO` y `DIRECCION` permiten que un cliente tenga múltiples domicilios guardados, con coordenadas para el cálculo de rutas y costos de envío.

**Catálogo del restaurante** — `RESTAURANTE`, `CATEGORIA_MENU` y `PRODUCTO` modelan la estructura jerárquica del menú (ej: *Pizzas → Margarita*), con campos de disponibilidad y horario de atención.

**Ciclo del pedido** — `PEDIDO` es la entidad central. Relaciona al usuario, restaurante, dirección de entrega y repartidor asignado. El campo `estado` recorre el ciclo: `pendiente → confirmado → en_preparacion → en_camino → entregado`. `ITEM_PEDIDO` guarda la foto fija del precio al momento de comprar (importante: no se debe tomar el precio actual del producto, sino el que tenía al pedirlo).

**Pagos** — `PAGO` tiene relación uno a uno con `PEDIDO`. El campo `referencia_externa` almacena el ID que devuelve la pasarela de pago (Stripe, Conekta, etc.) para conciliación.

**Repartidor** — `REPARTIDOR` guarda su ubicación en tiempo real (`latitud_actual`, `longitud_actual`) para el tracking del pedido.

**Post-entrega** — `RESENA` separa la calificación de la comida y la de la entrega, lo cual permite retroalimentación diferenciada para el restaurante y el repartidor. `NOTIFICACION` permite registrar todos los eventos enviados al usuario (push, SMS, email) para auditoría y reenvío.
