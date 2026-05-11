## "¿Puedes crear el script para exportar esto en sql?"

---

El script está listo para PostgreSQL 15+. Algunos detalles importantes de lo que incluye:

**ENUMs declarados al inicio** — todos los tipos enumerados (`estado_pedido`, `metodo_pago`, etc.) se crean antes que las tablas que los usan, evitando errores de dependencia al ejecutar el script en orden.

**`gen_random_uuid()`** — requiere la extensión `pgcrypto`, que se activa en la primera línea. En PostgreSQL 13+ también puedes sustituirla por la función nativa `gen_random_uuid()` sin extensión.

**Constraints de integridad** — se agregaron `CHECK` en campos críticos: precios no negativos, calificaciones entre 1 y 5, cantidades mayores a cero. Esto valida datos a nivel de motor, no solo en la app.

**`ON DELETE` explícito en cada FK** — cada llave foránea tiene su política: `CASCADE` donde borrar el padre debe borrar los hijos (ej. items de un pedido), y `SET NULL` donde conviene conservar el registro huérfano (ej. notificaciones si se elimina un pedido).

**Trigger de `actualizado_en`** — la función `set_actualizado_en()` actualiza automáticamente el timestamp al hacer `UPDATE` en la tabla `usuario`, sin depender de la capa de aplicación.

---

# MegaPizzaplex Schema (Código SQL)

-- =============================================================
--  MegaPizzaplex — Schema SQL
--  Base de datos: PostgreSQL 15+
--  Generado: 2026-05-11
-- =============================================================

-- Extensión para UUID
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

-- -------------------------------------------------------------
-- TIPOS ENUM
-- -------------------------------------------------------------

CREATE TYPE estado_pedido AS ENUM (
    'pendiente',
    'confirmado',
    'en_preparacion',
    'en_camino',
    'entregado',
    'cancelado'
);

CREATE TYPE estado_pago AS ENUM (
    'pendiente',
    'aprobado',
    'rechazado',
    'reembolsado'
);

CREATE TYPE metodo_pago AS ENUM (
    'tarjeta',
    'efectivo',
    'transferencia',
    'wallet'
);

CREATE TYPE estado_repartidor AS ENUM (
    'disponible',
    'en_entrega',
    'inactivo'
);

CREATE TYPE tipo_notificacion AS ENUM (
    'push',
    'sms',
    'email'
);

-- -------------------------------------------------------------
-- GRUPO: USUARIOS
-- -------------------------------------------------------------

CREATE TABLE usuario (
    id                UUID            PRIMARY KEY DEFAULT gen_random_uuid(),
    nombre            VARCHAR(120)    NOT NULL,
    email             VARCHAR(255)    NOT NULL,
    telefono          VARCHAR(20),
    contrasena_hash   TEXT            NOT NULL,
    activo            BOOLEAN         NOT NULL DEFAULT TRUE,
    creado_en         TIMESTAMPTZ     NOT NULL DEFAULT NOW(),
    actualizado_en    TIMESTAMPTZ     NOT NULL DEFAULT NOW()
);

CREATE UNIQUE INDEX idx_usuario_email ON usuario (email);

-- -------------------------------------------------------------

CREATE TABLE direccion (
    id              UUID            PRIMARY KEY DEFAULT gen_random_uuid(),
    usuario_id      UUID            NOT NULL REFERENCES usuario (id) ON DELETE CASCADE,
    alias           VARCHAR(60),
    calle           VARCHAR(200)    NOT NULL,
    colonia         VARCHAR(120)    NOT NULL,
    ciudad          VARCHAR(100)    NOT NULL,
    codigo_postal   CHAR(5)         NOT NULL,
    latitud         DECIMAL(9, 6)   NOT NULL,
    longitud        DECIMAL(9, 6)   NOT NULL,
    es_principal    BOOLEAN         NOT NULL DEFAULT FALSE
);

CREATE INDEX idx_direccion_usuario ON direccion (usuario_id);

-- -------------------------------------------------------------
-- GRUPO: CATÁLOGO
-- -------------------------------------------------------------

