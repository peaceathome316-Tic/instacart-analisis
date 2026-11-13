# Instacart: preparar y estructurar datos con Python y Pandas

**Aprendiz:** David Vargas Franco · **Ficha:** 3169901 · **Actividad:** GA1-220501094-01-AA1-EV06 (SENA)

## Objetivo
Aplicar técnicas de limpieza, filtrado, agrupación y combinación de datos con Python y Pandas sobre los cinco archivos CSV de Instacart, para responder la pregunta orientadora: **¿cómo preparar y estructurar los datos de Instacart para obtener información confiable sobre los hábitos de compra?**

## Datos de entrada
Cinco archivos CSV (separador `;`), disponibles en la [carpeta de la actividad](https://drive.google.com/drive/folders/19vYGYdE0hI1DzvpBcCZskzJ0u2TZjwas?usp=drive_link):

| Archivo | Una fila es | Filas |
|---|---|---|
| `instacart_orders.csv` | Un pedido | 478.967 |
| `order_products.csv` | Un producto dentro de un pedido | 4.545.007 |
| `products.csv` | Un producto del catálogo | 49.694 |
| `aisles.csv` | Un pasillo | 134 |
| `departments.csv` | Un departamento | 21 |

Los CSV **no se suben al repositorio** (están en `.gitignore`). Descárgalos y guárdalos en `data/raw/`.

## Estructura del repositorio
```
instacart-analisis/
├── README.md
├── .gitignore
├── requirements.txt
├── data/
│   ├── raw/                              # 5 CSV originales (ignorados por Git)
│   └── processed/                        # archivos generados
├── notebooks/
│   ├── 00_entorno.ipynb                  # comprobación del entorno
│   ├── 01_exploracion_instacart.ipynb    # actividad anterior (exploración)
│   └── pandas_labs.ipynb                 # lecciones 2, 3 y 4: laboratorios y retos
└── reports/
    └── processing_report.md              # informe del procesamiento
```

## Cómo ejecutar el proyecto
Requisitos: Python 3.12 o superior y Git.

```bash
git clone https://github.com/peaceathome316-Tic/instacart-analisis.git
cd instacart-analisis
python -m venv .venv
.venv\Scripts\activate             # macOS/Linux: source .venv/bin/activate
python -m pip install -r requirements.txt
```
1. Copia los cinco CSV a `data/raw/`.
2. Abre JupyterLab con `jupyter lab` y abre `notebooks/pandas_labs.ipynb`.
3. Ejecuta todo con **Kernel → Restart Kernel and Run All Cells**.

La ejecución completa tarda varios minutos y necesita memoria suficiente para la tabla maestra (4,5 millones de filas, 14 columnas).

## Archivos generados en `data/processed/`
| Archivo | Lección | Contenido | ¿En el repositorio? |
|---|---|---|---|
| `cleaning_report.csv` | 2 | Informe de cambios del pipeline de limpieza | Sí |
| `orders_clean.parquet` | 2 | Pedidos sin duplicados y con tipos corregidos | Sí |
| `products_clean.parquet` | 2 | Catálogo con nombres estandarizados | Sí |
| `orders_by_weekday.csv` | 3 | Pedidos, clientes y espera media por día | Sí |
| `customer_segments_summary.csv` | 3 | Resumen por segmento de cliente | Sí |
| `time_of_day_profile_by_segment.csv` | 3 | Franja horaria de compra por segmento (%) | Sí |
| `department_summary.csv` | 4 | Artículos y tasa de recompra por departamento | Sí |
| `product_profile.parquet` | 4 | Una fila por producto con sus métricas | Sí |
| `department_leaders.csv` | 4 | Producto con mayor recompra por departamento | Sí |
| `instacart_master.parquet` | 4 | Tabla maestra (4.545.007 filas × 14 columnas) | **No**: pesa unos 104 MB; se regenera ejecutando el notebook |

## Resultados principales
- **Limpieza:** se eliminaron 15 pedidos duplicados, se redujo la memoria de `orders` de 23,0 a 7,2 MB y la de `order_prods` de 145,4 a 54,5 MB, y se conservaron con justificación los nulos de `days_since_prior_order` (primeros pedidos) y de `add_to_cart_order` (la columna solo registra hasta la posición 64).
- **Hábitos de compra:** el 9,3 % de los pedidos se hace exactamente 7 días después del anterior (compra semanal), y la tasa de recompra baja con la posición en el carrito (0,679 en la primera posición y 0,617 en la quinta).
- **Combinación:** la tabla maestra se construyó sin perder ni duplicar filas, y se encontraron 28.906 pedidos sin productos registrados.

El detalle está en [`reports/processing_report.md`](reports/processing_report.md).

## Flujo de Git
Rama `feature/pandas-data-processing`, creada desde `develop`, con un commit por lección:

| Avance | Mensaje del commit |
|---|---|
| Lección 2 | `feat: complete lesson 2 data cleaning` |
| Lección 3 | `feat: complete lesson 3 data analysis` |
| Lección 4 | `feat: complete lesson 4 data merging` |
| Documentación | `docs: update README and processing report` |

La rama se fusiona con `develop` al finalizar.
