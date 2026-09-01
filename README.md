# Transformacion-de-Datos-en-Power-BI
# Preparación y Limpieza de Datos de Ventas Legacy — Power BI

## 1. Orden de Las Transformaciones Realizadas
1. **Importación y Remoción de Filas Inválidas:** Se cargó el archivo Excel original `Ventas_export_legacy.xlsx` y se eliminaron las filas completamente vacías y las 48 filas transaccionales duplicadas.
2. **Normalización de Atributos:** Se estandarizó el formato de la columna `canal_venta` aplicando un formato de texto capitalizado para eliminar discrepancias de mayúsculas y minúsculas.
3. **Renombrado de Columnas:** Se convirtió la nomenclatura técnica legacy a un esquema `snake_case` en español legible para el usuario final.
4. **Casting de Tipos de Datos:** Se definieron tipos explícitos para asegurar la compatibilidad con el motor de modelado DAX.
5. **Tratamiento e Imputación de Nulos:** Se gestionaron vacíos informativos en datos de contacto y se recalcularon montos transaccionales incompletos.
6. **Separación Dimensional:** Se dividió el dataset plano en dos entidades independientes (`Dim_Clientes` y `Fact_Ventas`).

## 2. Justificación Técnica de Tipos de Datos
* **Claves e Identificadores (`id_operacion`, `id_cliente`, `id_producto`):** Se definieron como `Texto`. Aunque contengan caracteres numéricos, no son valores sobre los que se realicen operaciones matemáticas (sumas o promedios) y formatearlos como texto evita la pérdida de ceros a la izquierda.
* **Fechas (`fecha_venta`, `fecha_alta_cliente`):** Se configuraron explícitamente como tipo `Fecha` (Date) para permitir inteligencia de tiempo en DAX, jerarquías de fechado y filtros temporales interactivos.
* **Métricas Cuantitativas (`cantidad`, `precio_unitario`, `total_venta`):** Se establecieron como `Número entero` para unidades físicas y `Número decimal` para valores monetarios para garantizar precisión en los agregados sumatorios y de promedio.

## 3. Resolución de Nulos y Duplicados
* **Duplicados:** Se aplicó la remoción de duplicados en la etapa inicial para evitar la duplicación de ingresos facturados en el modelo final.
* **Valores Nulos Informativos:** Los faltantes en email y teléfono se completaron con etiquetas descriptivas (`"Sin Email"`, `"Sin Teléfono"`) para conservar la integridad del registro del cliente sin distorsionar reportes de completitud de datos.
* **Valores Nulos Financieros:** Los nulos en descuento se asumieron como `0.00` (ausencia de descuento). En las filas donde `total_venta` estaba ausente, se aplicó la fórmula aritmética derivada $total\_venta = cantidad \times precio\_unitario \times (1 - porcentaje\_descuento)$ para asegurar que ninguna transacción distorsione la suma total de facturación.

## 4. Criterio de Normalización y Separación de Tablas
Se utilizó el principio de modelado dimensional (Esquema Estrella) para separar los atributos descriptivos de los eventos transaccionales:
* **Entidad `Dim_Clientes`:** Contiene los atributos estables del cliente con granularidad de una fila por `id_cliente` único. Previene la redundancia de datos de residencia y contacto en miles de filas transaccionales.
* **Tabla de Hechos `Fact_Ventas`:** Contiene los eventos de venta con sus métricas numéricas y la clave foránea `id_cliente`, optimizando el rendimiento de memoria de la motorización VertiPaq en Power BI.
