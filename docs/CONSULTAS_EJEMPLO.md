# Ejemplos de Consultas Northwind Modificado

A continuación tienes ejemplos prácticos para explorar y validar las principales funcionalidades de la base de datos.

---

## 1. Consultas básicas

```sql
-- Ver los 10 productos con menor stock
SELECT product_id, product_name, units_in_stock
FROM products
ORDER BY units_in_stock ASC
LIMIT 10;

-- Ver todos los clientes y su email
SELECT customer_id, company_name, email
FROM customers;
```

---

## 2. Consultas sobre subcategorías y jerarquía

```sql
-- Ver productos con su subcategoría y categoría principal
SELECT p.product_id, p.product_name, s.name AS subcategoria, c.category_name
FROM products p
JOIN subcategories s ON p.subcategory_id = s.subcategory_id
JOIN categories c ON s.category_id = c.category_id
ORDER BY c.category_name, s.name, p.product_name;
```

---

## 3. Consultas sobre descuentos por volumen

```sql
-- Ver reglas de descuento por volumen
SELECT * FROM volume_discounts;

-- Calcular descuento aplicado en cada detalle de pedido
SELECT od.order_id, od.product_id, od.quantity,
       calcular_descuento_volumen(od.product_id, od.quantity) AS descuento_aplicado
FROM order_details od
WHERE od.quantity >= 10;
```

---

## 4. Consultas sobre auditoría y cambios en productos

```sql
-- Ver historial de cambios en productos
SELECT audit_id, product_id, changed_at, old_data, new_data
FROM product_audit
ORDER BY changed_at DESC;
```

---

## 5. Consultas sobre historial de precios

```sql
-- Ver historial de precios de un producto concreto
SELECT product_id, old_price, new_price, changed_at
FROM product_price_history
WHERE product_id = 1
ORDER BY changed_at DESC;
```

---

## 6. Consultas sobre alertas de stock

```sql
-- Ver alertas de stock pendientes de resolver
SELECT sa.alert_id, p.product_name, sa.current_stock, sa.min_stock, sa.alert_date
FROM stock_alerts sa
JOIN products p ON sa.product_id = p.product_id
WHERE sa.resolved = FALSE
ORDER BY sa.alert_date DESC;
```

---

## 7. Consultas sobre productos borrados lógicamente (soft delete)

```sql
-- Ver productos activos
SELECT * FROM products WHERE deleted = FALSE;

-- Ver productos borrados lógicamente
SELECT * FROM products WHERE deleted = TRUE;

-- Restaurar un producto borrado lógicamente
UPDATE products SET deleted = FALSE WHERE product_id = 1;
```

---

## 8. Consultas sobre vistas de análisis

```sql
-- Productos con stock bajo
SELECT * FROM productos_stock_bajo;

-- Ventas mensuales (vista normal)
SELECT * FROM ventas_mensuales;

-- Top productos vendidos
SELECT * FROM top_productos_vendidos;

-- Análisis de clientes por ventas
SELECT * FROM analisis_clientes;

-- Ventas mensuales (vista materializada)
SELECT * FROM ventas_mensuales_mat;
```

---

## 9. Consultas y updates con JSONB

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

---

## 10. Validación de emails

```sql
-- Intentar insertar un cliente con email inválido (debería fallar)
INSERT INTO customers (customer_id, company_name, email)
VALUES ('ZZZZZ', 'Cliente Prueba', 'noesuncorreo');

-- Insertar un cliente con email válido (debería funcionar)
INSERT INTO customers (customer_id, company_name, email)
VALUES ('YYYYY', 'Cliente Correcto', 'cliente@ejemplo.com');
```

---

## 11. Refrescar la vista materializada

```sql
-- Refrescar los datos de la vista materializada de ventas mensuales
REFRESH MATERIALIZED VIEW ventas_mensuales_mat;
```

---

**¡Explora, prueba y aprende pero tampoco te olvides de disfrutar!**  
Estas consultas te ayudarán a validar y aprovechar todas las funcionalidades avanzadas de Northwind Modificado.
