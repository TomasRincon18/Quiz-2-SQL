# 📚 Quiz SQL 2 - Consultas 

## 📌 Punto 1: Exploración Básica de Datos
**Contexto:** El líder del equipo solicita un reconocimiento rápido de la tabla `TIC2023` antes de realizar análisis más profundos.

### Consulta A: Ver todos los registros
Muestra todos los registros de la tabla `TIC2023` sin filtros, órdenes o límites. Solo para ver los datos contenidos.

```sql
SELECT * FROM TIC2023;
```

### Consulta B: Extraer valores únicos
Muestra únicamente los valores distintos y no repetidos que existen en la columna `actividad_nombre` de la tabla `TIC2023`.

```sql
SELECT DISTINCT actividad_nombre FROM TIC2023;
```

---

## 📌 Punto 2: Reporte de Reconocimiento
**Contexto:** Como analista junior, se requiere producir un reporte básico de exploración sobre la tabla `TIC2022`. Se pide extraer columnas específicas, ordenadas numéricamente y mostrando únicamente los primeros 50 registros.

```sql
SELECT sede_codigo, periodo_anio, actividad_codigo, actividad_nombre 
FROM TIC2022
ORDER BY sede_codigo ASC
LIMIT 50;
```

---

## 📌 Punto 3: Creación de Tabla de Consolidación
**Contexto:** El equipo de arquitectura de datos necesita que diseñes la estructura de una nueva tabla llamada `tic_sedes_resumen`. Ésta servirá para consolidar el número de actividades TIC registradas por sede y año, y debe llevar una restricción para asegurar que una sede no tenga dos resúmenes del mismo año.

*(Nota: Sustituida la consulta del punto 2 repetida en tu archivo, por la creación de la tabla correspondiente al Quiz).*

```sql
CREATE TABLE tic_sedes_resumen (
    resumen_id INTEGER PRIMARY KEY,
    sede_codigo INTEGER NOT NULL,
    anio INTEGER NOT NULL,
    Total_actividades INTEGER,
    Tiene_internet BOOLEAN,
    Fecha_carga DATETIME,
    UNIQUE (sede_codigo, anio)
);
```

---

## 📌 Punto 4: Informe de Consultas por Internet
**Contexto:** La Dirección de Cobertura y Equidad necesita saber cuántas sedes distintas de cada departamento registraron la actividad de consulta por internet (código de actividad `'05'`) en 2023, buscando identificar brechas de conectividad. Mostrando solo departamentos superando las 500 sedes en el conteo.

### Consulta de Ajuste / Previa
Reemplazo preventivo en caso de que los valores `'5'` se encuentren como datos nulos en la tabla de 2023.

```sql
UPDATE TIC2023
SET actividad_codigo = '5'
WHERE actividad_codigo IS NULL;
```

### Consulta Principal (Conteo por Departamento)
```sql
SELECT 
    SUBSTR(sede_codigo, 1, 2) AS codigo_departamento, 
    COUNT(DISTINCT sede_codigo) AS total_sedes_unicas 
FROM TIC2023
WHERE actividad_codigo = '5'
GROUP BY codigo_departamento
HAVING total_sedes_unicas > 500
ORDER BY total_sedes_unicas DESC;
```

---

## 📌 Punto 5: Comparativo Evolutivo 2022 vs 2023
**Contexto:** Comparar el comportamiento de las sedes educativas entre los periodos 2022 y 2023 para el informe de gestión al Congreso. Requiere determinar para las sedes recurrentes en ambos años cuánto fue la diferencia en su uso TIC y tipificar la tendencia (`CRECIÓ`, `DECRECIÓ` o `SIN CAMBIO`).

```sql
WITH resumen_2022 AS (
    SELECT sede_codigo, COUNT(*) AS total_act_2022
    FROM TIC2022
    GROUP BY sede_codigo
), resumen_2023 AS (
    SELECT sede_codigo, COUNT(*) AS total_act_2023
    FROM TIC2023
    GROUP BY sede_codigo
), comparativo_final AS (
    SELECT 
        resumen_2022.sede_codigo, 
        resumen_2022.total_act_2022, 
        resumen_2023.total_act_2023, 
        (resumen_2023.total_act_2023 - resumen_2022.total_act_2022) AS diferencia
    FROM resumen_2022
    INNER JOIN resumen_2023 
        ON resumen_2022.sede_codigo = resumen_2023.sede_codigo
    WHERE resumen_2022.total_act_2022 >= 2
)
SELECT 
    sede_codigo, 
    total_act_2022, 
    total_act_2023, 
    diferencia,
    CASE 
        WHEN diferencia > 0 THEN 'CRECIÓ'
        WHEN diferencia < 0 THEN 'DECRECIÓ'
        ELSE 'SIN CAMBIO'
    END AS tendencia
FROM comparativo_final
ORDER BY diferencia DESC
LIMIT 30;
```