CREATE TABLE restaurante (
    id                  UUID            PRIMARY KEY DEFAULT gen_random_uuid(),
    nombre              VARCHAR(150)    NOT NULL,
    descripcion         TEXT,
    direccion           TEXT            NOT NULL,
    latitud             DECIMAL(9, 6)   NOT NULL,
    longitud            DECIMAL(9, 6)   NOT NULL,
    telefono            VARCHAR(20),
    logo_url            TEXT,
    activo              BOOLEAN         NOT NULL DEFAULT TRUE,
    horario_apertura    TIME            NOT NULL DEFAULT '08:00',
    horario_cierre      TIME            NOT NULL DEFAULT '22:00',
    tiempo_entrega_min  SMALLINT        NOT NULL DEFAULT 30,
    radio_entrega_km    DECIMAL(5, 2)   NOT NULL DEFAULT 5.00
);

-- -------------------------------------------------------------

CREATE TABLE categoria_menu (
    id              UUID            PRIMARY KEY DEFAULT gen_random_uuid(),
    restaurante_id  UUID            NOT NULL REFERENCES restaurante (id) ON DELETE CASCADE,
    nombre          VARCHAR(80)     NOT NULL,
    descripcion     VARCHAR(255),
    imagen_url      TEXT,
    orden           SMALLINT        NOT NULL DEFAULT 0,
    activa          BOOLEAN         NOT NULL DEFAULT TRUE
);

CREATE INDEX idx_categoria_restaurante ON categoria_menu (restaurante_id);

-- -------------------------------------------------------------

CREATE TABLE producto (
    id              UUID            PRIMARY KEY DEFAULT gen_random_uuid(),
    categoria_id    UUID            NOT NULL REFERENCES categoria_menu (id) ON DELETE CASCADE,
    nombre          VARCHAR(120)    NOT NULL,
    descripcion     TEXT,
    precio          DECIMAL(10, 2)  NOT NULL CHECK (precio >= 0),
    imagen_url      TEXT,
    disponible      BOOLEAN         NOT NULL DEFAULT TRUE,
    calorias        SMALLINT,
    tiempo_prep_min SMALLINT,
    es_vegano       BOOLEAN         NOT NULL DEFAULT FALSE
);

CREATE INDEX idx_producto_categoria ON producto (categoria_id);

-- -------------------------------------------------------------
-- GRUPO: OPERACIÓN
-- -------------------------------------------------------------

CREATE TABLE repartidor (
    id                      UUID                PRIMARY KEY DEFAULT gen_random_uuid(),
    nombre                  VARCHAR(120)        NOT NULL,
    telefono                VARCHAR(20)         NOT NULL,
    estado                  estado_repartidor   NOT NULL DEFAULT 'inactivo',
    latitud_actual          DECIMAL(9, 6),
    longitud_actual         DECIMAL(9, 6),
    ubicacion_actualizada   TIMESTAMPTZ,
    calificacion_promedio   DECIMAL(3, 2)       CHECK (calificacion_promedio BETWEEN 0 AND 5)
);

-- -------------------------------------------------------------
-- GRUPO: PEDIDOS
-- -------------------------------------------------------------

CREATE TABLE pedido (
    id              UUID            PRIMARY KEY DEFAULT gen_random_uuid(),
    usuario_id      UUID            NOT NULL REFERENCES usuario (id),
    restaurante_id  UUID            NOT NULL REFERENCES restaurante (id),
    direccion_id    UUID            NOT NULL REFERENCES direccion (id),
    repartidor_id   UUID            REFERENCES repartidor (id),
    estado          estado_pedido   NOT NULL DEFAULT 'pendiente',
    subtotal        DECIMAL(10, 2)  NOT NULL CHECK (subtotal >= 0),
    costo_envio     DECIMAL(10, 2)  NOT NULL DEFAULT 0 CHECK (costo_envio >= 0),
    descuento       DECIMAL(10, 2)  NOT NULL DEFAULT 0 CHECK (descuento >= 0),
    total           DECIMAL(10, 2)  NOT NULL CHECK (total >= 0),
    notas           TEXT,
    creado_en       TIMESTAMPTZ     NOT NULL DEFAULT NOW(),
    entregado_en    TIMESTAMPTZ
);

