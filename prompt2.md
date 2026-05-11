# "Desglosa la estructura de las tablas/documentos de forma que se puedan visualizar las entidades junto con tipos de datos y atributos"
---

<img width="1134" height="6510" alt="image" src="https://github.com/user-attachments/assets/15cd8277-ef30-4416-9840-035cee8a974d" />

El explorador permite filtrar las tablas por grupo funcional. Algunos puntos de diseño destacables:

**Tipos de datos con criterio** — se usa `UUID` en lugar de `INT` autoincremental para las llaves primarias, lo que facilita la generación distribuida de IDs sin colisiones. Los precios son `DECIMAL(10,2)` para evitar errores de punto flotante típicos de `FLOAT`. Las coordenadas geográficas usan `DECIMAL(9,6)` para precisión centimétrica.

**`ENUM` para estados controlados** — `pedido.estado`, `pago.estado` y `repartidor.estado` usan `ENUM` en lugar de `VARCHAR` libre, lo que garantiza integridad a nivel de motor sin depender de validaciones en la aplicación.

**`nombre_snapshot` en `item_pedido`** — si un producto cambia de nombre o precio después de un pedido, el registro histórico debe conservar los datos tal como eran en ese momento. Por eso `item_pedido` almacena `nombre_snapshot` y `precio_unitario` como copia fija, no como referencia dinámica al catálogo.

**Índices estratégicos** — las columnas marcadas con `IDX` son candidatas a índice porque son filtros frecuentes en queries reales: `usuario_id` en pedidos, `estado` en pedidos, `enviada_en` en notificaciones, y `email` en usuario para el login.

**`TIMESTAMPTZ` sobre `TIMESTAMP`** — se usa la variante con zona horaria para soportar operaciones en múltiples ciudades sin ambigüedad en los registros de tiempo.
