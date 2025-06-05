# Funcionalidades y Mejoras Implementadas

## 1. Sistema de Categorías Jerárquicas

```sql
-- Crear tabla de subcategorías
CREATE TABLE subcategories (
    subcategory_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    category_id INT NOT NULL,
    FOREIGN KEY (category_id) REFERENCES categories(category_id)
);

-- Añadir columna subcategory_id a products y crear la relación
ALTER TABLE products ADD COLUMN subcategory_id INT;
ALTER TABLE products
    ADD CONSTRAINT fk_products_subcategories
    FOREIGN KEY (subcategory_id) REFERENCES subcategories(subcategory_id);

-- Insertar ejemplos de subcategorías
INSERT INTO subcategories (name, category_id) VALUES
('Refrescos', 1), ('Zumos', 1), ('Quesos duros', 2), ('Quesos blandos', 2),
('Café', 1), ('Tés', 1), ('Embutidos', 3), ('Carnes curadas', 3);

-- Asignar subcategorías a productos (ejemplo)
UPDATE products SET subcategory_id = 1 WHERE product_id = 1;
UPDATE products SET subcategory_id = 2 WHERE product_id = 2;
```

---

## 2. Control de Stock Avanzado

```sql
-- Añadir campo de stock mínimo
ALTER TABLE products ADD COLUMN min_stock INT DEFAULT 10;

-- Crear tabla de alertas de stock
CREATE TABLE stock_alerts (
    alert_id SERIAL PRIMARY KEY,
    product_id INT NOT NULL,
    alert_date TIMESTAMP DEFAULT now(),
    current_stock INT,
    min_stock INT,
    resolved BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- Función y trigger para alertas automáticas
CREATE OR REPLACE FUNCTION check_stock_alert()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.units_in_stock < NEW.min_stock THEN
        INSERT INTO stock_alerts(product_id, current_stock, min_stock)
        VALUES (NEW.product_id, NEW.units_in_stock, NEW.min_stock);
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_stock_alert
AFTER UPDATE OF units_in_stock ON products
FOR EACH ROW
EXECUTE FUNCTION check_stock_alert();
```

---

## 3. Descuentos por Volumen

```sql
-- Crear tabla de descuentos por volumen
CREATE TABLE volume_discounts (
    discount_id SERIAL PRIMARY KEY,
    product_id INT NOT NULL,
    min_quantity INT NOT NULL,
    discount_percent NUMERIC(5,2) NOT NULL,
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- Insertar reglas de ejemplo
INSERT INTO volume_discounts (product_id, min_quantity, discount_percent) VALUES
(1, 10, 5.00), (1, 50, 10.00), (2, 20, 7.50);

-- Función para calcular el descuento
CREATE OR REPLACE FUNCTION calcular_descuento_volumen(pid INT, cantidad INT)
RETURNS NUMERIC AS $$
DECLARE
    descuento NUMERIC := 0;
BEGIN
    SELECT COALESCE(MAX(discount_percent), 0)
    INTO descuento
    FROM volume_discounts
    WHERE product_id = pid AND cantidad >= min_quantity;
    RETURN descuento;
END;
$$ LANGUAGE plpgsql;
```

---

## 4. Auditoría Completa de Cambios en Productos

```sql
-- Crear tabla de auditoría
CREATE TABLE product_audit (
    audit_id SERIAL PRIMARY KEY,
    product_id INT,
    old_data JSONB,
    new_data JSONB,
    changed_at TIMESTAMP DEFAULT now()
);

-- Función y trigger de auditoría
CREATE OR REPLACE FUNCTION audit_product_changes()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO product_audit(product_id, old_data, new_data)
    VALUES (
        NEW.product_id,
        to_jsonb(OLD),
        to_jsonb(NEW)
    );
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_audit_products
AFTER UPDATE ON products
FOR EACH ROW
EXECUTE FUNCTION audit_product_changes();
```

---

## 5. Ejemplo de uso de JSONB en Products

```sql
-- Actualizar producto con datos JSONB
UPDATE products
SET caracteristicas_json = '{"color": "rojo", "peso": "1kg", "subcategoria": "Refrescos"}'
WHERE product_id = 1;

-- Consultar productos por subcategoría en JSONB
SELECT product_id, product_name
FROM products
WHERE caracteristicas_json->>'subcategoria' = 'Refrescos';

-- Índice GIN para búsquedas rápidas en JSONB
CREATE INDEX idx_products_caracteristicas_json ON products USING gin(caracteristicas_json);
```

---

## 6. Propuesta personal del alumno (ejemplo)

```sql
-- Soft delete en productos
ALTER TABLE products ADD COLUMN deleted BOOLEAN DEFAULT FALSE;

-- Para "borrar" un producto:
UPDATE products SET deleted = TRUE WHERE product_id = 1;

-- Consultar solo productos activos:
SELECT * FROM products WHERE deleted = FALSE;
```

---

## 7. Historial de Cambios de Precio

```sql
-- Crear tabla de historial de precios
CREATE TABLE product_price_history (
    history_id SERIAL PRIMARY KEY,
    product_id INT NOT NULL,
    old_price NUMERIC(10,2),
    new_price NUMERIC(10,2),
    changed_at TIMESTAMP DEFAULT now(),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- Trigger para registrar cambios de precio
CREATE OR REPLACE FUNCTION track_price_changes()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.unit_price <> OLD.unit_price THEN
        INSERT INTO product_price_history(product_id, old_price, new_price)
        VALUES (NEW.product_id, OLD.unit_price, NEW.unit_price);
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_price_history
AFTER UPDATE OF unit_price ON products
FOR EACH ROW
EXECUTE FUNCTION track_price_changes();
```

---

## 8. Validación de Emails en Clientes

```sql
-- Añadir columna email a customers
ALTER TABLE customers ADD COLUMN email VARCHAR(255);

-- Añadir restricción para emails válidos en customers
ALTER TABLE customers
ADD CONSTRAINT chk_email_valid
CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');
```

---

## 9. Descuento Automático por Registro de Pedido

```sql
-- Trigger para descontar stock automáticamente al insertar un detalle de pedido
CREATE OR REPLACE FUNCTION descontar_stock()
RETURNS TRIGGER AS $$
BEGIN
    UPDATE products
    SET units_in_stock = units_in_stock - NEW.quantity
    WHERE product_id = NEW.product_id;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_descontar_stock
AFTER INSERT ON order_details
FOR EACH ROW
EXECUTE FUNCTION descontar_stock();
```

---

## 10. Ventas Mensuales

```sql
-- Crear una vista materializada de ventas por mes
CREATE MATERIALIZED VIEW ventas_mensuales_mat AS
SELECT DATE_TRUNC('month', o.order_date) AS mes,
       SUM(od.quantity * od.unit_price * (1 - od.discount)) AS total_ventas
FROM orders o
JOIN order_details od ON o.order_id = od.order_id
GROUP BY mes;
```
