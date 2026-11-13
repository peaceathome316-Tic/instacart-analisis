# Informe de procesamiento de datos de Instacart

**Aprendiz:** David Vargas Franco · **Ficha:** 3169901 · **Actividad:** GA1-220501094-01-AA1-EV06
**Fecha:** 13 de noviembre de 2026 · **Cuaderno:** `notebooks/pandas_labs.ipynb`

## 1. Objetivo
Preparar y estructurar los datos de Instacart (limpieza, filtrado, agrupación y combinación) para obtener información confiable sobre los hábitos de compra.

## 2. Datos de entrada
Cinco archivos CSV con separador `;`: `instacart_orders.csv` (478.967 filas), `order_products.csv` (4.545.007), `products.csv` (49.694), `aisles.csv` (134) y `departments.csv` (21).

## 3. Lección 2: limpieza y estandarización

| Problema | Evidencia | Decisión |
|---|---|---|
| Duplicados exactos en `orders` | 15 pedidos repetidos (30 filas, todas del miércoles a las 2 a. m.) | **Eliminar**: quedan 478.952 pedidos |
| Tipos de dato | Memoria de `orders`: 23,0 → 7,2 MB; de `order_prods`: 145,4 → 54,5 MB | **Corregir** con enteros pequeños y `Int16` |
| Nulos en `days_since_prior_order` | 28.817, que son los primeros pedidos de cada cliente | **Conservar** el nulo |
| Nulos en `product_name` | 1.258, todos del pasillo 100 y departamento 21 (`missing`) | **Corregir**: rellenar con `Unknown` |
| Nulos en `add_to_cart_order` | 836 en 70 pedidos de 65 a 127 productos; el máximo registrado es 64 | **Conservar** |
| Texto sin estandarizar | 403 nombres con espacios dobles y 16 con espacio invisible; 6.321 nombres cambian al estandarizar | **Corregir** con `.str` |
| Duplicados implícitos | 141 nombres repetidos con `product_id` distinto | **Conservar** y agrupar por `product_id` |
| Productos `Unknown` | 9.547 filas en 8.254 pedidos | **Conservar** y mostrarlos aparte |

El pipeline de limpieza se encadenó con `pipe`, registró el estado de cada tabla en `cleaning_report.csv` y se validó con `assert`. Las tablas resultantes coinciden con las de la limpieza manual.

## 4. Lección 3: filtrado, ordenamiento y agrupación
- **Hora y día:** la hora con más pedidos es las 10 a. m. (40.578 pedidos). Los domingos, las tres horas pico son las 15 h (7.585 pedidos), 14 h (7.533) y 13 h (7.483).
- **Frecuencia de compra:** el 9,3 % de los pedidos (44.577) se hace exactamente 7 días después del anterior, lo que sugiere un hábito de compra semanal. La espera mediana es de 7 días entre semana y de 8 en fin de semana. 6.327 clientes tienen 10 pedidos o más.
- **Segmentos de cliente:** se clasificó a los clientes en ocasionales (1 pedido), regulares (2 a 4) y frecuentes (5 o más). Los frecuentes son cerca del 19 % de los clientes y hacen casi la mitad de los pedidos; compran más en la mañana que los ocasionales.
- **Recompra y carrito:** la tasa de recompra baja con la posición en el carrito (0,679, 0,677, 0,659, 0,638 y 0,617 en las posiciones 1 a 5): el cliente agrega primero lo que ya conoce.
- **Pedidos grandes:** 49.420 pedidos tienen 20 o más productos; el mayor (el 61355) tiene 127.

## 5. Lección 4: combinación de datasets
- **Tabla maestra:** `order_prods` se unió con `products`, `aisles`, `departments` y `orders` usando `merge` con `how='left'` y `validate='m:1'`. Cada unión conservó las 4.545.007 filas; el resultado tiene 14 columnas.
- **Auditoría:** el `merge` con `indicator=True` mostró 450.046 pedidos con productos y **28.906 pedidos (6 %) sin productos registrados**. Un `inner` los habría eliminado sin aviso, y la diferencia entre `left` e `inner` (4.573.913 frente a 4.545.007 filas) es exactamente esa cifra.
- **Tabla por producto:** 45.573 productos con veces pedido, clientes, tasa de recompra y posición media en el carrito. Los más pedidos son Banana (66.050), Bag Of Organic Bananas (53.297) y Organic Strawberries (37.039).
- **Productos más fieles:** entre los 739 productos pedidos al menos 1.000 veces, se identificó el líder de recompra de cada uno de los 18 departamentos que tienen alguno; por ejemplo, Whole Organic Omega 3 Milk en lácteos (0,863) y Banana en productos frescos (0,844).
- **Pasillos:** con al menos 10.000 artículos, los de mayor recompra son milk (0,781), water seltzer sparkling water (0,731) y fresh fruits (0,718); el de menor, spices seasonings (0,152).
- **Clientes:** la tabla por cliente tiene 149.626 clientes, con 30,4 artículos en promedio (el máximo, 580).
- **Validación final:** ninguna unión cambió el número de filas, todo artículo tiene departamento y pertenece a un pedido existente.

## 6. Respuesta a la pregunta orientadora
Para obtener información confiable sobre los hábitos de compra, los datos se prepararon en cuatro pasos: **explorar** (tamaños, tipos, nulos y duplicados), **limpiar con una decisión justificada** para cada problema (eliminar, corregir o conservar), **analizar** con filtros y agrupaciones formuladas como "métrica por grupo" y **combinar** las tablas con uniones validadas. Las comprobaciones (`validate`, `indicator` y `assert`) evitan que los errores pasen en silencio, y el informe de cambios documenta qué se le hizo a los datos.

## 7. Limitaciones
- 28.906 pedidos no tienen productos registrados, así que no entran en los análisis de artículos.
- `add_to_cart_order` solo registra hasta la posición 64: en los 70 pedidos más grandes se desconoce el orden de los productos siguientes.
- 1.258 productos no tienen nombre (`Unknown`) y 141 nombres están repetidos con identificadores distintos; los análisis agrupan por `product_id`.
- Los datos no incluyen fechas ni precios: solo permiten analizar hábitos de día, hora, frecuencia y recompra.

## 8. Archivos generados
En `data/processed/`: `cleaning_report.csv`, `orders_clean.parquet`, `products_clean.parquet`, `orders_by_weekday.csv`, `customer_segments_summary.csv`, `time_of_day_profile_by_segment.csv`, `department_summary.csv`, `product_profile.parquet` y `department_leaders.csv`. La tabla maestra `instacart_master.parquet` (unos 104 MB) no se versiona: se regenera ejecutando el cuaderno.