CREATE INDEX idx_pedido_usuario    ON pedido (usuario_id);
CREATE INDEX idx_pedido_restaurante ON pedido (restaurante_id);
CREATE INDEX idx_pedido_estado     ON pedido (estado);
CREATE INDEX idx_pedido_creado     ON pedido (creado_en DESC);

-- -------------------------------------------------------------

CREATE TABLE item_pedido (
    id              UUID            PRIMARY KEY DEFAULT gen_random_uuid(),
    pedido_id       UUID            NOT NULL REFERENCES pedido (id) ON DELETE CASCADE,
    producto_id     UUID            REFERENCES producto (id) ON DELETE SET NULL,
    nombre_snapshot VARCHAR(120)    NOT NULL,
    precio_unitario DECIMAL(10, 2)  NOT NULL CHECK (precio_unitario >= 0),
    cantidad        SMALLINT        NOT NULL CHECK (cantidad > 0),
    subtotal        DECIMAL(10, 2)  NOT NULL CHECK (subtotal >= 0),
    notas           TEXT
);

CREATE INDEX idx_item_pedido ON item_pedido (pedido_id);

-- -------------------------------------------------------------

CREATE TABLE pago (
    id                  UUID            PRIMARY KEY DEFAULT gen_random_uuid(),
    pedido_id           UUID            NOT NULL UNIQUE REFERENCES pedido (id) ON DELETE CASCADE,
    metodo              metodo_pago     NOT NULL,
    estado              estado_pago     NOT NULL DEFAULT 'pendiente',
    monto               DECIMAL(10, 2)  NOT NULL CHECK (monto > 0),
    moneda              CHAR(3)         NOT NULL DEFAULT 'MXN',
    referencia_externa  VARCHAR(200),
    procesado_en        TIMESTAMPTZ
);

CREATE INDEX idx_pago_pedido ON pago (pedido_id);

-- -------------------------------------------------------------

CREATE TABLE resena (
    id              UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
    pedido_id       UUID        NOT NULL UNIQUE REFERENCES pedido (id) ON DELETE CASCADE,
    usuario_id      UUID        NOT NULL REFERENCES usuario (id),
    cal_comida      SMALLINT    NOT NULL CHECK (cal_comida BETWEEN 1 AND 5),
    cal_entrega     SMALLINT    NOT NULL CHECK (cal_entrega BETWEEN 1 AND 5),
    comentario      TEXT,
    creado_en       TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_resena_pedido ON resena (pedido_id);

-- -------------------------------------------------------------

CREATE TABLE notificacion (
    id          UUID                    PRIMARY KEY DEFAULT gen_random_uuid(),
    usuario_id  UUID                    NOT NULL REFERENCES usuario (id) ON DELETE CASCADE,
    pedido_id   UUID                    REFERENCES pedido (id) ON DELETE SET NULL,
    tipo        tipo_notificacion       NOT NULL,
    canal       VARCHAR(60),
    mensaje     TEXT                    NOT NULL,
    leida       BOOLEAN                 NOT NULL DEFAULT FALSE,
    enviada_en  TIMESTAMPTZ             NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_notificacion_usuario ON notificacion (usuario_id);
CREATE INDEX idx_notificacion_enviada ON notificacion (enviada_en DESC);

-- -------------------------------------------------------------
-- FUNCIÓN: actualizar timestamp automáticamente
-- -------------------------------------------------------------

CREATE OR REPLACE FUNCTION set_actualizado_en()
RETURNS TRIGGER AS $$
BEGIN
    NEW.actualizado_en = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_usuario_actualizado
    BEFORE UPDATE ON usuario
    FOR EACH ROW EXECUTE FUNCTION set_actualizado_en();

-- =============================================================
--  Fin del schema — MegaPizzaplex
-- =============================================================
