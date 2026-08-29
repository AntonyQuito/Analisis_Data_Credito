# Analisis_Data_Credito

## Resumen
*(Escribe aquí una breve descripción general de tu proyecto, qué herramientas utilizaste y qué problema estás resolviendo con el análisis de datos).*

## Objetivo de Proyecto 
*(Define cuál es la meta principal que buscas alcanzar con este análisis financiero o de créditos).*

## Preguntas

**1. ¿Cuántos son los clientes que tienen más edad?**
```sql
Select
         edad,
         COUNT(*) as Cantidad_Clientes
From [Proyecto_Personal].[dbo].[clientes_credito]
Group by edad
Order By edad 
```

**2. ¿Cuáles niveles educativos tienen un limite_credito promedio superior a $8,500, considerando solo aquellos grupos que tengan más de 2000 clientes en la base de datos?**
```sql
Select 
         educacion,
         AVG(limite_credito) as Limite_Credito,
         COUNT(id) as cantidad_de_clientes 
from [Proyecto_Personal].[dbo].[clientes_credito]
Group by educacion
Having 
         AVG(limite_credito) > 8500
         AND COUNT(id)>2000
```
**3. ¿Qué grupos según su estado civil y sexo tienen un promedio de meses inactivo mayor a 1, pero a la vez generan un volumen total de transacciones superior a $100,000?**
```sql
Select 
         estado_civil,
         sexo,
         AVG(meses_inactivo_12m) as promedio_Meses_Inactivo,
         SUM(valor_transaccion_12m) as valor_transaccion
From [Proyecto_Personal].[dbo].[clientes_credito]
Group by 
         estado_civil,
         sexo
Having 
         AVG(meses_inactivo_12m)>1
         AND SUM(valor_transaccion_12m)>1000000
```
**4. ¿Cuáles son los id y el tipo_tarjeta de los clientes que realizaron una cantidad de transacciones (qtd_transaccion_12m) superior al promedio de transacciones de los clientes que tienen tarjeta 'Blue'?**
```sql
Select
         id,
         tipo_tarjeta,
         qtd_transaccion_12m
From [Proyecto_Personal].[dbo].[clientes_credito]
Where
         qtd_transaccion_12m>(Select AVG(qtd_transaccion_12m) From [Proyecto_Personal].[dbo].[clientes_credito] Where tipo_tarjeta='blue')

```
**5. ¿Cuáles son los id de los clientes cuyo valor de transacción anual supera en al menos el doble (200%) el promedio de gasto de su mismo segmento demográfico (definido por estado civil y salario anual)?**

```sql
WITH CalculoPromedios AS (
    SELECT 
        id,
        estado_civil,
        salario_anual,
        valor_transaccion_12m,
        AVG(valor_transaccion_12m) OVER (PARTITION BY estado_civil, salario_anual) AS promedio_segmento
    FROM 
        [Proyecto_Personal].[dbo].[clientes_credito]
)
SELECT 
    id,
    estado_civil,
    salario_anual,
    valor_transaccion_12m,
    promedio_segmento
FROM 
    CalculoPromedios
WHERE 
    valor_transaccion_12m >= (promedio_segmento * 2);

```
**6. ¿Cuáles son los clientes que se ubican en el percentil 95 o superior de inactividad, cuentan con 4 o más productos, y cuyo límite de crédito intacto supera la media global de toda la cartera?**
```sql
With CalculoClientes As (
         Select
                 id,
                 meses_inactivo_12m,
                 qtd_productos,
                 limite_credito,
                 valor_transaccion_12m,
                 ( limite_credito-valor_transaccion_12m) as Limite_Credito_Intacto,
                 PERCENT_RANK() OVER (ORDER BY meses_inactivo_12m ASC) as p_Inactividad
         From [Proyecto_Personal].[dbo].[clientes_credito]
)
Select 
         id,
         meses_inactivo_12m,
         qtd_productos,
         Limite_Credito_Intacto
From CalculoClientes
Where
         p_Inactividad>=0.95
         AND qtd_productos>=4
         AND Limite_Credito_Intacto >(
         Select AVG(limite_credito-valor_transaccion_12m) From [Proyecto_Personal].[dbo].[clientes_credito]
         )
```
**7. ¿Cuál es el límite de crédito promedio y cuántos clientes hay dependiendo de qué tanto exprimen su tarjeta? Clasifica a los clientes en 'Uso Alto' (gastan más del 70% de su límite), 'Uso Medio' (entre 30% y 70%) y 'Uso Bajo' (menos del 30%).**
```sql
WITH CategoriasUso AS (
    SELECT 
        id,
        limite_credito,
        CASE 
            WHEN (valor_transaccion_12m / limite_credito) > 0.70 THEN '1. Uso Alto'
            WHEN (valor_transaccion_12m / limite_credito) BETWEEN 0.30 AND 0.70 THEN '2. Uso Medio'
            ELSE '3. Uso Bajo'
        END AS nivel_de_uso
    FROM [Proyecto_Personal].[dbo].[clientes_credito]
)
SELECT 
    nivel_de_uso,
    COUNT(id) AS total_clientes,
    AVG(limite_credito) AS limite_credito_promedio
FROM CategoriasUso
GROUP BY nivel_de_uso
ORDER BY nivel_de_uso;
```
**8. ¿Quiénes son los 3 clientes con la mayor cantidad de transacciones anuales dentro de cada tipo de tarjeta para incluirlos en una campaña de recompensas?**
```sql
WITH RankingClientes AS (
    SELECT 
        id,
        tipo_tarjeta,
        qtd_transaccion_12m,
        ROW_NUMBER() OVER (PARTITION BY tipo_tarjeta ORDER BY qtd_transaccion_12m DESC) AS puesto
    FROM [Proyecto_Personal].[dbo].[clientes_credito]
)
SELECT 
    tipo_tarjeta,
    puesto,
    id,
    qtd_transaccion_12m
FROM RankingClientes
WHERE puesto <= 3;
```
**9. ¿Cuál es la tasa exacta de morosidad, junto con el volumen total de clientes y los casos específicos de impago, segmentando la cartera por estado civil y sexo?**
```sql
SELECT 
    estado_civil,
    sexo,
    COUNT(id) AS total_clientes,
    SUM(CASE WHEN [is_default] = 1 THEN 1 ELSE 0 END) AS clientes_morosos,
    (SUM(CASE WHEN [is_default] = 1 THEN 1.0 ELSE 0.0 END) / COUNT(id)) * 100 AS tasa_morosidad_porcentaje
FROM [Proyecto_Personal].[dbo].[clientes_credito]
GROUP BY 
    estado_civil,
    sexo
ORDER BY 
    tasa_morosidad_porcentaje DESC;
```
**10. ¿Qué clientes generan un volumen de transacciones superior al promedio global, pero experimentan alta fricción con el banco al registrar 4 o más iteraciones de servicio?**
```sql
SELECT 
    id,
    tipo_tarjeta,
    valor_transaccion_12m,
    iteraciones_12m
FROM [Proyecto_Personal].[dbo].[clientes_credito]
WHERE 
    iteraciones_12m >= 4
    AND valor_transaccion_12m > (
        SELECT AVG(valor_transaccion_12m) 
        FROM [Proyecto_Personal].[dbo].[clientes_credito]
    )
ORDER BY 
    valor_transaccion_12m DESC;
```

## Conclusiones 
* Conclusión 1 basada en los hallazgos de tus consultas SQL y datos.
* Conclusión 2.

## Recomendaciones 
* Recomendación 1 orientada al negocio o a la toma de decisiones.
* Recomendación 2.
