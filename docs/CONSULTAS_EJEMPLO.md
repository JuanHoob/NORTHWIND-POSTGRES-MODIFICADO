# Ejemplos de Consultas

## 1. Productos con subcategoría y categoría

```sql
SELECT p.product_id, p.product_name, s.name AS subcategoria, c.category_name
FROM products p
JOIN subcategories s ON p.subcategory_id = s.subcategory_id
JOIN categories c ON s.category_id = c.category_id
ORDER BY c.category_name, s.name, p.product_name;
```

## 2. Productos con stock bajo

```sql
SELECT p.product_id, p.product_name, p.units_in_stock, p.min_stock
FROM products p
WHERE p.units_in_stock < p.min_stock;
```

## 3. Descuentos aplicados en pedidos

```sql
SELECT od.order_id, od.product_id, od.quantity,
       calcular_descuento_volumen(od.product_id, od.quantity) AS descuento_aplicado
FROM order_details od
WHERE od.quantity >= 10;
```

## 4. Consultar auditoría de productos

```sql
SELECT audit_id, product_id, changed_at, old_data, new_data
FROM product_audit
ORDER BY changed_at DESC;
```

## 5. Consultas sobre vistas de análisis

```sql
SELECT * FROM productos_stock_bajo;
SELECT * FROM ventas_mensuales;
SELECT * FROM top_productos_vendidos;
SELECT * FROM analisis_clientes;
```

## 6. Consultas y updates con JSONB

```sql
-- Añadir campo al JSON existente
UPDATE products
SET caracteristicas_json = jsonb_set(caracteristicas_json, '{envase}', '"Lata"')
WHERE product_id = 1;

-- Buscar productos que tengan el campo 'color'
SELECT product_id, product_name, caracteristicas_json
FROM products
WHERE caracteristicas_json ? 'color';

-- Extraer valor específico del JSONB
SELECT product_id, product_name,
       caracteristicas_json->>'color' AS color
FROM products
WHERE caracteristicas_json->>'color' IS NOT NULL;
```

## 7. Ejemplos prácticos para activar triggers y probar funcionalidades

### a) Cambiar stock para activar el trigger de alerta

```sql
-- Disminuir el stock de un producto por debajo del mínimo para activar el trigger de alerta
UPDATE products
SET units_in_stock = 2
WHERE product_id = 1;
-- Esto insertará una alerta en la tabla stock_alerts si el valor es menor que min_stock
```

### b) Actualizar un producto para activar la auditoría

```sql
-- Cambiar el nombre o el precio de un producto para activar el trigger de auditoría
UPDATE products
SET product_name = 'Nuevo nombre', unit_price = unit_price + 1
WHERE product_id = 1;
-- Esto insertará un registro en la tabla product_audit
```

### c) Probar el trigger de descuentos por volumen

```sql
-- Insertar un detalle de pedido con cantidad suficiente para aplicar descuento
INSERT INTO order_details (order_id, product_id, unit_price, quantity, discount)
VALUES (11078, 1, 18.00, 50, 0);
-- Luego consulta el descuento aplicado:
SELECT order_id, product_id, quantity,
       calcular_descuento_volumen(product_id, quantity) AS descuento_aplicado
FROM order_details
WHERE order_id = 11078;
```

### d) Actualizar JSONB y consultar cambios

```sql
-- Añadir o modificar un campo en el JSONB
UPDATE products
SET caracteristicas_json = jsonb_set(caracteristicas_json, '{material}', '"Vidrio"')
WHERE product_id = 1;

-- Consultar productos cuyo JSONB contiene un campo específico
SELECT product_id, product_name
FROM products
WHERE caracteristicas_json ? 'material';
```

---

**Recuerda:**  
- Cambios en `units_in_stock` activan el trigger de alertas de stock.
- Cambios en cualquier campo de `products` activan la auditoría.
- Cambios en cantidades de `order_details` pueden activar descuentos por volumen si consultas con la función.
- Updates en `caracteristicas_json` permiten probar búsquedas y el índice GIN si está creado.